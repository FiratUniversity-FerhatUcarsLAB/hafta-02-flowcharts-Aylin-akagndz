BAŞLA // Ana Sistem Başlangıcı

// =================================================================
// I. ANA MENÜ VE MODÜL SEÇİMİ
// =================================================================

sistem_devam = DOGRU

ANA_SISTEM_DONGUSU: DÖNGÜ (DURUM: sistem_devam == DOGRU)

    YAZ "Hoş geldiniz. Lütfen yapmak istediğiniz işlemi seçin:"
    YAZ "1: Randevu Al"
    YAZ "2: Tahlil Sonuçları Görüntüle"
    YAZ "3: Çıkış"
    OKU ana_secim

    EĞER (ana_secim == "1")
        ÇALIŞTIR Modul_Randevu_Alma()
    DEĞİLSE EĞER (ana_secim == "2")
        ÇALIŞTIR Modul_Tahlil_Sonuclari()
    DEĞİLSE EĞER (ana_secim == "3")
        sistem_devam = YANLIS
    DEĞİLSE
        YAZ "Hata: Geçersiz seçim. Lütfen tekrar deneyin."
    END_EĞER

END_DÖNGÜ // ANA_SISTEM_DONGUSU

YAZ "Sistemden çıkılıyor. İyi günler dileriz."

BİTİR


// ***************************************************************
// MODÜL 1: RANDEVU ALMA İŞLEVİ
// ***************************************************************

FONKSİYON Modul_Randevu_Alma()

    YAZ "--- Randevu Alma Sistemi ---"
    kullanici_dogrulandi = YANLIS

    // I. KİMLİK DOĞRULAMA (Tekrar Kullanıcı Girişi İstenir)
    KIMLIK_DOGRULAMA_DONGUSU: DÖNGÜ (DURUM: kullanici_dogrulandi == YANLIS)
        YAZ "Lütfen TC Kimlik Numaranızı ve Şifrenizi/Doğum Tarihinizi girin:"
        OKU tc_kimlik, sifre_dogum

        EĞER (KimlikBilgisiDogrula(tc_kimlik, sifre_dogum) == DOGRU)
            kullanici_dogrulandi = DOGRU
            YAZ "Kimlik doğrulama başarılı."
        DEĞİLSE
            YAZ "Hata: Bilgileriniz hatalı. Lütfen tekrar deneyin."
        END_EĞER
    END_DÖNGÜ 

    // II. RANDEVU SEÇİM AŞAMALARI
    randevu_hazir = YANLIS
    RANDEVU_OLUSTURMA_DONGUSU: DÖNGÜ (DURUM: randevu_hazir == YANLIS)

        // Poliklinik Seçimi
        // ... (Poliklinik Seçimi, Geçerlilik Kontrolü ve Hata Mesajları) ...
        OKU secilen_poliklinik

        // Doktor Seçimi
        doktor_listesi = GetUygunDoktorlar(secilen_poliklinik)
        EĞER (doktor_listesi BOŞ_DEĞİL)
            OKU secilen_doktor
            
            // Uygun Saatleri Gösterme
            uygun_saatler = GetUygunSaatler(secilen_doktor)
            EĞER (uygun_saatler BOŞ_DEĞİL)
                OKU secilen_saat_id
                EĞER (SaatIDGecerliMi(secilen_saat_id) == DOGRU)
                    randevu_hazir = DOGRU
                DEĞİLSE
                    YAZ "Hata: Geçersiz saat seçimi."
                END_EĞER
            DEĞİLSE
                YAZ "Bu doktor için uygun saat yok. Yeni seçim yapın."
            END_EĞER
        DEĞİLSE
            YAZ "Bu poliklinikte uygun doktor yok. Yeni seçim yapın."
        END_EĞER
    END_DÖNGÜ 

    // III. RANDEVU ONAYLAMA VE BİLDİRİM
    YAZ "Randevuyu onaylıyor musunuz? (Evet/Hayır)"
    OKU onay_cevap

    EĞER (onay_cevap == "Evet")
        randevu_kayit_basarili = KaydetRandevu(tc_kimlik, secilen_doktor, secilen_saat_id)
        
        EĞER (randevu_kayit_basarili == DOGRU)
            YAZ "Randevu başarıyla oluşturulmuştur."
            
            // SMS Gönderme
            cep_tel = GetKullaniciTelefon(tc_kimlik)
            sms_icerik = "Randevunuz: " & secilen_doktor & " " & SaatIDBilgisiAl(secilen_saat_id)
            GonderSMS(cep_tel, sms_icerik)
        DEĞİLSE
            YAZ "HATA: Randevu kaydı sırasında bir sorun oluştu."
        END_EĞER
    DEĞİLSE
        YAZ "Randevu oluşturma işlemi iptal edildi."
    END_EĞER
    
    YAZ "Randevu alma modülü tamamlandı."

END_FONKSİYON // Modul_Randevu_Alma

// ***************************************************************
// MODÜL 2: TAHLİL SONUÇLARI GÖRÜNTÜLEME İŞLEVİ
// ***************************************************************

FONKSİYON Modul_Tahlil_Sonuclari()

    YAZ "--- Tahlil Sonuçları Sistemi ---"
    kullanici_dogrulandi = YANLIS

    // I. KİMLİK DOĞRULAMA (Randevu modülü ile aynı)
    DOG_DONGUSU: DÖNGÜ (DURUM: kullanici_dogrulandi == YANLIS)
        YAZ "Lütfen TC Kimlik Numaranızı ve Şifrenizi/Doğum Tarihinizi girin:"
        OKU tc_kimlik, sifre_dogum

        EĞER (KimlikBilgisiDogrula(tc_kimlik, sifre_dogum) == DOGRU)
            kullanici_dogrulandi = DOGRU
            YAZ "Kimlik doğrulama başarılı."
        DEĞİLSE
            YAZ "Hata: Bilgileriniz hatalı. Lütfen tekrar deneyin."
        END_EĞER
    END_DÖNGÜ

    // II. TAHLİL VARLIĞI VE DURUM KONTROLÜ
    
    // Tahlil Varlığı Kontrolü
    EĞER (TahlilVarMi(tc_kimlik) == YANLIS)
        YAZ "Size ait beklemede veya tamamlanmış tahlil bulunmamaktadır."
        ÇIKIŞ FONKSİYONDAN
    END_EĞER
    
    // Sonuç Hazırlık Kontrolü
    EĞER (SonucHazirMi(tc_kimlik) == DOGRU)
        YAZ "Tahlil sonuçlarınız hazırdır."
        
        // Görüntüleme ve İndirme
        YAZ "Tahlil sonuçları ekranda gösteriliyor..."
        YAZ "Sonucu PDF olarak indirmek ister misiniz? (Evet/Hayır)"
        OKU pdf_cevap
        
        EĞER (pdf_cevap == "Evet")
            DosyaIndir(tc_kimlik, "TahlilSonucu.pdf")
            YAZ "Tahlil sonuçlarınız PDF olarak indirildi."
        END_EĞER
        
    DEĞİLSE
        // Bekleme Mesajı
        YAZ "Tahlil sonuçlarınız henüz hazır değildir."
        tahmini_sure = GetTahminiSure()
        YAZ "Tahmini sonuçlanma süresi: ", tahmini_sure
    END_EĞER
    
    YAZ "Tahlil sonuçları görüntüleme modülü tamamlandı."

END_FONKSİYON // Modul_Tahlil_Sonuclari
