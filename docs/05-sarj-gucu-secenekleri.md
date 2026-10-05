# Şarj gücü karşılaştırması: 100 W SPR ve 140 W EPR

Durum: 5 Ekim 2026. Kullanıcı standart USB Power Delivery ve ayrı USB-C şarj soketi istedi. Gerçek paketin izin verilen **şarj C değeri** ve sıcaklık aralığı üretici veri sayfasından henüz doğrulanmadı; aşağıdaki 4,4 A yalnız tasarım seçeneğidir. Referans 6S 6200 mAh pakette üst gerilim olarak 25,2 V varsayıldı.

## 1. 20 V/6 A ne anlama gelir?

Standart USB-C PD **SPR** profilinde 20 V için üst sınır **5 A / 100 W**'tır. Adaptör kasasında `20 V 6 A` yazması, bu akımın USB PD sözleşmesi ile sıradan Type-C kabloda alınabileceğini kanıtlamaz; üreticiye özel protokol olabilir. Bu projede şarj soketi USB-C ve **standart PD** olduğundan 20 V/6 A tasarlanmayacak. 5 A için e-marker uygun kablo gerekir. Standart daha yüksek profil USB PD 3.1 **EPR 28 V/5 A = 140 W** ve EPR uyumlu kaynak/kablodur. [USB-IF PD bilgisi](https://www.usb.org/usb-charger-pd), [TI TPS26750 EPR veri sayfası](https://www.ti.com/lit/ds/symlink/tps26750.pdf).

`120 W` kaynağın varlığında bile bataryaya **6 A** gitmez: 25,2 V × 6 A = **151,2 W** akü gücü; %90 varsayımsal verimde girişte **168 W** gerekir. 20 V × 6 A = 120 W girişte aynı verimle tam doluya yakın en çok `120 × 0,90 / 25,2 ≈ 4,29 A` elde edilir; kontrol/kablo/ısı payı bu sınırı daha da aşağı çeker.

| Senaryo | Standart USB-C? | 25,2 V noktasında hedef akü akımı | Yaklaşık giriş gücü, %90 örnek verim | İdeal yalnız CC süresi `6,2 Ah/I` |
| --- | --- | --- | --- | --- |
| Şimdiki 20 V/5 A, 100 W SPR | Evet | 3,0 A | 84 W / 4,2 A giriş | **2,07 saat** |
| 20 V/6 A, 120 W | **Hayır**, standart PD profili değil | En çok 4,29 A teorik; pratik daha az | 120 W tam sınır | 1,45 saat teorik alt sınır |
| 28 V/5 A, 140 W EPR | Evet, EPR kaynak/kablo ile | **4,4 A önerilen ön hedef** | 123,2 W / 4,40 A giriş | **1,41 saat** |
| 28 V/5 A, 140 W EPR | Evet | 5,0 A | 140 W / 5 A giriş; **hiç pay yok** | 1,24 saat teorik, uygun hedef değil |

Bu süreler yalnız `kapasite/akım` hesaplarıdır; **0 V hücreden şarj** anlamına gelmez ve CV evresindeki akım düşüşünü içermez. Gerçek `0–100%` süresi her iki tasarımda da daha uzun olur; üretici verisi ve sonlandırma akımı olmadan dakikası hesaplanamaz. 3 A'den 4,4 A'e geçişin ideal CC kazancı yaklaşık **40 dakika**dır. 3 A ≈ **0,48 C**, 4,4 A ≈ **0,71 C**; paket şarj akımı onayı şarttır.

## 2. 28 V/5 A EPR için değişim listesi

| Şimdiki 20 V/5 A tasarımı | 28 V/5 A için karar |
| --- | --- |
| `STUSB4500QTR` PD sink; en çok 20 V | **Değişir.** Ön aday **TI TPS26750SRSMR**, 32 pin 4 × 4 mm VQFN; 28/36/48 V EPR sink destekler. Seçtiğimiz otonom açılış yaklaşımı için üretici kılavuzundaki **harici EEPROM** ve EPR port koruması (TI'nin örnek devresinde `TPD4S480`) gerekir; alternatif olarak MCU'nun PD firmware'ini açılışta yüklemesi ayrı bir tasarımdır. Bu alt devrenin şeması ve BOM'u yeniden çizilir. |
| Şarj USB-C soketi, 20 V VBUS anahtarı, TVS/ESD ve giriş kapasitörleri | **Yeniden seçilir/doğrulanır.** 28 V çalışma ve EPR geçiş/surge gerilimleri, 5 A kontak ve kablo, 50 V EPR kablo gereği, güç anahtarının VDS/akım/ters akım ve giriş koruma derecesi kontrol edilir. |
| `BQ25756RRVR` 4 anahtarlı CC/CV buck-boost denetleyicisi | **Kalır.** Denetleyicinin 4,2–70 V giriş aralığı 28 V'u kapsar; 28 V girişte 25,2 V'a doğru çoğunlukla buck çalışır. Yalnız 28 V sözleşmesiyle şarj edilecekse buck-only denetleyiciye geçmek mümkün bir yeniden tasarımdır; mevcut 20 V fallback ve seçilmiş güç katını korumak için BQ25756 ile devam edilir. İki ayrı USB portu ve şarjdayken yük kilidi aynı gereksinim olarak kalır. |
| Dört `SIR680LDP-T1-RE3` 80 V MOSFET; `IHLP6767GZER100M01` 10 µH bobin; iki 5 mΩ şönt | **İlk parça seçimi korunabilir**, ancak ~4,4 A akü akımında ve 28 V girişte FET geçiş/switching kaybı, bobin sıcaklığı/doyması, şönt gücü ve PCB bakırı ölçülür. 5 mΩ şöntte 4,4 A kaybı ≈0,097 W; 4,63 A giriş sınırında ≈0,107 W. Sıcaklık sonucu başarısızsa ilgili parça değişir. |
| `R_ICHG=16,9 kΩ` ve `R_ILIM=4,42 kΩ`, 5 mΩ şönt | **Değişir.** Ön değerler `R_ICHG=11,3 kΩ` → tipik 4,42 A, `R_ILIM=4,32 kΩ` → tipik 4,63 A giriş limiti. TI faktör ve %1 direnç toleransı ile 5 A kablo sınırının altında kaldığı, prototipte ve firmware limitlerinde kontrol edilir. 20 V/5 A fallback sözleşmesinde MCU akü akımını yeniden **≈3 A veya altına** düşürür. |
| 18–22 V `ACUV/ACOV` bölücüsü | **Değişir.** 28 V EPR'ye özel şarj için yaklaşık 25–30 V pencere: `R_AC1=1 MΩ`, `R_AC2=4,22 kΩ`, `R_AC3=41,8 kΩ` (%1, TI bölücü topolojisi). 20 V fallback şarjı istenirse donanım penceresi tek başına 25 V'a sabitlenemez; alternatif bölücü/denetim tasarlanır. |
| 249 kΩ/16,2 kΩ FB bölücüsü, 133 kΩ frekans direnci | **Kalabilir.** Aynı 6S hücre üst gerilimi ve yaklaşık 250 kHz anahtarlama; gerçek hücre gerilimi doğrulanınca CV ayarı yeniden gözden geçirilir. |
| Batarya AFE, ana sigorta, 50 A yük anahtarı | Şarj akımına bağlı **AFE şarj eşikleri, ayrı şarj kolu kesicisi ve şarj kablosu/izleri** yeniden boyutlandırılır. 50 A yük yolu değerleri 4,4 A şarj katı parçalarıyla karıştırılmaz. |

PD denetleyicisi değişimi küçük bir yazılım ayarı değildir: TPS26750'nin EPR sözleşmesi, EEPROM firmware'i, CC/VBUS koruması ve kablo algısı için `04_PD_INPUT` sayfası tekrar tasarlanır. TI veri sayfası EPR kaynak/sink ve 28 V'u; ayrıca EEPROM'dan konfigürasyon yükleme şartlarını açıklar. [TPS26750 veri sayfası](https://www.ti.com/lit/ds/symlink/tps26750.pdf), [BQ25756 veri sayfası](https://www.ti.com/lit/ds/symlink/bq25756.pdf).

## 3. Proje kararı

**Bu aşamada 20 V/5 A, yaklaşık 3 A şarjı temel revizyon olarak tutmak daha mantıklı.** EPR, şarjın ideal CC kısmında yaklaşık 40 dakika kazandırır; yeni PD denetleyicisi, EEPROM, EPR koruma/kablo, 28 V güç yolu ve ek doğrulama gerektirir. Gerçek paketin şarj C değeri bilinmediğinden 4,4 A'yı bugün kesin gereksinim yapamayız. Eğer ölçülmüş toplam süre kabul edilemezse ve pil üreticisi ≥4,4 A şarjı doğrularsa **28 V/5 A EPR** ikinci revizyon hedefi olur. 20 V/6 A USB-C seçeneği standart PD kapsamına alınmaz.
