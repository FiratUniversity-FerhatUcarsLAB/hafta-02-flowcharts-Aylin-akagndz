öğrenci no:250541011 
aylin akagündüz

BAŞLA

// Sabitler ve Başlangıç Değerleri
dogru_PIN = 1234      // Varsayılan doğru PIN
MAX_PIN_HAKKI = 3
GUNLUK_LIMIT_MAX = 1000 

// Değişkenler
PIN_hakki = MAX_PIN_HAKKI
hesap_bakiye = 1500   
gunluk_limit_kalan = GUNLUK_LIMIT_MAX
islem_tekrar = "Evet" 
kart_bloke = YANLIS
PIN_DOGRULANDI = YANLIS

// =================================================================
// 1. PIN DOĞRULAMA AŞAMASI
// =================================================================

PIN_DOGRULAMA_DONGUSU: DO_WHILE (PIN_hakki > 0 VE PIN_DOGRULANDI == YANLIS)
    YAZ "Lütfen PIN’inizi girin (Kalan hak: ", PIN_hakki, "):"
    OKU girilen_PIN

    EĞER (girilen_PIN == dogru_PIN)
        PIN_DOGRULANDI = DOGRU
        YAZ "PIN Doğrulama Başarılı."
    DEĞİLSE
        PIN_hakki = PIN_hakki - 1
        YAZ "Hatalı PIN."
        
        EĞER (PIN_hakki == 0)
            kart_bloke = DOGRU
            YAZ "PIN hakkınız bitti. Kart bloke edilmiştir."
        END_EĞER
    END_EĞER
END_DO_WHILE // PIN_DOGRULAMA_DONGUSU

// PIN başarısız olduysa veya kart bloke ise BİTİR
EĞER (PIN_DOGRULANDI == YANLIS)
    GIT_BITIR

// =================================================================
// 2. ANA İŞLEM DÖNGÜSÜ (Para Çekme ve Tekrar)
// =================================================================

ANA_ISLEM_DONGUSU: DO_WHILE (islem_tekrar == "Evet")

    ISLEM_TAMAMLANDI = YANLIS

    PARA_CEKME_DONGUSU: DO_WHILE (ISLEM_TAMAMLANDI == YANLIS)
        YAZ "Çekmek istediğiniz tutarı girin (20 TL katları):"
        OKU cekilen_tutar

        // A. Kontroller Sırası
        EĞER (cekilen_tutar % 20 != 0)
            YAZ "HATA: Tutar 20 TL katları olmalıdır."
        DEĞİLSE EĞER (cekilen_tutar > hesap_bakiye)
            YAZ "HATA: Yetersiz bakiye."
        DEĞİLSE EĞER (cekilen_tutar > gunluk_limit_kalan)
            YAZ "HATA: Günlük limit aşıldı."
        DEĞİLSE
            // D. İşlem Başarılı
            hesap_bakiye = hesap_bakiye - cekilen_tutar
            gunluk_limit_kalan = gunluk_limit_kalan - cekilen_tutar
            YAZ "İşlem başarılı. Kalan Bakiye: ", hesap_bakiye
            ISLEM_TAMAMLANDI = DOGRU 
        END_EĞER
    END_DO_WHILE // PARA_CEKME_DONGUSU

    // İşlem Tekrarı Seçeneği
    YAZ "Başka işlem yapmak ister misiniz? (Evet/Hayır):"
    OKU islem_tekrar
    
END_DO_WHILE // ANA_ISLEM_DONGUSU

GIT_BITIR:
YAZ "İşlemleriniz bitti. İyi günler dileriz."

BİTİR
