# 6S LiPo için USB-C PD şarj ve balanslama: ön mimari

Durum: uygulanabilirlik araştırması; şematik veya onaylı parça listesi değildir. Standart 4,20 V üst sınırlı LiPo hücre varsayımı kullanılır. Hücre veri sayfası farklı bir üst gerilim, şarj akımı veya sıcaklık aralığı verirse hesaplar değişir.

## Temel karar

6 seri hücreyi **paket olarak CC/CV yöntemiyle** şarj etmek ve her hücreyi ayrı ölçüp gerektiğinde pasif balanslamak aynı PCB'de mümkündür. 6S paket tam doluyken `6 × 4,20 = 25,20 V` olur. Her hücreye bağımsız şarj cihazı koymak, tek bir seri paket şarjı ve balanslama ile aynı şey değildir; bu ilk mimaride altı bağımsız şarj kanalı öngörülmez.

```text
USB-C soket → EPR/SPR PD alıcısı → VBUS koruması ve kontrollü güç yolu
            → akım sınırlamalı buck-boost CC/CV şarj katı → 6S paket
                                                         ↕
                                           6 hücre tap'i, NTC'ler, şönt
                                                         ↕
                                    hücre izleme/koruma AFE'si
                                    ├─ pasif balans anahtarları ve dirençleri
                                    ├─ şarj/deşarj kesme yolu
                                    └─ MCU: durum, PD/şarj koordinasyonu, kayıt
```

Şarj katı paket gerilimi ve akımını düzenler. AFE altı hücre gerilimini ayrı izler, balanslar ve hücre aşırı gerilimi/sıcaklığı gibi durumlarda şarjı fiziksel olarak engeller. Yalnız paket toplamının 25,2 V altında olması, tek tek hücrelerin güvenli olduğunu kanıtlamaz. PD veya MCU yazılımı kilitlendiğinde de BMS koruma yolu etkin kalmalıdır. Şarj akımının hücre veri sayfası, seri/paralel düzen ve termal tasarıma göre ayarlanması gerekir.

## Adaptör ve güç bütçesi

GaN, adaptörün güç dönüştürme teknolojisidir; sunulan USB PD gerilimlerini tek başına söylemez. Adaptör etiketindeki PDO/AVS profilleri ve kablo ayrıca kontrol edilir.

| PD sözleşmesi | Giriş gücü | %90 örnek verimde, 25,2 V paket ucunda teorik şarj akımı üst sınırı |
| --- | ---: | ---: |
| 20 V × 3 A (SPR) | 60 W | 2,14 A |
| 20 V × 5 A (SPR) | 100 W | 3,57 A |
| 28 V × 5 A (EPR) | 140 W | 5,00 A |

Bu değerler yalnızca `I ≈ η × Pin / Vbat` güç hesabıdır; hücrenin izin verdiği şarj akımı, kablo, konnektör, MOSFET, bobin, akım algılayıcı ve ısı kayıpları daha düşük sınır koyabilir. Şarjın CC aşamasında batarya gerilimi daha düşük olur; akım sınırı ayrıca uygulanmalıdır. 20 V girişten 25,2 V'a çıkmak için **boost veya buck-boost** gerekir. 28 V EPR ile üst uçta buck şarj mümkündür; tüm SOC aralığı ve daha düşük PD profillerini desteklemek için buck-boost seçimi esnektir. 5 A için uygun elektronik işaretli kablo ve EPR için uyumlu kablo/adaptör gerekir. İlk 5 V bağlantısında yüksek akımlı şarj açılmaz; geçerli PD sözleşmesi ve limitler doğrulandıktan sonra açılır. Sözleşme düşerse veya kablo ayrılırsa şarj derhal durur.

## İncelenecek parça sınıfları

- [TI BQ25756](https://www.ti.com/lit/gpn/BQ25756): üretici veri sayfasında 1–14 seri Li-ion/Li-polymer için buck-boost CC/CV şarj denetleyicisi, 4,2–70 V giriş ve şarj akımı/gerilimi ayarı belirtiliyor. Harici güç MOSFET'leri ve bobin gerektirir. **Bu IC hücre balanslama AFE'sinin yerini almaz.**
- [TI BQ76952](https://www.ti.com/lit/gpn/BQ76952): 3–16 seri hücre izleme/koruma ve pasif balanslama adayı; 6S bağlantısı, kullanılmayan VC pinleri, hücre tap sırası ve kesme yolu veri sayfasına göre tasarlanmalı. **Bu IC tek başına USB-C PD şarj cihazı değildir.**
- [TI TPS26750](https://www.ti.com/lit/gpn/TPS26750): USB PD 3.1 EPR sink ve 28/36/48 V müzakeresi için aday. Veri sayfasındaki **VBUS pininin önerilen çalışma üst sınırı 22 V** olduğundan 28 V doğrudan bu pine uygulanamaz. EPR uygulaması için [TPD4S480](https://www.ti.com/product/TPD4S480) ile CC koruması/VBUS seviye çevirme ve harici yüksek gerilim güç anahtarı gerekir; veri sayfası bölüm 8.2.1.5'teki EPR topolojisi esas alınmalı. PD denetleyicisi ile şarj IC'sinin I²C/MCU koordinasyonu ve açılış koşulları ayrıca doğrulanmalı.

Bunlar işlevleri karşılayan adaylardır; kesin bağlantı ve birlikte çalışabilirlik, referans tasarım ve eval kartlarıyla doğrulanmadan parça seçimi tamamlanmış sayılmaz.

## Şematikten önce netleşecek girdiler

1. Hücre modeli/veri sayfası, kapasite (Ah), varsa paralel sayısı ve izin verilen şarj/deşarj akımları.
2. Hedef şarj süresi/akımı; kullanılacak adaptörün gerçek PD profilleri ve kablo tipi.
3. Yükün sürekli/tepe akımı, şarj olurken yükün çalışıp çalışmayacağı ve rejeneratif akım.
4. Paket NTC yerleri, sıcaklık aralığı, PCB hacmi/soğutma, sigorta ve konnektörler.
5. BMS–şarj kesme sinyali, PD kaybı/yeniden bağlanma davranışı, arıza kaydı ve CAN ihtiyacı.

Doğrulama planında en az hücre dengesizliği, hücre/NTC kablosu kopması, tek hücre aşırı gerilimi, kısa devre, şarj sırasında adaptör çekilmesi, PD'nin 5/20/28 V arasında geçişi ve EPR koruma devresinin termal durumu bulunmalıdır. LTspice güç katı ve arıza geçişlerini inceleyebilir; USB PD protokolünün doğrulanması için ayrıca gerçek PD analizörü ve donanım testi gerekir.
