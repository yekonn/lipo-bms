# Linux'ta KiCad 10 + Konnect + Codex kullanımı

Bu rehber, BMS projesindeki KiCad şeması ve PCB'sini Konnect üzerinden Codex ile düzenlemek içindir. Konnect'in KiCad **Plugin and Content Manager** içine eklenmesi ile **Codex MCP bağlantısı** iki ayrı kurulum adımıdır. Eklentinin PCB Editor menüsünde görünmesi, Codex'in MCP araçlarını otomatik gördüğü anlamına gelmez.

## 1. Kurulumu denetle

Linux terminalinde:

```bash
kicad-cli version       # 10.x beklenir
konnect --version       # ör. konnect 0.13.0
codex mcp list          # konnect kaydı var mı?
```

`konnect` komutu bulunmuyorsa PCM paketi yalnız KiCad eklenti dizinine kurulmuş olabilir. KiCad PCB Editor → **Tools → External Plugins → Konnect** menüsüyle eklentinin kurulduğunu doğrulayın. Codex kaydı için Konnect'in **sunucu ikilisinin mutlak yolunu** kullanın. Resmi [Konnect Releases](https://github.com/mixelpixx/Konnect/releases) sayfasında Linux için ayrıca `konnect-v...-x86_64-unknown-linux-gnu.tar.gz` bulunur; PCM zip dosyasından farklıdır. İndirilen sürüm ile `konnect --version` çıktısını kontrol edin.

Bu projede ilk Codex bağlantısının genel komutu:

```bash
codex mcp add konnect -- /mutlak/yol/konnect --client codex
codex mcp get konnect
```

Konnect zaten kayıtlıysa önce `codex mcp get konnect` ile komut yoluna bakın. Yanlış kaydı düzeltirken `codex mcp remove konnect` ardından doğru mutlak yolla `add` kullanın. Codex'in açık oturumunda araç listesi önbelleğe alınmış olabilir; **yeni Codex oturumu** başlatın. Bu sohbetin Linux bulut oturumuna eklenen kayıt, kendi Linux masaüstünüzün Codex ayarına otomatik kopyalanmaz.

İsteğe bağlı Konnect Codex rehber becerileri için, ayrı `konnect-codex` companion eklentisi kurulmamışsa `konnect init --client codex` ve `konnect status --client codex` kullanılır. Companion eklentisi kuruluysa onun yönergeleri izlenir; iki farklı rehber paketini üst üste kurmayın. MCP bağlantısı için rehber becerilerini kurmak zorunlu değildir.

## 2. Bağlantıyı gerçekten doğrula

Yeni Codex oturumunda şu talimatı verin:

> Konnect `get_installation_info` aracını çağır. Konnect sürümünü, kullanılan sunucu dosyasının yolunu, tespit edilen `kicad-cli` yolunu ve KiCad sürümünü göster. Sonra `list_toolboxes` çağır; hiçbir proje dosyasını değiştirme.

Beklenen: Konnect sürümü döner ve `kicad.version` alanı **10.x** olur. Yalnız `codex mcp list` sonucunun `enabled` olması araç çağrısının çalıştığını kanıtlamaz. `kicad.version=9.x` ise Codex/ Konnect başka bir `kicad-cli` buluyordur; Konnect yapılandırmasında `kicad_cli` yolunu KiCad 10 ikilisine ayarlayın ve yeni oturumda tekrar deneyin.

Şema işlemleri `.kicad_sch` dosyasına doğrudan yazılabilir. Çoğu PCB yerleşim/yönlendirme işi için KiCad 10 **PCB Editor'da ilgili `.kicad_pcb` dosyası açık** olmalı; KiCad Preferences → Plugins içinde **KiCad API/IPC** etkin olmalı. Konnect `check_kicad_ui` bu bağlantıyı kontrol eder. `get_installation_info` CLI sürümünün 10 olmasını, `check_kicad_ui` ise canlı editör IPC bağlantısını ayrı ayrı gösterir.

## 3. BMS projesinde örnek çalışma akışı

Bu depoda henüz gerçek `.kicad_pro/.kicad_sch/.kicad_pcb` dosyaları oluşturulmadı. [Modül sırası](03-kicad-moduller-ve-sema-plani.md), [seçilmiş şarj entegreleri/kılıfları](04-parca-secimi-ve-sarj-kati.md) ve [BOM](bom-sarj-kati.csv) tasarım girdisidir.

1. **Proje ve hiyerarşi:** Codex'e Konnect'in `project` ve `sch_hierarchy` araçlarıyla proje oluşturmasını, `00_TOP` ve 01–08 modül sayfalarını eklemesini söyleyin. Sayfa pinleri, `BAT_FUSED+`, `SYS_GND`, `LOAD+`, `CHARGE_PERMIT` gibi belgelenmiş netlerle eşleşsin. Var olan dosyaların üzerine yazılmasın.
2. **Özel kütüphane:** `library` araçlarıyla standart KiCad kütüphanesinde bulunmayan **BQ25756RRVR** (1–36 + pad 37) ve **BQ7694202PFBR** sembollerini oluşturun. Üretici veri sayfasındaki pin numarası, elektrik tipi, package/footprint pad haritasını ayrı inceleyin. STUSB4500QTR ve STM32G0B1CBTx stok sembollerinin seçilen sipariş kodlarına uyumunu denetleyin.
3. **Şarj katı:** `sch_components`, `sch_wiring` ve toplu yerleştirme araçlarıyla `05_CHARGER` sayfasında dört MOSFET, 10 µH bobin, giriş/akü 5 mΩ şöntleri, geri besleme ve akım sınır dirençlerini bağlayın. `CE` açılışta kapalı; USB-C PD ile şarj izni ve AFE kesme hattı ayrı kalsın.
4. **Her sayfa sonrası kontrol:** `run_erc`, `generate_netlist`, `validate_sheet_pins`, `audit_connections` ve `audit_power_rails` çıktısını kaydedin. Hatasız ERC'den sonra netlistte özellikle B0–B6 sıra, şönt Kelvin uçları, PD VBUS ile veri USB VBUS ayrımı ve şarj/yük güç yolları gözden geçirilsin.
5. **PCB:** Şemadan PCB'ye aktarımı önce `update_pcb_from_schematic` **dry-run** ile inceleyin. 50 A/90 A yol, şönt Kelvin izleri, FET ısı yayılımı, sigorta ve XT60 yerleşimini elle mühendislik incelemesinden geçirin. Ardından yerleştirme/yönlendirme, `run_drc`, Gerber ve montaj dosyaları gelir.

Konnect'in referans `buck_converter` şablonu bu BMS'nin **20 V girişten 25,2 V aküye** şarj eden dört anahtarlı buck-boost devresinin yerine geçmez. ERC/DRC; MOSFET SOA, sigorta DC kesme kapasitesi, 50 A sıcaklık artışı, USB PD sözleşmesi veya hücre şarj güvenliğini tek başına doğrulamaz. Bu hesaplar ve prototip ölçümleri ayrıca tamamlanır.

## 4. Örnek Codex isteği

> Konnect ile yalnız `01_PACK_INPUT` ve `02_CELL_AFE` KiCad şema sayfalarını oluştur. Önce `get_installation_info` ile KiCad 10'u doğrula. [modül planındaki](03-kicad-moduller-ve-sema-plani.md) netleri ve [parça seçimindeki](04-parca-secimi-ve-sarj-kati.md) BQ7694202PFBR kodunu kullan. Eksik pin/footprint bilgisi varsa datasheet'ten doğrula. Her mutasyondan sonra proje dosyalarını göster; iş sonunda ERC, hiyerarşi pin denetimi ve netlist özeti ver. 50 A PCB güç yolunu bu adımda çizme.

Kaynak: [Konnect README](https://github.com/mixelpixx/Konnect), [Konnect araç kataloğu](https://github.com/mixelpixx/Konnect/blob/main/tool-directory.md).
