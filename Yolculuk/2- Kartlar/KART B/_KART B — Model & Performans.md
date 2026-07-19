> **Bu kart serinin 2. parçasıdır.** Kart A'nın ürettiği temiz veriyi (`veri_hazirla`) alıp üzerine modeli kurar: kaç ay geriye bakılacağını (p) AIC/BIC ile seçer, kayan pencereyle tahmin üretir, **CV ile en iyi pencereyi** bulur ve performansı beş metrikle ölçer. Kart C bu kartın `tahmin_yap` ve `cv_pencere_sec` fonksiyonlarını otomatik panele bağlayacak.
> 
> **Ön koşul:** Kart A'daki fonksiyonlar yüklü olmalı. Dosyanın başında bir kez:

```r
source("veri_altyapisi.R")   # Kart A'nın fonksiyonları
veri <- veri_hazirla("ENFLASYON")
```

> **Bu kartın sonunda elinde:** `model.R` adlı, içinde `p_sec`, `tahmin_yap`, `cv_pencere_sec`, `metrikler`, `model_calistir` fonksiyonları olan çalışan bir dosya.
> 
> **Kural:** Sırayla yaz, çalıştır, "Doğrula"yı gör. Pratik öncelikli.

---

## B0 — Hazırlık

- [ ] Yeni R Script: `model.R`. Başına:

```r
source("veri_altyapisi.R")            # Kart A
veri <- veri_hazirla("ENFLASYON")     # temiz veri hazır (artık AYLIK)
```

- [ ] **Doğrula:** `veri` tablosu dolu, `head(veri)` lag sütunlarını gösteriyor.

> **Not (Kart A güncellemesi):** `veri_hazirla` artık günlük veriyi `aya_indirge` ile aylığa indirgeyip veriyor — yani `veri`'deki her satır bir ay, önceki (günlük) sürümde olduğu gibi aynı ayın farklı günleri değil. Ayrıca tabloda `FAIZ` sütunu da var (TÜBİTAK önerisindeki kontrol değişkeni, Bölüm 2.2); bu kartın modelleri şimdilik sadece hedefin kendi geçmişini (`ENFLASYON_lag*` gibi) kullanıyor, `FAIZ`'i regresyona sokmuyor — istersen `formul_kur`'u genişletip kontrol değişkeni olarak ekleyebilirsin, bu bir sonraki iterasyon için iyi bir genişletme fikri.

---

## B1 — Bir EKK modeli kurmak (en küçük parça)

Her şeyin temelinde tek bir doğrusal regresyon (EKK / öneri Bölüm 2.4) var: enflasyonu kendi geçmişiyle açıkla.

```r
m1 <- lm(ENFLASYON ~ ENFLASYON_lag1 + ENFLASYON_lag2, data = veri)
summary(m1)
```

> **Satır satır:**
> 
> - `lm(...)` → linear model = EKK regresyonu. R bunu `β̂ = (X'X)⁻¹X'Y` formülüyle (öneri Bölüm 2.4) arka planda çözer.
> - `ENFLASYON ~ ENFLASYON_lag1 + ENFLASYON_lag2` → "enflasyonu, bir ve iki ay önceki enflasyonla açıkla". `~` = "şunu şunlarla açıkla".
> - `summary(m1)` → katsayıları (φ₁, φ₂), anlamlılığı, R²'yi gösterir.

- [ ] **Doğrula:** `summary` çıktısında `Coefficients` tablosu var; `ENFLASYON_lag1` katsayısı görünüyor.
- [ ] Katsayılara bak: `lag1` katsayısı genelde pozitif ve büyük olur (bu ayın enflasyonu geçen aya çok benzer). Not al.

---

## B2 — Formülü otomatik kuran yardımcı

Lag sayısını elle yazmak yerine, p verince formülü üreten küçük bir yardımcı. Bu, hem `p_sec` hem `tahmin_yap` tarafından kullanılacak.

```r
formul_kur <- function(hedef, p) {
  lag_isimleri <- paste0(hedef, "_lag", 1:p)
  as.formula(paste(hedef, "~", paste(lag_isimleri, collapse = " + ")))
}

# Dene:
formul_kur("ENFLASYON", 3)
```

> **Satır satır:**
> 
> - `paste0(hedef, "_lag", 1:p)` → `c("ENFLASYON_lag1","ENFLASYON_lag2","ENFLASYON_lag3")` üretir.
> - `paste(..., collapse = " + ")` → bunları `+` ile birleştirir.
> - `as.formula(...)` → metni R'ın model formülüne çevirir: `ENFLASYON ~ ENFLASYON_lag1 + ENFLASYON_lag2 + ENFLASYON_lag3`.

- [ ] **Doğrula:** Çıktı `ENFLASYON ~ ENFLASYON_lag1 + ENFLASYON_lag2 + ENFLASYON_lag3` formülü.

---

## B3 — AIC/BIC ile optimal p seçimi

Öneri Bölüm 2.6. "Kaç ay geriye bakmalı?" sorusunu döngüyle, her adayı deneyip en düşük bilgi kriterini seçerek yanıtlıyoruz. Çok lag = aşırı uyum (model gürültüyü ezberler); az lag = yetersiz. AIC/BIC dengeyi kurar.

```r
p_sec <- function(veri, hedef, max_lag = 6, kriter = AIC) {
  skorlar <- numeric(max_lag)
  for (p in 1:max_lag) {
    model <- lm(formul_kur(hedef, p), data = veri)
    skorlar[p] <- kriter(model)
  }
  en_iyi <- which.min(skorlar)
  cat(deparse(substitute(kriter)), "skorları:", round(skorlar, 1), "\n")
  cat("Seçilen p:", en_iyi, "\n")
  list(p = en_iyi, skorlar = skorlar)
}

# Dene — hem AIC hem BIC:
aic_sonuc <- p_sec(veri, "ENFLASYON", max_lag = 6, kriter = AIC)
bic_sonuc <- p_sec(veri, "ENFLASYON", max_lag = 6, kriter = BIC)

p_secilen <- aic_sonuc$p
cat("AIC p:", aic_sonuc$p, "| BIC p:", bic_sonuc$p, "\n")
```

> **Satır satır:**
> 
> - Döngü 1..max_lag her aday p için model kurar (`formul_kur` ile) ve skorunu kaydeder.
> - `kriter(model)` → AIC ya da BIC. `kriter` bir parametre olduğu için aynı fonksiyon ikisi için de çalışır.
> - `which.min(skorlar)` → **en düşük** skorun indeksi = optimal p.
> - `deparse(substitute(kriter))` → kullanılan kriterin adını ("AIC"/"BIC") ekrana yazdırmak için küçük bir numara.

> **AIC vs BIC:** BIC'in cezası (`k·ln(n)`) daha ağır olduğundan genelde AIC'e eşit ya da daha küçük p seçer. İki sonuç farklıysa rapora "AIC p=3 seçerken BIC daha sade p=2 önerdi" diye yaz — bu metodolojik bir gözlemdir.

- [ ] **Doğrula:** İki p değeri ve skor dizileri gördün. Skorların hangi p'de dibe vurduğunu (en düşük) işaretle.

---

## B4 — Kayan pencere tahmin fonksiyonu (öneri Bölüm 2.5)

Tüm geçmişi tek seferde kullanmak yerine, her tahmin için sadece **son w aylık pencereyi** kullanırız; pencere her adımda bir ay kayar. Bu, modelin değişen koşullara uyumunu sağlar (yapısal kırılmalar). CV bu fonksiyonu defalarca çağıracak, o yüzden sağlam ve hızlı olmalı.

```r
tahmin_yap <- function(veri, hedef, pencere, p) {
  formul <- formul_kur(hedef, p)
  n <- nrow(veri)
  if (n <= pencere + 1) return(NULL)     # pencere veriden büyükse dürüstçe boş dön

  sonuc <- data.frame(tarih  = as.Date(character()),
                      gercek = numeric(),
                      tahmin = numeric())

  for (i in (pencere + 1):n) {
    egitim <- veri[(i - pencere):(i - 1), ]    # SADECE son `pencere` ay (geçmiş)
    model  <- lm(formul, data = egitim)
    tah    <- predict(model, newdata = veri[i, ])   # bir sonraki ayı tahmin et
    sonuc  <- rbind(sonuc,
                    data.frame(tarih  = veri$tarih[i],
                               gercek = veri[[hedef]][i],
                               tahmin = as.numeric(tah)))
  }
  sonuc
}

# Dene:
deneme <- tahmin_yap(veri, "ENFLASYON", pencere = 36, p = p_secilen)
head(deneme); nrow(deneme)
```

> **Satır satır — bu fonksiyon işin motoru:**
> 
> - `if (n <= pencere + 1) return(NULL)` → **kritik koruma:** pencere veriden büyükse hata fırlatmaz, boş döner. CV sırasında büyük adayları güvenle eleyebilmemizi sağlar.
> - `for (i in (pencere + 1):n)` → ilk tahmin edilebilir aydan sona kadar her ay.
> - `egitim <- veri[(i - pencere):(i - 1), ]` → o aydan **önceki** son `pencere` ayı alır. Geleceği görmez — bu yüzden dürüst.
> - `lm(formul, data = egitim)` → SADECE bu pencereyle model kurar (tüm geçmişle değil — kayan pencerenin tüm farkı bu).
> - `predict(model, newdata = veri[i, ])` → o ayı, kendisinden önceki pencereyle tahmin eder.
> - `rbind(...)` → tarih, gerçek, tahmini biriktirir.

- [ ] **Doğrula:** `deneme` tablosunda `tarih`, `gercek`, `tahmin` dolu; satır sayısı ≈ (toplam satır − pencere).

---

## B5 — Metrikler (öneri Bölüm 2.7'nin tamamı)

Beş performans ölçütünü tek fonksiyonda topluyoruz.

```r
metrikler <- function(tahmin) {
  h <- tahmin$gercek - tahmin$tahmin
  data.frame(
    MAE     = mean(abs(h)),
    MSE     = mean(h^2),
    RMSE    = sqrt(mean(h^2)),
    MAPE    = mean(abs(h / tahmin$gercek)) * 100,
    Theil_U = sqrt(sum(h^2)) / sqrt(sum(tahmin$gercek^2))
  )
}

print(metrikler(deneme))
```

> **Her metrik ne ölçer:**
> 
> - `MAE` → ortalama mutlak sapma (yorumu en kolay; "ortalama şu kadar yanıldık").
> - `MSE` → kare ortalama (büyük hataları sertçe cezalandırır).
> - `RMSE` → MSE'nin karekökü; hedefle aynı birimde.
> - `MAPE` → yüzde hata ("ortalama %X yanıldık"); göstergeler arası kıyas için iyi.
> - `Theil_U` → naif tahminle (dünkü değeri tekrarla) kıyas. `< 1` = modelimiz naiften iyi; `> 1` = naif kadar bile değil.

- [ ] **Doğrula:** Beş metrikli tek satırlık tablo. Theil U'yu yorumla (1'in altında mı?).

---

## B6 — CV İLE PENCERE SEÇİMİ (bu kartın kalbi — geniş bölüm)

Öneri Bölüm 2.5: _"Pencere uzunluğu (w) çapraz doğrulama ile optimal belirlenecektir... literatürde 36–60 ay."_ Sabit pencere her göstergeye uymaz. Burada her adayı **görmediği veride** dürüstçe test edip en iyisini seçeriz.

### B6.1 — CV neden dürüsttür?

Kayan pencere tahmini her ayı, o aydan **önceki** veriyle tahmin eder. Yani model test ettiği ayı eğitimde hiç görmez. Bu yüzden bir pencerenin kayan-pencere RMSE'si, o pencerenin **gerçek (örnek-dışı)** performansıdır — şişirilmiş değil. CV, her aday pencerenin bu dürüst RMSE'sini hesaplayıp kıyaslar.

### B6.2 — CV fonksiyonu

> **Not (Kart A güncellemesi):** `veri` artık aylık olduğu için (bkz. B0 notu), aşağıdaki `adaylar = seq(36, 60, by = 6)` gerçekten **ay** sayısını ifade ediyor — yani 36-60 ay = 3-5 yıllık eğitim penceresi, tam olarak öneri Bölüm 2.5'teki literatür aralığı. Veri günlükken (eski Kart A) aynı `pencere` değeri günü ifade ediyordu ve pencereler istemeden çok daha kısa (birkaç haftalık) eğitim setlerine karşılık geliyordu — bu yüzden aya indirgeme adımı burada da doğruluğu sağlıyor.

```r
cv_pencere_sec <- function(veri, hedef, p, adaylar = seq(36, 60, by = 6),
                           min_tahmin = 12) {
  cat("=== CV ile pencere seçimi (", hedef, ", p =", p, ") ===\n")
  tablo <- data.frame(pencere = integer(), n_tahmin = integer(), rmse = numeric())

  for (w in adaylar) {
    tahmin <- tahmin_yap(veri, hedef, pencere = w, p = p)

    if (is.null(tahmin) || nrow(tahmin) < min_tahmin) {
      cat(sprintf("  pencere %2d -> yetersiz tahmin, ELENDI\n", w))
      tablo <- rbind(tablo, data.frame(pencere = w, n_tahmin = 0, rmse = Inf))
      next
    }

    h <- tahmin$gercek - tahmin$tahmin
    rmse_w <- sqrt(mean(h^2))
    cat(sprintf("  pencere %2d -> %3d tahmin, örnek-dışı RMSE = %.4f\n",
                w, nrow(tahmin), rmse_w))
    tablo <- rbind(tablo, data.frame(pencere = w, n_tahmin = nrow(tahmin), rmse = rmse_w))
  }

  en_iyi <- tablo$pencere[which.min(tablo$rmse)]
  cat(sprintf("--> SEÇİLEN pencere: %d ay (en düşük RMSE = %.4f)\n\n",
              en_iyi, min(tablo$rmse)))
  list(en_iyi = en_iyi, tablo = tablo)
}

# Dene:
cv <- cv_pencere_sec(veri, "ENFLASYON", p = p_secilen, adaylar = seq(36, 60, by = 6))
en_iyi_pencere <- cv$en_iyi
print(cv$tablo)
```

> **Satır satır:**
> 
> - `adaylar = seq(36, 60, by = 6)` → denenecek pencereler: 36, 42, 48, 54, 60 (literatür aralığı).> - Döngü her `w` için `tahmin_yap`'ı çağırır — yani her aday görmediği veride sınanır.
> - `if (is.null(...) || nrow(tahmin) < min_tahmin)` → pencere çok büyük olup yeterli (en az 12) tahmin üretmezse `Inf` verip eler. Adil kıyas: az tahminle "şanslı" düşük RMSE'yi engeller.
> - `sqrt(mean(h^2))` → o pencerenin örnek-dışı RMSE'si.
> - `which.min(tablo$rmse)` → **en düşük RMSE'li** pencereyi seçer.
> - Fonksiyon hem seçimi hem tüm adayların tablosunu döndürür (Kart C'de grafiğe dökeceğiz).

### B6.3 — CV sonucunu yorumla (RMSE eğrisi)

```r
gecerli <- cv$tablo[is.finite(cv$tablo$rmse), ]
plot(gecerli$pencere, gecerli$rmse, type = "b", pch = 19,
     main = "Pencere uzunluğu vs örnek-dışı RMSE",
     xlab = "Pencere (ay)", ylab = "RMSE")
points(en_iyi_pencere, min(gecerli$rmse), col = "red", pch = 19, cex = 2)
```

> **Ne gösterir?** Her pencerenin örnek-dışı hatası. Kırmızı nokta = seçilen. Eğri "U" şekliyse ortada bir tatlı nokta var (ne çok kısa ne çok uzun ideal). Sürekli düşüyorsa en uzun pencere en iyi; sürekli artıyorsa en kısa pencere kazanıyor.

- [ ] **Doğrula:** Konsolda her pencere için RMSE satırı, en sonda "SEÇİLEN pencere: X ay", ve bir RMSE eğrisi grafiği gördün. Eğrinin şeklini bir cümleyle yorumla.

### B6.4 — İddiayı test et: her gösterge farklı pencere mi ister?

Önerinin gerekçesi buydu. USD/TRY için tekrarlayıp kıyasla:

```r
veri_usd <- veri_hazirla("USDTRY")
p_usd    <- p_sec(veri_usd, "USDTRY", max_lag = 6, kriter = AIC)$p
cv_usd   <- cv_pencere_sec(veri_usd, "USDTRY", p = p_usd, adaylar = seq(36, 60, by = 6))

cat("ENFLASYON en iyi pencere:", en_iyi_pencere, "| USDTRY en iyi pencere:", cv_usd$en_iyi, "\n")
```

- [ ] **Doğrula:** İki gösterge için seçilen pencereler **farklı** çıkabilir. Çıktıysa, "göstergeye özel pencere" iddian kendi verinde doğrulanmış demektir — raporun bulgular kısmına güçlü bir cümle.

---

## B7 — Tüm modeli tek çağrıda toplayan fonksiyon

Kart C'nin (Shiny) kullanacağı tek-düğme. Veri zaten Kart A'dan geliyor; bu fonksiyon p ve pencereyi otomatik seçip final tahmini üretir.

```r
model_calistir <- function(veri, hedef = "ENFLASYON",
                           max_lag = 6, adaylar = seq(36, 60, by = 6)) {
  ps <- p_sec(veri, hedef, max_lag = max_lag, kriter = AIC)
  cv <- cv_pencere_sec(veri, hedef, p = ps$p, adaylar = adaylar)
  tahmin <- tahmin_yap(veri, hedef, pencere = cv$en_iyi, p = ps$p)

  list(hedef    = hedef,
       p        = ps$p,
       pencere  = cv$en_iyi,
       cv_tablo = cv$tablo,
       tahmin   = tahmin,
       metrik   = metrikler(tahmin),
       zaman    = Sys.time())
}

# Tek komutla:
rapor <- model_calistir(veri, "ENFLASYON")
cat("p:", rapor$p, "| pencere:", rapor$pencere, "\n")
print(rapor$metrik)
```

> **Ne yaptı?** p seç → CV ile pencere seç → final tahmin → metrik adımlarını tek `list`'te topladı. `zaman` da içinde (ne zaman üretildi — otomasyon için lazım). Kart C bu `rapor` nesnesini doğrudan ekrana basacak.

- [ ] **Doğrula:** Tek komutla p, pencere, CV tablosu, tahmin ve metrikler üretildi; `rapor` içinde hepsi var.

---

## Kart B teslim

- [ ] `model.R` dosyasında tanımlı ve çalışır: `formul_kur`, `p_sec`, `tahmin_yap`, `metrikler`, `cv_pencere_sec`, `model_calistir`.
- [ ] `model_calistir(veri, "ENFLASYON")` tek satırla p+pencere seçip tahmin ve metrik üretiyor.
- [ ] Defterinde: AIC vs BIC p farkı, CV eğrisinin şekli, enflasyon vs USD/TRY pencere farkı, Theil U yorumu.

Bu kart, önerinin Hedef 2 (CV ile AR-EKK) ve Hedef 3'ünü (performans metrikleri) — Çalışma Takvimi'nin model+performans faaliyetini — kendi elinle kanıtlar. Kart C, `model_calistir`'ı otomatik panele bağlayacak.

---

## Hata çıkarsa

- [ ] `object 'veri' not found` → B0'daki `source` ve `veri_hazirla` satırlarını çalıştırmadın.
- [ ] CV'de tüm pencereler `Inf` → veri çok kısa; `adaylar`ı küçült (`seq(24, 48, by = 6)`) ya da Kart A'da başlangıç tarihini erkene çek.
- [ ] `predict` farklı uzunlukta hata → lag sütun adları formülle uyuşmuyor; `formul_kur` çıktısını ve `veri` sütun adlarını karşılaştır.
- [ ] AIC hep en büyük p'yi seçiyor → normal olabilir; BIC ile de dene, daha sade seçer.
- [ ] RMSE eğrisi düz çizgi → adaylar birbirine çok yakın; aralığı genişlet (`seq(24, 72, by = 12)`).