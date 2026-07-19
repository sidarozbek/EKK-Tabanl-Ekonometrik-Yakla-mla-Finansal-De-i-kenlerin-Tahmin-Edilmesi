> **Bu kart serinin 1. parçasıdır.** Seri üç karttan oluşur ve hepsi **tek bir mini-projeye** hizmet eder: kendi kendine çalışan bir tahmin sistemi.
> 
> Bu kartta R'ı hiç bilmiyor olabilirsin — sorun değil. Her adım tek bir şey yapacak, hiçbir adımda birden fazla yeni kavramla aynı anda karşılaşmayacaksın.
> 
> **Bu kartın sonunda elinde olacak:** `veri_altyapisi.R` adlı bir dosya. İçinde 8 tane küçük "makine" (fonksiyon) olacak. Her makine bir işi yapıyor: veri çekmek, birleştirmek, günlükten aylığa indirgemek, eksikleri doldurmak, geçmiş ay sütunları üretmek.
> 
> **Altın kural:** Bir kod bloğunu yaz → çalıştır → "Doğrula" satırındaki şeyi gördüğünden emin ol → sonraki bloğa geç. Bir bloğu atlayıp ileri gitme; sonraki blok bir öncekine ihtiyaç duyuyor.

---

## Önce: Bu kartta karşılaşacağın 3 temel kavram

Koda girmeden önce üç kelimeyi anlayalım. Bunlar kartın her yerinde çıkacak.

**1. Değişken (`<-`)**

```r
x <- 5
```

Bu satır "x diye bir kutuya 5 sayısını koy" demektir. `<-` işareti "şunu şuna ata" anlamına gelir. Sağdaki değer, soldaki isme kaydedilir.

**2. Fonksiyon** Fonksiyon, bir işi tekrar tekrar yapabileceğin bir "makine"dir. Girdi alır, bir iş yapar, çıktı verir.

```r
topla <- function(a, b) {
  a + b
}
topla(3, 4)
```

`topla` artık bir makine. İçine `3` ve `4` koydun, o da sana `7` verdi. Bu kartta kuracağımız her şey (`fred_cek`, `doldur`, vb.) aynı mantıkla çalışan makineler.

**3. Data frame (tablo)** R'da tablo halindeki veriye `data.frame` denir. Excel'deki bir sayfa gibi düşün: satırlar ve sütunlar var.

Bu üç kavramı gördükçe tanıyacaksın. Şimdi kuruluma geçelim.

---

## A0 — Kurulum

### A0.1 — Dosyayı oluştur

- [ ] RStudio'da: **File → New File → R Script**.
- [ ] Dosyayı `veri_altyapisi.R` adıyla kaydet.

> Bu dosya, tüm kodlarımızı yazacağımız yer. Yukarıdan aşağıya çalışır.

### A0.2 — Paketleri kur (sadece bir kez, hayatında bir kez)

**Paket nedir?** R'ın kendisi her şeyi yapamaz. Paket, başkalarının yazıp paylaştığı hazır araç kutusudur. Örneğin "FRED'den veri çek" işini biz sıfırdan yazmayacağız; `fredr` adlı paket bunu bizim için yapacak.

```r
install.packages(c("httr", "jsonlite", "fredr", "zoo", "lubridate"))
```

- [ ] Bu satırı çalıştır (Ctrl+Enter / Cmd+Enter). İnternetten indirme olacak, birkaç dakika sürebilir.
- [ ] **Doğrula:** Konsolda kırmızı `Error` yazmadan işlem bitti (sarı uyarılar sorun değil).

> Bu adımı bir daha yapmana gerek yok — paketler bilgisayarına kalıcı kuruldu.

### A0.3 — Paketleri her oturumda çağır

Kurmak ile çağırmak farklı şeyler. Kurmak = mağazadan satın almak (bir kez). Çağırmak = dolaptan çıkarıp masaya koymak (her oturumda).

```r
library(httr)
library(jsonlite)
library(fredr)
library(zoo)
library(lubridate)
```

- [ ] Bu 5 satırı çalıştır.
- [ ] **Doğrula:** Hiçbir satır kırmızı `Error` vermedi.

> Bu 5 satırı dosyanın en tepesine yaz. RStudio'yu her kapatıp açtığında bu satırları yeniden çalıştırman gerekecek (kurulum değil, çağırma).

### A0.4 — API anahtarlarını gir

**API anahtarı nedir?** Bazı veri servisleri (FRED, EVDS) "sen kimsin" diye sorar; anahtar senin kimliğin. Kendi anahtarını ilgili sitelerden ücretsiz alabilirsin.

```r
fredr_set_key("KENDI_FRED_ANAHTARIN")
EVDS_ANAHTARI <- "KENDI_EVDS_ANAHTARIN"
```

- [ ] Tırnak içindeki metni kendi anahtarınla değiştir.
- [ ] Çalıştır.
- [ ] **Doğrula:** Hata almadan geçti. (Anahtar yanlışsa hatayı birazdan A1/A2'de göreceğiz, şimdilik sorun yok.)

---

## A1 — FRED'den veri çeken makineyi kur

**Amaç:** ABD Merkez Bankası'nın (FRED) veri tabanından, örneğin dolar kuru gibi bir seriyi çekip düzgün bir tabloya koyan bir makine yazacağız.

Bunu tek seferde yazmak yerine, 4 küçük parçaya bölelim.

### A1.1 — Boş iskeleti yaz

```r
fred_cek <- function(seri_kodu, std_ad, baslangic = "2010-01-01") {

}
```

> Bu, henüz içi boş bir makine tanımı. `seri_kodu` (hangi veri), `std_ad` (bu veriye ne isim vereceğiz) ve `baslangic` (hangi tarihten itibaren) olmak üzere 3 girdi alıyor. `baslangic`'in yanındaki `= "2010-01-01"` demek: "eğer sen bir tarih vermezsen, ben otomatik 2010'dan başlarım."

- [ ] **Doğrula:** Bu bloğu çalıştırınca hata almadın (henüz bir şey yapmıyor, sadece tanımlanıyor).

### A1.2 — FRED'e isteği at

```r
fred_cek <- function(seri_kodu, std_ad, baslangic = "2010-01-01") {
  ham <- fredr(series_id = seri_kodu,
               observation_start = as.Date(baslangic))
  ham
}
```

> `fredr(...)` FRED'e "bana bu seriyi ver" diye istek atar. `as.Date(baslangic)` yazı halindeki tarihi ("2010-01-01") R'ın anladığı gerçek bir tarihe çevirir. Sonucu şimdilik `ham` adlı kutuya koyup direkt geri döndürüyoruz (henüz düzenlemedik).

- [ ] Dene:

```r
test1 <- fred_cek("DEXUSAL", "USDTRY")
head(test1)
```

- [ ] **Doğrula:** `test1` tablosu göründü; içinde `date` ve `value` gibi sütunlar var ama isimler henüz bizim istediğimiz gibi değil. Normal — sıradaki adımda düzelteceğiz.

### A1.3 — Sadece işimize yarayan 2 sütunu al

```r
fred_cek <- function(seri_kodu, std_ad, baslangic = "2010-01-01") {
  ham <- fredr(series_id = seri_kodu,
               observation_start = as.Date(baslangic))
  out <- data.frame(tarih = ham$date,
                    deger = ham$value)
  out
}
```

> FRED'in verdiği tabloda gereğinden çok sütun var. Biz sadece tarih ve değeri istiyoruz. `data.frame(tarih = ham$date, deger = ham$value)` yeni, sade bir tablo kurar: sadece iki sütun.

- [ ] Dene:

```r
test2 <- fred_cek("DEXUSAL", "USDTRY")
head(test2)
```

- [ ] **Doğrula:** `test2` tablosunda artık sadece `tarih` ve `deger` sütunları var.

### A1.4 — Sütuna standart bir isim ver

```r
fred_cek <- function(seri_kodu, std_ad, baslangic = "2010-01-01") {
  ham <- fredr(series_id = seri_kodu,
               observation_start = as.Date(baslangic))
  out <- data.frame(tarih = ham$date,
                    deger = ham$value)
  names(out)[2] <- std_ad
  out
}
```

> Şu ana kadar ikinci sütunun adı hep `deger` idi. Ama biz sonradan birden fazla göstergeyi (dolar kuru, enflasyon, faiz...) yan yana koyacağız; hepsi `deger` adında olursa birbirine karışır. `names(out)[2] <- std_ad` satırı "ikinci sütunun adını, bana verdiğin isimle değiştir" der.

- [ ] Son haliyle dene:

```r
kur <- fred_cek("DEXUSAL", "USDTRY")
head(kur); nrow(kur)
```

- [ ] **Doğrula:** `kur` tablosunda `tarih` ve `USDTRY` sütunları var. `nrow(kur)` sıfırdan büyük bir sayı verdi (kaç satır veri geldiğini gösterir).

**A1 tamamlandı.** Artık elinde, herhangi bir FRED serisini iki temiz sütuna çeviren bir makine var.

---

## A2 — EVDS'den veri çeken makineyi kur

**Amaç:** Türkiye Merkez Bankası'nın (EVDS) veri tabanından bir seriyi (örneğin enflasyon) çekmek. EVDS, FRED'den farklı "konuşuyor": anahtarı farklı yerde ister, veriyi farklı biçimde (JSON) döner, tarih formatı bazen "2010-01" gibi gün olmadan, bazen de "01-01-2010" gibi tam gelir. Bu farklılıkları makinenin içine gizleyeceğiz — dışarıdan bakan, `fred_cek` ile aynı davranan bir şey görecek.

> **Not:** EVDS'nin iki sürümü var: eski **EVDS2** ve güncel **EVDS3**. Bu kartta doğrudan çalışan **EVDS3** uç noktasını kullanıyoruz.

Bunu 6 küçük parçaya bölelim.

### A2.1 — Seri kodunu normalize et, adresi (URL) kur

EVDS'ye vereceğimiz seri kodları bazen `"TP_FG_J0"` gibi alt çizgiyle, EVDS'nin kendisi ise noktayla (`"TP.FG.J0"`) yazılmış kod bekler. Bunu elle her seferinde düzeltmek yerine, makinenin kendisi çevirsin.

```r
evds_cek <- function(seri_kodu, std_ad,
                     baslangic = "01-01-2010", bitis = "01-01-2025") {

  api_kodu <- gsub("-1$", "", gsub("_", ".", seri_kodu))

  url <- paste0(
    "https://evds3.tcmb.gov.tr/igmevdsms-dis/series=",
    api_kodu,
    "&startDate=", baslangic,
    "&endDate=", bitis,
    "&type=json"
  )
  url
}
```

> `gsub("_", ".", seri_kodu)` → seri kodundaki her `_` işaretini `.` ile değiştirir (`"TP_FG_J0"` → `"TP.FG.J0"`). `gsub("-1$", "", ...)` → eğer kodun sonunda yanlışlıkla `"-1"` kalmışsa onu siler (`$` işareti "sadece sonda ise" demek). `paste0(...)` bu düzeltilmiş kodu ve tarihleri birleştirip EVDS3'ün adresini (`evds3.tcmb.gov.tr`) kurar.

- [ ] Dene:

```r
evds_cek("TP_FG_J0", "ENFLASYON")
```

- [ ] **Doğrula:** İçinde `TP.FG.J0` geçen (alt çizgi değil, nokta) uzun bir internet adresi göründü.

### A2.2 — İsteği at ve hatayı erken yakala

```r
evds_cek <- function(seri_kodu, std_ad,
                     baslangic = "01-01-2010", bitis = "01-01-2025") {

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
  cevap
}
```

> `GET(url, add_headers(key = EVDS_ANAHTARI))` → tarayıcıya adresi yazmışsın gibi isteği atar; anahtarı da "header" denen görünmez bir bölmede yollar (EVDS böyle istiyor).
> 
> `status_code(cevap)` → sunucunun cevap kodu. **200 = her şey yolunda.** Başka bir sayıysa bir sorun var demektir.
> 
> `if (...) stop(...)` → sorun varsa programı burada durdurup hemen bir hata mesajı yazdırır. Bunu neden yapıyoruz? Çünkü hata erken ve anlaşılır olursa, ileride "neden çalışmıyor" diye saatlerce uğraşmayız.

- [ ] Dene:

```r
test_cevap <- evds_cek("TP_FG_J0", "ENFLASYON")
status_code(test_cevap)
```

- [ ] **Doğrula:** `status_code(test_cevap)` → `200` gördün. 200 değilse: anahtarını (`EVDS_ANAHTARI`) kontrol et.

### A2.3 — Cevabı metne, sonra tabloya çevir

```r
evds_cek <- function(seri_kodu, std_ad,
                     baslangic = "01-01-2010", bitis = "01-01-2025") {

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
  veri
}
```

> Sunucudan gelen cevap önce ham metin (`content(...)`), sonra bu metin JSON formatında bir tablo (`fromJSON(...)$items`). JSON, internette veri taşımak için kullanılan yaygın bir format — şimdilik "veri paketleme şekli" olarak düşünebilirsin.

- [ ] Dene:

```r
test_veri <- evds_cek("TP_FG_J0", "ENFLASYON")
head(test_veri)
names(test_veri)
```

- [ ] **Doğrula:** Bir tablo göründü; sütun isimleri arasında `Tarih` var, yanında bir de değer sütunu var (adı seriye göre değişebilir — bu yüzden sıradaki adımda onu elle "2." sütun diye sabitlemek yerine otomatik bulacağız).

### A2.4 — Değer sütununu otomatik bul

EVDS3'te değer sütununun adı her seride aynı olmuyor (`TP_FG_J0` için farklı, `TP_APIFON4` için farklı olabilir). Sütunu sabit "2. sütun" diye almak kırılgan; onun yerine "Tarih olmayan sütun hangisiyse o" diyoruz.

```r
evds_cek <- function(seri_kodu, std_ad,
                     baslangic = "01-01-2010", bitis = "01-01-2025") {

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

  deger_kolonu <- setdiff(names(veri), "Tarih")[1]
  deger_kolonu
}
```

> `names(veri)` → tablodaki tüm sütun isimlerini listeler. `setdiff(names(veri), "Tarih")` → bu listeden `"Tarih"`'i çıkarır, geriye kalanlar değer adayı sütunlardır. `[1]` → geriye kalanların ilkini alır — genelde tek bir değer sütunu olduğu için bu yeterli.

- [ ] Dene:

```r
evds_cek("TP_FG_J0", "ENFLASYON")
```

- [ ] **Doğrula:** Değer sütununun adı (örn. `"TP_FG_J0"` gibi bir metin) ekrana geldi.

### A2.5 — Tarihi iki farklı formata da dayanıklı şekilde çevir

EVDS bazen tarihi `"2010-01"` (yıl-ay, gün yok), bazen `"01-01-2010"` (gün-ay-yıl) biçiminde döndürebilir. İkisini de yakalayan bir kontrol yazıyoruz.

```r
evds_cek <- function(seri_kodu, std_ad,
                     baslangic = "01-01-2010", bitis = "01-01-2025") {

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

  deger_kolonu <- setdiff(names(veri), "Tarih")[1]

  tarih <- veri$Tarih
  if (grepl("^\\d{4}-\\d+$", tarih[1])) {
    tarih <- as.Date(paste0(tarih, "-01"))
  } else {
    tarih <- as.Date(tarih, format = "%d-%m-%Y")
  }

  out <- data.frame(tarih = tarih,
                    deger = as.numeric(veri[[deger_kolonu]]))
  out
}
```

> `grepl("^\\d{4}-\\d+$", tarih[1])` → ilk tarih değerine bakıp "4 rakam, tire, bir veya daha çok rakam" kalıbına uyuyor mu diye sorar (yani "2010-01" gibi mi?). `^` başlangıç, `$` bitiş demek — baştan sona bu kalıba uysun istiyoruz. Uyuyorsa → sonuna `"-01"` ekleyip tam tarihe çeviriyoruz (A2'nin eski mantığıyla aynı). Uymuyorsa → `"01-01-2010"` formatında geldiğini varsayıp `format = "%d-%m-%Y"` ile okuyoruz. `as.numeric(veri[[deger_kolonu]])` → bir önceki adımda bulduğumuz değer sütununu sayıya çevirir.

- [ ] Dene:

```r
test3 <- evds_cek("TP_FG_J0", "ENFLASYON")
head(test3)
```

- [ ] **Doğrula:** `tarih` sütunu gerçek bir tarih (örn. `2010-01-01`), `deger` sütunu sayı.

### A2.6 — Sütuna standart isim ver (A1.4 ile aynı fikir)

```r
evds_cek <- function(seri_kodu, std_ad,
                     baslangic = "01-01-2010", bitis = "01-01-2025") {

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

  deger_kolonu <- setdiff(names(veri), "Tarih")[1]

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
```

> Aynı A1.4'teki mantık: ikinci sütunun adını `deger` yerine bizim verdiğimiz standart isme (`"ENFLASYON"`, `"FAIZ"` vb.) çeviriyoruz. Böylece birden fazla göstergeyi birleştirdiğimizde sütunlar birbirine karışmıyor.

- [ ] Son haliyle dene:

```r
enf <- evds_cek("TP_FG_J0", "ENFLASYON")
head(enf); nrow(enf)
```

- [ ] **Doğrula:** `enf` tablosunda `tarih` ve `ENFLASYON` sütunları var, satır sayısı sıfırdan büyük.

> **Not:** `evds_cek` genel bir makine — aynı fonksiyonu kontrol değişkenimiz olan **FAIZ** serisi için de kullanacağız (TÜBİTAK önerisindeki "kontrol değişkeni", Bölüm 2.2). Deneyelim:

```r
faiz <- evds_cek("TP_APIFON4", "FAIZ")
head(faiz); nrow(faiz)
```

- [ ] **Doğrula:** `faiz` tablosunda `tarih` ve `FAIZ` sütunları var. Not: bu seri **günlük** geliyor (enflasyon aylık) — bu farkı A3.5'te ele alacağız.

**A2 tamamlandı.** Artık üç farklı kaynaktan (FRED'den kur, EVDS'den enflasyon ve faiz) veri çeken, ama dışarıdan bakınca aynı şekilde davranan makinelerin var.

---

## A3 — İki tabloyu birleştiren makine

**Amaç:** `kur`, `enf` ve `faiz` tablolarını, ortak `tarih` sütununa göre yan yana koymak.

### A3.1 — Neden `merge` gerekiyor, önce gözlemle

Üç tablonun tarihleri birebir aynı satır sayısında olmayabilir (üstelik `kur` ve `faiz` günlük, `enf` aylık geliyor). Bu yüzden basitçe yan yana yapıştıramayız; tarihe göre eşleştirmemiz lazım.

```r
nrow(kur)
nrow(enf)
nrow(faiz)
```

- [ ] Çalıştır ve üç sayıya bak.
- [ ] **Doğrula:** Sayılar muhtemelen birbirinden farklı. (Aynıysa da sorun değil, birleştirme yine gerekli çünkü tarihler tam örtüşmeyebilir.)

### A3.2 — Tek makineyle birleştir

```r
birlestir <- function(...) {
  tablolar <- list(...)
  g <- Reduce(function(a, b) merge(a, b, by = "tarih", all = TRUE), tablolar)
  g <- g[order(g$tarih), ]          # tarihe göre sırala (zaman serisinde ŞART)
  rownames(g) <- NULL
  g
}
```

> `...` demek "kaç tane tablo verirsen ver, hepsini kabul ederim" (2, 3, 5 tablo — fark etmez).
> 
> `merge(a, b, by = "tarih", all = TRUE)` iki tabloyu `tarih` sütununa göre eşleştirir. `all = TRUE` demek: "bir tarih sadece birinde varsa, o satırı yine de tut, diğer sütunu boş (`NA`) bırak — veriyi kaybetme."
> 
> `Reduce(...)` bu birleştirmeyi, kaç tablo varsa hepsine sırayla uygular (2 tablo varsa bir kez, 3 tablo varsa iki kez birleştirir).
> 
> `g[order(g$tarih), ]` satırları eskiden yeniye sıralar. Bu şart — ileride "geçen ay" hesaplarken sıra karışıksa sonuç yanlış çıkar.

- [ ] Dene:

```r
gunluk <- birlestir(kur, enf, faiz)
head(gunluk); nrow(gunluk)
```

- [ ] **Doğrula:** `gunluk` tablosunda `tarih`, `USDTRY`, `ENFLASYON`, `FAIZ` sütunları var, tarihler küçükten büyüğe sıralı. Bazı hücreler boş (`NA`) olabilir — normal, sıradaki kartta çözeceğiz.

**A3 tamamlandı.**

---

## A3.5 — Günlük veriyi AYLIĞA indirgeyen makine (önemli — veri sızıntısını önler)

**Sorun ne?** `ENFLASYON` zaten aylık bir seri; ama `USDTRY` ve `FAIZ` **günlük** geliyor. A3'te `birlestir` ile birleştirince, `ENFLASYON`'ın tek bir aylık değeri, o ay içindeki **her güne** kopyalanmış oluyor (boşluk doldurma bunu yapıyor). Bu görünüşte zararsız ama aslında tehlikeli: aynı ayın farklı günlerini modelde "farklı gözlem" gibi göstermiş oluruz, bu da geçmiş-ay (`lag`) sütunlarını neredeyse birbirinin aynısı yapar — buna **veri sızıntısı (data leakage)** denir ve model performansını olduğundan iyi gösterir.

**Çözüm:** Birleştirme bittikten sonra tabloyu **aya indirgiyoruz** — her ay için o ayın **son gözlenen** (en güncel) değerini alıyoruz.

```r
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
```

> `format(tablo$tarih, "%Y-%m")` → her tarihi "2010-01" gibi yıl-ay etiketine indirger. `aggregate(...)` bu etikete göre satırları gruplar; her grup (her ay) için verdiğimiz fonksiyonu çalıştırır. `na.omit(x)` boş değerleri atar, `tail(x, 1)` kalanların **sonuncusunu** (o ayın en güncel günü) alır. Sonunda tabloyu tekrar gerçek bir `tarih` sütunuyla (ayın 1'i) düzenliyoruz.

- [ ] Dene:

```r
aylik <- aya_indirge(gunluk)
head(aylik); nrow(aylik)
```

- [ ] **Doğrula:** `aylik` tablosunda artık her satır bir **aya** karşılık geliyor (günlere değil); satır sayısı `gunluk`'tan çok daha az. Sütunlar aynı (`tarih`, `USDTRY`, `ENFLASYON`, `FAIZ`).

**A3.5 tamamlandı.** Bundan sonraki tüm adımlarda (`bosluk_olc`, `doldur`, `lag_ekle`) `gunluk` yerine `aylik` tablosunu kullanacağız.

---

## A4 — Eksik verileri ele almak (2 aşama: önce ölç, sonra doldur)

Neden iki ayrı aşama? Çünkü "ne kadar boşluk vardı" bilgisini kaybetmeden, doldurma işlemini şeffaf yapmak istiyoruz. Raporda "şu kadar boşluk vardı, şöyle doldurduk" diyebilmek için.

### A4.1 — Önce say: kaç boşluk var?

```r
bosluk_olc <- function(tablo) {
  toplam <- sum(is.na(tablo))
  sutun  <- colSums(is.na(tablo))
  cat("Toplam boş hücre:", toplam, "\n")
  print(sutun)
  invisible(toplam)
}
```

> `is.na(tablo)` her hücreye "boş mu?" diye sorar, cevap `TRUE`/`FALSE` olur. `sum(...)` toplam boş hücre sayısını verir. `colSums(...)` her sütunda ayrı ayrı kaç boşluk olduğunu verir.

- [ ] Dene:

```r
baslangic_bosluk <- bosluk_olc(aylik)
```

- [ ] **Doğrula:** Bir toplam sayı ve sütun sütun dağılım gördün. Bu sayıyı not al — A4.3'te "sıfıra indi mi" diye buna bakacağız.

### A4.2 — Doldurma mantığını 3 katmanda anla (önce tek tek dene)

Üç farklı boşluk türü var, üçüne de farklı çözüm lazım:

**Ortadaki boşluk** → iki bilinen nokta arasına düz çizgi çek (ara değer bulma):

```r
na.approx(c(1, NA, NA, 4), na.rm = FALSE)
```

- [ ] Çalıştır. **Doğrula:** Sonuç `1, 2, 3, 4` — aradaki boşluklar kademeli dolduruldu.

**Sondaki boşluk** → son bilinen değeri ileri taşı:

```r
na.locf(c(1, 2, NA, NA), na.rm = FALSE)
```

- [ ] Çalıştır. **Doğrula:** Sonuç `1, 2, 2, 2` — son bilinen değer (2) tekrarlandı.

**Baştaki boşluk** → ilk bilinen değeri geriye taşı:

```r
na.locf(c(NA, NA, 3, 4), fromLast = TRUE, na.rm = FALSE)
```

- [ ] Çalıştır. **Doğrula:** Sonuç `3, 3, 3, 4` — ilk bilinen değer (3) geriye taşındı.

### A4.3 — Üç katmanı tek makinede birleştir

```r
doldur <- function(x) {
  x <- na.approx(x, na.rm = FALSE)                  # 1) ortadaki boşluk
  x <- na.locf(x, na.rm = FALSE)                    # 2) sondaki boşluk
  x <- na.locf(x, fromLast = TRUE, na.rm = FALSE)   # 3) baştaki boşluk
  x
}
```

> Üç satır, A4.2'de tek tek denediğin üç işlemi sırayla uyguluyor. Sıra önemli: önce ortayı, sonra sonu, sonra başı dolduruyoruz.

- [ ] Tüm sütunlara uygula:

```r
for (s in setdiff(names(aylik), "tarih")) {
  aylik[[s]] <- doldur(aylik[[s]])
}
```

> `setdiff(names(aylik), "tarih")` demek: "tüm sütun isimlerini al, ama tarih'i çıkar" (tarihi doldurmaya gerek yok, zaten dolu). `for (s in ...)` döngüsü bu listedeki her sütun adını sırayla `s`'e koyup, o sütuna `doldur` makinesini uygular.

- [ ] Sonucu kontrol et:

```r
son_bosluk <- sum(is.na(aylik))
cat("Doldurma sonrası boş:", son_bosluk, "| Çözüldü mü:", son_bosluk == 0, "\n")
```

- [ ] **Doğrula:** `Çözüldü mü: TRUE` gördün. `FALSE` ise A4.1'deki dağılıma dön, hangi sütun hâlâ boş bak.

> **Not (raporunda kullan):** Bu doldurma bir **tahmindir**, gerçek gözlem değil. Rapor yazarken bunu açıkça belirt.

**A4 tamamlandı.**

---

## A5 — Geçmiş ay (lag) sütunları üretmek

**Neden gerekli?** Bir sonraki kartta "bu ayın enflasyonu, geçen ayın enflasyonuna bağlı mı?" diye modelleyeceğiz. Bunun için "geçen ayki değer" bilgisini ayrı bir sütun olarak elimizde bulundurmamız lazım. Buna **lag** (gecikme) denir.

### A5.1 — Tek bir lag'i elle, küçük bir örnekle anla

```r
ornek <- c(10, 20, 30, 40, 50)
c(NA, head(ornek, -1))
```

- [ ] Çalıştır. **Doğrula:** Sonuç `NA, 10, 20, 30, 40`. Yani her değer bir sağa kaydı — "bir önceki ay"ı temsil ediyor. Başa `NA` koyduk çünkü ilk ayın "öncesi" yok.

### A5.2 — Bunu genelleştiren makineyi yaz

```r
lag_ekle <- function(tablo, sutun, max_lag = 6) {
  for (k in 1:max_lag) {
    yeni_ad <- paste0(sutun, "_lag", k)
    tablo[[yeni_ad]] <- c(rep(NA, k), head(tablo[[sutun]], -k))
  }
  tablo
}
```

> `for (k in 1:max_lag)` → 1'den 6'ya kadar her gecikme miktarı için bir sütun üretir (1 ay önce, 2 ay önce, ... 6 ay önce). `head(tablo[[sutun]], -k)` → sütunun son `k` değeri **hariç** hepsini alır. `c(rep(NA, k), ...)` → başına `k` tane boşluk (`NA`) koyup değerleri `k` satır aşağı kaydırır. Neden 6 lag üretiyoruz ama hepsini kullanmayacağız? Çünkü kaç ay geriye bakmanın en iyi olduğuna Kart B'de karar vereceğiz; şimdilik hepsini hazırlıyoruz.

- [ ] Dene:

```r
aylik <- lag_ekle(aylik, "ENFLASYON", 6)
aylik <- lag_ekle(aylik, "USDTRY", 6)
head(aylik, 8)
```

- [ ] **Doğrula:** `ENFLASYON_lag1` ... `ENFLASYON_lag6` ve `USDTRY_lag1` ... `USDTRY_lag6` sütunları oluştu. İlk satırların lag hücreleri `NA` — bu doğru ve beklenen.

> **Not:** Bu sadece bir alıştırma — `veri_hazirla` ana makinesi (A6) lag'i yalnızca **hedef** değişken (`hedef` parametresi neyse, örn. sadece `ENFLASYON`) için üretecek, `USDTRY` ve `FAIZ` için değil. Çünkü modelin tahmin etmeye çalıştığı şey `hedef`; diğerleri sadece o anki (lag'siz) değerleriyle veride kalıyor.

**A5 tamamlandı.**

---

## A6 — Hepsini tek düğmede toplayan ana makine

Şimdiye kadar 6 ayrı makine kurduk: `fred_cek`, `evds_cek`, `birlestir`, `bosluk_olc`, `doldur`, `lag_ekle`. Şimdi bunları sırayla çağıran **tek bir makine** yazacağız — Kart B ve C bunu tek satırla kullanacak.

### A6.1 — İskeleti kur, adımları `message` ile işaretle

```r
veri_hazirla <- function(hedef = "ENFLASYON", max_lag = 6) {
  message("[1/5] FRED ve EVDS'den veri çekiliyor...")
  kur  <- fred_cek("DEXUSAL", "USDTRY")
  enf  <- evds_cek("TP_FG_J0", "ENFLASYON")
  faiz <- evds_cek("TP_APIFON4", "FAIZ")

  message("[2/5] Birleştiriliyor...")
  g <- birlestir(kur, enf, faiz)

  g   # şimdilik burada dur, sadece ilk 2 adımı test edelim
}
```

- [ ] Dene:

```r
test_g <- veri_hazirla()
head(test_g)
```

- [ ] **Doğrula:** Konsolda `[1/5]` ve `[2/5]` mesajlarını gördün; `test_g` tablosu `kur`, `enf` ve `faiz`'in birleşimi (`tarih`, `USDTRY`, `ENFLASYON`, `FAIZ` sütunları var).

### A6.2 — Aylığa indirgeme adımını ekle (veri sızıntısını önlemek için)

```r
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

  g
}
```

> `USDTRY` ve `FAIZ` günlük, `ENFLASYON` aylık geldiği için, A3.5'te anlattığımız veri sızıntısı riskini burada `aya_indirge` ile kapatıyoruz. Bu adımdan sonra tablo tamamen aylık.

- [ ] Dene:

```r
test_g2 <- veri_hazirla()
nrow(test_g2)
```

- [ ] **Doğrula:** `[3/5]` mesajı ve "Aylık gözlem sayısı: X" satırı göründü; `test_g2`'nin satır sayısı A3'teki günlük tablodan çok daha az.

### A6.3 — Doldurma adımını ekle

```r
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

  g
}
```

- [ ] Dene:

```r
test_g3 <- veri_hazirla()
sum(is.na(test_g3))
```

- [ ] **Doğrula:** `[4/5]` mesajı ve "Boşluk: X -> 0" satırı göründü. `sum(is.na(test_g3))` → `0`.

### A6.4 — Lag adımını ekle

```r
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

  g
}
```

> `hedef` parametresi hangi göstergenin lag'lerini üreteceğimizi belirliyor. Böylece aynı makineyi hem `"ENFLASYON"` hem `"USDTRY"` için kullanabiliyoruz — `USDTRY` ve `FAIZ` diğer durumda da tabloda kalır, sadece lag'i alınmaz (bkz. A5.2 notu).

- [ ] Dene:

```r
test_g4 <- veri_hazirla("ENFLASYON")
names(test_g4)
```

- [ ] **Doğrula:** Sütun isimleri arasında `ENFLASYON_lag1` ... `ENFLASYON_lag6` görünüyor, `USDTRY` ve `FAIZ` de hâlâ (lag'siz) sütun olarak duruyor.

### A6.5 — Son temizlik: yarım (eksik) satırları at

```r
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
```

> Lag sütunlarının ilk birkaç satırı `NA` (çünkü "öncesi" yok, bkz. A5.1). Model bu boş satırları kabul etmez. `complete.cases(...)` sadece hedef sütun ve tüm lag sütunları **dolu olan** satırları seçer; geri kalanları atar.

- [ ] Son haliyle dene:

```r
veri <- veri_hazirla("ENFLASYON")
head(veri); tail(veri)
```

- [ ] **Doğrula:** `[1/5]` → `[5/5]` mesajları sırayla göründü, "Aylık gözlem sayısı" ve "Hazır veri: X satır, Y sütun" yazdı, `veri` tablosunda kullanılan sütunlarda hiç `NA` yok.

**A6 tamamlandı — Kart A bitti.**

---

## Kart A teslim

- [ ] `veri_altyapisi.R` dosyasında şu 8 makine tanımlı ve çalışıyor: `fred_cek`, `evds_cek`, `birlestir`, `aya_indirge`, `bosluk_olc`, `doldur`, `lag_ekle`, `veri_hazirla`.
- [ ] `veri <- veri_hazirla("ENFLASYON")` tek satırla temiz, aylık, modele hazır veri üretiyor (USDTRY + ENFLASYON + FAIZ hepsi bir arada).
- [ ] Bonus: `veri_hazirla("USDTRY")` da çalışıyor mu dene (hedefi değiştirince lag o göstergeden üretiliyor mu?).

Bu kartı bitiren kişi, projenin **veri altyapısını** kendi eliyle kurmuş olur. Kart B, `veri_hazirla` makinesini aynen kullanarak modeli kuracak.

---

## Hata çıkarsa (sırayla kontrol et)

- [ ] `could not find function "fredr"` → A0.3'teki `library(fredr)` satırını çalıştırmadın; yeniden başlattıysan tekrar çalıştır.
- [ ] EVDS `stop` ile hata veriyor → A0.4'teki anahtarını kontrol et. Adres `evds3.tcmb.gov.tr` olmalı — eski `evds2` adresi artık çalışmayabilir.
- [ ] Değer sütunu bulunamıyor / hep `NA` geliyor → A2.4'teki `deger_kolonu` kontrolüne bak; EVDS'nin döndürdüğü sütun adı beklenenden farklı olabilir, `names(veri)` ile gerçek adı gör.
- [ ] `na.approx` hatası → `library(zoo)` çalıştırdın mı (A0.3)?
- [ ] Çok fazla `NA` kalıyor → serilerin sıklığı (günlük/aylık) çok farklı olabilir; A3.5'teki `aya_indirge` bunu zaten aylığa indirgeyerek çözüyor, A4.3'teki doldurma kalanı kapatır.
- [ ] `aggregate` hatası veya `aya_indirge` çalışmıyor → `birlestir` adımını atlamışsındır; `aya_indirge`'ye ham (günlük) değil, `birlestir(kur, enf, faiz)` çıktısı olan tabloyu vermelisin.
- [ ] `TP_APIFON4` (FAIZ) için EVDS hata veriyor → seri kodu değişmiş olabilir, EVDS sitesinden güncel kodu kontrol et.
- [ ] `veri` çok az satır döndü → `baslangic` tarihini erkene çek (örn. `"2005-01-01"`).
- [ ] Bir bloğu çalıştırdın ama önceki bloktaki değişken "bulunamadı" diyor → o bloğu atlamışsındır, yukarı dön ve sırayla tekrar çalıştır.