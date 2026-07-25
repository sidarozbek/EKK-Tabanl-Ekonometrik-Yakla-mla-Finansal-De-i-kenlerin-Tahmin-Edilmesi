## Kavramsal Temellerden Uygulamaya ##

### Rastgele Değişken Kavramı

Bir **rastgele değişken (RV)**, bir deneyin olası sonuçlarına sayısal değerler atayan bir fonksiyondur:

- **Kesikli RV**: Örnek — bir zar atışının sonucu (1,2,...,6)
- **Sürekli RV**: Örnek — bir hisse senedinin günlük getirisi

Zaman serisi analizinde her bir gözlem (Y_t), zamanın belirli bir noktasında ortaya çıkan **ayrı bir rastgele değişken** olarak kabul edilir. Bu nedenle bir zaman serisi ({Y_1, Y_2, ..., Y_t}), rastgele değişkenlerin oluşturduğu bir dizi, yani bir **stokastik süreç** olarak tanımlanır.

### Beklenen Değer ve Doğrusallık Özelliği

**Beklenen değer**, bir rastgele değişkenin olasılık ağırlıklı ortalamasıdır:

```
E[X] = Σ xᵢ P(xᵢ)         (kesikli)
E[X] = ∫ x f(x) dx        (sürekli)
```

**Doğrusallık özelliği**, AR modelinin matematiksel iskeletini kurmamızı sağlayan en kritik özelliktir:

```
E[aX + bY + c] = aE[X] + bE[Y] + c
```

Bu özellik, X ve Y'nin **bağımsız olup olmadığına bakmaksızın** geçerlidir. AR(1) modelinde:

```
Yₜ = c + φ₁Yₜ₋₁ + εₜ
```

modelinin her iki tarafının beklenen değerini alırken doğrusallığı kullanırız:

```
E[Yₜ] = c + φ₁E[Yₜ₋₁] + E[εₜ]
```

### Varyans

Varyans, bir rastgele değişkenin ortalaması etrafındaki yayılımını ölçer:

```
Var(X) = E[(X − E[X])²] = E[X²] − (E[X])²
```

Doğrusal dönüşümler altında:

```
Var(aX + b) = a² Var(X)
```

İki bağımsız değişken toplamının varyansı:

```
Var(X + Y) = Var(X) + Var(Y)     (yalnızca Cov(X,Y) = 0 ise)
```

AR modelinde hata teriminin varyansı σ², sürecin ne kadar "gürültülü" olduğunu; Yₜ'nin varyansı ise sürecin toplam belirsizliğini gösterir. AR(1) için durağanlık altında:

```
Var(Yₜ) = σ² / (1 − φ₁²)
```

Bu formül, |φ₁| < 1 koşulunun (durağanlık koşulu) neden gerekli olduğunu da gösterir — aksi halde payda sıfır veya negatif olur.

### Kovaryans

Kovaryans, iki rastgele değişkenin birlikte nasıl değiştiğini ölçer:

```
Cov(X,Y) = E[(X − E[X])(Y − E[Y])] = E[XY] − E[X]E[Y]
```

Zaman serisinde bu kavram **otokovaryansa** dönüşür:

```
γ(k) = Cov(Yₜ, Yₜ₋ₖ) = E[(Yₜ − μ)(Yₜ₋ₖ − μ)]
```

Burada k gecikme (lag) sayısıdır. γ(k)'nın k'ya göre nasıl azaldığı, AR modelinin derecesi hakkında önemli ipuçları verir.

Kovaryansın matris cebirindeki genellemesi olan **kovaryans matrisi**, EKK tahmincisinin dağılımını ve BLUE özelliğini kanıtlarken doğrudan kullanılacaktır.

### Basit Doğrusal Regresyon Hatırlatması

Basit doğrusal regresyon modeli:

```
Yᵢ = β₀ + β₁Xᵢ + εᵢ
```

EKK (En Küçük Kareler / OLS) tahmincileri:

```
β̂₁ = Σ(Xᵢ − X̄)(Yᵢ − Ȳ) / Σ(Xᵢ − X̄)²  = Cov(X,Y) / Var(X)

β̂₀ = Ȳ − β̂₁X̄
```

AR(1) modeli aslında **Yₜ'nin Yₜ₋₁ üzerine basit regresyonundan başka bir şey değildir**:

```
Yₜ = c + φ₁Yₜ₋₁ + εₜ   ⟺   Y = β₀ + β₁X + ε
```

### Matris Cebiri Temelleri

Çoklu regresyon ve AR(p) modelleri, tek değişkenli formüllerle yönetilemeyecek kadar çok parametre içerir. Bu yüzden matris gösterimine geçilir.

**Transpoz**: Bir matrisin satır ve sütunlarının yer değiştirmesi. (Xᵀ)ᵢⱼ = Xⱼᵢ

Kurallar:
```
(A + B)ᵀ = Aᵀ + Bᵀ
(AB)ᵀ = BᵀAᵀ
```

**Matris tersi**: A kare ve tekil olmayan (non-singular) ise, AA⁻¹ = A⁻¹A = I olacak şekilde tek bir A⁻¹ vardır. EKK tahmincisinde (XᵀX)⁻¹ teriminin var olabilmesi için XᵀX'in **tam ranklı** olması gerekir — yani açıklayıcı değişkenler arasında tam çoklu doğrusal bağlantı (perfect multicollinearity) olmamalıdır.

**Türev kuralları** (vektör/matris kalkülüsü — EKK türetiminde doğrudan kullanılacak):

```
∂(Aβ)/∂β = Aᵀ

∂(βᵀAβ)/∂β = 2Aβ     (A simetrik ise)

∂(βᵀAᵀAβ)/∂β = 2AᵀAβ
```

## AR Modelinin Kurulması

### Tek Gecikmeli Model: AR(1)

AR(1) modeli, bir değişkenin bugünkü değerinin **kendi bir dönem önceki değerine** bağlı olduğu en basit zaman serisi modelidir:

```
Yₜ = c + φ₁Yₜ₋₁ + εₜ
```

- **c**: sabit terim
- **φ₁**: otoregresif katsayı (Yₜ₋₁'in Yₜ üzerindeki etkisi)
- **εₜ**: hata terimi (beyaz gürültü)

Bu model yapısı, basit doğrusal regresyon modeline benzer bir yapıya sahiptir. Aralarındaki temel fark, bağımsız değişkenin ((X_i)) dışarıdan alınan bir değişken yerine aynı zaman serisinin geçmiş değeri ((Y_{t-1})) olmasıdır. Bu nedenle AR (AutoRegressive) modeli, serinin kendi geçmiş değerleriyle kurulan bir regresyon modeli olduğu için **kendisiyle regresyon** olarak adlandırılır.

### Gecikme Kavramının Netleştirilmesi

**Gecikme (lag)**, bir serinin geçmiş bir zaman noktasındaki değerini ifade eder. **Gecikme operatörü (Lag/Backshift operator) L** şu şekilde tanımlanır:

```
LYₜ = Yₜ₋₁
L²Yₜ = Yₜ₋₂
LᵏYₜ = Yₜ₋ₖ
```

Bu operatör, AR modellerini kompakt biçimde yazmamızı sağlar. AR(1) modeli operatör gösterimiyle:

```
Yₜ = c + φ₁LYₜ + εₜ
(1 − φ₁L)Yₜ = c + εₜ
```

### 2.3 Genel Model: p'inci Dereceden AR(p)

AR(1)'in doğal genellemesi, birden fazla geçmiş değerin modele dahil edilmesidir:

```
Yₜ = c + φ₁Yₜ₋₁ + φ₂Yₜ₋₂ + ... + φₚYₜ₋ₚ + εₜ
```

Operatör gösterimiyle:

```
Φ(L)Yₜ = c + εₜ,     Φ(L) = 1 − φ₁L − φ₂L² − ... − φₚLᵖ
```

Bu model artık **çoklu doğrusal regresyon** yapısındadır — açıklayıcı değişkenler Yₜ₋₁, Yₜ₋₂, ..., Yₜ₋ₚ'di

### 2.4 Ortalamanın Türetilmesi (μ = c/(1−φ₁))

AR(1) sürecinin durağan (stationary) olduğunu varsayalım; yani E[Yₜ] = E[Yₜ₋₁] = μ (zamana bağlı olmayan sabit bir ortalama).

Modelin beklenen değerini alalım:

```
E[Yₜ] = E[c + φ₁Yₜ₋₁ + εₜ]
      = c + φ₁E[Yₜ₋₁] + E[εₜ]
```

εₜ'nin beklenen değeri 0 olduğundan ve durağanlık altında E[Yₜ] = E[Yₜ₋₁] = μ olduğundan:

```
μ = c + φ₁μ
μ − φ₁μ = c
μ(1 − φ₁) = c
```

Buradan:

```
μ = c / (1 − φ₁)
```

AR(p) için genelleştirme benzer mantıkla:

```
μ = c / (1 − φ₁ − φ₂ − ... − φₚ)
```

### Hata Terimi Varsayımları (εₜ ∼ N(0, σ²))

EKK tahmincisinin istatistiksel özelliklerinin (yansızlık, BLUE, güven aralıkları) geçerli olabilmesi için hata terimi üzerine şu varsayımlar yapılır:

1. **Sıfır ortalama**: E[εₜ] = 0 — sistematik bir yanlılık yoktur.
2. **Sabit varyans (homoskedastisite)**: Var(εₜ) = σ² (tüm t için sabit)
3. **Otokorelasyonsuzluk**: Cov(εₜ, εₛ) = 0, t ≠ s için — hatalar birbirinden bağımsızdır.
4. **Normallik**: εₜ ∼ N(0, σ²) — bu varsayım özellikle **olabilirlik fonksiyonu türetimi** ve hipotez testleri için gereklidir; EKK'nın kendisi (nokta tahmini için) normallik gerektirmez, ama çıkarım (inference) için gerekir.
5. **Dışsallık**: εₜ, Yₜ₋₁, Yₜ₋₂, ... geçmiş değerlerinden bağımsızdır (Cov(εₜ, Yₜ₋ₖ) = 0, k ≥ 1).

Bu beş varsayım topluca "beyaz gürültü + normallik" olarak anılır ve **Gauss-Markov teoreminin** dayanağını oluşturur.

### Basit Regresyondan AR(p)'ye Geçiş

| Basit Regresyon | AR(p) Karşılığı |
|---|---|
| Yᵢ = β₀ + β₁Xᵢ + εᵢ | Yₜ = c + φ₁Yₜ₋₁ + ... + φₚYₜ₋ₚ + εₜ |
| Tek açıklayıcı değişken X | p tane açıklayıcı değişken: Yₜ₋₁, ..., Yₜ₋ₚ |
| Gözlemler bağımsız (i.i.d. varsayımı) | Gözlemler zaman içinde bağımlı (seri kendisiyle ilişkili) |
| β̂ = Cov(X,Y)/Var(X) | β̂ = (XᵀX)⁻¹XᵀY (matris formunda) |

Tek fark, açıklayıcı değişkenlerin **rastgele örneklenmiş bağımsız veriler değil, aynı serinin gecikmeli halleri** olmasıdır. Bu durum, klasik regresyonun "gözlemler bağımsızdır" varsayımını ihlal eder — ancak EKK tahmincisi yine de tutarlı kalır, çünkü Yₜ₋ₖ ile εₜ arasında ilişki olmadığı sürec) matematiksel türetim aynı şekilde işler.

### Gauss-Markov Teoremi ve BLUE Özelliği

**Gauss-Markov Teoremi**: Eğer

1. E[ε] = 0
2. Var(ε) = σ²I (homoskedastisite + otokorelasyonsuzluk)
3. X ile ε ilişkisiz (dışsallık)
4. X tam ranklı

varsayımları sağlanıyorsa, EKK tahmincisi β̂ = (XᵀX)⁻¹XᵀY, β'nın **doğrusal ve yansız tahmincileri arasında en küçük varyansa sahip olanıdır**. Bu özellik **BLUE** (Best Linear Unbiased Estimator) olarak adlandırılır.

> **AR modeline özgü uyarı**: Klasik Gauss-Markov teoremi, X'in **sabit (deterministic)** olduğunu varsayar. AR modelinde X, geçmiş Y değerlerinden oluştuğu için **stokastiktir**. Bu durumda BLUE özelliği tam anlamıyla geçerli olmasa da, εₜ'nin geçmiş Yₜ₋ₖ değerlerinden bağımsız olması ( dışsallık varsayımı) koşuluyla EKK tahmincisi **tutarlı (consistent)** ve **asimptotik olarak yansız** kalır — büyük örneklemlerde benzer güvenilirlik sağlar.

## 4. Model Derecesinin (p) Belirlenmesi

### Az/Çok Gecikme Sorunu

AR(p) modelinde p'nin seçimi bir denge problemidir:

- **Çok az gecikme (p küçük)**: Model, seride gerçekte var olan otokorelasyon yapısını yakalayamaz. Bu durum **eksik belirtime (underfitting)** yol açar
- **Çok fazla gecikme (p büyük)**: Model, serinin yanı sıra rassal gürültüyü de öğrenmeye başlar. Bu durum **aşırı öğrenme (overfitting)** olarak adlandırılır.

Bu ikilem, "model karmaşıklığı ile veri uyumu arasındaki değiş tokuş" (bias-variance trade-off) probleminin AR modellerindeki özel görünümüdür.

### Olabilirlik Fonksiyonunun Kurulması

Normallik varsayımını (εₜ ∼ N(0,σ²)) kullanarak, Yₜ'nin geçmiş değerleri verildiğinde koşullu dağılımı da normaldir:

```
Yₜ | Yₜ₋₁,...,Yₜ₋ₚ ∼ N(c + φ₁Yₜ₋₁ + ... + φₚYₜ₋ₚ, σ²)
```

Tek bir gözlemin olabilirliği (normal dağılımın yoğunluk fonksiyonu):

```
f(Yₜ) = (1/√(2πσ²)) · exp(−εₜ²/(2σ²))
```

n adet gözlemin **birleşik olabilirlik fonksiyonu** (bağımsızlık varsayımı altında, hataların çarpımı):

```
L(β,σ²) = Π f(Yₜ) = (2πσ²)^(−n/2) · exp(−Σεₜ²/(2σ²))
```

Hesaplamayı kolaylaştırmak için **log-olabilirlik** alınır:

```
ln L(β,σ²) = −(n/2)ln(2π) − (n/2)ln(σ²) − (1/(2σ²))Σεₜ²
```

Bu ifadede Σεₜ² = SSE'dir. Dikkat edilirse, β'yı SSE'yi minimize edecek şekilde seçmek, log-olabilirliği maksimize etmekle **birebir aynı sonucu** verir — yani **normallik varsayımı altında EKK tahmincisi ile Maksimum Olabilirlik (MLE) tahmincisi çakışır**. Bu, EKK ve olabilirlik tabanlı kriterlerin (AIC/BIC) neden birlikte tutarlı çalıştığını açıklar.

σ²'nin MLE tahmincisi:

```
σ̂² = SSE/n = (1/n)Σε̂ₜ²
```

### AIC ve BIC Türetimi

Bilgi kriterleri, log-olabilirliği **parametre sayısına göre cezalandırarak** karşılaştırılabilir bir skor üretir. Genel form:

```
Kriter = −2 ln L(β̂, σ̂²) + Ceza(k, n)
```

Log-olabilirliğin maksimum noktasında (σ̂² = SSE/n yerine konursa) şu sadeleştirilmiş ifadeye ulaşılır:

```
−2 ln L̂ = n ln(σ̂²) + n ln(2π) + n
```

n ln(2π) + n terimi, tüm modeller için **sabit** olduğundan (modeller arası karşılaştırmada rol oynamaz), pratikte şu forma indirgenir:

```
−2 ln L̂  ≈  n ln(σ̂²)  + sabit
```

**Akaike Bilgi Kriteri (AIC)**:

```
AIC(p) = n ln(σ̂²) + 2k
```

**Bayesian Bilgi Kriteri (BIC / Schwarz Kriteri)**:

```
BIC(p) = n ln(σ̂²) + k ln(n)
```

**AIC ile BIC arasındaki temel fark**, ceza teriminde yatar:

| Kriter | Ceza terimi | Örneklem büyüklüğüyle davranış |
|---|---|---|
| AIC | 2k | n'den bağımsız, sabit ceza |
| BIC | k·ln(n) | n arttıkça ceza büyür |

Bu yüzden **BIC, büyük örneklemlerde AIC'den daha "cimri" (parsimonious)** modeller seçme eğilimindedir — yani daha düşük p tercih eder. AIC ise göreceli olarak daha karmaşık (yüksek p) modellere daha toleranslıdır ve **tahmin (forecasting) performansı** açısından genelde tercih edilir; BIC ise **doğru modeli bulma (model seçim tutarlılığı)** açısından asimptotik olarak daha güçlü bir teorik özelliğe (consistency) sahiptir.

### 4.4 Karar Kuralı

Model derecesi seçim prosedürü:

1. Bir üst sınır p_max belirlenir
2. p = 1'den p_max'a kadar her bir p değeri için AR(p) modeli EKK ile tahmin edilir.
3. Her model için AIC(p) ve BIC(p) hesaplanır.
4. **Karar kuralı: AIC(p) veya BIC(p) değerini minimize eden p seçilir.**

```
p* = argmin_p  AIC(p)      veya      p* = argmin_p  BIC(p)
```

## 5. Kayan Pencere Yaklaşımı

### 5.1 Sabit Modelin Yetersizliği

β̂ = (XᵀX)⁻¹XᵀY, elimizdeki **tüm örneklemi tek seferde kullanarak** sabit bir φ̂ katsayı seti üretir. Bu yaklaşımın örtük varsayımı, sürecin **parametrelerinin zaman içinde değişmediğidir** (parameter stability / stationarity of coefficients).

Gerçek dünya verilerinde bu varsayım sıklıkla ihlal edilir:

- **Yapısal kırılmalar (structural breaks)**: Bir ekonomik kriz, politika değişikliği veya rejim geçişi, φ katsayılarının değerini kalıcı olarak değiştirebilir.
- **Zamanla değişen dinamikler (time-varying dynamics)**: Bazı süreçlerde otokorelasyon yapısı yavaşça evrilir (örneğin volatilite kümelenmesi gösteren finansal seriler).
- **Rejim değişimleri**: Seri, farklı dönemlerde farklı istatistiksel davranışlar sergileyebilir (yüksek volatilite / düşük volatilite rejimleri gibi).

Tüm örneklemle tahmin edilen **sabit (tek) bir model**, bu değişimleri ortalayarak "yumuşatır" — bu da hem yakın geçmişteki dinamikleri tam yansıtamaz hem de gelecek tahminlerinde sistematik hatalara yol açar. Çözüm, modeli **belirli bir zaman penceresi içindeki veriyle tekrar tekrar tahmin etmektir** — bu da bizi kayan pencere (rolling window) yaklaşımına götürür.

### 5.2 Pencere Mekanizmasının İşleyişi

Kayan pencere yaklaşımında, sabit uzunlukta bir **pencere (window)**, w gözlem içerecek şekilde tanımlanır. Model, bu pencere içindeki veriyle tahmin edilir; sonra pencere bir adım ileri kaydırılır (en eski gözlem düşürülür, en yeni gözlem eklenir) ve model **yeniden tahmin edilir**.

Matematiksel olarak, t anındaki pencere:

```
Pencere_t = {Y_{t-w+1}, Y_{t-w+2}, ..., Y_t}
```

Bu pencere kullanılarak β̂_t = (XᵀX)⁻¹XᵀY hesaplanır. Sonraki adımda:

```
Pencere_{t+1} = {Y_{t-w+2}, Y_{t-w+3}, ..., Y_{t+1}}
```

Bu, "kayan (rolling)" pencere olarak adlandırılır çünkü pencere **sabit genişlikte** kalır, sadece konumu kayar.
Her pencerede yeniden tahmin edilen β̂_t, o zaman dilimine **özgü** katsayıları temsil eder ve böylece modelin zamanla değişen dinamiklere uyum sağlamasına (adaptivity) imkan tanır.

### 5.3 Örnek Dışı Tahminin Metodolojik Önemi

Kayan pencere yaklaşımının asıl gücü, **örnek dışı (out-of-sample) tahmin** üretmeye doğal olarak uygun olmasıdır:

1. Pencere_t içindeki veriyle model tahmin edilir (β̂_t elde edilir).
2. Bu model kullanılarak **pencerenin dışındaki** bir sonraki gözlem için tahmin üretilir: Ŷ_{t+1} = ĉ_t + φ̂₁,ₜY_t + ... + φ̂ₚ,ₜY_{t-p+1}
3. Gerçekleşen Y_{t+1} değeri ile Ŷ_{t+1} karşılaştırılarak tahmin hatası hesaplanır: e_{t+1} = Y_{t+1} − Ŷ_{t+1}
4. Pencere kaydırılır, süreç tekrarlanır.

### 5.4 Pencere Uzunluğunun Seçimi

Pencere uzunluğu w'nin seçimi, kendi başına bir denge (trade-off) problemidir:

- **Kısa pencere (w küçük)**:
  - (+) Modele en güncel dinamikleri daha hızlı yansıtır, yapısal değişimlere daha çabuk adapte olur.
  - (−) Az gözlemle tahmin edildiği için β̂'nin varyansı artr ;ayrıca p büyükse (XᵀX)⁻¹'in hesaplanabilmesi için w ≥ p+1 zorunludur, w küçüldükçe bu sınıra yaklaşılabilir.

- **Uzun pencere (w büyük)**:
  - (+) Daha fazla gözlem, daha düşük tahmin varyansı, daha istikrarlı katsayılar.
  - (−) Eski (artık geçerliliğini yitirmiş) veriyi de içerdiğinden, yapısal kırılmalara veya rejim değişimlerine yavaş tepki verir; aşırı uzun pencere, aslında Bölüm 5.1'de eleştirilen "sabit model" durumuna geri döner.

## Tahmin Performansı Metrikleri

Üretilen örnek dışı tahmin hataları (e_t = Y_t − Ŷ_t), tek tek incelendiğinde yorumlanması zor, dağınık sayılardır. Bu bölümde, bu hataları **tek bir özet sayıya** indirgeyen üç temel metrik ele alınacaktır. n_test, örnek dışı tahmin yapılan gözlem sayısını göstersin.

### MAE (Mean Absolute Error / Ortalama Mutlak Hata)

```
MAE = (1/n_test) Σ|Yₜ − Ŷₜ|
```

**Özellikleri**:
- Hataların **mutlak değerini** aldığı için pozitif ve negatif hatalar birbirini götürmez.
- Tüm hatalara **eşit ağırlık** verir — büyük bir hata ile küçük bir hata, hatanın büyüklüğüyle **doğru orantılı** şekilde etkiler.
- **Aykırı değerlere (outliers) karşı görece dayanıklıdır** (RMSE'ye kıyasla), çünkü büyük hatalar karesi alınmadan toplanır.

### RMSE (Root Mean Squared Error / Karekök Ortalama Kare Hata)

```
RMSE = √[ (1/n_test) Σ(Yₜ − Ŷₜ)² ]
```

**Özellikleri**:
- Hataların **karesini** aldığı için büyük hatalar **orantısız şekilde daha ağır cezalandırılır**.
- Bu nedenle RMSE, **büyük/nadir hatalardan kaçınmanın kritik olduğu** uygulamalarda tercih edilir.
- Aykırı değerlere karşı **MAE'den daha hassastır** — bu bazen istenen bir özellik (büyük hataları öne çıkarmak), bazen bir dezavantajdır (tek bir aşırı gözlem metriği domine edebilir).

### 6.3 MAPE (Mean Absolute Percentage Error / Ortalama Mutlak Yüzde Hata)

```
MAPE = (1/n_test) Σ |（Yₜ − Ŷₜ) / Yₜ| × 100
```

**Özellikleri**:
- Hatayı gerçek değere **oranlayarak yüzdesel** bir ölçüt üretir — bu da **farklı ölçeklerdeki serileri karşılaştırmayı** mümkün kılar.
- **Kısıtlar**:
  - Yₜ = 0 (veya sıfıra çok yakın) olduğunda tanımsız veya aşırı büyük değerler üretir — payda sıfıra yaklaştıkça patlar.
  - Negatif değerler içeren serilerde yorumlanması problemlidir.
  
### 6.4 Metriklerin Karşılaştırılması ve Birlikte Kullanımı

| Metrik | Birim | Büyük hatalara duyarlılık | Ölçekler arası karşılaştırma | Sıfıra yakın değerlerde |
|---|---|---|---|---|
| MAE | Orijinal birim | Düşük (doğrusal ceza) | Uygun değil | Sorun yok |
| RMSE | Orijinal birim | Yüksek (kareli ceza) | Uygun değil | Sorun yok |
| MAPE | Yüzde (%) | Orta (oransal) | Uygun | Tanımsız/patlar |

- **MAE ve RMSE birlikte** yorumlandığında, aralarındaki fark bilgi verir: RMSE, MAE'ye çok yakınsa hatalar **homojen** dağılmıştır (aşırı sapma yoktur); RMSE, MAE'den belirgin şekilde büyükse, veri setinde **bazı büyük hatalar (outlier tahminler)** olduğuna işaret eder.
