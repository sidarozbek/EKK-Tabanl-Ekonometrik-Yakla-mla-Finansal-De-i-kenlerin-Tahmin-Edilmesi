## Kavramsal Temellerden Uygulamaya ##

### Rastgele Değişken Kavramı

Bir **rastgele değişken (RV)**, bir deneyin olası sonuçlarına sayısal değerler atayan bir fonksiyondur:

- **Kesikli RV**: Örnek — bir zar atışının sonucu (1,2,...,6)
- **Sürekli RV**: Örnek — bir hisse senedinin günlük getirisi

Zaman serisi analizinde her bir gözlem $Y_t$, zamanın belirli bir noktasında ortaya çıkan **ayrı bir rastgele değişken** olarak kabul edilir. Bu nedenle bir zaman serisi $\{Y_1, Y_2, \ldots, Y_t\}$, rastgele değişkenlerin oluşturduğu bir dizi, yani bir **stokastik süreç** olarak tanımlanır.

### Beklenen Değer ve Doğrusallık Özelliği

**Beklenen değer**, bir rastgele değişkenin olasılık ağırlıklı ortalamasıdır:

$$
E[X] = \sum_{i} x_i P(x_i) \qquad \text{(kesikli)}
$$

$$
E[X] = \int_{-\infty}^{\infty} x\,f(x)\,dx \qquad \text{(sürekli)}
$$

**Doğrusallık özelliği**, AR modelinin matematiksel iskeletini kurmamızı sağlayan en kritik özelliktir:

$$
E[aX + bY + c] = aE[X] + bE[Y] + c
$$

Bu özellik, X ve Y'nin **bağımsız olup olmadığına bakmaksızın** geçerlidir. 

AR(1) modelinde:

$$
Y_t = c + \phi_1 Y_{t-1} + \varepsilon_t
$$

modelinin her iki tarafının beklenen değerini alırken doğrusallığı kullanırız:

$$
E[Y_t] = c + \phi_1 E[Y_{t-1}] + E[\varepsilon_t]
$$

### Varyans

Varyans, bir rastgele değişkenin ortalaması etrafındaki yayılımını ölçer:

$$
Var(X) = E[(X - E[X])^2] = E[X^2] - (E[X])^2
$$

Doğrusal dönüşümler altında:

$$
Var(aX+b)=a^2 Var(X)
$$

İki bağımsız değişken toplamının varyansı:

$$
Var(X+Y)=Var(X)+Var(Y) \qquad \text{(yalnızca } Cov(X,Y)=0 \text{ ise)}
$$

AR modelinde hata teriminin varyansı $\sigma^2$, sürecin ne kadar "gürültülü" olduğunu; $Y_t$'nin varyansı ise sürecin toplam belirsizliğini gösterir. 

AR(1) için durağanlık altında:

$$
Var(Y_t)=\frac{\sigma^2}{1-\phi_1^2}
$$

Bu formül, $|\phi_1| < 1$ koşulunun (durağanlık koşulu) neden gerekli olduğunu da gösterir — aksi halde payda sıfır veya negatif olur.

### Kovaryans

Kovaryans, iki rastgele değişkenin birlikte nasıl değiştiğini ölçer:

$$
Cov(X,Y)=E[(X-E[X])(Y-E[Y])] = E[XY]-E[X]E[Y]
$$

Zaman serisinde bu kavram **otokovaryansa** dönüşür:

$$
\gamma(k)=Cov(Y_t,Y_{t-k})=E[(Y_t-\mu)(Y_{t-k}-\mu)]
$$

Burada $k$ gecikme (lag) sayısıdır. $\gamma(k)$'nın $k$'ya göre nasıl azaldığı, AR modelinin derecesi hakkında önemli ipuçları verir.

Kovaryansın matris cebirindeki genellemesi olan **kovaryans matrisi**, EKK tahmincisinin dağılımını ve BLUE özelliğini kanıtlarken doğrudan kullanılacaktır.

### Basit Doğrusal Regresyon Hatırlatması

Basit doğrusal regresyon modeli:

$$
Y_i=\beta_0+\beta_1X_i+\varepsilon_i
$$

EKK (En Küçük Kareler / OLS) tahmincileri:

\[
\hat{\beta}_1
=
\frac{\sum_{i=1}^{n}(X_i-\bar{X})(Y_i-\bar{Y})}
{\sum_{i=1}^{n}(X_i-\bar{X})^2}
\]

\[
\hat{\beta}_1
=
\frac{\operatorname{Cov}(X,Y)}
{\operatorname{Var}(X)}
\]

\[
\hat{\beta}_1
=
\frac{S_{XY}}{S_X^2}
\]

$$
\hat{\beta}_0=\bar{Y}-\hat{\beta}_1\bar{X}
$$

AR(1) modeli aslında **$Y_t$'nin $Y_{t-1}$ üzerine basit regresyonundan başka bir şey değildir**:

$$
Y_t=c+\phi_1Y_{t-1}+\varepsilon_t
\qquad \Longleftrightarrow \qquad
Y=\beta_0+\beta_1X+\varepsilon
$$

### Matris Cebiri Temelleri

Çoklu regresyon ve AR(p) modelleri, tek değişkenli formüllerle yönetilemeyecek kadar çok parametre içerir. Bu yüzden matris gösterimine geçilir.

**Transpoz**: Bir matrisin satır ve sütunlarının yer değiştirmesi.

$$
(X^T)_{ij}=X_{ji}
$$

Kurallar:

$$
(A+B)^T=A^T+B^T
$$

$$
(AB)^T=B^T A^T
$$

**Matris tersi**: A kare ve tekil olmayan (non-singular) ise,

$$
AA^{-1} = A^{-1}A = I
$$

olacak şekilde tek bir $A^{-1}$ vardır. EKK tahmincisinde $(X^TX)^{-1}$ teriminin var olabilmesi için $X^TX$'in **tam ranklı** olması gerekir — yani açıklayıcı değişkenler arasında tam çoklu doğrusal bağlantı (perfect multicollinearity) olmamalıdır.

**Türev kuralları** (vektör/matris kalkülüsü — EKK türetiminde doğrudan kullanılacak):

$$
\frac{\partial(A\beta)}{\partial\beta}=A^T
$$

$$
\frac{\partial(\beta^T A\beta)}{\partial\beta}=2A\beta
\qquad \text{(A simetrik ise)}
$$

$$
\frac{\partial(\beta^T A^T A\beta)}{\partial\beta}=2A^T A\beta
$$

## AR Modelinin Kurulması

### Tek Gecikmeli Model: AR(1)

AR(1) modeli, bir değişkenin bugünkü değerinin **kendi bir dönem önceki değerine** bağlı olduğu en basit zaman serisi modelidir:

$$
Y_t = c + \phi_1 Y_{t-1} + \varepsilon_t
$$

- **c**: sabit terim
- **$\phi_1$**: otoregresif katsayı ($Y_{t-1}$'in $Y_t$ üzerindeki etkisi)
- **$\varepsilon_t$**: hata terimi (beyaz gürültü)

Bu model yapısı, basit doğrusal regresyon modeline benzer bir yapıya sahiptir. Aralarındaki temel fark, bağımsız değişkenin ($X_i$) dışarıdan alınan bir değişken yerine aynı zaman serisinin geçmiş değeri ($Y_{t-1}$) olmasıdır. Bu nedenle AR (AutoRegressive) modeli, serinin kendi geçmiş değerleriyle kurulan bir regresyon modeli olduğu için **kendisiyle regresyon** olarak adlandırılır.

### Gecikme Kavramının Netleştirilmesi

**Gecikme (lag)**, bir serinin geçmiş bir zaman noktasındaki değerini ifade eder. 

**Gecikme operatörü (Lag/Backshift operator) L** şu şekilde tanımlanır:

$$
LY_t = Y_{t-1}
$$

$$
L^2Y_t = Y_{t-2}
$$

$$
L^kY_t = Y_{t-k}
$$

Bu operatör, AR modellerini kompakt biçimde yazmamızı sağlar. AR(1) modeli operatör gösterimiyle:

$$
Y_t = c + \phi_1 LY_t + \varepsilon_t
$$

$$
(1 - \phi_1 L)Y_t = c + \varepsilon_t
$$

### Genel Model: p'inci Dereceden AR(p)

AR(1)'in doğal genellemesi, birden fazla geçmiş değerin modele dahil edilmesidir:

$$
Y_t = c + \phi_1 Y_{t-1} + \phi_2 Y_{t-2} + \ldots + \phi_p Y_{t-p} + \varepsilon_t
$$

Operatör gösterimiyle:

$$
\Phi(L)Y_t = c + \varepsilon_t, \qquad \Phi(L) = 1 - \phi_1 L - \phi_2 L^2 - \ldots - \phi_p L^p
$$

Bu model artık **çoklu doğrusal regresyon** yapısındadır — açıklayıcı değişkenler $Y_{t-1}, Y_{t-2}, \ldots, Y_{t-p}$'dir.

### 2.4 Ortalamanın Türetilmesi $\mu=\frac{c}{1-\phi_1}$

AR(1) sürecinin durağan (stationary) olduğunu varsayalım; yani $E[Y_t] = E[Y_{t-1}] = \mu$ (zamana bağlı olmayan sabit bir ortalama).

Modelin beklenen değerini alalım:

$$
E[Y_t] = E[c + \phi_1 Y_{t-1} + \varepsilon_t] = c + \phi_1 E[Y_{t-1}] + E[\varepsilon_t]
$$

$\varepsilon_t$'nin beklenen değeri 0 olduğundan ve durağanlık altında $E[Y_t] = E[Y_{t-1}] = \mu$ olduğundan:

$$
\mu = c + \phi_1 \mu
$$

$$
\mu - \phi_1 \mu = c
$$

$$
\mu(1 - \phi_1) = c
$$

Buradan:

$$
\mu = \frac{c}{1 - \phi_1}
$$

AR(p) için genelleştirme benzer mantıkla:

$$
\mu = \frac{c}{1 - \phi_1 - \phi_2 - \ldots - \phi_p}
$$

### Hata Terimi Varsayımları ($\varepsilon_t \sim N(0, \sigma^2)$)

EKK tahmincisinin istatistiksel özelliklerinin (yansızlık, BLUE, güven aralıkları) geçerli olabilmesi için hata terimi üzerine şu varsayımlar yapılır:

1. **Sıfır ortalama**: $E[\varepsilon_t] = 0$ — sistematik bir yanlılık yoktur.
2. **Sabit varyans (homoskedastisite)**: $Var(\varepsilon_t) = \sigma^2$ (tüm t için sabit)
3. **Otokorelasyonsuzluk**: $Cov(\varepsilon_t, \varepsilon_s) = 0$, $t \neq s$ için — hatalar birbirinden bağımsızdır.
4. **Normallik**: $\varepsilon_t \sim N(0, \sigma^2)$ — bu varsayım özellikle **olabilirlik fonksiyonu türetimi** ve hipotez testleri için gereklidir; EKK'nın kendisi (nokta tahmini için) normallik gerektirmez, ama çıkarım (inference) için gerekir.
5. **Dışsallık**: $\varepsilon_t$, $Y_{t-1}, Y_{t-2}, \ldots$ geçmiş değerlerinden bağımsızdır ($Cov(\varepsilon_t, Y_{t-k}) = 0$, $k \geq 1$).

Bu beş varsayım topluca "beyaz gürültü + normallik" olarak anılır ve **Gauss-Markov teoreminin** dayanağını oluşturur.

### Basit Regresyondan AR(p)'ye Geçiş

| Basit Regresyon | AR(p) Karşılığı |
|---|---|
| $Y_i = \beta_0 + \beta_1 X_i + \varepsilon_i$ | $Y_t = c + \phi_1 Y_{t-1} + \ldots + \phi_p Y_{t-p} + \varepsilon_t$ |
| Tek açıklayıcı değişken $X$ | p tane açıklayıcı değişken: $Y_{t-1}, \ldots, Y_{t-p}$ |
| Gözlemler bağımsız (i.i.d. varsayımı) | Gözlemler zaman içinde bağımlı (seri kendisiyle ilişkili) |
| $\hat{\beta} = Cov(X,Y)/Var(X)$ | $\hat{\beta} = (X^TX)^{-1}X^TY$ (matris formunda) |

Tek fark, açıklayıcı değişkenlerin **rastgele örneklenmiş bağımsız veriler değil, aynı serinin gecikmeli halleri** olmasıdır. Bu durum, klasik regresyonun "gözlemler bağımsızdır" varsayımını ihlal eder — ancak EKK tahmincisi yine de tutarlı kalır, çünkü $Y_{t-k}$ ile $\varepsilon_t$ arasında ilişki olmadığı sürece matematiksel türetim aynı şekilde işler.

### Gauss-Markov Teoremi ve BLUE Özelliği

**Gauss-Markov Teoremi**: Eğer

1. $E[\varepsilon] = 0$
2. $Var(\varepsilon) = \sigma^2 I$ (homoskedastisite + otokorelasyonsuzluk)
3. $X$ ile $\varepsilon$ ilişkisiz (dışsallık)
4. $X$ tam ranklı

varsayımları sağlanıyorsa, EKK tahmincisi

$$
\hat{\beta} = (X^TX)^{-1}X^TY
$$

$\beta$'nın **doğrusal ve yansız tahmincileri arasında en küçük varyansa sahip olanıdır**. Bu özellik **BLUE** (Best Linear Unbiased Estimator) olarak adlandırılır.

> **AR modeline özgü uyarı**: Klasik Gauss-Markov teoremi, X'in **sabit (deterministic)** olduğunu varsayar. AR modelinde X, geçmiş Y değerlerinden oluştuğu için **stokastiktir**. Bu durumda BLUE özelliği tam anlamıyla geçerli olmasa da, $\varepsilon_t$'nin geçmiş $Y_{t-k}$ değerlerinden bağımsız olması (dışsallık varsayımı) koşuluyla EKK tahmincisi **tutarlı (consistent)** ve **asimptotik olarak yansız** kalır — büyük örneklemlerde benzer güvenilirlik sağlar.

## Model Derecesinin (p) Belirlenmesi

### Az/Çok Gecikme Sorunu

AR(p) modelinde p'nin seçimi bir denge problemidir:

- **Çok az gecikme (p küçük)**: Model, seride gerçekte var olan otokorelasyon yapısını yakalayamaz. Bu durum **eksik belirtime (underfitting)** yol açar
- **Çok fazla gecikme (p büyük)**: Model, serinin yanı sıra rassal gürültüyü de öğrenmeye başlar. Bu durum **aşırı öğrenme (overfitting)** olarak adlandırılır.

Bu ikilem, "model karmaşıklığı ile veri uyumu arasındaki değiş tokuş" (bias-variance trade-off) probleminin AR modellerindeki özel görünümüdür.

### Olabilirlik Fonksiyonunun Kurulması

Normallik varsayımını ($\varepsilon_t \sim N(0,\sigma^2)$) kullanarak, $Y_t$'nin geçmiş değerleri verildiğinde koşullu dağılımı da normaldir:

$$
Y_t \mid Y_{t-1},\ldots,Y_{t-p} \sim N(c + \phi_1 Y_{t-1} + \ldots + \phi_p Y_{t-p},\ \sigma^2)
$$

Tek bir gözlemin olabilirliği (normal dağılımın yoğunluk fonksiyonu):

$$
f(Y_t) = \frac{1}{\sqrt{2\pi\sigma^2}} \cdot \exp\left(-\frac{\varepsilon_t^2}{2\sigma^2}\right)
$$

n adet gözlemin **birleşik olabilirlik fonksiyonu** (bağımsızlık varsayımı altında, hataların çarpımı):

$$
L(\beta,\sigma^2) = \prod_{t} f(Y_t) = (2\pi\sigma^2)^{-n/2} \cdot \exp\left(-\frac{\sum \varepsilon_t^2}{2\sigma^2}\right)
$$

Hesaplamayı kolaylaştırmak için **log-olabilirlik** alınır:

$$
\ln L(\beta,\sigma^2) = -\frac{n}{2}\ln(2\pi) - \frac{n}{2}\ln(\sigma^2) - \frac{1}{2\sigma^2}\sum \varepsilon_t^2
$$

Bu ifadede $\sum \varepsilon_t^2 = SSE$'dir. Dikkat edilirse, $\beta$'yı SSE'yi minimize edecek şekilde seçmek, log-olabilirliği maksimize etmekle **birebir aynı sonucu** verir — yani **normallik varsayımı altında EKK tahmincisi ile Maksimum Olabilirlik (MLE) tahmincisi çakışır**. Bu, EKK ve olabilirlik tabanlı kriterlerin (AIC/BIC) neden birlikte tutarlı çalıştığını açıklar.

$\sigma^2$'nin MLE tahmincisi:

$$
\hat{\sigma}^2 = \frac{SSE}{n} = \frac{1}{n}\sum \hat{\varepsilon}_t^2
$$

### AIC ve BIC Türetimi

Bilgi kriterleri, log-olabilirliği **parametre sayısına göre cezalandırarak** karşılaştırılabilir bir skor üretir. Genel form:

$$
\text{Kriter} = -2\ln L(\hat{\beta}, \hat{\sigma}^2) + \text{Ceza}(k, n)
$$

Log-olabilirliğin maksimum noktasında ($\hat{\sigma}^2 = SSE/n$ yerine konursa) şu sadeleştirilmiş ifadeye ulaşılır:

$$
-2\ln \hat{L} = n\ln(\hat{\sigma}^2) + n\ln(2\pi) + n
$$

$n\ln(2\pi) + n$ terimi, tüm modeller için **sabit** olduğundan (modeller arası karşılaştırmada rol oynamaz), pratikte şu forma indirgenir:

$$
-2\ln \hat{L} \approx n\ln(\hat{\sigma}^2) + \text{sabit}
$$

**Akaike Bilgi Kriteri (AIC)**:

$$
AIC(p) = n\ln(\hat{\sigma}^2) + 2k
$$

**Bayesian Bilgi Kriteri (BIC / Schwarz Kriteri)**:

$$
BIC(p) = n\ln(\hat{\sigma}^2) + k\ln(n)
$$

**AIC ile BIC arasındaki temel fark**, ceza teriminde yatar:

| Kriter | Ceza terimi | Örneklem büyüklüğüyle davranış |
|---|---|---|
| AIC | $2k$ | n'den bağımsız, sabit ceza |
| BIC | $k\ln(n)$ | n arttıkça ceza büyür |

Bu yüzden **BIC, büyük örneklemlerde AIC'den daha "cimri" (parsimonious)** modeller seçme eğilimindedir — yani daha düşük p tercih eder. AIC ise göreceli olarak daha karmaşık (yüksek p) modellere daha toleranslıdır ve **tahmin (forecasting) performansı** açısından genelde tercih edilir; BIC ise **doğru modeli bulma (model seçim tutarlılığı)** açısından asimptotik olarak daha güçlü bir teorik özelliğe (consistency) sahiptir.

### Karar Kuralı

Model derecesi seçim prosedürü:

1. Bir üst sınır $p_{max}$ belirlenir
2. $p = 1$'den $p_{max}$'a kadar her bir $p$ değeri için AR(p) modeli EKK ile tahmin edilir.
3. Her model için $AIC(p)$ ve $BIC(p)$ hesaplanır.
4. **Karar kuralı: $AIC(p)$ veya $BIC(p)$ değerini minimize eden $p$ seçilir.**

$$
p^*=\underset{p}{\operatorname{argmin}}\,AIC(p)
\qquad \text{veya} \qquad
p^*=\underset{p}{\operatorname{argmin}}\,BIC(p)
$$

## Kayan Pencere Yaklaşımı

### Sabit Modelin Yetersizliği

$$
\hat{\beta} = (X^TX)^{-1}X^TY
$$

elimizdeki **tüm örneklemi tek seferde kullanarak** sabit bir $\hat{\phi}$ katsayı seti üretir. Bu yaklaşımın örtük varsayımı, sürecin **parametrelerinin zaman içinde değişmediğidir** (parameter stability / stationarity of coefficients).

Gerçek dünya verilerinde bu varsayım sıklıkla ihlal edilir:

- **Yapısal kırılmalar (structural breaks)**: Bir ekonomik kriz, politika değişikliği veya rejim geçişi, $\phi$ katsayılarının değerini kalıcı olarak değiştirebilir.
- **Zamanla değişen dinamikler (time-varying dynamics)**: Bazı süreçlerde otokorelasyon yapısı yavaşça evrilir (örneğin volatilite kümelenmesi gösteren finansal seriler).
- **Rejim değişimleri**: Seri, farklı dönemlerde farklı istatistiksel davranışlar sergileyebilir (yüksek volatilite / düşük volatilite rejimleri gibi).

Tüm örneklemle tahmin edilen **sabit (tek) bir model**, bu değişimleri ortalayarak "yumuşatır" — bu da hem yakın geçmişteki dinamikleri tam yansıtamaz hem de gelecek tahminlerinde sistematik hatalara yol açar. Çözüm, modeli **belirli bir zaman penceresi içindeki veriyle tekrar tekrar tahmin etmektir** — bu da bizi kayan pencere (rolling window) yaklaşımına götürür.

### Pencere Mekanizmasının İşleyişi

Kayan pencere yaklaşımında, sabit uzunlukta bir **pencere (window)**, w gözlem içerecek şekilde tanımlanır. Model, bu pencere içindeki veriyle tahmin edilir; sonra pencere bir adım ileri kaydırılır (en eski gözlem düşürülür, en yeni gözlem eklenir) ve model **yeniden tahmin edilir**.

Matematiksel olarak, $t$ anındaki pencere:

$$
\text{Pencere}_t = \{Y_{t-w+1}, Y_{t-w+2}, \ldots, Y_t\}
$$

Bu pencere kullanılarak $\hat{\beta}_t = (X^TX)^{-1}X^TY$ hesaplanır. Sonraki adımda:

$$
\text{Pencere}_{t+1} = \{Y_{t-w+2}, Y_{t-w+3}, \ldots, Y_{t+1}\}
$$

Bu, "kayan (rolling)" pencere olarak adlandırılır çünkü pencere **sabit genişlikte** kalır, sadece konumu kayar.
Her pencerede yeniden tahmin edilen $\hat{\beta}_t$, o zaman dilimine **özgü** katsayıları temsil eder ve böylece modelin zamanla değişen dinamiklere uyum sağlamasına (adaptivity) imkan tanır.

### 5.3 Örnek Dışı Tahminin Metodolojik Önemi

Kayan pencere yaklaşımının asıl gücü, **örnek dışı (out-of-sample) tahmin** üretmeye doğal olarak uygun olmasıdır:

1. $\text{Pencere}_t$ içindeki veriyle model tahmin edilir ($\hat{\beta}_t$ elde edilir).
2. Bu model kullanılarak **pencerenin dışındaki** bir sonraki gözlem için tahmin üretilir:

$$
\hat{Y}_{t+1} = \hat{c}_t + \hat{\phi}_{1,t}Y_t + \ldots + \hat{\phi}_{p,t}Y_{t-p+1}
$$

3. Gerçekleşen $Y_{t+1}$ değeri ile $\hat{Y}_{t+1}$ karşılaştırılarak tahmin hatası hesaplanır:

$$
e_{t+1} = Y_{t+1} - \hat{Y}_{t+1}
$$

4. Pencere kaydırılır, süreç tekrarlanır.

### 5.4 Pencere Uzunluğunun Seçimi

Pencere uzunluğu $w$'nin seçimi, kendi başına bir denge (trade-off) problemidir:

- **Kısa pencere (w küçük)**:
  - (+) Modele en güncel dinamikleri daha hızlı yansıtır, yapısal değişimlere daha çabuk adapte olur.
  - (−) Az gözlemle tahmin edildiği için $\hat{\beta}$'nin varyansı artar; ayrıca p büyükse $(X^TX)^{-1}$'in hesaplanabilmesi için $w \geq p+1$ zorunludur, w küçüldükçe bu sınıra yaklaşılabilir.

- **Uzun pencere (w büyük)**:
  - (+) Daha fazla gözlem, daha düşük tahmin varyansı, daha istikrarlı katsayılar.
  - (−) Eski (artık geçerliliğini yitirmiş) veriyi de içerdiğinden, yapısal kırılmalara veya rejim değişimlerine yavaş tepki verir; aşırı uzun pencere, aslında Bölüm 5.1'de eleştirilen "sabit model" durumuna geri döner.

## Tahmin Performansı Metrikleri

Üretilen örnek dışı tahmin hataları ($e_t = Y_t - \hat{Y}_t$), tek tek incelendiğinde yorumlanması zor, dağınık sayılardır. Bu bölümde, bu hataları **tek bir özet sayıya** indirgeyen üç temel metrik ele alınacaktır. $n_{test}$, örnek dışı tahmin yapılan gözlem sayısını göstersin.

### MAE (Mean Absolute Error / Ortalama Mutlak Hata)

$$
MAE = \frac{1}{n_{test}} \sum |Y_t - \hat{Y}_t|
$$

**Özellikleri**:
- Hataların **mutlak değerini** aldığı için pozitif ve negatif hatalar birbirini götürmez.
- Tüm hatalara **eşit ağırlık** verir — büyük bir hata ile küçük bir hata, hatanın büyüklüğüyle **doğru orantılı** şekilde etkiler.
- **Aykırı değerlere (outliers) karşı görece dayanıklıdır** (RMSE'ye kıyasla), çünkü büyük hatalar karesi alınmadan toplanır.

### RMSE (Root Mean Squared Error / Karekök Ortalama Kare Hata)

$$
RMSE = \sqrt{\frac{1}{n_{test}} \sum (Y_t - \hat{Y}_t)^2}
$$

**Özellikleri**:
- Hataların **karesini** aldığı için büyük hatalar **orantısız şekilde daha ağır cezalandırılır**.
- Bu nedenle RMSE, **büyük/nadir hatalardan kaçınmanın kritik olduğu** uygulamalarda tercih edilir.
- Aykırı değerlere karşı **MAE'den daha hassastır** — bu bazen istenen bir özellik (büyük hataları öne çıkarmak), bazen bir dezavantajdır (tek bir aşırı gözlem metriği domine edebilir).

### 6.3 MAPE (Mean Absolute Percentage Error / Ortalama Mutlak Yüzde Hata)

$$
MAPE = \frac{1}{n_{test}} \sum \left| \frac{Y_t - \hat{Y}_t}{Y_t} \right| \times 100
$$

**Özellikleri**:
- Hatayı gerçek değere **oranlayarak yüzdesel** bir ölçüt üretir — bu da **farklı ölçeklerdeki serileri karşılaştırmayı** mümkün kılar.
- **Kısıtlar**:
  - $Y_t = 0$ (veya sıfıra çok yakın) olduğunda tanımsız veya aşırı büyük değerler üretir — payda sıfıra yaklaştıkça patlar.
  - Negatif değerler içeren serilerde yorumlanması problemlidir.

### 6.4 Metriklerin Karşılaştırılması ve Birlikte Kullanımı

| Metrik | Birim | Büyük hatalara duyarlılık | Ölçekler arası karşılaştırma | Sıfıra yakın değerlerde |
|---|---|---|---|---|
| MAE | Orijinal birim | Düşük (doğrusal ceza) | Uygun değil | Sorun yok |
| RMSE | Orijinal birim | Yüksek (kareli ceza) | Uygun değil | Sorun yok |
| MAPE | Yüzde (%) | Orta (oransal) | Uygun | Tanımsız/patlar |

- **MAE ve RMSE birlikte** yorumlandığında, aralarındaki fark bilgi verir: RMSE, MAE'ye çok yakınsa hatalar **homojen** dağılmıştır (aşırı sapma yoktur); RMSE, MAE'den belirgin şekilde büyükse, veri setinde **bazı büyük hatalar (outlier tahminler)** olduğuna işaret eder.
