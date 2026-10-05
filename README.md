# LiPo BMS tasarım çalışması

Bu depo, 6S LiPo paket için bir batarya yönetim sisteminin (BMS) gereksinimlerini, KiCad şematik ve PCB tasarımını, LTspice analizlerini ve doğrulama planını bir araya getirir.

Araştırma: [BMS işlevleri, mimari seçenekleri ve önceki çalışmalar](docs/01-bms-arastirma.md).

Bu proje için verilen 6S1P/50 A gereksinimleri ve ilk blok mimarisi: [gereksinimler ve ön mimari](docs/02-gereksinimler-ve-on-mimari.md).

KiCad'de çizilecek modüller, devre taslakları, arayüzler, seçim kapıları ve sıra: [modül ve şema planı](docs/03-kicad-moduller-ve-sema-plani.md). Ön bileşen listesi: [BOM adayları](docs/bom-adaylari.csv). Bunlar üretime hazır şema/BOM değildir.

Şarj dönüştürücüsünün tam parça kodları, kılıfları, ilk devre değerleri ve KiCad footprint adayları: [6S şarj katı parça seçimi](docs/04-parca-secimi-ve-sarj-kati.md) ve [ayrıntılı şarj BOM'u](docs/bom-sarj-kati.csv). Daha yüksek güçlü USB-C seçeneğinin etkisi: [şarj gücü karşılaştırması](docs/05-sarj-gucu-secenekleri.md).

Linux'ta KiCad 10 ve Konnect'i Codex'e bağlayıp bu BMS üzerinde kullanma adımları: [Konnect + Codex rehberi](docs/06-konnect-codex-linux.md).

Yerel bilgisayarda yeni Codex sohbetine verilecek proje özeti ve ilk görev: [yerel Codex başlangıç promptu](docs/07-lokal-codex-baslangic-promptu.md).

Hedef paket 6S1P ve yaklaşık 6 Ah olarak tarif edildi. Şarj ve veri için iki ayrı USB-C soketi seçildi. Hücre modelinin üretici veri sayfası, gerçek konnektör varyantları ve mekanik/termal sınırlar henüz kesinleşmedi; bu konularda parça numaraları ve koruma eşikleri tasarım kararı sayılmaz.
