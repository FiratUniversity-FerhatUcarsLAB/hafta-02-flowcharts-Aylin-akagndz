BAŞLA // Ana Sistem Başlangıcı

// =================================================================
// I. KULLANICI / KİMLİK DOĞRULAMA
// =================================================================

kullanici_dogrulandi = YANLIS

KIMLIK_DOGRULAMA_DONGUSU: DÖNGÜ (DURUM: kullanici_dogrulandi == YANLIS)
    YAZ "Lütfen TC Kimlik Numaranızı ve Şifrenizi/Doğum Tarihinizi girin:"
    OKU tc_kimlik, sifre_dogum

    EĞER (KimlikBilgisiDogrula(tc_kimlik, sifre_dogum) == DOGRU)
        kullanici_dogrulandi = DOGRU
        YAZ "Kimlik doğrulama başarılı. Hoş geldiniz!"
    DEĞİLSE
        YAZ "Hata: Bilgileriniz hatalı. Lütfen tekrar deneyin."
    END_EĞER
END_DÖNGÜ // KIMLIK_DOGRULAMA_DONGUSU

// =================================================================
// II. POLİKLİNİK VE DOKTOR SEÇİMİ
// =================================================================

randevu_hazir = YANLIS

RANDEVU_OLUSTURMA_DONGUSU: DÖNGÜ (DURUM: randevu_hazir == YANLIS)

    // 1. Poliklinik Seçimi
    YAZ "Lütfen randevu almak istediğiniz polikliniği seçin (ID):"
    poliklinik_listesi = GetPoliklinikListesi()
    YAZ poliklinik_listesi
    OKU secilen_poliklinik

    EĞER (PoliklinikGecerliMi(secilen_poliklinik) == YANLIS)
        YAZ "Hata: Geçersiz poliklinik seçimi. Lütfen tekrar deneyin."
        DEVAM_ET // Döngünün başına dön
    END_EĞER

    // 2. Doktor Listesi Görüntüleme
    YAZ "Poliklinikteki uygun doktorlar listeleniyor:"
    doktor_listesi = GetUygunDoktorlar(secilen_poliklinik)
    
    EĞER (doktor_listesi BOŞ_DEĞİL)
        YAZ doktor_listesi
        YAZ "Lütfen randevu almak istediğiniz doktoru seçin (ID):"
        OKU secilen_doktor
        
        EĞER (DoktorGecerliMi(secilen_doktor) == YANLIS)
            YAZ "Hata: Geçersiz doktor seçimi."
            DEVAM_ET
        END_EĞER
        
        // 3. Uygun Saatleri Gösterme
        YAZ "Seçilen doktor için uygun tarih ve saatler listeleniyor:"
        uygun_saatler = GetUygunSaatler(secilen_doktor)
        
        EĞER (uygun_saatler BOŞ_DEĞİL)
            YAZ uygun_saatler
            YAZ "Lütfen randevu tarih ve saatinizi seçin (Tarih-Saat ID):"
            OKU secilen_saat_id
            
            EĞER (SaatIDGecerliMi(secilen_saat_id) == DOGRU)
                randevu_hazir = DOGRU // **Döngüden çıkış koşulu ayarlandı**
            DEĞİLSE
                YAZ "Hata: Geçersiz saat seçimi."
            END_EĞER
            
        DEĞİLSE
            YAZ "Bu doktor için uygun randevu saati bulunmamaktadır. Lütfen yeni bir seçim yapın."
        END_EĞER

    DEĞİLSE
        YAZ "Bu poliklinikte şu anda randevu verebilecek uygun doktor bulunmamaktadır."
    END_EĞER

END_DÖNGÜ // RANDEVU_OLUSTURMA_DONGUSU

// =================================================================
// III. RANDEVU ONAYLAMA VE BİLDİRİM
// =================================================================

// Randevu hazır olmadan bu aşamaya gelirse, direkt bitir (olası hata durumlarında)
EĞER (randevu_hazir == YANLIS)
    YAZ "Randevu oluşturma işlemi başarıyla tamamlanamadı."
    GIT_BITIR 
END_EĞER

// Randevu Özeti
YAZ "Randevu Özeti:"
YAZ "Poliklinik: ", secilen_poliklinik
YAZ "Doktor: ", secilen_doktor
YAZ "Tarih/Saat: ", SaatIDBilgisiAl(secilen_saat_id)

YAZ "Randevuyu onaylıyor musunuz? (Evet/Hayır)"
OKU onay_cevap

EĞER (onay_cevap == "Evet")
    // 1. Randevu Kaydı ve Onay
    randevu_kayit_basarili = KaydetRandevu(tc_kimlik, secilen_doktor, secilen_saat_id)
    
    EĞER (randevu_kayit_basarili == DOGRU)
        YAZ "Randevunuz başarıyla oluşturulmuştur."
        
        // 2. SMS Gönderme
        cep_tel = GetKullaniciTelefon(tc_kimlik)
        sms_icerik = "Randevunuz: " & secilen_doktor & " " & SaatIDBilgisiAl(secilen_saat_id)
        
        EĞER (GonderSMS(cep_tel, sms_icerik) == DOGRU)
            YAZ "Randevu bilgileri SMS ile gönderildi."
        DEĞİLSE
            YAZ "UYARI: SMS gönderimi başarısız oldu. Lütfen bilgilerinizi kontrol edin."
        END_EĞER
    DEĞİLSE
        YAZ "HATA: Randevu kaydı sırasında bir sorun oluştu."
    END_EĞER

DEĞİLSE
    YAZ "Randevu oluşturma işlemi kullanıcı tarafından iptal edildi."
END_EĞER

GIT_BITIR:
YAZ "İşlem tamamlandı. Teşekkür ederiz."

BİTİR
