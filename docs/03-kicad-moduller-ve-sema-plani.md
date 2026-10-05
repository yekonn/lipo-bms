# 6S BMS — KiCad modülleri, ön devre taslakları ve çizim sırası

Durum: 5 Ekim 2026, **şema çizimine giriş tanımı**. Aşağıdaki çizimler işlevsel devre taslaklarıdır; henüz pin düzeyinde KiCad şeması, PCB veya üretim onaylı BOM yoktur. Parça grupları [ön BOM'da](bom-adaylari.csv), sistem hedefleri [gereksinimlerde](02-gereksinimler-ve-on-mimari.md). Her güç elemanının değeri ve ayak izi gerçek üretici veri sayfası, hesap, simülasyon ve prototip ölçümüyle seçilecek.

## 1. Sistemin sınırı ve değişmez kuralları

Referans paket 6S1P, 6200 mAh, 22,2 V nominaldir. 25,2 V tam dolu gerilim **hücre kimyası doğrulanana kadar varsayımdır**. Çıkış 50 A sürekli ve 90 A/1–2 s taşır; 3 A şarj ayrı USB-C PD soketinden gelir. İkinci USB-C yalnız USB 2.0 veri cihazı olarak PC/Raspberry Pi host'una bağlanır. Harici XT60 çıkışı, motor sürücüsü ve 8,4 V servolar/Raspberry Pi için alt gerilim dönüştürücüleri içeren güç dağıtım kartını besler; bu düşük gerilim rayları BMS kartının kapsamı dışındadır. PDB'nin giriş kapasitesi ve frenleme sırasında geri akım olasılığı bilinmiyor.

1. Hücre aşırı/düşük gerilim, akım veya sıcaklık tehlikesinde **AFE donanım koruması**, MCU/USB yazılımından bağımsız olarak ilgili güç yolunu kapatabilmeli. Açılış, reset ve kablo/PD arızasında güç yolları güvenli varsayılan durumda kalmalı.
2. Geçerli şarj bağlantısında önce **yük pozitif yolu** açılmalı; XT60 çıkış gerilimi aküden ayrılmış ve yeterince düşmüş olarak ölçülmeden şarj güç katı etkinleşmemeli. PDB giriş kondansatörleri için kontrollü deşarj/bleed yolu ve zaman aşımı tasarlanmalı. Şarj boyunca yük izni donanım mantığında engellenmeli; `CHARGE_DONE` bu engeli kaldırmamalı. USB veri soketi takılması yükü kesmemeli.
3. Şarj cihazının çıkışı ile XT60 yük çıkışı birbirine doğrudan bağlanmamalı. Her iki kolun ters akım, MOSFET gövde diyodu ve tekil kısa devre arıza yolları çizimde izlenmeli. Şarj varken XT60'un **0 V** olması yalnız BMS denetimindeki batarya kaynaklı enerji için doğrulanabilir; harici PDB'nin kendi kondansatörü veya rejeneratif enerjisi XT60 tarafını canlı tutabilir. Şarj başlatma için ölçülen çıkış gerilimine izin eşiği ve zaman sınırı konmalı; engel kalkmıyorsa şarj başlamamalı.
4. Ana akım tüm kollar için 4 uçlu şöntten geçmeli. Hücre tap/JST, USB kılıfı veya MCU toprağı ana akım dönüş yoluna dönüşmemeli. Harici Pi/PDB ve şarj adaptörü aynı anda takılıyken olası toprak çevrimleri incelenecek.
5. Herhangi bir hücre aşırı gerilim/sıcaklığa çıkarsa balans beklenmeden şarj kesilmeli. Normal `CHARGE_DONE` kararı CV akım azalması **ve** hücreler arası gerilim farkı/sıcaklık/zaman koşullarıyla verilecek. Kesin eşikler hücre veri sayfasından gelecek.

## 2. Üst seviye güç ve kontrol taslağı

```text
Paket XT60 + ── F_MAIN ── BAT_FUSED+ ──┬── iki yönlü kesebilen YÜK ANAHTARI ── XT60 erkek + ── PDB
                                      ├── denetimli ŞARJ KOLU ◄── 6S CC/CV buck-boost ◄── PD güç anahtarı ◄── USB-C #1
                                      └── küçük korumalı yardımcı besleme ── 3V3 / MCU / AFE arayüzü
Paket XT60 − ── R_SHUNT (Kelvin) ── SYS_GND ──────────────────────────── XT60 erkek − / şarj dönüşü

JST 7 pin B0…B6 ── tap koruma/RC ── BQ76942 ── I²C + ALERT ── MCU ── USB-C #2 D+/D−
                           NTC ─────┘      CHG/DSG güvenlik izni ── donanım kilidi ── sürücüler
                                            XT60_VSENSE + PD_OK ────────┘
```

Bu çizim **topoloji ve net isimleri** tanımlar; FET sayısı/yönü, AFE CHG/DSG sürücü uyumu, şöntün koruma algılama aralığı ve USB toprak yolu henüz şematik düzeyde çözülmedi. BQ76942'nin ayrı şarj/yük portları için *parallel FET* ayarı (`SFET=0`) değerlendirilecek. Üretici EVM'si bu düzen için farklı parça yerleşimi gerektirir; doğrudan kopyalanacak hazır 50 A çözümü sunmaz. 50 A yolu için iki bağımsız kesme elemanı veya doğrulanmış eşdeğer arıza toleransı tasarım incelemesinde kararlaştırılacak.

## 3. KiCad sayfaları ve her sayfanın devre görevi

Sayfa adları önerilen hiyerarşik `.kicad_sch` dosya adlarıdır. Şemada konnektör pinleri, `BAT_RAW+`, `BAT_FUSED+`, `B−`, `SYS_GND`, `LOAD+`, `PD_VBUS`, `CHARGER_OUT+`, `3V3`, `CHARGE_REQUEST`, `CHARGE_PERMIT`, `LOAD_PERMIT`, `AFE_FAULT`, `XT60_VSENSE` gibi adlandırılmış hiyerarşik portlar kullanılacak. `B−` ile `SYS_GND` yalnız tanımlanmış şönt üzerinden bağlanır; gerçek yerleşimde Kelvin ve güç dönüşleri ayrılır.

| Sayfa | İşlevsel devre ve şema gereksinimi | Çıkış / bitiş koşulu |
| --- | --- | --- |
| `00_TOP.kicad_sch` | 01–08 alt sayfaları, net yönleri, güç alanları, çalışma/hata durumları. Şarj ve veri USB-C konnektörleri farklı isim ve silk etiketli. | Açık akım çevrimleri ve yanlış porta takma senaryoları gözden geçirilmiş. |
| `01_PACK_INPUT.kicad_sch` | Batarya XT60 karşı konnektörü → bataryaya mümkün olduğunca yakın **DC kesme kapasiteli ana sigorta** → `BAT_FUSED+`. 7 pin JST `B0…B6` yalnız ölçüm; sıra/polarite doğrulaması, tap başına akım sınırlama/filtre ve açık/kısa tap tanısı. Paket ve güç katı NTC girişleri. | Giriş konnektörü gerçek akım/termal değerleri, kablo kesiti, JST fiziksel eşleşmesi ve sigortadan önceki kablo koruması belli. |
| `02_CELL_AFE.kicad_sch` | B0…B6 → üretici 6S uygulama devresindeki hücre giriş RC'leri → BQ76942. Altı pasif balans kanalı; 4 uçlu şöntün Kelvin uçları; NTC'ler; I²C, ALERT, CHG/DSG izinleri; uygun kullanılmayan cell pin bağlantıları ve besleme kapasitörleri. | Hücre ölçüm doğruluğu, balans direnci/gücü, NTC açık/kısa tanısı ve AFE konfigürasyon tablosu tamam. |
| `03_LOAD_SWITCH.kicad_sch` | `BAT_FUSED+` → ters akımı güvenli kesen ve 50 A/90 A taşıyan anahtar bankı → `LOAD+` → PCB XT60 erkek. Anahtar kaynak/gate ölçüm noktaları, ayrı gate deşarjı, sıcaklık algısı, çıkış gerilim bölücüsü. PDB kondansatörü için akım sınırlı **ön şarj** kolu ve anahtar kapanmadan önce çıkış gerilim onayı; açıldıktan sonra kontrollü çıkış deşarjı. | MOSFET yönleri, SOA/kısa devre açma süresi, soğutma, PCB/kablo akımı, ön şarj enerjisi ve kapama davranışı ölçülmüş. |
| `04_PD_INPUT.kicad_sch` | Şarj USB-C soketi: CC1/CC2 → STUSB4500, VBUS ESD/TVS ve uygun giriş koruması → denetimli VBUS güç anahtarı → şarj dönüştürücüsü. Programlı NVM PDO **20 V/5 A**, 5 A kablo ve gerçek PD sözleşmesi doğrulaması; düşük/yanlış PD'de 3 A şarj izni yok. | PD analizörüyle sözleşme ve kablo sınırı görüldü; bağlantı/kopma anında güç yolu güvenli. |
| `05_CHARGER.kicad_sch` | BQ25756 + dört anahtarlı buck-boost güç katı, indüktör, giriş/şarj akım algısı, 6S CC/CV geri beslemesi → denetimli CHG kolu → `BAT_FUSED+`. Aküden USB-C'ye ters güç akışı donanım/ayar ile engellenir; AFE şarj kesme hattı dönüştürücüyü kapatır. | 20 V/5 A girişte 3 A/25,2 V sınır koşulu, kayıp/ısı, 6S CV ayarı, terminasyon ve koruma ölçüldü. EVM'nin varsayılan **29,4 V/10 A** ayarı kullanılmaz. |
| `06_AUX_SUPPLY.kicad_sch` | Aküden MCU/AFEnin uygun raylarına korumalı düşük akımlı besleme; gerekli 3V3 regülasyon, filtre ve güç iyi sinyali. USB veri VBUS'u güç katına ulaşmaz; iki USB soketi üzerinden geri besleme engellenir. | En düşük paket geriliminde açılış, uyku/kapalı halde aküden çekilen akım ve reset davranışı ölçüldü. |
| `07_MCU_USB.kicad_sch` | USB FS + I²C'li STM32G0B1 adayı; USB-C #2 UFP CC dirençleri, D+/D− ESD/seri elemanlar, VBUS algısı; SWD, reset, watchdog, AFE ALERT, PD/şarj telemetrisi. Raspberry Pi **USB host** soketinden bağlanır, Pi güç girişinden değil. | CDC/ikili protokol ve hata telemetrisi çalışır; MCU kapanması korumayı devre dışı bırakmaz. |
| `08_INTERLOCK_TEST.kicad_sch` | PD geçerli/şarj isteği, AFE güvenlik izni, XT60 gerilim onayı ve anahtar geri bildiriminden **donanım izin kapıları**. `CHARGE_PERMIT` yalnız yük kesilmiş/gerilimi düşmüşken; `LOAD_PERMIT` yalnız şarj fiziksel olarak devre dışıyken. Varsayılan pull-down/off, ayrı test noktaları ve hata girişleri. | MCU kilitlenmesi, reset, AFE hata, PD kaybı, tek FET kısa/açık, dış PDB geri besleme senaryosu hata ağacında izlenmiş. |

AFE'nin dahili yüksek taraf FET sürücüleri ile ek ters akım kesici/gate sürücüsünün voltaj referansı **üretici uygulama şemasıyla** eşleştirilecek. FET'ler veya konnektörler seçilmeden `03_LOAD_SWITCH` bağlantısı pin seviyesinde sonlandırılamaz. Ön şarj direnci, sürücü ve zaman aşımı PDB'nin ölçülmüş `C_in` değeri ile hesaplanır: ilk yaklaşım `E=½·C_in·V_pack²`; 25,2 V ve 10 mF yalnız örnek alınırsa yaklaşık **3,18 J**. Rejeneratif yük varsa açık çıkışın geri beslenmesi ve hücre doluyken fren enerjisinin nereye gideceği ayrıca çözülür; çıkışı kapatmak tek başına bunu çözmez.

## 4. İşaretleşme, koruma ve ölçüm

| Sinyal | Kaynak → hedef | İşlev |
| --- | --- | --- |
| `AFE_FAULT`, CHG/DSG izinleri | BQ76942 → 08/03/05 | Hücre, sıcaklık, akım tehlikesinde ilgili güç yolunu MCU'dan bağımsız kesmek. |
| `I2C_SCL/SDA`, `ALERT` | AFE ↔ MCU | Hücre/akım/sıcaklık telemetrisi, balans ayarı ve hata kaydı. I²C kopması korumayı kaldırmamalı. |
| `PD_20V_5A_OK` | STUSB4500 + VBUS/kontrat doğrulama → 08/05 | 5 V veya 20 V/3 A kaynakta 3 A şarj başlatmamak. STUSB4500'un genel `VBUS_EN_SNK` çıkışı tek başına **20 V/5 A kanıtı değildir**. |
| `CHARGE_REQUEST` / `CHARGE_PERMIT` | MCU → 08 → 05/CHG kolu | İstenen çalışma ile donanımca izin verilen çalışma ayrımı. |
| `LOAD_REQUEST` / `LOAD_PERMIT` | MCU → 08 → 03 | Şarj sırasında veya güç açılışında yükün kapalı kalması. |
| `XT60_VSENSE`, FET geri bildirim | 03 → 08/MCU | Çıkışın gerçekten kesildiğini doğrulamak; dışarıdan beslenen çıkışta şarjı engellemek. |
| `USB_D+/D−` | 07 ↔ PC/Pi | İlk sürüm USB 2.0 cihaz protokolü; CAN yok. |

Test noktaları: B0…B6'ya güvenli ve korumalı erişim, şönt Kelvin çiftleri, `BAT_FUSED+`, `B−`, `SYS_GND`, `LOAD+`, `XT60_VSENSE`, PD giriş/anahtar sonrası VBUS, şarj giriş/çıkış akım algısı, 3V3, AFE ALERT/I²C, CHG/DSG gate-source ve SWD. Canlı hücre tap'larına standart topraklı osiloskop probu bağlanmadan önce referans ve kısa devre yolu kontrol edilir. Yüksek akım küçük test pad'inden geçirilmez.

## 5. BOM durumu ve kaynak kısıtı

[CSV BOM](bom-adaylari.csv) **işlev ve adet düzeyinde ön listedir**. `secildi_sch` tam sipariş kodunun ilk şema revizyonuna seçildiğini, `referans` yalnız değerlendirme kartında kullanımı, `açık` ise satın alma parçası seçilmediğini ifade eder. Şarj katının tam kod/kılıf/değerleri [04 — parça seçimi](04-parca-secimi-ve-sarj-kati.md) ve [ayrıntılı BOM'da](bom-sarj-kati.csv) bulunur; [100 W/140 W seçenekleri](05-sarj-gucu-secenekleri.md) ayrıca değerlendirildi. Özellikle sigorta, XT60, 50 A sistem şöntü, deşarj FET'leri, gate sürücüsü, PCB bakırı, TVS ve NTC gerçek maliyetlendirme/üretim BOM'u değildir. TI değerlendirme kartlarının örnek güç elemanları bu kartın 50 A çıkışı için seçilmiş sayılmaz.

- [BQ76942 veri sayfası](https://www.ti.com/lit/gpn/BQ76942), [teknik kılavuz](https://www.ti.com/lit/pdf/SLUUBY1), [BQ76942EVM kılavuzu / şema-BOM](https://www.ti.com/lit/pdf/SLUUC32): 6S ölçüm, dengeleme, NTC/şönt bağlantısı ve paralel FET düzeni için referans. EVM BOM'unda `CSD19532Q5B` FET, `103AT-2` NTC vardır; bunlar bizim güç yolu onayı değildir.
- [BQ25756 veri sayfası](https://www.ti.com/lit/gpn/BQ25756), [BQ25756EVM kılavuzu / şema-BOM](https://www.ti.com/lit/pdf/SLUUCT7): EVM, 29,4 V/10 A, dört `SIR680LDP-T1-RE3` MOSFET ve 10 µH `IHLP6767GZER100M01` indüktörle kurulmuş; bizim 6S/3 A düzenimiz için yeniden hesaplanacak. EVM kılavuzu kartta sigorta olmadığını açıkça belirtir.
- [STUSB4500 veri sayfası](https://www.st.com/resource/en/datasheet/stusb4500.pdf): 20 V/5 A'ya kadar PDO ve NVM ayarı; `VBUS_EN_SNK` ile belirli PDO/akım kabulünü birbirinden ayırma gereği.

## 6. Çizim ve doğrulama sırası

| Sıra | Teslim | Sonraki adıma geçiş kanıtı |
| --- | --- | --- |
| 0 | Gerçek paket/konnektör fotoğraf-ölçü/veri sayfası; PDB `C_in`/geri akım bilgisi; DC sigorta ve 50 A XT60 uygunluğu; güç yolu ile FET arıza analizi. | Konnektör, sigorta, şönt ve yük kesme topolojisi gerçek parça adaylarıyla mümkün görünüyor. Eksik PDB verisi konservatif aralık ve ölçüm planıyla açık tutulabilir, ancak ön şarj/sürme değeri kesinleştirilemez. |
| 1 | `00_TOP` + `01_PACK_INPUT` | 6S tap sıra tablosu, giriş/çıkış/şarj/USB portları ve güç dönüşü anlaşılır. |
| 2 | `02_CELL_AFE` | 6S üretici bağlantısı, filtre, balans, NTC/şönt ve AFE izin mantığı kontrol edilmiş. |
| 3 | `03_LOAD_SWITCH` + `08_INTERLOCK_TEST` | Şarj varken XT60 kesme ve ön şarj/ters akım planı; tekil arıza ve geri besleme incelemesi. Yük yolu projenin 50 A/90 A belirleyici riskidir. |
| 4 | `04_PD_INPUT` + `05_CHARGER` | 20 V/5 A PD, 3 A CC/CV, AFE şarj kesme ve ters güç engeli. Önce tezgâhta bağımsız charger prototipi düşünülebilir. |
| 5 | `06_AUX_SUPPLY` + `07_MCU_USB` | USB veri, I²C, telemetri, reset/uyku; yazılım kesilince donanım koruması sürer. |
| 6 | Tüm sayfaların ERC'si, sembol/footprint-üretici pin karşılaştırması, `B0…B6`/güç net denetimi, BOM revizyonu. Ardından LTspice güç yolu-ön şarj-şarj/arıza köşeleri ve PCB yerleşimi/DRC. | Simülasyon ile veri sayfası SOA/ısı sınırları; ilk prototipte akım sınırlı hücre simülatörü, PD analizörü ve ardından kontrollü gerçek pil testi. |

İlk KiCad şemasında `01` ile `02` birlikte gözden geçirilecek; AFE'nin kullanılmayan hücre pinleri, termistör pin çoklama ve CHG/DSG ayarları üretici dokümanına göre değer/pin düzeyinde doldurulacak. Ardından en kritik iki kanıt, **şarj varken çıkışın enerjisiz kalması** ve **50 A sürekli ısıl dayanım**, şema tamamlandı diye kapanmış sayılmayacak.
