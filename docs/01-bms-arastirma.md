# LiPo BMS: işlevler, haberleşme ve önceki çalışmalar

Durum: üretici veri sayfaları, uygulama notları, açık tasarımlar ve bir akademik derleme ile desteklenen mimari araştırma. Bu belge bir üretim şeması veya güvenlik onayı değildir. Sonradan belirlenen 6S paket ve USB-C PD ile şarj hedefi [ayrı ön mimari notunda](02-6s-usb-c-sarj.md) ele alınır.

## 1. BMS nedir?

BMS, batarya paketinin her hücresini ve güç yolunu izleyerek hücrelerin izin verilen çalışma alanında kalmasını sağlayan donanım ve yazılımdır. "LiPo" çoğu zaman polimer poşet hücre biçimini anlatır; hücrenin kesin şarj/boşaltma sınırlarını tek başına belirlemez. Gerilim ve sıcaklık eşikleri seçilen hücrenin veri sayfasından, paket düzeninden ve gerekli paylardan türetilmelidir. BMS, hücreye uygun CC/CV şarj cihazının yerini almaz.

Temel güvenlik zinciri: **hücre/sıcaklık/akım algılama → bağımsız koruma kararı → şarj veya deşarj yolunu açma → hatayı kaydetme ve üst sisteme bildirme**. Ana bilgisayarın yazılımı kilitlense veya haberleşme kesilse bile yerel koruma çalışmalıdır.

## 2. Olası işlevler

| İşlev | Tipik uygulama | İlk sürüm için öncelik |
| --- | --- | --- |
| Tek tek hücre gerilimi | Çok hücreli analog ön uç (AFE), hücre tap bağlantıları ve giriş filtreleri | Zorunlu |
| Paket akımı ve kısa devre/aşırı akım | Şönt/Hall ölçümü; hızlı donanım koruması; sigorta | Zorunlu |
| Hücre ve güç elemanı sıcaklığı | Hücrelere ve MOSFET/şönt bölgesine NTC; şarj ve deşarj için ayrı sınırlar | Zorunlu |
| Şarj/deşarj kesme | Ayrı kontrol edilebilen MOSFET'ler veya kontaktör; arıza halinde güvenli duruma geçiş | Zorunlu |
| Hücre dengeleme | Önce pasif dirençli dengeleme; direnç gücü ve PCB ısısı hesabı | Seri hücreli paket için güçlü aday |
| Durum tahmini | Akım integrasyonu (coulomb counting), dinlenmiş açık devre gerilimiyle düzeltme; daha sonra SOH | İzleme için yararlı, korumanın temeli değil |
| Arıza kaydı ve öz denetim | Watchdog, sensör/kablo kopma kontrolü, hata sayacı, durum makinesi | Zorunlu |
| Ön şarj | Büyük giriş kapasiteli yükte ilk akımı sınırlama | Yük mimarisine bağlı |
| Uyku/düşük tüketim | Depolama ve uzun bekleme için düşük tüketim, kontrollü uyandırma | Uygulamaya bağlı |
| İkinci koruma katmanı | Bağımsız koruyucu/sigorta veya yüksek riskli uygulamada yedek ölçüm | Risk analizine bağlı |

Koruma eşiği, histerezis, gecikme ve yeniden bağlanma koşulları her hata için ayrı tanımlanmalıdır. Örneğin aşırı akım ile hücre aşırı geriliminin zaman ölçekleri farklıdır. **Kablo kopması, ters/yanlış sıralı hücre bağlantısı, NTC arızası, MOSFET kısa devresi ve haberleşme kaybı** ayrıca hata analizine girmelidir. TI'nin [BQ76952 teknik kılavuzu](https://www.ti.com/lit/pdf/SLUUBY2) birincil/ikincil korumaları, hücre hattı kopması algısını ve ayrı CHG/DSG FET kararlarını tanımlar; etkinleştirilecek korumalar ve kurtarma davranışı varsayılan ayarlara bırakılmamalıdır.

Pasif dengeleme, yüksek gerilimli hücreyi direnç üzerinden kontrollü boşaltır. Örneğin **yalnızca hesap örneği olarak** 4,2 V hücrede 100 Ω direnç yaklaşık 42 mA ve 0,176 W üretir (`I=V/R`, `P=V²/R`); gerçek hücre üst gerilimi ve direnç seçimi ayrıca belirlenecektir. [TI dengeleme notu](https://www.ti.com/lit/pdf/SLUAA81) iç ve harici anahtar seçeneklerini, dengeleme sırasında ölçüm doğruluğunu ve ana bilgisayar algoritmasının IC'ye zarar vermemesi için gerekli sınırları açıklar. Dengeleme akımı ile süresi, MOSFET/direnç sıcaklığı ve ölçüm zamanlaması birlikte doğrulanmalıdır.

SOC tahmini için akımı zamanla toplamak başlangıç yöntemi olabilir; akım ofseti ve başlangıç SOC hatası birikir. Dinlenmiş hücre açık devre gerilimiyle düzeltme, kimyaya ve sıcaklığa özgü karakterizasyon gerektirir. [SOC yöntemleri derlemesi](https://doi.org/10.3390/en12030446) bataryanın zamana bağlı ve doğrusal olmayan davranışı nedeniyle tek bir yöntemin her koşulda doğru olmadığını vurgular. SOC/SOH gösterimi güvenlik kesme kararının tek dayanağı olmamalıdır.

## 3. Mimari ve haberleşme

```text
Hücreler, NTC'ler, akım algılayıcı
             │
             ▼
       BMS AFE / koruma ───────► MOSFET veya kontaktör ───► yük/şarj cihazı
             │                         ▲
       I²C/SPI veya AFE'ye             │ yerel koruma
       özgü yalıtımlı hat               │
             ▼                         │
       BMS mikrodenetleyicisi ─────────┘
             │
       CAN / RS-485 / SMBus
             ▼
       Ana bilgisayar veya sistem denetleyicisi
```

**BMS içi:** Tek kartlı düşük/orta gerilim paketinde hücre AFE'si ile mikrodenetleyici arasında parçanın desteklediği I²C/SPI benzeri sayısal hat kullanılabilir. [BQ76952 veri sayfası](https://www.ti.com/lit/gpn/BQ76952) I²C, SPI ve HDQ seçeneklerini; ana mikrodenetleyici olmadan işleyebilen korumaları ve yüksek tarafta N-MOSFET sürücülerini belirtir. Bu bağımsızlık **üst sistem MCU'sundan bağımsızlıktır**; AFE'nin bütün arızalara karşı tek başına yeterli olduğu anlamına gelmez. Veri bütünlüğü, watchdog, güç açılış sırası ve haberleşme kaybı davranışı ayrıca tasarlanır. Birden fazla hücre izleme kartı veya yüksek paket gerilimi gerekiyorsa [LTC6813-1'in isoSPI hattı](https://www.analog.com/media/en/technical-documentation/data-sheets/ltc6813-1.pdf) veya [MC33771C'nin transformatörlü fiziksel hattı (TPL)](https://www.nxp.com/docs/en/data-sheet/MC33771C.pdf) gibi zincirlenebilir bağlantılar ve ayrı master mimarisi değerlendirilir. Birden fazla işlemci varsa aralarında açık görev bölüşümü, heartbeat ve hata halinde kimin güç yolunu kapatacağı tanımlanır.

**Ana bilgisayar ile:** İlk aday CAN'dir; gürültülü kablo ortamına ve birden çok düğüme uygundur. Hat sonlandırması, ortak mod aralığı, toprak düzeni ve gerektiğinde galvanik yalıtım ayrıca tasarlanır. RS-485 uzun/yalın nokta bağlantısı için seçenek; SMBus ise akıllı batarya ekosistemine uyulacaksa seçenek olabilir. USB yalnızca masaüstü geliştirme ve servis için düşünülebilir. Bağlantı türü, sistemin kablo uzunluğu, gerilim alanları ve mevcut ağ standardıyla seçilir.

Üst sistemin alması gereken asgari bilgiler: hücrelerin en düşük/en yüksek gerilimi ve kimliği, paket gerilimi/akımı, sıcaklıklar, SOC tahmini, izin verilen şarj/deşarj durumu, aktif ve geçmiş hata kodları, BMS sürümü ve haberleşme canlılığı. Üst sistem şarj veya yük talebi iletebilir; tehlikeli durumda yerel BMS'nin kesme kararı her zaman üstün olmalıdır. Mesajların kimliği, birimi/ölçeği, geçerlilik bayrağı, zaman aşımı, sayaç/heartbeat ve sürüm uyumu protokol taslağında yazılmalıdır. CAN'ın çerçeve CRC'si tek başına uygulama seviyesindeki eski veri veya yanlış durum yorumunu çözmez. Şarj cihazıyla konuşma gerekiyorsa BMS–şarj cihazı izinleri ve akım/gerilim sınırları ayrı bir arayüz sözleşmesine bağlanmalıdır.

| Üst sistem bağlantısı | Güçlü olduğu durum | Ek tasarım kararı |
| --- | --- | --- |
| CAN/CAN FD | Gürültülü araç veya makine ağı, çok düğüm | Transceiver, sonlandırma, kablo/toprak, gerektiğinde izolasyon; uygulama mesaj şeması |
| RS-485 | Uzun kablo veya basit iki uçlu cihaz ağı | Adresleme, sıra/çakışma yönetimi, CRC ve zaman aşımı protokolde tanımlanmalı |
| SMBus | Mevcut akıllı batarya ana sistemi | Smart Battery veri modeli ve elektriksel/topolojik uyum doğrulanmalı |
| USB/UART | Laboratuvar ayarı ve hata ayıklama | Ana ürün haberleşmesi ve koruma yolu olarak tek başına düşünülmemeli |

Başlangıçta tek BMS MCU'su, AFE ile protokol/telemetri görevlerini üstlenebilir. İkinci işlemci ancak açık bir ihtiyaç varsa eklenmeli: örneğin güvenlik denetleyicisi ile kullanıcı arayüzü ayrılacaksa her işlemcinin güç kesme yetkisi, paylaşılan verinin sahibi, başlatma sırası, watchdog/heartbeat ve iletişim kopma davranışı yazılı olarak belirlenmeli. Yerel durum makinesinde en az **kapalı/bekleme, çalışmaya hazır, şarj, deşarj, geçici hata, kilitli hata** durumları ile geçiş koşulları bulunmalıdır; kesin durumlar uygulamaya göre netleşir.

## 4. Daha önce yapılmış çalışmalar

| Çalışma | Gözlenen yaklaşım | Bu projeye katkısı ve sınırı |
| --- | --- | --- |
| [Libre Solar BMS C1](https://github.com/LibreSolar/bms-c1) | Açık kaynak tek kart; belgelerinde 3–16 seri Li-ion hücre, TI bq76952, ESP32-C3, MOSFET anahtarlama, CAN/RS-485/USB ve çeşitli ölçümler belirtiliyor. [Ayrı firmware](https://github.com/LibreSolar/bms-firmware) Zephyr tabanlı. | Orta hücre sayısında AFE + MCU + yerel kesme + telemetri için yakın örnek. Belirtilen 70–100 A değeri kendi MOSFET/soğutucu koşullarına bağlı; bizim kartımıza taşınamaz. |
| [foxBMS 2](https://github.com/foxBMS/foxbms-2) ve [donanım listesi](https://github.com/foxBMS/foxbms-2/blob/master/hardware/README.md) | Açık kaynak modüler master/interface/slave mimarisi. Donanım listesi LTC6811-1 (12 hücre), LTC6813-1 (18 hücre) ve farklı AFE aileleriyle slave kartlarını; master ve Altium kaynaklarını tanımlıyor. | Çok sayıda seri hücre ve dağıtık ölçüm için referans. İlk küçük paket için maliyet/karmaşıklığı yüksek olabilir. Tasarımlar doğrudan kopyalanmadan lisans, hata revizyonları ve test raporları incelenmeli. |
| [TI PMP23329](https://www.ti.com/tool/PMP23329) | 10–16 seri LiFePO₄ paket için BQ76952, ayrı yakıt göstergesi, ikincil koruma ve Cortex-M0+ kullanan referans tasarım. | Ölçüm, SOC ve ikinci koruma işlevlerini ayırmanın somut örneği. Hücre kimyası ve akım koşulları bizimkilerle eşit varsayılmamalı. |
| [TI TIDA-010247](https://www.ti.com/tool/TIDA-010247) | Üst tarafta N-MOSFET kumandalı, istiflenmiş BQ769x2 izleme ve CAN içeren yüksek gerilim referans tasarımı. [Uygulama notu](https://www.ti.com/lit/pdf/SLUAAN9) istifleme, yalıtımlı haberleşme, güç beslemesi ve hızlı FET kapatmayı inceliyor. | 16 seri hücre sınırı aşıldığında zorlukların yalnızca ilave AFE seçmekten ibaret olmadığını gösterir; ilk küçük prototipe doğrudan ölçeklenmemeli. |
| [SOC yöntemleri derlemesi](https://doi.org/10.3390/en12030446) ve [batarya eşdeğer devre model karşılaştırması](https://doi.org/10.1016/j.jpowsour.2011.10.013) | Şarj durumu kestirimi ve gerilim-akım davranışı modellemesi için literatür. | LTspice hücre modelinin ve daha sonra yazılacak SOC algoritmasının ölçülmüş hücre verisiyle eşlenmesi gerektiğini gösterir. |

Bu örnekler mimariyi karşılaştırmak içindir; bir tasarımın açık kaynak olması bizim hücre, akım, izolasyon ve üretim koşullarımızda güvenli çalışacağını kanıtlamaz. Üretici siteleri ve bağlantılı veri sayfaları bu araştırma sırasında erişilebilir durumdaydı. Kesin seçim anında errata, üretim durumu, devre modeli ve stok yeniden kontrol edilmelidir.

## 5. Aday AFE aileleri ve ilk mimari önerisi

| Aday | Üretici belgesinde doğrulanan özellikler | Tasarım sonucu |
| --- | --- | --- |
| [TI BQ76952](https://www.ti.com/lit/gpn/BQ76952) | 3–16 seri hücre; bağımsız koruma, yüksek tarafta N-MOSFET sürme, otonom veya MCU denetimli pasif dengeleme; I²C/SPI/HDQ. | Tek kartlı, en çok 16S paket için güçlü ilk aday. MCU yine protokol, kayıt, SOC ve gözetim işlerini üstlenir. |
| [ADI LTC6813-1](https://www.analog.com/media/en/technical-documentation/data-sheets/ltc6813-1.pdf) | 18 seri hücre ölçümü, isoSPI zincirleme, pasif dengeleme (veri sayfasında azami 200 mA). | Dağıtık/yüksek gerilimli izleme için aday; paket kesme ve sistem koruması ayrıca mimarilenmeli. |
| [NXP MC33771C](https://www.nxp.com/docs/en/data-sheet/MC33771C.pdf) | 7–14 hücre, SPI/TPL, çift yönlü zincirleme ve pasif dengeleme (veri sayfasında azami 300 mA); otomatik gerilim/sıcaklık hata algısı. | Modüler otomotiv/endüstriyel yaklaşım için aday; master, güç kesme ve yalıtım ayrıca tasarlanmalı. |

Veri sayfasındaki dengeleme akımı **azami bileşen özelliğidir**; hücrenin, direncin, PCB'nin ve termal ortamın izin verdiği tasarım akımı değildir. Üç aday aynı işi aynı kapsamda yapmaz: BQ76952 paket koruyucu işlevlerini de bir araya getirirken diğer iki parça özellikle hücre izleme zincirinin parçasıdır.

Hedef **tek kartta 3–16 seri hücre** ve MOSFET ile yönetilebilir güç ise koruma özellikli BQ76952 sınıfı AFE + ayrı MCU + pasif dengeleme + CAN telemetri ile başlanması makul. Libre Solar C1 bu sınıf için karşılaştırma örneğidir. 1–2S için farklı bir entegre ailesi gerekir. Hücre sayısı, paket gerilimi veya güvenlik/izolasyon gereği büyürse foxBMS benzeri master ve yalıtımlı hücre izleme kartları değerlendirilmeli. Bu bir parça onayı değildir; hedef gereksinimler belirlenince AFE, MCU, güç kesici, şönt, transceiver ve besleme seçenekleri matrisle karşılaştırılacaktır.

Güç yolunun kararı yalnız akıma bağlı değildir. Örneğin **hesap örneği** olarak 40 A ve 1 mΩ şönt, 40 mV ölçüm gerilimi ve 1,6 W ısı üretir (`V=IR`, `P=I²R`). MOSFET iletkenlik kaybı, açma/kapama geçişi ve kısa devre esnasındaki güvenli çalışma alanı (SOA) ayrıca hesaplanmalıdır. Yükün büyük giriş kondansatörü varsa ön şarj devresi olmadan MOSFET zorlanabilir.

## 6. Tasarımı başlatmak için eksik girdiler

1. Hücrenin üreticisi/modeli, gerçek kimyası, veri sayfası ve 6S paket içindeki paralel düzeni (P).
2. Nominal/en yüksek paket gerilimi; sürekli ve tepe şarj/deşarj akımı ve süreleri.
3. Şarj cihazı türü, yükün giriş kapasitansı, rejeneratif akım olup olmadığı.
4. Çalışma/depolama sıcaklığı, mekanik hacim, soğutma ve hücre NTC yerleşimi.
5. Ana bilgisayarın var olan haberleşme hattı, kablo uzunluğu ve güvenlik/izolasyon gereksinimi.
6. Bekleme akımı hedefi, servis/ürün kullanım senaryosu ve doğrulanacak standartlar.

## 7. Altium ve LTspice için sonraki adımlar

Gereksinimler kesinleşince Altium'da hücre tap/filtre, AFE, MCU, akım algılama, güç kesici, besleme, sıcaklık, haberleşme ve konnektörler ayrı şema sayfaları olarak çizilmeli. PCB'de yüksek akım döngüsü, Kelvin şönt ölçümü, analog referans ve anahtarlama akımları, hücre tap koruması, ısı yayılımı ve gerekli açıklık/izolasyon mesafeleri gözden geçirilmeli.

LTspice çalışmaları için ilk test matrisi: (1) en düşük/nominal/en yüksek hücre geriliminde giriş filtrelerinin kararlılığı ve tap kopması; (2) MOSFET açma/kapama, kısa devre ve ön şarj anındaki gerilim/akım ile SOA karşılaştırması; (3) şönt ve pasif dengeleme direnci güçleri; (4) NTC açık/kısa durumlarında algılama; (5) besleme ve yük geçişleri. Modeller hücre veri sayfası veya ölçümünden türetilmelidir. Tam AFE modeli bulunacağı varsayılmamalı; gerekiyorsa AFE kararları davranışsal blokla temsil edilip bu sınır raporlanmalı. Simülasyon, hücre veri sayfası incelemesi ve fiziksel hata enjeksiyonlu prototip testinin yerine geçmez.

Doğrulama sırası: gereksinim ve hata matrisi → şema incelemesi → LTspice sınır durumları → akım sınırlı hücre simülatörüyle ilk açılış → gerçek hücrelerle kontrollü test → termal ve hata enjeksiyon testleri. Ürün uygulaması belirlendiğinde geçerli güvenlik, taşıma ve EMC standartları ayrıca eşleştirilmelidir.
