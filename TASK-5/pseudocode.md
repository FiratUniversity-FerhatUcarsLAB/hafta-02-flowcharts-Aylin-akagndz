BAŞLA // Sistem Başlangıcı

// =================================================================
// I. SİSTEM BAŞLANGIÇ VE AYARLAR
// =================================================================

SISTEM_MODU = "PASİF" // Kullanıcı tarafından kontrol edilir: PASİF, AKTİF
ALARM_DURUMU = YANLIS
CEP_TELEFONU = "5XXXXXXXXX"

YAZ "Akıllı Ev Güvenlik Sistemi Başlatıldı. Mod: PASİF."

// =================================================================
// II. ANA KONTROL DÖNGÜSÜ (7/24 Çalışır)
// =================================================================

ANA_DONGU: DÖNGÜ (DURUM: SISTEM_MODU != "KAPALI") // Sürekli çalışır

    // A. Kullanıcı Komutlarını ve Sistemi Yönetme
    KOMUT = GetKullaniciKomutu()
    
    EĞER (KOMUT == "AKTİF_ET")
        SISTEM_MODU = "AKTİF"
        YAZ "Sistem AKTİF edildi."
    DEĞİLSE EĞER (KOMUT == "PASİF_ET" VEYA KOMUT == "SUSTUR")
        SISTEM_MODU = "PASİF"
        EĞER (ALARM_DURUMU == DOGRU)
            AlarmSustur()
            ALARM_DURUMU = YANLIS
            YAZ "Alarm susturuldu."
        END_EĞER
    DEĞİLSE EĞER (KOMUT == "KAPAT")
        SISTEM_MODU = "KAPALI"
        ÇIKIŞ DÖNGÜDEN
    END_EĞER

    // B. SENSÖR OKUMA VE TEHDİT KONTROLÜ
    TEHDIT_TIPI = "YOK"
    
    // 1. Kritik Sensörler (7/24 Aktif Olmalı)
    EĞER (OkuSensor("Yangın") == TEHDIT)
        TEHDIT_TIPI = "YANGIN"
    DEĞİLSE EĞER (OkuSensor("Gaz") == TEHDIT)
        TEHDIT_TIPI = "GAZ"
    END_EĞER

    // 2. Güvenlik Sensörleri (Sadece AKTİF modda)
    EĞER (SISTEM_MODU == "AKTİF" VE TEHDIT_TIPI == "YOK")
        EĞER (OkuSensor("Hareket") == TEHDIT)
            TEHDIT_TIPI = "HAREKET"
        DEĞİLSE EĞER (OkuSensor("KapiPencere") == TEHDIT)
            TEHDIT_TIPI = "ZORLA GİRİŞ"
        END_EĞER
    END_EĞER

    // C. TEHDİT MÜDAHALE (Alarm ve Bildirim)
    EĞER (TEHDIT_TIPI != "YOK")
        
        // 1. Alarmı Çal ve Durumu Güncelle
        EĞER (ALARM_DURUMU == YANLIS)
            AlarmCalistir()
            ALARM_DURUMU = DOGRU
            YAZ "ALARM: ", TEHDIT_TIPI, " algılandı!"
        END_EĞER
        
        // 2. Bildirim Gönderme
        EĞER (BildirimGonderimSüresiDolduMu() == DOGRU) 
            SMS_Icerigi = "ACİL! Evinizde " & TEHDIT_TIPI & " algılandı."
            GonderBildirim(CEP_TELEFONU, SMS_Icerigi)
        END_EĞER

        // 3. Acil Servis Çağrısı (Sadece KRİTİK tehditlerde)
        EĞER (TEHDIT_TIPI == "YANGIN" VEYA TEHDIT_TIPI == "GAZ")
            AcilServisAra(TEHDIT_TIPI)
        END_EĞER
    END_EĞER

    BEKLE(2_SANİYE) // Kontrol periyodunu belirler

END_DÖNGÜ // ANA_KONTROL_DONGUSU

BITIR // Sistem KAPALI komutuyla sonlandı
