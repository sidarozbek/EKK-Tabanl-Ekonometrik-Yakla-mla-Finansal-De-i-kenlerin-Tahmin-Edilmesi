> Bu dosya, **KART A — Veri Altyapısı** kartındaki tüm alt-adımların sonunda ulaşılan **son hali**dir. Kart boyunca kod parça parça, alt-adımlarla inşa edildi; burada sadece çalışan **nihai sürüm** tek parça halinde veriliyor.
> 
> Kullanım: Aşağıdaki kod bloğunu olduğu gibi kopyalayıp `veri_altyapisi.R` adlı bir dosyaya yapıştır. Kart B ve Kart C bu dosyayı `source("veri_altyapisi.R")` ile çağıracak.

```r
# =========================================================
# veri_altyapisi.R
# KART A — Veri Altyapısı
# =========================================================

# --- A0.3: Paketleri çağır (kurulum A0.2'de bir kez yapılmış olmalı) ---
library(httr)
library(jsonlite)
library(fredr)
library(zoo)
library(lubridate)

# --- A0.4: API anahtarları ---
fredr_set_key("KENDI_FRED_ANAHTARIN")
EVDS_ANAHTARI <- "KENDI_EVDS_ANAHTARIN"


# =========================================================
# A1 — FRED'den veri çeken makine
# =========================================================
fred_cek <- function(seri_kodu, std_ad, baslangic = "2010-01-01") {
  ham <- fredr(series_id = seri_kodu,
               observation_start = as.Date(baslangic))
  out <- data.frame(tarih = ham$date,
                    deger = ham$value)
  names(out)[2] <- std_ad     # değer sütununa standart adı ver
  out
}


# =========================================================
# A2 — EVDS'den veri çeken makine (EVDS3 uç noktası)
# =========================================================
evds_cek <- function(seri_kodu, std_ad,
                     baslangic = "01-01-2010", bitis = "01-01-2025") {

  # Seri kodunu normalize et: "TP_FG_J0" -> "TP.FG.J0"
  api_kodu <- gsub("-1$", "", gsub("_", ".", seri_kodu))

  url <- paste0(
    "https://evds3.tcmb.gov.tr/igmevdsms-dis/series=",
    api_kodu,
    "&startDate=", baslangic,
    "&endDate=", bitis,
    "&type=json"
  )

  cevap <- GET(url, add_headers(key = EVDS_ANAHTARI))
  if (status_code(cevap) != 200) {
    stop("EVDS Hatası: ", status_code(cevap))
  }

  veri <- fromJSON(content(cevap, as = "text", encoding = "UTF-8"))$items

  # Değer sütununu otomatik bul (adı seriye göre değişebiliyor)
  deger_kolonu <- setdiff(names(veri), "Tarih")[1]

  # Tarihi iki olası formata (YYYY-MM veya DD-MM-YYYY) dayanıklı çevir
  tarih <- veri$Tarih
  if (grepl("^\\d{4}-\\d+$", tarih[1])) {
    tarih <- as.Date(paste0(tarih, "-01"))
  } else {
    tarih <- as.Date(tarih, format = "%d-%m-%Y")
  }

  out <- data.frame(tarih = tarih,
                    deger = as.numeric(veri[[deger_kolonu]]))
  names(out)[2] <- std_ad
  out
}


# =========================================================
# A3 — İki (veya daha çok) tabloyu tarihe göre birleştiren makine
# =========================================================
birlestir <- function(...) {
  tablolar <- list(...)
  g <- Reduce(function(a, b) merge(a, b, by = "tarih", all = TRUE), tablolar)
  g <- g[order(g$tarih), ]          # tarihe göre sırala (zaman serisinde ŞART)
  rownames(g) <- NULL
  g
}


# =========================================================
# A3.5 — Günlük veriyi AYLIĞA indirgeyen makine
# =========================================================
# ENFLASYON zaten aylık; USDTRY ve FAIZ günlük geldiği için, birleştirme
# sonrası ENFLASYON değeri o ay içindeki her güne kopyalanıyor (na.locf
# doldurması nedeniyle). Bu, aynı ayın farklı günlerini "farklı gözlem"
# gibi gösterip lag sütunlarını neredeyse aynı değere eşitliyor (veri
# sızıntısı). Bu makine, birleştirme sonrasında tabloyu aya indirger:
# her ay için o ayın SON gözlenen değeri alınır.
aya_indirge <- function(tablo) {
  ay <- format(tablo$tarih, "%Y-%m")
  sutunlar <- setdiff(names(tablo), "tarih")

  agg <- aggregate(tablo[sutunlar],
                   by = list(ay = ay),
                   FUN = function(x) {
                     x <- na.omit(x)
                     if (length(x) == 0) return(NA_real_)
                     tail(x, 1)          # ayın son (en güncel) değeri
                   })

  agg$tarih <- as.Date(paste0(agg$ay, "-01"))
  agg$ay <- NULL
  agg <- agg[order(agg$tarih), c("tarih", sutunlar)]
  rownames(agg) <- NULL
  agg
}


# =========================================================
# A4.1 — Eksik hücreleri ölçen makine
# =========================================================
bosluk_olc <- function(tablo) {
  toplam <- sum(is.na(tablo))
  sutun  <- colSums(is.na(tablo))
  cat("Toplam boş hücre:", toplam, "\n")
  print(sutun)
  invisible(toplam)
}


# =========================================================
# A4.3 — Eksik hücreleri dolduran makine (3 katman)
# =========================================================
doldur <- function(x) {
  x <- na.approx(x, na.rm = FALSE)                  # 1) ortadaki boşluk
  x <- na.locf(x, na.rm = FALSE)                    # 2) sondaki boşluk
  x <- na.locf(x, fromLast = TRUE, na.rm = FALSE)   # 3) baştaki boşluk
  x
}


# =========================================================
# A5.2 — Geçmiş ay (lag) sütunları üreten makine
# =========================================================
lag_ekle <- function(tablo, sutun, max_lag = 6) {
  for (k in 1:max_lag) {
    yeni_ad <- paste0(sutun, "_lag", k)
    tablo[[yeni_ad]] <- c(rep(NA, k), head(tablo[[sutun]], -k))
  }
  tablo
}


# =========================================================
# A6.4 — Hepsini tek çağrıda toplayan ana makine
# =========================================================
# NOT: Değişken seti, Melek Sevimli'nin TÜBİTAK 2209-A önerisindeki
# "Finansal Değişkenler" ile uyumludur: Döviz kuru (USD/TRY), Enflasyon
# (TÜFE) ve Faiz oranları (kontrol değişkeni, öneri Bölüm 2.2).
veri_hazirla <- function(hedef = "ENFLASYON", max_lag = 6) {
  message("[1/5] FRED ve EVDS'den veri çekiliyor...")
  kur  <- fred_cek("DEXUSAL", "USDTRY")
  enf  <- evds_cek("TP_FG_J0", "ENFLASYON")
  faiz <- evds_cek("TP_APIFON4", "FAIZ")

  message("[2/5] Birleştiriliyor...")
  g <- birlestir(kur, enf, faiz)

  message("[3/5] Aylığa indirgeniyor...")
  g <- aya_indirge(g)
  cat("    Aylık gözlem sayısı:", nrow(g), "\n")

  message("[4/5] Eksikler dolduruluyor...")
  basl <- sum(is.na(g))
  for (s in setdiff(names(g), "tarih")) g[[s]] <- doldur(g[[s]])
  cat("    Boşluk:", basl, "->", sum(is.na(g)), "\n")

  message("[5/5] Lag türetiliyor...")
  g <- lag_ekle(g, hedef, max_lag)

  # Modele verilecek tam-dolu satırlar (öncesi olmayan baş satırları at)
  lag_kolon <- paste0(hedef, "_lag", 1:max_lag)
  v <- g[complete.cases(g[, c(hedef, lag_kolon)]), ]
  rownames(v) <- NULL

  cat("Hazır veri:", nrow(v), "satır,", ncol(v), "sütun\n")
  v
}


# =========================================================
# Tek komutla baştan sona çalıştırma
# =========================================================
veri <- veri_hazirla("ENFLASYON")
head(veri); tail(veri)
```

---

## Bu dosyadaki 8 makinenin özeti

|Makine|Ne yapar|Girdi|Çıktı|
|---|---|---|---|
|`fred_cek`|FRED'den tek bir seri çeker (USD/TRY: `DEXUSAL`)|seri kodu, standart isim, başlangıç tarihi|`tarih` + `std_ad` sütunlu tablo|
|`evds_cek`|EVDS'den tek bir seri çeker (ENFLASYON: `TP_FG_J0`, FAIZ: `TP_APIFON4`)|seri kodu, standart isim, tarih aralığı|`tarih` + `std_ad` sütunlu tablo|
|`birlestir`|Kaç tablo verirsen ver, tarihe göre yan yana koyar|tablolar (`...`)|tek, sıralı, geniş tablo|
|`aya_indirge`|Günlük/karışık sıklıktaki tabloyu aya indirger (her ay için son gözlem); veri sızıntısını önler|tablo (günlük+aylık karışık)|her satırı bir ay olan tablo|
|`bosluk_olc`|Kaç boş hücre olduğunu sayar/raporlar|bir tablo|toplam boşluk sayısı (görünmez döner)|
|`doldur`|Tek bir sütundaki boşlukları 3 katmanda doldurur|bir sütun (vektör)|dolu sütun|
|`lag_ekle`|Bir sütunun geçmiş `k` ay öncesini yeni sütun yapar|tablo, sütun adı, kaç lag|lag sütunları eklenmiş tablo|
|`veri_hazirla`|Yukarıdaki 7'sini sırayla çalıştırıp modele hazır, AYLIK veri üretir (USDTRY + ENFLASYON + FAIZ)|hedef gösterge, max lag|temiz, aylık, tam dolu, lag'li son tablo|

## Kullanım (Kart B ve C için)

```r
source("veri_altyapisi.R")
veri <- veri_hazirla("ENFLASYON")
```

Bu iki satır, Kart B'nin ve Kart C'nin başında hep aynı şekilde yer alacak.