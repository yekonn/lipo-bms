# 6S1P BMS — gereksinimler ve ön mimari

Durum: kavramsal tasarım, 5 Ekim 2026. Şema, PCB ve üretim için onaylanmış BOM değildir. EDA aracı **KiCad** olarak seçildi; analog güç ve hata senaryoları LTspice'ta değerlendirilecek. Ayrıntılı modül planı: [03 — KiCad modülleri ve şema planı](03-kicad-moduller-ve-sema-plani.md).

## 1. Kullanıcıdan alınan hedefler

| Konu | Hedef / durum |
| --- | --- |
| Paket | 6S1P, yaklaşık 6 Ah LiPo. [Güncel referans ürün](https://www.motorobit.com/222v-6s-6200mah-45c-lipo-batarya): Profuse, 22,2 V, 6200 mAh, 45C olarak pazarlanıyor; ürün kodu `LIP.6S.01.000024`. |
| Şarj | Bataryaya yaklaşık 3 A; yalnız şarj için ayrılmış bir USB Type-C soketi kullanılacak. |
| Deşarj | 50 A sürekli, 90 A tepe en fazla 1–2 saniye. |
| Şarj/yük kilidi | Şarj etkinse XT60 yük çıkışı fiziksel olarak kapalı olacak; yük yalnız bataryadan beslenecek. |
| Yük konnektörü | PCB üzerinde XT60 erkek. Kesin üretici ve PCB montaj varyantı henüz seçilmedi. |
| Hücre ölçüm konnektörü | 7 pin JST soket, B0…B6 hücre tap hatları. Kesin JST serisi ve karşı konnektör doğrulanacak. |
| Bilgisayar/master | İkinci, yalnız veri için ayrılmış USB Type-C soketiyle PC ve Raspberry Pi'ye telemetri ve yapılandırma; CAN ilk sürümde istenmiyor. |
| Diğer | Uygun yerde sigorta ve test noktaları; sıcaklık aralığı, PCB hacmi ve termal düzen henüz açık. Referans paketin ilan edilen ölçüsü 160 × 50 × 47 mm, ağırlığı 827 g. |
| Harici yük | XT60 çıkışına motor sürücüsü ile Raspberry Pi ve 8,4 V servoların beslemelerini barındıran bir güç dağıtım kartı bağlanacak. Sürücü giriş kapasitesi ve rejeneratif akım bilgisi henüz yok. |

Referans ürünün satıcı sayfası **XT60 güç ucu** ve 6S/6200 mAh/45C bilgisini veriyor; 6,2 Ah × 45C = **279 A** yalnızca ilan edilen C değerinin aritmetik karşılığıdır. Hedef 50 A sürekli ≈ **8,1C**, 90 A/1–2 s ≈ **14,5C** eder; paketin bu koşullardaki sıcaklığı, gerilim düşümü ve ömrü ilandan doğrulanamaz. Sayfadaki [SGS raporu](https://dosya.motorobit.com/pdf/22-2V-6S-6200mAh-45C-Lipo-Battery.pdf) Avrupa pil direktifi kimyasal içerik sınırlarına ilişkin; [MSDS](https://dosya.motorobit.com/pdf/22-2V-Lipo-Battery-MSDS-Report.pdf) çok sayıda modeli kapsıyor. İkisi de bu paketin elektriksel performans veri sayfasının yerini tutmuyor. Gerçek üretici hücre/paket veri sayfası ve güç/balans konnektörlerinin fiziksel eşleşmesi doğrulanmalı. **Batarya girişi** için mevcut XT60'ın karşı parçası seçilecek; kullanıcının istediği **XT60 erkek yük çıkışı** bundan ayrı olacak. İki XT60'un yanlış eşleştirilmemesi ve enerjili uçların erişilebilirliği mekanik tasarımda kontrol edilmeli. 7 pin JST yalnız hücre ölçümü ve dengeleme içindir, 50 A taşımaz.

## 2. Ölçek ve güç hesabı

İlandaki 22,2 V nominal değer 6 × 3,7 V ile uyumlu; nominal paket enerjisi yaklaşık **137,6 Wh**. Satıcı sayfası hücre üst sınırını 4,20 V olarak anlatıyor; üretici veri sayfası doğrulanana kadar 6 × 4,2 V = **25,2 V** tasarım varsayımıdır. Buna göre şarj sonuna yakın 3 A × 25,2 V = **75,6 W** batarya gücü gerekir. %90 örnek verimle USB girişinden yaklaşık **84 W** alınır. 20 V/3 A USB PD kaynağı (60 W) 3 A şarja yetmez; 20 V/5 A (100 W) kaynağı ve uygun 5 A kablo ilk güç hedefidir. [USB-IF](https://www.usb.org/usb-charger-pd), 20 V × 5 A profilini 100 W USB PD sınırı olarak açıklar. Gerçek kablo/kaynak ve termal kayıplar doğrulanmalıdır.

| Durum | Yaklaşık değer | Anlamı |
| --- | --- | --- |
| Nominal deşarj gücü | 22,2 V × 50 A = 1,11 kW | Konnektör, kablo, PCB bakırı ve MOSFET ısısı sürekli koşulda değerlendirilecek. |
| Üst gerilimde sürekli güç | 25,2 V × 50 A = 1,26 kW | Anahtarlama elemanının gerilim/akım kapasitesi bundan türetilmez; geçiş ve hata durumları ayrıca gerekir. |
| Üst gerilimde tepe güç | 25,2 V × 90 A = 2,27 kW, 1–2 s | Darbe ve kısa devre SOA, sigorta eğrisi ve kablo/konnektör sıcaklığı kritik. |
| 0,5 mΩ **örnek** şönt | 50 A'de 1,25 W; 90 A'de 4,05 W | 4 uçlu ölçüm, darbe gücü ve termal kapasite doğrulanacak; değer henüz seçilmedi. |
| 6200 mAh / 3 A | ideal CC kısmı yaklaşık 2,1 saat | CV evresi nedeniyle gerçek tam şarj daha uzun olur. |

XT60 erkek isteği korunuyor, ancak **kesin XT60 modelinin** üretici sürekli/darbe akım derecesi, PCB lehim bağlantısı ve sıcaklık artışı 50 A/90 A hedefleri için doğrulanmadan kabul edilemez. Gerekirse montaj biçimi veya konnektör seçimi revize edilecek.

## 3. Önerilen işlevsel bloklar

```text
6S paket ana güç girişi (XT60 karşı parça) ─ sigorta/koruma ─┬─ yük kesici ─ XT60 erkek ─ yük
                                                    │
                                                    ├─ 6S CC/CV şarj katı ◄─ USB-C #1 (yalnız PD şarj)
                                                    │
                                                    └─ BMS/MCU beslemesi
7 pin JST: B0…B6 ─ giriş koruma/filtre ─ 6S AFE ─ I²C ─ USB MCU ─ USB 2.0 D+/D− ─ USB-C #2 (yalnız veri)
                              NTC / şönt ───────┘          │
                                             şarj izni + yük kilidi + hata kaydı
```

Bu yalnız blok ilişkisidir; şarj katının batarya tarafı, akım ölçüm noktası, MOSFET yönleri ve CHG/DSG gate sürme topolojisi henüz şema seviyesinde seçilmedi. Ayrı şarj ve yük güç yolları gerekir: ortak bir güç portunda şarj cihazı XT60 yük pinlerini besleyebilir. [BQ76942 teknik kılavuzu](https://www.ti.com/lit/pdf/SLUUBY1), ayrı charger/load bağlantılı **paralel CHG/DSG FET** düzenini destekler (`Settings:FET:FET Options[SFET]=0`). Seri FET düzeninde iletimde bırakılan tek FET'in gövde diyodu üzerinden akım algılanırsa diğer FET de otomatik iletime alınabilir; bu davranış yük kilidi için uygun değildir. Paralel mod seçimi tek başına fiziksel izolasyon kanıtı değildir: her kolun MOSFET gövde diyodu, ters besleme ve tekil kısa devre davranışı doğrulanacak. **Şarj modu** algılandığında DSG/yük yolu şarj katı etkinleşmeden önce açık devreye alınarak XT60 izole edilmeli; bağlı şarj cihazı varken şarj tamamlandığında yükün otomatik yeniden enerjilendirilmesi önerilmiyor. Bilgisayara yalnız USB veri bağlantısı kurulduğunda yükü kapatmak gerekmez.

### Ön bileşen adayları

| Blok | Aday ve doğrulanmış kapsam | Açık tasarım işi |
| --- | --- | --- |
| Hücre AFE / koruma | [TI BQ76942](https://www.ti.com/lit/gpn/BQ76942): 3–10S, hücre/akım/sıcaklık izlemesi, koruma, yüksek tarafta N-FET sürme ve pasif dengeleme; teknik kılavuzda ayrı şarj/yük kolları için paralel FET modu var. 6S için kullanılmayan hücre girişleri veri sayfasındaki düzene göre bağlanır. | 6S tap sırası, NTC sayısı, kalibrasyon, her kolun ters akım engellemesi ve 50 A ısıl tasarım. |
| USB-C PD güç alıcısı | [ST STUSB4500](https://www.st.com/resource/en/datasheet/stusb4500.pdf): MCU olmadan PD sink anlaşması, en çok 20 V/5 A PDO; 5 V bağlantıda 3 A şarj varsayılmaz. | 20 V/5 A sözleşmesi, 5 A kablo tanıma, VBUS koruması ve şarj etkinleştirme. |
| 6S şarj dönüştürücü | [TI BQ25756](https://www.ti.com/lit/gpn/BQ25756): 1–14 seri Li-ion için buck-boost CC/CV şarj denetleyicisi, 4,2–70 V giriş. 20 V girişten 25,2 V pakete yükseltme kapasitesi var. | Anahtarlama MOSFET/indüktör/şönt/soğutma, 3 A ayarı ve ters güç akışının kapatılması. |
| BMS/USB mikrodenetleyicisi | [STM32G0B1 ailesi](https://www.st.com/en/microcontrollers-microprocessors/stm32g0b1cb.html): USB 2.0 FS ve I²C çevre birimleri. | Kesin varyant/pin sayısı, bootloader, watchdog, USB sınıfı ve bekleme akımı. |
| Güç yolu | Ana sigorta, şönt, düşük dirençli MOSFET veya uygun başka kesici, sürücü ve ısıl algı | DC kesme kapasitesi, MOSFET kısa devre SOA, 50 A ısı ve 90 A/2 s darbeye göre seçim. |

Bu tabloda başlayan adaylar için tam sipariş kodları ve kılıflar [şarj katı parça seçiminde](04-parca-secimi-ve-sarj-kati.md) verildi. Bu seçimler hazır ve doğrulanmış birleşik devre değildir. BQ76942 paralel modunun şema bağlantıları, gerekli ek ters akım kesiciler ve DFETOFF/CHG izin yolu üretici uygulama devresiyle doğrulanmadan kesinleştirilmeyecek.

## 4. Şarj sonlandırma ve yük kilidi

Şarj cihazı hücreye uygun **CC/CV** profilini uygular. Dengeleme, yüksek hücreleri küçük bir akımla baypas eder; 3 A şarj akımını tek başına kesen veya "tam dolu" kararı veren işlem değildir. **CHARGE_ENABLE** (şarj izni) ve **CHARGE_DONE** (normal tamamlanma bildirimi) ayrı mantık durumları olacaktır. Normal tamamlanma için hücre gerilimleri, hücreler arası fark, akımın CV evresinde düşmesi, sıcaklık ve zaman sınırı birlikte değerlendirilir. Şarj denetleyicisi akım azaldığı için dengeleme tamamlanmadan durursa BMS bunu CHARGE_DONE saymaz; güvenli koşulda dengelemeye devam eder, gerekiyorsa denetimli tamamlama şarjı yapar veya zaman aşımı/hata bildirir. Herhangi bir hücre aşırı gerilime/sıcaklığa ulaşırsa BMS, dengelemenin bitmesini beklemeden şarjı kapatır. Kesin eşikler gerçek hücre veri sayfası ve ölçüm doğruluğu bütçesiyle belirlenecek. [TI dengeleme uygulama notu](https://www.ti.com/lit/pdf/SLUAA81) otonom/MCU kontrollü balans ve ölçüm zamanlamasını anlatır.

Ön durum makinesi (USB-C #1 şarj kaynağı, USB-C #2 veri host'u):

1. **Boşta/veri:** #2 üzerinden bilgisayar veya Pi host bağlantısı varsa telemetri açık; #1 üzerinde geçerli 20 V/5 A PD sözleşmesi yoksa şarj devresi kapalı.
2. **Şarja geçiş:** Geçerli şarj sözleşmesi algılanır; önce XT60'a giden DSG/yük yolu açık devreye alınarak yük izole edilir, durum ölçülür, ardından şarja izin verilir.
3. **Şarj ve dengeleme:** 3 A hedefli CC/CV; AFE bağımsız koruma sınırlarını gözetir. Dengeleme uygun hücre gerilimi/sıcaklığı ve ölçüm zamanlamasında yürütülür.
4. **Tamamlandı:** Şarj denetleyicisinin sonlandırması ve BMS'nin hücre denge/durum koşulları birlikte sağlanınca CHARGE_DONE üretilir; BMS yük çıkışını enerjisiz tutar. Şarj kaynağı ayrıldıktan sonra uygun koruma koşullarında yük yeniden etkinleştirilebilir.
5. **Arıza:** Hücre/akım/sıcaklık veya PD güç kaybında şarj kapatılır; tehlikeli koşulda yük de kapatılır. Yeniden bağlanmanın ayrıntılı kullanıcı politikası sonra yazılabilir, ancak güç kaybında şarjın güvenli biçimde durması zorunludur.

Yük kilidi yalnız MCU komutu olmamalı: PD/şarj etkin sinyali ile AFE koruma izni, sürücüde güvenli mantıkla birleştirilmeli ve açılışta yük kapalı başlamalı. Bunun gerçek devre davranışı hata enjeksiyonu ve masa testleriyle doğrulanacak.

## 5. USB, Raspberry Pi ve test noktaları

**İki fiziksel USB-C soketi seçildi:** #1 PD sink + 6S şarj girişi, #2 USB 2.0 cihaz (UFP) + telemetri. #1'in USB veri hatları kullanılmayacak; #2'nin VBUS'u şarj katına bağlanmayacak. Böylece yanlışlıkla #2'ye PD adaptörü takılması 6S şarjı başlatmaz; #1'e bilgisayar bağlanması da 5 V ile 3 A şarjı başlatmaz. Her iki soketin CC/VBUS/ESD koruması ve dışarıdan kolay ayırt edilen etiketleri ayrı tasarlanacak.

USB #2 için CDC veya kısa bir ikili protokol seçilebilir. PC ve Raspberry Pi **USB host** olarak bağlanır; Pi'nin USB-C güç girişini host bağlantısı saymamak gerekir. BMS kendi bataryasından beslenir. Şarj sırasında #2 üzerinden telemetri açık kalabilir ve XT60 yük çıkışı kapalı tutulur. USB veri soketinin toprağı ile batarya eksi ucu arasındaki bağlantı, bilgisayar gövdesinden yüksek akım dönmeyecek şekilde düzenlenmeli; gerekiyorsa USB galvanik izolasyonu değerlendirilmeli.

Önerilen test noktaları: B0…B6 (korumalı/erişim sınırlarıyla), şönt Kelvin uçları, BAT+/BAT−, CHG/DSG gate ve source, PD VBUS, giriş/çıkış şarj akımı ölçümü, 3,3/5 V rayları, I²C ve MCU SWD. Yüksek akım hatları için güvenli, uygun akım dereceli ölçüm yöntemi ayrıca seçilmeli; küçük test pad'i akım enjekte etmek için kullanılmamalı.

## 6. Tasarım kapıları ve doğrulama

- **Pil kanıtı:** Gerçek hücre/paket veri sayfası, 6S tap dizilimi, batarya üzerindeki XT60 cinsiyeti ve JST karşılığı, şarj üst gerilimi ve sıcaklık sınırları.
- **Güç kanıtı:** XT60 ve batarya girişi kesin parça veri sayfası; 50 A sürekli ısıl sınama, 90 A/2 s darbe, ana sigortanın DC kesme kapasitesi ve zaman-akım eğrisi. Ana sigorta bataryaya olabildiğince yakın bulunmalı; giriş kablosunun sigortadan önceki kısmı korunmasız kalmamalı.
- **USB PD kanıtı:** #1'de 20 V/5 A kaynak ve kablo algısı, geçersiz/yetersiz PD'de şarjın başlamaması; #2'de 5 V PC/Pi bağlantısında veri çalışması; yanlış porta takma ve iki portun eş zamanlı kullanımının güvenli olması.
- **Şarj/yük kanıtı:** Şarj başlangıcında ve arıza anında XT60'un enerjisiz kalması; MOSFET tekil arızası dahil yük kilidi hata analizi.
- **Analog/simülasyon:** LTspice ile şönt ve MOSFET ısısı, açma/kapama geçişi, ön şarj ihtiyacı, indüktör ve şarj anahtarlama stresleri. PD protokolü ve BMS firmware'i LTspice'ta tam doğrulanmış sayılmaz.
- **Kart:** DRC/ERC yanında yüksek akım bakır/kablo yolu, Kelvin ölçümü, balans tap sırası ve ısı dağılımı gözden geçirme; prototipte termokupl ve akım sınırlı ilk açılış.

Kesin şema/BOM öncesi açık kararlar: örnek bataryanın gerçekten kullanılacak model olup olmadığı, şarj kaynağının minimum PD profili, mekanik hacim/soğutma, yükün giriş kondansatörü ve rejeneratif akım olasılığı. KiCad seçimi kesinleşti.
