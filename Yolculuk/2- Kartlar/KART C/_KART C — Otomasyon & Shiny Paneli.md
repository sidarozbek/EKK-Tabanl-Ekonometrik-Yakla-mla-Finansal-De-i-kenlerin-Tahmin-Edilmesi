> **Bu kart serinin 3. ve son parçasıdır.** Kart A (veri) ve Kart B (model) ürettiğin fonksiyonları, kendi kendine çalışan bir web paneline bağlar. Önerideki Hedef 4'ün tam karşılığı: _"web tabanlı arayüz, API ile otomatik güncelleme, gerçek zamanlı yenileme, teknik bilgi gerektirmeden."_
> 
> **Ön koşul:** Kart A ve B'nin dosyaları yanında olmalı (`veri_altyapisi.R`, `model.R`).
> 
> **Bu kartın sonunda elinde:** Açılışta kendi verisini çeken, p ve pencereyi kendisi seçen, zamanlayıcıyla kendini tazeleyen, sonucu diske önbellekleyen **otomasyonlu** bir `app.R` Shiny uygulaması.
> 
> **Kural:** Sırayla yaz, çalıştır, "Doğrula"yı gör. Bu kart pratik ağırlıklı; her Shiny parçasının ne yaptığını kısa kısa açıklayacağım ama asıl iş kodu çalıştırıp davranışı gözlemlemek.

---

## C0 — Hazırlık ve mimari

- [ ] Paketleri kur (bir kez):

```r
install.packages(c("shiny", "DT"))  
```

- [ ] Klasör yapısını anla. Tek bir `app.R` yazacağız; başında Kart A ve B'yi `source` edecek. Yani uygulama açılırken tüm motoru yüklüyor:

```
mini-proje/  
├── veri_altyapisi.R   (Kart A)  
├── model.R            (Kart B)  
├── app.R              (Kart C — bu kart)  
└── cache/             (otomatik oluşacak; veri önbelleği)  
```

> **Mimari mantık:** Shiny iki parçadır. `ui` = kullanıcının gördüğü (butonlar, grafikler). `server` = arka planda çalışan beyin (hesap, çizim). Aralarındaki köprü **reaktif** kavramı: bir girdi değişince ona bağlı çıktılar **otomatik** yeniden hesaplanır. Otomasyonun sırrı budur.

---

## C1 — İskelet: en basit çalışan Shiny

Önce boş ama çalışan bir iskelet kuralım; üstüne ekleyeceğiz.

- [ ] Yeni dosya `app.R`:

```r
library(shiny)
source("veri_altyapisi.R")   # Kart A fonksiyonları
source("model.R")            # Kart B fonksiyonları

ui <- fluidPage(
  titlePanel("Otomatik Ekonomik Tahmin Paneli"),
  sidebarLayout(
    sidebarPanel(
      p("Panel buraya gelecek.")
    ),
    mainPanel(
      p("Sonuçlar buraya gelecek.")
    )
  )
)

server <- function(input, output, session) {
  # Henüz boş
}

shinyApp(ui, server)  
```

- [ ] Çalıştır: `shiny::runApp("app.R")`
- [ ] **Doğrula:** Tarayıcıda başlıklı, iki panelli boş bir sayfa açıldı. Kapat, devam edelim.

---

## C2 — Önbellekli veri çekme (otomasyonun temeli)

Her açılışta API'yi yormamak için veriyi diske kaydedip, tazeyse oradan okuyacağız. Önerideki "veri güncellenme zamanları izlenecek" ve Risk 1'deki dayanıklılık mantığı.

- [ ] `app.R`'da `source` satırlarının altına ekle:

```r
TAZELIK_SAAT <- 12      # cache kaç saatte bayatlasın

# NOT: Cache dosyası göstergeye (hedef) göre ayrı tutulmalı.
# Tek bir dosya kullanılırsa (ör. hep "cache/veri_enf.rds"), ENFLASYON için
# hesaplanmış veri (sadece ENFLASYON_lag1..6 sütunlarını içerir) USDTRY
# seçildiğinde de okunur ve "USDTRY_lag1 not found" hatası verir.
cache_dosya_yolu <- function(hedef) {
  file.path("cache", paste0("veri_", tolower(hedef), ".rds"))
}

veri_getir <- function(hedef = "ENFLASYON", zorla = FALSE) {
  if (!dir.exists("cache")) dir.create("cache")
  cache_dosya <- cache_dosya_yolu(hedef)

  taze_mi <- file.exists(cache_dosya) &&
    difftime(Sys.time(), file.mtime(cache_dosya), units = "hours") < TAZELIK_SAAT

  if (taze_mi && !zorla) {
    message("Cache'ten okunuyor (", hedef, ")")
    return(readRDS(cache_dosya))
  }

  message("API'den taze veri çekiliyor (", hedef, ")")
  v <- veri_hazirla(hedef)         # Kart A
  saveRDS(v, cache_dosya)
  v
} 
```

> **Satır satır:**
> 
> - `cache_dosya_yolu(hedef)` → her gösterge (`ENFLASYON`, `USDTRY`...) için **ayrı** bir dosya adı üretir (`cache/veri_enflasyon.rds`, `cache/veri_usdtry.rds`). Tek bir sabit dosya kullanılsaydı, bir göstergenin lag sütunlarıyla üretilmiş veri diğer göstergede okunur ve `"USDTRY_lag1" not found` gibi bir hatayla karşılaşırdık.
> - `file.mtime(...)` → dosyanın son değişiklik zamanı. `difftime(... , units="hours")` ile şu andan farkı saat cinsinden hesaplanır.
> - `taze_mi` → o göstergenin cache'i var **ve** 12 saatten yeni mi? Öyleyse API'ye gitme, diskten oku (hızlı + API dostu).
> - `zorla = TRUE` → "Şimdi Yenile" butonu bunu kullanacak; cache taze olsa bile yeniden çeker.
> - `saveRDS(...)` → taze veriyi diske yazar, bir sonraki açılış hızlı olur.

- [ ] Test et (Shiny dışında, konsolda):

```r
v1 <- veri_getir("ENFLASYON")   # ilk: API'den çeker  
v2 <- veri_getir("ENFLASYON")   # ikinci: cache'ten okur (mesaja bak)  
v3 <- veri_getir("USDTRY")      # farklı gösterge, kendi cache dosyasına yazar
```

- [ ] **Doğrula:** İlk çağrı "API'den taze veri", ikinci çağrı "Cache'ten okunuyor" mesajı verdi. `v3` çağrısı da "API'den taze veri" dedi (çünkü USDTRY'nin kendi cache dosyası henüz yoktu) ve hatasız tamamlandı.

---

## C3 — UI: kontrolleri yerleştir

Artık arayüzü dolduruyoruz. Otomasyon hedefi: kullanıcı en az şey yapsın. Gösterge seçici, otomatik yenileme süresi, manuel yenile butonu, ve durum bilgisi.

- [ ] `app.R`'daki `ui`yi şununla değiştir:

```r
ui <- fluidPage(
  titlePanel("Otomatik Ekonomik Tahmin Paneli"),
  sidebarLayout(
    sidebarPanel(
      width = 3,
      selectInput("gosterge", "Gösterge:",
                  choices = c("ENFLASYON", "USDTRY")),
      sliderInput("yenileme_dk", "Otomatik yenileme (dakika):",
                  min = 1, max = 60, value = 15, step = 1),
      actionButton("yenile", "🔄 Şimdi Yenile", class = "btn-primary"),
      hr(),
      strong("Durum"),
      verbatimTextOutput("durum"),
      hr(),
      strong("Otomatik seçilen parametreler"),
      verbatimTextOutput("parametreler")
    ),
    mainPanel(
      width = 9,
      fluidRow(
        column(6,
          h4("Performans metrikleri"),
          tableOutput("metrik_tablo")
        ),
        column(6,
          h4("CV: pencere seçimi"),
          plotOutput("cv_grafik", height = "220px")
        )
      ),
      h4("Tahmin: Gerçek vs Model"),
      plotOutput("tahmin_grafik", height = "320px"),
      h4("CV detay tablosu"),
      tableOutput("cv_tablo")
    )
  )
) 
```

> **Ne var?** Sol panelde: gösterge seçimi, otomatik yenileme kaydırıcısı, manuel yenile butonu, durum ve parametre kutuları. Sağda: metrik tablosu + CV grafiği yan yana, altında büyük tahmin grafiği, en altta CV detay tablosu. Her `*Output(...)` bir boşluktur; `server` bunları dolduracak.

- [ ] **Doğrula:** Çalıştırınca arayüz görünüyor (henüz boş çıktılarla); buton ve kaydırıcı yerinde.

---

## C4 — Server: otomatik hesaplama kalbi

İşin otomasyon merkezi burası. Üç tetikleyici tanımlıyoruz: zamanlayıcı, buton, gösterge değişimi. Üçünden herhangi biri olunca sistem baştan çalışır.

- [ ] `server` fonksiyonunu şununla değiştir:

```r
server <- function(input, output, session) {

  # --- 1) ZAMANLAYICI: seçilen dakikada bir kendini tetikler ---
  zamanlayici <- reactive({
    invalidateLater(input$yenileme_dk * 60 * 1000, session)   # dakika -> ms
    Sys.time()
  })

  # --- 2) ANA HESAP: butona / göstergeye / zamanlayıcıya bağlı ---
  rapor <- reactive({
    g <- input$gosterge          # gösterge değişimine bağımlılık
    zamanlayici()                # zamanlayıcıya bağımlılık
    input$yenile                 # butona bağımlılık

    withProgress(message = paste(g, "için sistem çalışıyor..."), value = 0.5, {
      v <- veri_getir(g, zorla = (input$yenile > 0))   # buton basıldıysa zorla çek
      model_calistir(v, g)                              # Kart B: p+CV+tahmin+metrik
    })
  })

  # --- 3) ÇIKTILAR ---
  output$durum <- renderText({
    r <- rapor()
    paste0("Son güncelleme:\n", format(r$zaman, "%Y-%m-%d %H:%M:%S"),
           "\nVeri satırı: ", nrow(r$tahmin) + r$pencere)
  })

  output$parametreler <- renderText({
    r <- rapor()
    paste0("Gösterge : ", r$hedef,
           "\nAIC ile p : ", r$p,
           "\nCV pencere: ", r$pencere, " ay")
  })

  output$metrik_tablo <- renderTable({
    r <- rapor()
    data.frame(Metrik = names(r$metrik),
               Deger  = round(as.numeric(r$metrik[1, ]), 4))
  })

  output$cv_grafik <- renderPlot({
    t <- rapor()$cv_tablo
    t <- t[is.finite(t$rmse), ]
    plot(t$pencere, t$rmse, type = "b", pch = 19,
         xlab = "Pencere (ay)", ylab = "RMSE",
         main = "Örnek-dışı RMSE")
    points(rapor()$pencere, t$rmse[t$pencere == rapor()$pencere],
           col = "red", pch = 19, cex = 2)
  })

  output$tahmin_grafik <- renderPlot({
    t <- rapor()$tahmin
    plot(t$tarih, t$gercek, type = "l", lwd = 2,
         xlab = "Tarih", ylab = "Değer",
         main = paste(rapor()$hedef, "- Gerçek (siyah) vs Tahmin (kırmızı)"))
    lines(t$tarih, t$tahmin, col = "red", lwd = 2)
  })

  output$cv_tablo <- renderTable({ rapor()$cv_tablo }, digits = 4)
}
```

> **Otomasyonu kuran parçalar:**
> 
> - `reactive({...})` → bağımlı olduğu şeyler değişince **kendiliğinden** yeniden çalışan hesap. Elle çağırmazsın.
> - `invalidateLater(ms, session)` → **zamanlayıcı:** verilen süre sonra reaktifi "bayat" sayıp tetikler. Gerçek zamanlı güncellemenin benzetimi (önerideki cron job'un arayüz içi karşılığı).
> - `rapor` reaktifi üç şeye bağlı: `input$gosterge`, `input$yenile` (buton), `zamanlayici()`. Herhangi biri değişince → veri al + `model_calistir` → her çıktı tazelenir.
> - `veri_getir(g, zorla = ...)` → normalde cache'ten okur (hızlı); buton basılınca zorla API'den çeker (taze).
> - `withProgress(...)` → hesap sürerken kullanıcıya "çalışıyor..." gösterir.
> - CV grafiğinde seçilen pencere **kırmızı** vurgulanır — neden o pencere seçildi görsel olarak anlaşılır.

- [ ] Çalıştır: `shiny::runApp("app.R")`
- [ ] **Doğrula (otomasyon testleri):**
    - Açılışta **sen hiçbir şey yapmadan** ilk sonuç geldi (metrikler, grafikler doldu).
    - Göstergeyi USDTRY'ye çevir → panel kendiliğinden yeniden hesapladı, parametreler değişti.
    - "Otomatik yenileme"yi 1 dakikaya al, bekle → "Son güncelleme" zamanı kendiliğinden ilerledi.
    - "🔄 Şimdi Yenile"ye bas → taze veri çekildi (konsolda "API'den taze veri" mesajı).
    - CV grafiğindeki kırmızı nokta = seçilen pencere; CV tablosu tüm adayları gösteriyor.

---

## C5 — İnteraktif tablo ve indirme (cila)

Otomasyona görünürlük ve dışa aktarım ekleyelim: CV tablosunu sıralanabilir yap, tahminleri CSV indirilebilir yap.

- [ ] `ui`de `mainPanel` içine, en alta ekle:

```r
		hr(),
		downloadButton("indir", "Tahminleri CSV indir")
```

- [ ] `library(DT)`yi en üste ekle ve `cv_tablo` çıktısını DT'ye çevir (UI'da `tableOutput("cv_tablo")` yerine `DT::dataTableOutput("cv_tablo")`, server'da):

```r
output$cv_tablo <- DT::renderDataTable({
    DT::datatable(rapor()$cv_tablo, options = list(dom = "t"))
  })

  output$indir <- downloadHandler(
    filename = function() paste0("tahmin_", rapor()$hedef, "_",
                                 format(Sys.Date()), ".csv"),
    content  = function(file) write.csv(rapor()$tahmin, file, row.names = FALSE)
  )
```

> **Ne yaptı?** `DT::datatable` sıralanabilir, aranabilir bir tablo verir. `downloadHandler` → butona basınca tahmin tablosunu tarih damgalı bir CSV olarak indirir (önerideki "sonuçların paylaşılması" hedefi).

- [ ] **Doğrula:** CV tablosu artık interaktif; "Tahminleri CSV indir" butonu çalışan bir dosya veriyor.

## C6 — Tam `app.R` (birleşik kontrol)

- [ ] Yukarıdaki tüm parçaları tek `app.R`'da topladığından emin ol. Sıralama: `library(shiny); library(DT)` → `source("veri_altyapisi.R")` → `source("model.R")` → cache fonksiyonu (`veri_getir`) → `ui` → `server` → `shinyApp(ui, server)`.
- [ ] Son kez çalıştır ve baştan sona tüm akışı bir tur test et.
- [ ] **Doğrula:** Açılış otomatik sonuç → gösterge değişimi otomatik yeniden hesap → zamanlayıcı otomatik tazeleme → manuel yenile zorla çekme → CSV indirme. Hepsi çalışıyor.

---

## Kart C teslim (ve serinin tamamı)

- [ ] `app.R` — açılışta kendi verisini çeken (cache'li), p ve pencereyi otomatik seçen, zamanlayıcıyla kendini tazeleyen, CV'yi grafikleyen, CSV indirten otomasyonlu panel.
- [ ] `cache/veri_enflasyon.rds`, `cache/veri_usdtry.rds` — her gösterge için otomatik oluşan, birbirinden ayrı veri önbellekleri.

**Serinin bütünü:** Kart A veriyi hazırladı, Kart B modeli kurup CV ile pencere seçti, Kart C hepsini kendi kendine çalışan bir panele bağladı. Üçü birlikte, önerinin üç ana hedefini (Hedef 1 veri altyapısı, Hedef 2-3 model+performans, Hedef 4 otomatik arayüz) küçük ama **tam ve çalışan** bir maket olarak kanıtlar. Bu seriyi bitiren ekip, büyük projedeki her parçayı bir kez kendi eliyle kurmuş; herkes kendi takvim faaliyetini gerçek kodla yaşamış olur.

---

## Hata çıkarsa

- [ ] `could not find function "veri_hazirla"` / `model_calistir` → `source("veri_altyapisi.R")` ve `source("model.R")` satırlarını `app.R`'a koymadın ya da dosya adları farklı.
- [ ] Açılışta uzun donma → ilk çalıştırma CV yüzünden birkaç saniye sürer (normal); `withProgress` göstergesini bekle. Cache sonraki açılışları hızlandırır.
- [ ] Zamanlayıcı tetiklemiyor → `invalidateLater` içine `session` verdin mi; dakikayı çok büyük mü seçtin?
- [ ] "Şimdi Yenile" cache'i atlamıyor → `veri_getir(..., zorla = (input$yenile > 0))` bağlantısını kontrol et.
- [ ] `object 'USDTRY_lag1' not found` (gösterge değiştirince) → cache dosyası göstergeye göre ayrılmamış; `CACHE_DOSYA <- "cache/veri_enf.rds"` gibi sabit tek dosya kullanıyorsan `cache_dosya_yolu(hedef)` makinesine geç (C2). Eski/yanlış cache dosyaları kalmışsa `unlink("cache", recursive = TRUE)` ile temizleyip yeniden çalıştır.
- [ ] `there is no package called 'systemfonts'` / `'textshaping'` / `'ragg'` (grafiklerde) → bu bir R paket eksikliği, kod hatası değil. Konsolda `install.packages(c("systemfonts","textshaping","ragg"), type = "binary")` çalıştır, sonra R oturumunu (Session > Restart R) yeniden başlat.
- [ ] CSV boş → önce panel bir kez hesapladı mı (`rapor()` dolu mu)?
- [ ] DT tablosu görünmüyor → `library(DT)` üstte mi, UI'da `DT::dataTableOutput` kullandın mı?
- [ ] Değişiklik yansımıyor → `app.R`'ı kaydedip `runApp`'i yeniden başlat.