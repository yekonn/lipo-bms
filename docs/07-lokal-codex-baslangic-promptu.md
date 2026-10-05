# Yerel Codex oturumu için başlangıç promptu

Aşağıdaki metni, depoyu yerel Linux bilgisayarınıza klonlayıp KiCad 10 ve Konnect'i açtıktan sonra **yeni Codex sohbetinin ilk mesajı** olarak gönderin.

```text
Bu çalışma alanındaki yekonn/lipo-bms deposunda 6S1P LiPo BMS tasarımına devam edeceğiz. Önce README.md ve docs/01–06 dosyalarının tamamını, özellikle docs/03-kicad-moduller-ve-sema-plani.md, docs/04-parca-secimi-ve-sarj-kati.md, docs/bom-adaylari.csv ve docs/bom-sarj-kati.csv dosyalarını oku. Belgelerdeki değerleri üretim için doğrulanmış sayma; durum ve açık seçim sütunlarını koru.

Ortam yerel Linux'tur. KiCad 10 ve Konnect kuruludur. İlk olarak Konnect get_installation_info, list_toolboxes ve check_kicad_ui araçlarını çağır. KiCad sürümünün 10.x olduğunu, Konnect MCP bağlantısını ve açık KiCad proje/PCB durumunu raporla. Bağlantı kurulmamışsa dosyaları değiştirmeden kesin nedeni teşhis et. Konnect araçlarını göremiyorsan codex mcp get konnect çıktısını kontrol et ve yeni Codex oturumu gerekip gerekmediğini belirt.

Kesinleşen sistem hedefleri:
- Referans paket: 6S1P LiPo, 6200 mAh, 22,2 V nominal; 25,2 V tam dolu değeri gerçek üretici hücre veri sayfası gelene kadar varsayım.
- Yük: harici güç dağıtım kartı; motor sürücü, Raspberry Pi ve 8,4 V servo hatları içeriyor.
- Deşarj: 50 A sürekli, 90 A tepe 1–2 saniye.
- Şarj sırasında XT60 yük çıkışı kapalı kalacak; CHARGE_DONE yükü kendiliğinden açmayacak.
- İki ayrı USB-C: biri yalnız PD şarj, diğeri USB 2.0 veri/telemetri. CAN v1 kapsamında değil.
- Batarya hücre tap bağlantısı: B0…B6 için 7 pin JST. Yük çıkışı PCB üzerinde XT60 erkek. Ana sigorta ve test noktaları zorunlu.
- İlk revizyon şarj hedefi: standart USB PD SPR 20 V/5 A kaynaktan yaklaşık 3 A. 20 V/6 A standart PD değildir. 28 V/5 A EPR ve yaklaşık 4,4 A şarj docs/05 içinde ikinci revizyon seçeneğidir.
- PDB giriş kapasitesi ve motor rejeneratif akımı bilinmiyor; ön şarj/deşarj değerlerini kesinleştirme ve geri akım yolunu varsayma.

İlk şema revizyonu için seçilen ana parçalar:
- AFE: BQ7694202PFBR, TQFP-48, I2C + CRC, varsayılan REG1 3,3 V.
- Şarj denetleyicisi: BQ25756RRVR, 36 pin 5×6 mm VQFN/termal pad; dört anahtarlı buck-boost.
- PD sink: STUSB4500QTR, QFN-24 EP, 20 V/5 A NVM PDO.
- MCU: STM32G0B1CBT6, LQFP-48.
- Şarj MOSFETleri: SIR680LDP-T1-RE3 ×4; indüktör: IHLP6767GZER100M01 10 µH; şarj giriş/akü algısı: iki adet 5 mΩ WSL25125L000FEA.
- Bunlar 50 A yük anahtarı parçaları değildir. Ana sigorta, 50 A sistem şöntü, yük MOSFET bankı/gate sürücüsü, kesin XT60/JST, yardımcı 3,3 V regülatörü ve şarj kolu ters akım kesicisi hâlâ seçilecek.

İlk çalışma:
1. Mevcut dosyalara ve git durumuna bak; kullanıcı değişikliklerini koru.
2. Konnect ile proje yoksa lipo-bms KiCad 10 projesini oluştur. 00_TOP ve docs/03'te tanımlanan 01–08 hiyerarşik şema sayfalarını kur. Önce dry-run/plan çıktısını göster; mevcut dosyanın üzerine sessizce yazma.
3. BQ25756RRVR için pin 1–36 ve exposed pad 37 dahil, BQ7694202PFBR için 48 pinli proje sembollerini resmi güncel TI veri sayfalarından doğrulayarak yerel sembol kütüphanesinde oluştur. Footprint adaylarını üretici land pattern ölçüleriyle karşılaştır; yalnız gövde adı benziyor diye atama yapma.
4. İlk elektriksel çizim olarak 00_TOP, 01_PACK_INPUT ve 02_CELL_AFE sayfalarını hazırla. B0…B6 sırası, kullanılmayan AFE cell pinleri, hücre RC filtreleri, pasif balans, NTC, I2C/ALERT ve şönt Kelvin uçlarını üretici referans devresiyle doğrula.
5. Her anlamlı mutasyondan sonra dosya listesini ve değişiklik özetini ver. İş sonunda ERC, hiyerarşik pin doğrulaması, netlist özeti ve görsel şema çıktısını incele. ERC'nin elektriksel/termal güvenlik kanıtı olmadığını koru.
6. 50 A PCB güç yolunu, yük FET'lerini veya kesin koruma eşiklerini bu ilk adımda uydurma. Önce eksik gerçek parça/veri ve hata senaryolarını listele.

Türkçe iletişim kur. Üretici veri sayfalarını temel kaynak kullan. Doküman ile KiCad şeması çelişirse çelişkiyi açıkça bildir ve kanıtla düzelt. Çalışmayı yalnız bir planla bırakma; bağlantı uygunsa ilk üç KiCad şema sayfasını oluşturup doğrulamaya kadar ilerle.
```

## Yerel başlangıç sırası

```bash
git clone https://github.com/yekonn/lipo-bms.git
cd lipo-bms
kicad-cli version
konnect --version
codex mcp get konnect
```

KiCad 10'da proje/PCB açık ve API/IPC etkin olmalıdır. Konnect henüz Codex'e kayıtlı değilse [Linux bağlantı rehberindeki](06-konnect-codex-linux.md) `codex mcp add` komutunu uygulayıp yeni Codex sohbeti açın.
