# Neden Bu Dosya Yapısına İhtiyaç Duyuyoruz?

> Bu belge, `eco-forecast-ekk` projesindeki klasör ve dosya düzeninin **neden böyle olduğunu** anlatır. Kod yazmayı yeni öğrenen biri için yazılmıştır. Amaç "şunu yaz" demek değil, **"neden bu şekilde yazıyoruz"u** anlatmaktır.

---

## Önce büyük resim: Tek bir dev dosya neden olmaz?

Yeni başlayan herkesin aklına gelen ilk şey şudur:

> "Madem hepsi aynı projenin parçası, neden tek bir `proje.R` dosyasına yazmıyoruz? Her şey orada olsa, ararken de kolay olmaz mı?"

Çok mantıklı bir soru. Cevabı bir benzetmeyle verelim.

Bir **mutfağı** düşün. Her şeyi tek bir dolaba tıkıştırabilirsin: tabaklar, bıçaklar, baharatlar, çöp poşetleri, temizlik malzemeleri, hepsi aynı çekmecede. İlk gün belki çalışır. Ama:

- Bir bıçak ararken 200 parçanın arasında kaybolursun.
- Baharatı yenisiyle değiştirmek isteyince hangisi olduğunu bulamazsın.
- Bir arkadaşın "tuzluğu uzatır mısın" dediğinde tüm dolabı boşaltman gerekir.

İyi bir mutfakta her şeyin **bir yeri** vardır: bıçaklar bir çekmecede, baharatlar bir rafta, temizlik malzemeleri lavabonun altında. Bu düzen "süs" değildir — **bir şeyi bulmayı, değiştirmeyi ve birine açıklamayı kolaylaştırır.**

Yazılımda da aynısı geçerli. Kodu dosyalara bölmemizin tek bir sebebi var:

> **Her dosya tek bir işten sorumlu olsun ki; bir şeyi değiştireceğimizde, sadece o dosyaya bakalım, gerisine dokunmayalım.**

Buna programcılar **"sorumlulukların ayrılması"** (separation of concerns) der. Şimdi bu projede bu ilkenin nasıl uygulandığına dosya dosya bakalım.

---

## Projenin parçaları ve görevleri

Bu proje, ekonomik verileri internetten çekip, temizleyip, gelecek tahmini yapan ve bunu bir web ekranında gösteren bir sistem. Bu işi **5 adımda** yapıyor ve her adım kendi dosyasında yaşıyor:

| Dosya | Tek cümlelik görevi | Mutfak benzetmesi |
|-------|---------------------|-------------------|
| `config.R` | Tüm ayarları tutar | Tarif defteri / alışveriş listesi |
| `utils.R` | Tekrar tekrar kullanılan yardımcı araçlar | Kaşık, bıçak, ölçü kabı |
| `api_functions.R` | İnternetten ham veri çeker | Markete gidip malzeme almak |
| `data.prep.R` | Ham veriyi temizleyip kullanılır hale getirir | Sebzeleri yıkayıp doğramak |
| `models.R` | Temiz veriyle tahmin yapar | Yemeği pişirmek |
| `app/app.R` | Sonucu ekranda gösterir | Tabakta servis etmek |

Şimdi her birinin **neden ayrı bir dosya olması gerektiğini** tek tek görelim.

---

## 1. `config.R` — Neden ayarları ayrı tutuyoruz?

**Config**, "configuration" yani **ayarlar** demektir. Bu dosyada hiç iş yapılmaz; sadece projenin her yerinde kullanılacak **sabit bilgiler** durur:

- Hangi API adreslerine bağlanacağız?
- Hangi ekonomik göstergeleri çekeceğiz? (USD/TRY, enflasyon, faiz...)
- Model kaç ay geriye baksın? (lag sayısı)
- Verinin ne kadarı eğitim için kullanılsın? (`%80`)

### Neden bunları diğer kodun içine gömmüyoruz?

Diyelim ki tahmin penceresini "48 ay"dan "60 ay"a çıkarmak istiyorsun. Eğer bu sayı kodun 10 farklı yerine dağılmış olsaydı, 10 yeri tek tek bulup değiştirmen, birini unutursan da hata yaşaman gerekirdi.

Ama tek bir `config.R` dosyasında durursa:

> **Bir ayarı değiştirmek için tek bir satıra dokunursun, gerisi otomatik güncellenir.**

Buna programlamada **"sihirli sayılardan kaçınma"** denir. Kodun içine serpiştirilmiş, nereden geldiği belli olmayan sayılar ve adresler ("sihirli sayılar") bakım yapmayı zorlaştırır. Hepsini tek yere toplamak bu sorunu çözer.

### Bonus: Güvenlik

API anahtarları (şifreler gibi düşün) `config.R` üzerinden ama `.Renviron` adlı **gizli bir dosyadan** okunur. Anahtarı koda düz yazmıyoruz çünkü kod GitHub'a yüklenince herkesin eline geçer. Bu yüzden `.Renviron` dosyası `.gitignore` ile "asla yüklenmeyecekler" listesine konur.

> 🔑 **Altın kural:** Şifreler ve anahtarlar koda yazılmaz, ayrı gizli bir dosyada tutulur.

---

## 2. `utils.R` — Neden bir "yardımcı araçlar" dosyası?

**Utils**, "utilities" yani **yardımcı araçlar** demektir. Bu dosyada, projenin **birçok yerinde tekrar tekrar** ihtiyaç duyulan küçük araçlar bulunur:

- Ekrana ve dosyaya log (kayıt) yazma
- İnternetten çekilen veriyi diske kaydedip sonra geri okuma (cache)
- Bir model formülü kurma

### Neden bunları ayrı topladık?

Bir benzetme: Mutfakta **ölçü kabını** her tarifte kullanırsın — keki yaparken de, çorbayı yaparken de. Her seferinde yeni bir ölçü kabı icat etmezsin; bir tane var, herkes onu kullanır.

Yardımcı fonksiyonlar da öyle. `log_msg()` fonksiyonunu bir kez yazarsın, sonra `api_functions.R` de, `data.prep.R` de, `models.R` de onu çağırır. Eğer bu araçlar ayrı bir dosyada olmasaydı, aynı kodu her dosyaya kopyala-yapıştır yapman gerekirdi. Bu da şu kâbusa yol açar:

> Bir gün log biçimini değiştirmek istersin. Ama o kodu 5 farklı dosyaya kopyalamışsın. 5 yeri de tek tek düzeltmen, birini unutursan tutarsızlık yaşaman gerekir.

Programlamada bunun adı **DRY** ilkesidir: **D**on't **R**epeat **Y**ourself — "Kendini Tekrar Etme". Aynı işi yapan kodu tek bir yere koy, her yerden onu çağır.

---

## 3. `api_functions.R` — Neden "veri çekme" kendi dosyasında?

**API**, bir internet servisinden veri istemenin yoludur. Bu dosyanın tek görevi: **dış dünyadan ham veri getirmek.** Üç farklı kaynaktan veri çekiyoruz ve her birinin kuralı farklı:

- **FRED** (ABD merkezli) → anahtarı adresin içinde gönderir
- **EVDS** (TCMB) → anahtarı "başlık" (header) olarak gönderir
- **Dünya Bankası** → anahtar bile istemez

### Neden bu karmaşayı izole ediyoruz?

Bu dosya, projenin **dış dünyayla temas eden tek noktası**. İnternet kapris yapar: bağlantı kopar, servis çöker, veri eksik gelir. Tüm bu "kirli" ve güvenilmez işleri tek bir dosyaya hapsediyoruz ki, projenin geri kalanı temiz kalsın.

Buna **"sınır katmanı"** (boundary layer) denir. Bir benzetme: Eve giren herkesin **kapıda** ayakkabısını çıkarması gibi. Çamur kapıda kalır, evin içi temiz olur. `api_functions.R` projenin kapısıdır; dış dünyanın dağınıklığı orada durur.

### En güzel kısım: `fetch_indicator()`

Bu dosyadaki en akıllı fikir şu fonksiyon. Normalde bir veri çekmek isteyen kişinin şunu düşünmesi gerekirdi:

> "Enflasyon EVDS'den mi gelir, FRED'den mi? Hangi fonksiyonu çağırmalıyım?"

Ama `fetch_indicator("INFLATION")` dediğinde, fonksiyon `config.R`'a bakıp **doğru kaynağa kendisi yönlendirir.** Kullanan kişi kaynağı bilmek zorunda değildir.

Bu, iyi yazılımın özüdür: **karmaşıklığı saklamak.** Arabayı kullanırken motorun nasıl çalıştığını bilmen gerekmez; sadece gaza basarsın. `fetch_indicator` de öyle bir "gaz pedalı".

---

## 4. `data.prep.R` — Neden veriyi ayrıca temizliyoruz?

**Data prep**, "data preparation" yani **veri hazırlama** demektir. İnternetten gelen ham veri **doğrudan kullanılamaz**, çünkü şu sorunu vardır:

- USD/TRY **günlük** gelir (her gün bir değer)
- Enflasyon, faiz **aylık** gelir
- İşsizlik, büyüme **yıllık** gelir

Bunlar farklı takvimlerde olduğu için yan yana koyduğunda devasa boşluklar (NA'lar) oluşur. Bu dosyanın görevi:

1. Hepsini ortak bir **aylık takvime** indirgemek
2. Aradaki boşlukları **akıllıca doldurmak** (interpolasyon)
3. Modelin ihtiyacı olan ek sütunları **türetmek** (geçmiş değerler, fark, trend, mevsim)

### Neden bunu veri çekmekle aynı dosyaya koymuyoruz?

Çünkü bunlar **iki farklı iş**. Bir benzetme:

- Markete gidip sebze almak = `api_functions.R` (veri çekme)
- O sebzeleri yıkayıp, soyup, doğramak = `data.prep.R` (veri hazırlama)

Bunları ayırmanın somut faydası: Yarın enflasyon verisini başka bir kaynaktan çekmeye karar verirsen, sadece `api_functions.R`'ı değiştirirsin. **Temizleme mantığına hiç dokunman gerekmez**, çünkü o ayrı dosyada. Tersine, temizleme yöntemini değiştirmek istersen veri çekmeye dokunmazsın.

> İki iş ayrı dosyalardaysa, birini değiştirmek diğerini bozmaz.

---

## 5. `models.R` — Tahmin mantığı neden ayrı?

Bu dosya, projenin **asıl amacını** gerçekleştirir: temiz veriyle **gelecek tahmini** yapmak. Burada iki ana fikir var:

**AR-EKK modeli:** "Bir değişkenin yarınki değeri, büyük ölçüde geçmiş değerlerine bağlıdır" fikri. Bu ayki dolar kuru, geçen ayki kura çok benzer. Model bu ilişkiyi öğrenir.

**Kayan pencere (rolling window):** Tek bir sabit model yerine, **her ay son 48 ayın verisiyle yeni bir model kurup** bir sonraki ayı tahmin ederiz. Sonra pencereyi bir ay kaydırıp tekrarlarız. Böylece model, zamanla değişen ekonomik koşullara uyum sağlar.

### Neden ayrı dosya?

Çünkü tahmin yöntemi **en sık değişebilecek** kısımdır. Belki yarın AR-EKK yerine daha gelişmiş bir model denemek istersin. O zaman sadece `models.R`'ı açar, oradaki mantığı değiştirirsin. Veri çekme, temizleme ve arayüz **hiç etkilenmez** — onlar `models.R`'ın sadece sonucunu kullanır, içinde ne olduğunu bilmez.

Bu, iyi tasarımın bir sınavıdır: **"Bir parçayı söküp yenisini takabiliyor musun, gerisini bozmadan?"** Cevap evetse, dosyaları doğru ayırmışsın demektir.

---

## 6. `app/app.R` — Arayüz neden en sonda ve ayrı klasörde?

**Shiny**, R ile tarayıcıda çalışan interaktif web ekranları yapmanı sağlar. Bu dosya, tüm zincirin sonucunu **bir insana gösterir**: kullanıcı bir gösterge seçer, kayan pencere kaydırıcısını oynatır, grafiği ve tabloları görür.

### Neden ayrı bir `app/` klasöründe?

Çünkü arayüz, "işi yapan" kod değil, **"işi gösteren"** koddur. İkisini ayırmanın iki sebebi var:

1. **Arayüz değişse de motor aynı kalır.** Yarın grafiğin rengini değiştirmek istersen `models.R`'a dokunmazsın. Tahmin mantığı değişse de arayüzü baştan yazmazsın.
2. **Aynı motoru farklı yerlerde kullanabilirsin.** Bugün Shiny ekranı, yarın bir rapor, öbür gün bir e-posta. Hepsi aynı `models.R`'ı çağırır. Eğer tahmin mantığını arayüzün içine gömseydin, başka yerde kullanamazdın.

Buna **"mantık ve sunum ayrımı"** denir. Bir benzetme: Bir restoranda **mutfak** (yemeği yapan) ve **salon** (servis eden) ayrıdır. Garson değişse mutfak çalışmaya devam eder; menü değişse salon aynı kalır. `app.R` salondur, geri kalan dosyalar mutfaktır.

---

## Sıra neden önemli? (Bağımlılık zinciri)

Dosyaları her zaman **şu sırayla** yüklüyoruz:

```
config.R  →  utils.R  →  api_functions.R  →  data.prep.R  →  models.R
```

Bu sıra keyfi değil. Her dosya, kendinden **öncekilere muhtaç**. Buna **bağımlılık** (dependency) denir.

Bir benzetme: **Yemek tarifi.** Önce malzemeleri alırsın (config), sonra tezgâhı ve aletleri hazırlarsın (utils), sonra alışveriş yaparsın (api), sonra doğrarsın (data.prep), en sonunda pişirirsin (models). Pişirme adımını en başta yapamazsın — ortada malzeme yok!

Aynı şekilde:

- `utils.R`, `config.R`'daki ayarları kullanır → önce config gelmeli
- `api_functions.R`, hem config'in adreslerini hem utils'in cache aracını kullanır → ikisi de önce gelmeli
- `data.prep.R`, api'nin çektiği veriyi temizler → api önce gelmeli
- `models.R`, hazır veriyle tahmin yapar → hepsi önce gelmeli

### Bu yüzden her dosyanın en üstünde bir "bekçi" var

Dikkat edersen her dosyanın başında şuna benzer bir satır var:

```r
if (!exists("CONFIG")) stop("[data.prep.R] Önce config.R yükle")
```

Bu bir **güvenlik bekçisi**. Eğer dosyaları yanlış sırada yüklersen, kafası karışık bir hatayla saatlerce uğraşmak yerine, sana **net bir mesaj** verir: "Önce config'i yükle." Bu, eğitim materyali için çok değerli bir alışkanlıktır — hata mesajları açık ve yol gösterici olmalıdır.

---

## Destekleyici klasörler neden var?

Dosyaların yanında birkaç klasör de oluşturuyoruz. Her birinin sebebi var:

| Klasör | Neden var? |
|--------|------------|
| `R/` | Tüm kod dosyaları burada toplanır — kod ve veri karışmasın diye |
| `data/cache/` | İnternetten çekilen veri buraya kaydedilir; aynı veriyi tekrar tekrar çekip API'yi yormamak için |
| `data/processed/` | Temizlenmiş, kullanıma hazır veri burada durur; her seferinde baştan temizlememek için |
| `models/` | Tahmin sonuçlarının özeti buraya kaydedilir |
| `logs/` | Çalışma kayıtları buraya yazılır; bir şey ters giderse "ne oldu" diye buraya bakarsın |
| `tests/` | Kodun doğru çalıştığını otomatik kontrol eden dosyalar için ayrılmış yer |

**Cache** fikri özellikle güzel: Bir benzetme, markete her gidişinde aynı şeyi tekrar almak yerine **dolabına koymak** gibi. İkinci sefer dolaba bakarsın, market kapalıysa bile elinde malzeme olur. Cache de internet çalışmasa bile en son çektiğin veriyle devam etmeni sağlar.

---

## Özet: Tüm bu düzen tek bir şey için

Bu kadar dosya, klasör ve kuralın **tek bir amacı** var:

> **Değişimi kolaylaştırmak.**

Yazılım hiçbir zaman "bitmez". Her zaman bir ayar değişir, bir kaynak güncellenir, bir özellik eklenir. İyi dosya yapısı, bu değişimleri **korkmadan** yapabilmeni sağlar — çünkü her şey yerli yerindedir ve bir yeri değiştirmek başka yeri bozmaz.

Yeni başlayan biri için akılda tutulacak üç cümle:

1. **Her dosya tek bir işten sorumlu olmalı** (sorumlulukların ayrılması).
2. **Aynı kodu tekrar yazma, tek yere koy, her yerden çağır** (DRY).
3. **Ayarları, sırrı ve karmaşayı izole et** (config, gizli anahtarlar, sınır katmanı).

Bu üç ilkeyi kavradığında, sadece bu projeyi değil, karşına çıkacak her yazılım projesinin yapısını anlayabilirsin. Çünkü iyi projeler hep aynı sebeplerle aynı şekilde düzenlenir.
