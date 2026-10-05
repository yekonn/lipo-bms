# İlk revizyon için gerçek parça seçimi: 6S/3 A şarj katı

Durum: **şema çizimine esas ön seçim**, 5 Ekim 2026. Seçilen kodlar ve kılıflar üretici dokümanında doğrulandı; PCB ısıl/EMI testi ve gerçek pil veri sayfası gelmeden üretim onayı değildir. [Ayrıntılı şarj BOM'u](bom-sarj-kati.csv) bu sayfanın makinece okunabilir karşılığıdır. 20 V USB PD girişi, 6S paketin varsayılan 25,2 V tam dolu geriliminden düşük olduğundan bu devrede yalnız buck dönüştürücü **yeterli değildir**; dört MOSFET'li buck-boost gerekir.

## 1. Çizimde kullanılacak aktif parçalar ve kılıflar

| İşlev | Seçilen sipariş kodu | Gerçek kılıf | KiCad sembolü / footprint durumu |
| --- | --- | --- | --- |
| CC/CV buck-boost denetleyici | **TI BQ25756RRVR** | 36 pin VQFN, TI RRV0036A, **5 × 6 mm**, 0,5 mm adım, açık termal pad (37. bağlantı) | KiCad 9.0.2 standart kütüphanesinde sembol yok: **37 padli proje sembolü çizilecek**. `Package_DFN_QFN:QFN-36-1EP_5x6mm_P0.5mm_EP3.6x4.6mm` boyut adayı; pin-1 yönü, pad ve üretici önerilen land pattern karşılaştırılmadan atanmış footprint sayılmaz. |
| PD alıcı denetleyicisi | **ST STUSB4500QTR** | QFN-24 EP, **4 × 4 mm** | Stok `Interface_USB:STUSB4500QTR` sembolü ve `Package_DFN_QFN:QFN-24-1EP_4x4mm_P0.5mm_EP2.7x2.7mm` footprint'i var; üretici çizimiyle son kontrol. NVM'de 20 V/5 A PDO programlanır. |
| Hücre AFE | **TI BQ7694202PFBR** | TQFP-48, TI PFB, **7 × 7 mm** gövde, 0,5 mm adım | Proje sembolü çizilecek; `Package_QFP:TQFP-48_7x7mm_P0.5mm` footprint adayı. `02` varyantı I²C + CRC ve varsayılan 3,3 V REG1 sunar. REG1, MCU'nun tüm beslemesini taşıyacak varsayılmaz. |
| USB/I²C MCU | **ST STM32G0B1CBT6** | LQFP-48, **7 × 7 mm**, 0,5 mm adım | Stok `MCU_ST_STM32G0:STM32G0B1CBTx` + `Package_QFP:LQFP-48_7x7mm_P0.5mm`. 128 KB Flash; bu tam kodun -40…85 °C derecesi mekanik/termal şartla yeniden kontrol edilir. |

BQ25756'nın EVM'sinde `BQ25756RRVT` bulunur. Güncel veri sayfasındaki aktif 3000'lik reel sipariş kodu **RRVR** olduğu için BOM'da onu kullandık; ikisi aynı RRV kılıf ailesindedir. Bu dört entegre seçimiyle şemada **hangi pinlerin çizileceği artık bellidir**. Yardımcı 3,3 V regülatörü ve 50 A yük kesici bu şarj alt devresinin parçası değildir; ayrı sayfalarda seçilecektir.

## 2. Şarj güç katı: başlangıç parça takımı

| Konum | Ön seçim | Kılıf/footprint | Hesap ve sınır |
| --- | --- | --- | --- |
| Q_CH1–Q_CH4 | **Vishay SIR680LDP-T1-RE3**, 80 V N-MOSFET ×4 | PowerPAK SO-8; `Package_SO:PowerPAK_SO-8_Single` pad haritası üretici pinleriyle karşılaştırılacak | TI BQ25756EVM'deki dört anahtarın aynısı. Bu FET'ler **yalnız 3 A şarj dönüştürücüsüne** aittir; 50 A yük çıkışı için seçilmedi. Anahtarlama kaybı ve soğutma PCB'de doğrulanır. |
| L_CH | **Vishay IHLP6767GZER100M01**, 10 µH, EVM BOM'una göre 16,5 A | IHLP-6767; `Inductor_SMD:L_Vishay_IHLP-6767` | 20 V → 25,2 V ve 250 kHz örnek boost noktasında yaklaşık 1,65 A tepe-tepe bobin ripple; 3 A akü gücünde %90 örnek verimle giriş akımı 4,2 A ve bobin tepe akımı yaklaşık 5,03 A. Sıcak bobin doyum verisi ve kayıp yine kontrol edilir. |
| R_AC / R_BAT | **Vishay WSL25125L000FEA**, 5 mΩ, 1 W, %1 ×2 | 2512; `Resistor_SMD:R_2512_6332Metric` | Girişte 4,52 A sınırında 0,102 W; aküde 3 A'de 0,045 W. BQ25756 ayar denklemleri 5 mΩ için kullanılır. EVM akü tarafında daha büyük 5 W şönt kullanıyor; 3 A devresi için 1 W ön seçimi Kelvin izleri ve sıcaklıkla doğrulanır. |
| R_FSW | **Panasonic ERJ-6ENF1333V**, 133 kΩ, %1 | 0805 | Veri sayfasına göre yaklaşık 250 kHz. |
| R_FB_TOP / R_FB_BOT | 249 kΩ / 16,2 kΩ, **%0,1** | 0805 veya 0603, seri/parça kodu şema öncesi sabitlenecek | Varsayılan `VFB_REG=1,536 V` ile yaklaşık **25,10 V** CV. 25,2 V kesin hücre üst sınırı olarak doğrulanırsa `VFB_REG=1,542 V` yazılarak nominal ≈**25,20 V** elde edilir. BQ7694202 tek hücre OV koruması bağımsız kalır. |
| R_ICHG / R_ILIM | 16,9 kΩ / 4,42 kΩ, %1 | 0603; kesin direnç MPN'si şema öncesi sabitlenecek | 5 mΩ şöntle tipik sınırlar `50 A·kΩ / 16,9 kΩ ≈ 2,96 A` akü ve `20 A·kΩ / 4,42 kΩ ≈ 4,52 A` USB girişi. TI faktör toleransı ve direnç toleransında sınırlara pay bırakılır; gerçek giriş gücü/verim ölçülür. |
| R_AC1 / R_AC2 / R_AC3 | 1 MΩ / 6,98 kΩ / 58,0 kΩ, %1 | 0805/0603; kesin MPN şema öncesi sabitlenecek | Veri sayfasındaki ACUV/ACOV bölücüsünde hesaplanan yaklaşık **18,0 V alt**, **22,0 V üst** penceresi. PD sözleşmesindeki 5 A bilgisi bu analog pencereyle tek başına kanıtlanmaz. |
| C_REGN / C_DRV / C_VAC / C_BTST1,2 / C_SENSE | 4,7 µF / 4,7 µF / 1 µF / 0,1 µF ×2 / 0,47 µF ×2 | Veri sayfasındaki en az nominal değer ve DC bias sonrası etkin kapasite sağlanacak; gerilim sınıfı ve MPN şemada seçilecek | REGN/DRV_SUP, VAC, iki bootstrap ve ACP–ACN/SRP–SRN filtreleri; üretici yerleşim kuralına göre IC'ye yakın. Giriş/çıkış seramik ve bulk kapasitörleri ayrıca yerleşim/akım dalgalanmasına göre sayılacak. |

Bu güç katının 20 V/5 A giriş bütçesi yaklaşık 100 W'dır. 25,2 V × 3 A = **75,6 W** aküye gider; %90 varsayımsal verimde giriş yaklaşık **84 W/4,2 A** olur. 4,52 A giriş sınırı, verim daha düşükse akü akımını 3 A altına düşürür. Bu güvenli güç sınırlamasıdır; **her kaynakta sabit 3 A** garantisi değildir. 20 V/3 A PD kaynağında bu kart 3 A şarj başlatmaz.

## 3. `05_CHARGER.kicad_sch` için pin/net taslağı

```text
USB-C #1 VBUS → STUSB4500 kontrollü VBUS anahtarı → R_AC 5 mΩ → VAC/ACP/ACN
                                     │
                                     └─ giriş MLCC + bulk → PGND

                   Q_CH1 high (HIDRV1)                 Q_CH3 high (HIDRV2)
CHG_IN ────────────┬──────── SW1 ── L_CH 10 µH ── SW2 ────────┬── CHG_CONV_OUT
                   Q_CH2 low (LODRV1)                  Q_CH4 low (LODRV2)
                    │                                           │
                   PGND                                        PGND

CHG_CONV_OUT → R_BAT 5 mΩ (SRP/SRN Kelvin) → şarj kolu ters-akım kesici → BAT_FUSED+
BAT_FUSED+ → R_FB_TOP → FB → R_FB_BOT → FBG  (akü kutbuna duyarlı bağlantı kontrolü)
FSW_SYNC → 133 kΩ → PGND; ICHG → 16,9 kΩ → PGND; ILIM_HIZ → 4,42 kΩ → PGND
REGN → 4,7 µF → PGND; DRV_SUP → 4,7 µF → PGND; iki bootstrap ve sense filtreleri
CE (aktif düşük) ← CHARGE_PERMIT; varsayılan pull-up = şarj kapalı; I²C ↔ MCU
TS ← akü NTC/uygun bölücü; AFE'nin tek hücre ve sıcaklık kesme izni ayrı donanım hattında
```

Pin düzeyinde çizimde özellikle `VAC` **32 ve 33**, `PGND` **17, 22 ve termal pad 37**, `NC` **15, 16, 31** ayrı gösterilir. `CE` pin 7 aktiftir ve açıkta bırakılmaz; firmware `DIS_CE_PIN` özelliğini açmamalıdır. `EN_REV` yazılımda **0** tutulur; ayrıca USB-C'ye geri güç akımını önleyen fiziksel şarj/PD anahtarı bulunur. `VBUS_EN_SNK` yüksek güçlü sözleşmenin tek kanıtı değildir: MCU STUSB4500 sözleşme bilgisini okur, 18–22 V giriş penceresini doğrular ve ancak yük çıkışı kapalıyken `CHARGE_PERMIT` ister. MCU resetinde CE yüksek kalır. 5 A için uygun e-marked kablo ve gerçek 20 V/5 A PDO gerekir.

Şarj çıkışındaki şönt ve `FB` geri beslemesinin **şarj kesicisine göre tam konumu**, BQ25756'nın battery sensing/kaçak akım kılavuzu ve BQ76942'nin ortak şönt yerleşimiyle çizimde dondurulacak. Akü şöntü `R_BAT` 3 A şarj akımını ölçer; [sistem şöntü](03-kicad-moduller-ve-sema-plani.md) 50 A yük koruması için ayrı elemandır. İkisini aynı parça gibi bağlamak hatalı olur.

## 4. KiCad'de çizime geçiş için gerçek kalan işler

1. **BQ25756RRVR sembolü:** üreticinin 1–37 pin tablosundan yerel KiCad sembolü oluştur, NC pinleri ve pad 37 dahil ERC ile doğrula. BQ7694202 için de proje sembolü gerekir. Kılıf adaylarını TI/ST land pattern ölçüleriyle karşılaştır.
2. **Önce `04_PD_INPUT`, sonra `05_CHARGER`:** STUSB4500 NVM 20 V/5 A ve kablo/kontrat okuma, ardından BQ25756 dört anahtarlı güç katı. Üretici EVM'nin 7S/29,4 V ve 10 A jumper ayarlarını bu şemaya taşımak **hatalıdır**.
3. **Şarj kesicisi, USB soketi, yardımcı 3V3 ve 50 A yük yolu:** ayrı aktif parça seçimi gerekiyor. 3 A dönüştürücü FET'leri yük yolunda kullanılmaz. `CE` ve fiziksel geri akım kesici birlikte test edilmeden akü bağlanmaz.
4. LTspice veya eşdeğer güç devresi modelinde 20 V → 25,2 V geçişi, 3 A akım, kısa/açık şönt, PD kopması, gate sürüşü, indüktör/FET sıcaklık ve reverse-mode hata senaryoları; ardından KiCad ERC/PCB DRC ve prototip ölçümü.

Kaynaklar: [BQ25756 veri sayfası, pinler, denklemler ve RRV0036A](https://www.ti.com/lit/ds/symlink/bq25756.pdf); [BQ25756EVM şeması ve BOM'u, SLUUCT7D](https://www.ti.com/lit/pdf/SLUUCT7); [BQ76942 varyant ve kılıf tablosu](https://www.ti.com/lit/ds/symlink/bq76942.pdf); [STUSB4500 veri sayfası](https://www.st.com/resource/en/datasheet/stusb4500.pdf); [STM32G0B1 sipariş kodu ve pin haritası](https://www.st.com/resource/en/datasheet/stm32g0b1cb.pdf).
