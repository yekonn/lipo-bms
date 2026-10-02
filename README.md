# LiPo BMS tasarım çalışması

Bu depo, LiPo paketler için bir batarya yönetim sisteminin (BMS) gereksinimlerini, Altium Designer şematik ve PCB tasarımını, LTspice analizlerini ve doğrulama planını bir araya getirmek için açıldı.

İlk araştırma: [BMS işlevleri, mimari seçenekleri ve önceki çalışmalar](docs/01-bms-arastirma.md).

Hedef paket 6S LiPo. USB-C PD/GaN adaptörden şarj ve kart üzerinde hücre balanslama için [ön mimari ve güç hesabı](docs/02-6s-usb-c-sarj.md) hazırlandı.

Hücre modeli, paralel düzen, kapasite, izin verilen şarj akımı, yük akımı ve mekanik sınırlar henüz belirlenmedi. Bu bilgiler seçilmeden koruma eşikleri ve parça numaraları tasarım kararı sayılmaz.
