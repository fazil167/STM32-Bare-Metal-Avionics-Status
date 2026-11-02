# STM32 Bare-Metal Avionics Status & Fault Monitoring System

STM32F407 mikrodenetleyicisi üzerinde bare-metal (register düzeyinde) olarak geliştirilmiş, çok kanallı sensör izleme ve gerçek zamanlı öncelikli hata yönetimi (fault monitoring) sistemi.

---

 
Bu proje; STM32F407G-DISC1 kiti üzerinde HAL veya standart çevre birimi kütüphaneleri kullanılmadan, tamamen doğrudan CMSIS register erişimiyle geliştirilmiştir. Çoklu analog sensör verilerini donanımsal tarama modu (Scan Mode) ile toplayıp işleyen ve kritik eşik değerlerine göre durum/hata sinyali üreten bir aviyonik durum izleme mimarisidir.

---

##   Kullanılan Donanım ve Teknolojiler
- **Mikrodenetleyici:** STM32F407VGT6 (ARM Cortex-M4)
- **Geliştirme Kiti:** STM32F407G-DISC1
- **Geliştirme Ortamı:** STM32CubeIDE
- **Programlama Dili & Mimari:** Bare-Metal C, CMSIS Register-Level Programming
- **Bileşenler:**
  - NTC Sıcaklık Sensörü (Sıcaklık İzleme)
  - 2x Potansiyometre (Ana Güç ve Yedek Güç Simülasyonu)
  - 3x Durum LED'i (Yeşil, Sarı, Kırmızı)
  - Aktif Buzzer (Kritik Alarm)
  - 3 Pinli Sürgülü Anahtar (Sistem Güvenlik/Devreye Alma Kilidi)

---

##   Sistem Mimarisi ve Özellikler

### 1. Çoklu ADC Okuma (Hardware Scan Mode)
ADC1 birimi üzerinde yazılımsal kanal değiştirme gecikmelerini önlemek adına donanımsal **Scan Mode** yapılandırılmıştır. `SQR1` ve `SQR3` register'ları üzerinden 3 kanal (PA3, PA0, PA1) belirlenen sırada taranır ve `EOCS` bayrağı ile ardışık olarak okunur.

### 2. Öncelikli Hata Yönetimi (Priority Fault Management)
Sistem sensör verilerini belirli bir hiyerarşiye göre değerlendirir:
- **Kritik Durum:** Sıcaklık eşiği aşıldığında (`ntc > 150`) veya ana güç hattında düşüş yaşandığında (`pot1 < 50`), sistem anında **Kırmızı LED** ve **Buzzer** ile sesli/görsel acil durum uyarısına geçer.
- **Uyarı Durumu:** Kritik bir sorun yokken yedek güç seviyesi düştüğünde (`pot2 < 100`), **Sarı LED** ile uyarı verilir.
- **Normal Durum:** Bütün telemetri verileri güvenli aralıktaysa sadece **Yeşil LED** yanar.

### 3. Donanımsal Güvenlik Kilidi
PC0 pinine bağlı sürgülü anahtar üzerinden sistemin donanımsal devreye alma/kapatma kontrolü sağlanır. Anahtar pasif konumdayken tüm sistem güvenli bekleme moduna geçer.

---

##   Pin ve Donanım Bağlantı Tablosu

| Komponent / Sinyal | STM32 Pini | Mod / Konfigürasyon | Görev |
| :--- | :--- | :--- | :--- |
| **Sürgülü Anahtar** | `PC0` | Digital Input | Sistem Aktif / Pasif Güvenlik Girişi |
| **Yeşil LED** | `PD0` | Digital Output | Normal Durum Göstergesi |
| **Sarı LED** | `PD1` | Digital Output | İkaz / Uyarı Durum Göstergesi |
| **Kırmızı LED** | `PD2` | Digital Output | Kritik Hata Göstergesi |
| **Buzzer** | `PD3` | Digital Output | Sesli Alarm Çıkışı |
| **Sıcaklık Sensörü (NTC)** | `PA3` | Analog (ADC1_IN3) | SQ1 - Sıcaklık Telemetrisi |
| **Ana Güç Simülasyonu (Pot 1)** | `PA0` | Analog (ADC1_IN0) | SQ2 - Ana Güç Seviyesi |
| **Yedek Güç Simülasyonu (Pot 2)** | `PA1` | Analog (ADC1_IN1) | SQ3 - Yedek Güç Seviyesi |

---

##  Derleme ve Yükleme
1. Projeyi **STM32CubeIDE** içerisine `Existing Projects into Workspace` seçeneğiyle aktarın.
2. `Project -> Build Project` adımıyla derleyin.
3. Kartınızı ST-LINK üzerinden bilgisayara bağlayıp kodu yükleyin (Run/Debug).
4. İsteğe bağlı olarak IDE içerisindeki *Live Expressions* sekmesine `ntc`, `pot1` ve `pot2` değişkenlerini ekleyerek sensör verilerini canlı takip edebilirsiniz.
