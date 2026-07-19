> Bu dosya, **KART C — Otomasyon & Shiny Paneli** kartındaki tüm alt-adımların sonunda ulaşılan **son hali**dir. Kart boyunca kod parça parça, alt-adımlarla inşa edildi; burada sadece çalışan **nihai sürüm** tek parça halinde veriliyor.
> 
> Kullanım: Aşağıdaki kod bloğunu olduğu gibi kopyalayıp `app.R` adlı bir dosyaya yapıştır. `veri_altyapisi.R` ve `model.R` ile **aynı klasörde** olmalı (dosyanın başı onları `source(...)` ile çağırıyor). Çalıştırmak için: `shiny::runApp("app.R")`.

```r
# =========================================================
# app.R
# KART C — Otomatik Ekonomik Tahmin Paneli
# =========================================================

library(shiny)
library(DT)

source("veri_altyapisi.R")   # Kart A fonksiyonları
source("model.R")            # Kart B fonksiyonları


# =========================================================
# C2 — Önbellekli veri çekme
# =========================================================
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


# =========================================================
# UI
# =========================================================
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
      DT::dataTableOutput("cv_tablo"),
      hr(),
      downloadButton("indir", "Tahminleri CSV indir")
    )
  )
)


# =========================================================
# SERVER
# =========================================================
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

  output$cv_tablo <- DT::renderDataTable({
    DT::datatable(rapor()$cv_tablo, options = list(dom = "t"))
  })

  output$indir <- downloadHandler(
    filename = function() paste0("tahmin_", rapor()$hedef, "_",
                                 format(Sys.Date()), ".csv"),
    content  = function(file) write.csv(rapor()$tahmin, file, row.names = FALSE)
  )
}

shinyApp(ui, server)
```

---

## Bu dosyadaki parçaların özeti

|Parça|Ne yapar|Girdi|Çıktı|
|---|---|---|---|
|`cache_dosya_yolu`|Her gösterge için ayrı bir cache dosya adı üretir (`cache/veri_enflasyon.rds` vb.); göstergeler arası veri karışmasını önler|hedef|dosya yolu (string)|
|`veri_getir`|Cache'ten okur (taze ve zorla değilse) ya da `veri_hazirla` ile API'den çekip cache'e yazar|hedef, zorla|Kart A'nın ürettiği temiz/aylık tablo|
|`ui`|Gösterge seçici, yenileme kaydırıcısı, "Şimdi Yenile" butonu, durum/parametre kutuları, metrik tablosu, CV grafiği, tahmin grafiği, CV tablosu (DT), CSV indir butonu|—|kullanıcının gördüğü arayüz|
|`zamanlayici` (reactive)|Seçilen dakikada bir kendini tetikleyen zamanlayıcı|`input$yenileme_dk`|`Sys.time()` (her tetiklemede günceli)|
|`rapor` (reactive)|Gösterge/buton/zamanlayıcı değişince veri çekip `model_calistir`'ı çalıştıran ana hesap|`input$gosterge`, `input$yenile`, `zamanlayici()`|Kart B'nin ürettiği tam rapor (`p`, `pencere`, `tahmin`, `metrik`, `cv_tablo`, `zaman`)|
|`output$*`|`rapor()`'daki bilgiyi metin/tablo/grafiğe döken render fonksiyonları|`rapor()`|ekrandaki her bir çıktı kutusu|
|`downloadHandler`|"Tahminleri CSV indir" butonuna basılınca tahmin tablosunu tarih damgalı CSV olarak indirir|`rapor()$tahmin`|`.csv` dosyası|

## Kullanım

```r
# app.R'ın bulunduğu klasörde (veri_altyapisi.R ve model.R de aynı klasörde olmalı):
shiny::runApp("app.R")
```

Bu tek satır, Kart A → Kart B → Kart C zincirinin tamamını başlatır: veri çekilir/cache'ten okunur, p ve pencere otomatik seçilir, sonuç panele basılır ve zamanlayıcıyla kendini tazeler.