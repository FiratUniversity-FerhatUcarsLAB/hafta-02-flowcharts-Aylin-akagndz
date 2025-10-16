BAŞLA 

// =================================================================
// I. ÖĞRENCİ GİRİŞİ ve HAZIRLIK
// =================================================================

OTURUM_ACIK = YANLIS
KAYITLI_DERSLER = Boş_Liste
TOPLAM_KREDI = 0
MAX_KREDI_LIMITI = 35

KAYIT_HAZIRLIK_DONGUSU: DÖNGÜ (DURUM: OTURUM_ACIK == YANLIS)
    YAZ "Lütfen Öğrenci Numaranızı ve Şifrenizi girin:"
    OKU ogrenci_no, sifre

    EĞER (GirisDogrula(ogrenci_no, sifre) == DOGRU)
        OTURUM_ACIK = DOGRU
        YAZ "Giriş başarılı. Kayıt sayfasına yönlendiriliyorsunuz."
        OGRENCI_BILGI = GetOgrenciBilgileri(ogrenci_no)
        OGRENCI_GPA = OGRENCI_BILGI.GPA 
    DEĞİLSE
        YAZ "Hata: Öğrenci No veya Şifre hatalı."
    END_EĞER
END_DÖNGÜ 

// =================================================================
// II. DERS EKLEME/ÇIKARMA VE KONTROLLER
// =================================================================

kayit_islem_devam = DOGRU

DERS_ISLEM_DONGUSU: DÖNGÜ (DURUM: kayit_islem_devam == DOGRU)

    YAZ "Mevcut Kredi: ", TOPLAM_KREDI, " / ", MAX_KREDI_LIMITI
    YAZ "1: Ders Ekle | 2: Ders Çıkar | 3: Kaydı Onayla"
    OKU islem_secim

    // ------------------------------------
    // A. DERS EKLEME İŞLEMİ
    // ------------------------------------
    EĞER (islem_secim == "1")
        YAZ "Ders listesi görüntüleniyor: ", GetTumDersListesi()
        YAZ "Eklemek istediğiniz dersin Kodunu girin:"
        OKU eklenecek_ders_kodu
        ders_bilgi = GetDersBilgisi(eklenecek_ders_kodu)

        KOSUL_HATASI = YANLIS
        
        // Önce basit kontroller
        EĞER (DersKayitliMi(KAYITLI_DERSLER, eklenecek_ders_kodu) == DOGRU)
            YAZ "Hata: Bu derse zaten kayıtlısınız."
            KOSUL_HATASI = DOGRU
        END_EĞER
        
        // Detaylı Kontroller (Sadece ilk kontroller başarılı ise yapılır)
        EĞER (KOSUL_HATASI == YANLIS)
            YENI_TOPLAM_KREDI = TOPLAM_KREDI + ders_bilgi.Kredi

            // 1. KONTROL: Kontenjan Kontrolü
            EĞER (ders_bilgi.KontenjanDoluluk >= ders_bilgi.KontenjanMaks)
                YAZ "Hata: Dersin kontenjanı doludur."
                KOSUL_HATASI = DOGRU
            // 2. KONTROL: Ön Koşul Kontrolü
            DEĞİLSE EĞER (ÖnKosulGerekliMi(ders_bilgi) == DOGRU VE ÖnKosulDersiAlındıMı(ogrenci_no, ders_bilgi.ÖnKoşulKodu) == YANLIS)
                YAZ "Hata: Ön koşul dersini tamamlamalısınız."
                KOSUL_HATASI = DOGRU
            // 3. KONTROL: Zaman Çakışması Kontrolü
            DEĞİLSE EĞER (ZamanCakısmasıVarMı(KAYITLI_DERSLER, ders_bilgi.DersGünü, ders_bilgi.DersSaati) == DOGRU)
                YAZ "Hata: Ders saatleri çakışmaktadır."
                KOSUL_HATASI = DOGRU
            // 4. KONTROL: Kredi Limiti Kontrolü
            DEĞİLSE EĞER (YENI_TOPLAM_KREDI > MAX_KREDI_LIMITI)
                YAZ "Hata: Kredi limitiniz aşıldı."
                KOSUL_HATASI = DOGRU
            END_EĞER
        END_EĞER
        
        // DERSİ EKLEME
        EĞER (KOSUL_HATASI == YANLIS)
            EkleDers(KAYITLI_DERSLER, eklenecek_ders_kodu)
            TOPLAM_KREDI = YENI_TOPLAM_KREDI
            YAZ eklenecek_ders_kodu, " dersi başarıyla eklendi."
        END_EĞER

    // ------------------------------------
    // B. DERS ÇIKARMA İŞLEMİ
    // ------------------------------------
    DEĞİLSE EĞER (islem_secim == "2")
        YAZ "Çıkarmak istediğiniz dersin Kodunu girin:"
        OKU cikarilacak_ders_kodu
        
        EĞER (DersKayitliMi(KAYITLI_DERSLER, cikarilacak_ders_kodu) == DOGRU)
            cikarilan_ders_kredi = GetDersKredisi(cikarilacak_ders_kodu)
            CikarDers(KAYITLI_DERSLER, cikarilacak_ders_kodu)
            TOPLAM_KREDI = TOPLAM_KREDI - cikarilan_ders_kredi
            YAZ cikarilacak_ders_kodu, " dersi kayıttan çıkarıldı."
        DEĞİLSE
            YAZ "Hata: Belirttiğiniz ders kaydınızda yok."
        END_EĞER

    // ------------------------------------
    // C. KAYIT ONAYLAMA İŞLEMİ
    // ------------------------------------
    DEĞİLSE EĞER (islem_secim == "3")
        
        // 5. KONTROL: Danışman Onayı Gerekli mi?
        EĞER (OGRENCI_GPA < 2.5)
            KAYIT_DURUMU = "Danışman Onayı Bekleniyor"
            YAZ "UYARI: GPA < 2.5, Danışman Onayı zorunludur."
        DEĞİLSE
            KAYIT_DURUMU = "Kayıt Başarılı"
        END_EĞER
        
        // Kayıt Özeti ve Onay
        YAZ "---------------- KAYIT ÖZETİ ----------------"
        YAZ "Kayıtlı Dersler: ", KAYITLI_DERSLER
        YAZ "Durum: ", KAYIT_DURUMU
        
        OnaylaKayit(ogrenci_no, KAYITLI_DERSLER, KAYIT_DURUMU)
        kayit_islem_devam = YANLIS 
        
    DEĞİLSE
        YAZ "Geçersiz işlem seçimi."
    END_EĞER

END_DÖNGÜ 

YAZ "Ders kayıt işlemi tamamlandı."

BİTİR
