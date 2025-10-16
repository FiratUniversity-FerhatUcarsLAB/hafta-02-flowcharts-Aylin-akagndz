BAŞLA // Ana Sistem Başlangıcı

// =================================================================
// I. KULLANICI GİRİŞİ ve OTURUM AÇMA
// =================================================================

kullanıcı_oturum_açtı = YANLIS

KULLANICI_GİRİSİ_DONGUSU: DÖNGÜ (DURUM: kullanıcı_oturum_açtı == YANLIS)

    YAZ "Lütfen kullanıcı adınızı ve şifrenizi girin:"
    OKU kullanıcı_adi, şifre

    EĞER (KullanıcıDoğrula(kullanıcı_adi, şifre) == DOGRU)
        kullanıcı_oturum_açtı = DOGRU
        YAZ kullanıcı_adi, " hoş geldiniz!"
    DEĞİLSE
        YAZ "Hata: Kullanıcı adı veya şifre hatalı. Lütfen tekrar deneyin."
    END_EĞER
END_DÖNGÜ // KULLANICI_GİRİSİ_DONGUSU

// =================================================================
// II. SEPET YÖNETİMİ VE ÜRÜN EKLEME
// =================================================================

SEPET = Boş_Liste
sepete_ekle_devam = DOGRU

SEPET_YONETIM_DONGUSU: DÖNGÜ (DURUM: sepete_ekle_devam == DOGRU)

    YAZ "Lütfen sepete eklemek istediğiniz Ürün ID ve Miktarını girin:"
    OKU urun_id_yeni, miktar_yeni

    // A. Ürün ve Stok Kontrolü
    EĞER (UrunVarMi(urun_id_yeni) == DOGRU)
        stok_durumu = GetStok(urun_id_yeni)
        
        EĞER (miktar_yeni <= stok_durumu)
            // B. Ürünü Sepete Ekle
            EkleSepete(SEPET, urun_id_yeni, miktar_yeni)
            YAZ urun_id_yeni, " ürünü sepete eklendi."
        DEĞİLSE
            YAZ "Hata: Yeterli stok yok. Mevcut stok: ", stok_durumu
        END_EĞER
    DEĞİLSE
        YAZ "Hata: Belirtilen Ürün ID bulunamadı."
    END_EĞER

    YAZ "Başka ürün eklemek/işlem yapmak ister misiniz? (Evet/Hayır)"
    OKU cevap

    EĞER (cevap == "Hayır")
        sepete_ekle_devam = YANLIS
    END_EĞER
END_DÖNGÜ // SEPET_YONETIM_DONGUSU

// =================================================================
// III. ÖDEME SÜRECİ ÖNCESİ KONTROLLER
// =================================================================

// 1. Sepet Kontrolü
EĞER (SepetBoşMu(SEPET) == DOGRU)
    YAZ "Sepetiniz boş. İşlem sonlandırılıyor."
    GIT_BITIR // Ödeme aşamasına geçmeden bitir
END_EĞER

// 2. Sepet Toplamı Hesaplama
sepet_toplami = HesaplaSepetToplamı(SEPET)

// 3. İndirim Kodu Uygulama
YAZ "İndirim kodu uygulamak ister misiniz? (Evet/Hayır)"
OKU indirim_cevap

EĞER (indirim_cevap == "Evet")
    YAZ "Lütfen indirim kodunu girin:"
    OKU indirim_kodu

    EĞER (IndirimKoduGecerliMi(indirim_kodu) == DOGRU)
        indirim_miktari = HesaplaIndirim(sepet_toplami, indirim_kodu)
        sepet_toplami = sepet_toplami - indirim_miktari
        YAZ "İndirim kodu uygulandı. Yeni toplam: ", sepet_toplami
    DEĞİLSE
        YAZ "Hata: Geçersiz indirim kodu."
    END_EĞER
END_EĞER

// 4. Kargo Hesaplama
YAZ "Lütfen teslimat adresinizi girin (Kargo Hesaplama için):"
OKU teslimat_adresi
kargo_ucreti = HesaplaKargo(teslimat_adresi, sepet_toplami) 
GENEL_TOPLAM = sepet_toplami + kargo_ucreti

YAZ "Kargo Ücreti: ", kargo_ucreti
YAZ "Ödenecek Genel Toplam: ", GENEL_TOPLAM

// =================================================================
// IV. ÖDEME AŞAMASI
// =================================================================

ODEME_DURUMU = YANLIS
odeme_tekrar_deneme = DOGRU

ODEME_DONGUSU: DÖNGÜ (DURUM: ODEME_DURUMU == YANLIS VE odeme_tekrar_deneme == DOGRU)

    YAZ "Lütfen ödeme yöntemi seçin (Kredi Kartı/Havale):"
    OKU odeme_yontemi

    ODEME_BASARISIZ = YANLIS

    EĞER (odeme_yontemi == "Kredi Kartı")
        YAZ "Kart bilgilerinizi girin: (Kart No, CVV, Son Kullanma)"
        OKU kart_no, cvv, son_kullanma

        EĞER (OdemeIslemiYap(GENEL_TOPLAM, kart_no) == DOGRU)
            ODEME_DURUMU = DOGRU
            YAZ "Ödeme başarılı!"
        DEĞİLSE
            ODEME_BASARISIZ = DOGRU
            YAZ "Hata: Ödeme işlemi başarısız."
        END_EĞER

    DEĞİLSE EĞER (odeme_yontemi == "Havale")
        YAZ "Havale işlemi seçildi. Siparişiniz ödeme bekleniyor durumuna alındı."
        ODEME_DURUMU = DOGRU

    DEĞİLSE
        ODEME_BASARISIZ = DOGRU
        YAZ "Geçersiz ödeme yöntemi."
    END_EĞER
    
    // Ödeme başarısız ise tekrar sorma
    EĞER (ODEME_BASARISIZ == DOGRU)
        YAZ "Tekrar denemek ister misiniz? (Evet/Hayır)"
        OKU tekrar_cevap
        EĞER (tekrar_cevap == "Hayır")
            odeme_tekrar_deneme = YANLIS
        END_EĞER
    END_EĞER

END_DÖNGÜ // ODEME_DONGUSU

// =================================================================
// V. SİPARİŞ ONAYI VE BİTİRME
// =================================================================

EĞER (ODEME_DURUMU == DOGRU)
    YAZ "Siparişiniz başarıyla oluşturuldu."
    StoklarıGüncelle(SEPET)
    GonderSiparisOnayMaili(kullanıcı_adi)
DEĞİLSE
    YAZ "Ödeme işlemi iptal edildi veya tamamlanamadı. Sepet kaydedildi."
END_EĞER

GIT_BITIR:
YAZ "İşlem sonlandı. Teşekkür ederiz."

BİTİR // Ana Sistem Bitişi
