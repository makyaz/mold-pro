# MoldPro — Kalıp Talep, Maliyet ve Teklif Yönetimi

Plastik enjeksiyon kalıpçıları için Windows masaüstü uygulaması (C# WinForms, .NET 8, SQLite).
Bir müşteri talebini **numune kabulünden** **teklif PDF'ine** kadar tek ekranda yönetir.

## Süreç

```
Numune Alındı → Teknik Analiz → Maliyetlendirme → Teklif Verildi → Sipariş Alındı / Kaybedildi
                                                   ↘ Revizyon İstendi
```

Her talep 4 sekmeden oluşur:

| Sekme | İçerik |
|---|---|
| 1. Numune / Talep | Müşteri, parça adı/kodu, numune türü ve geliş tarihi, istenen termin, yıllık adet, ekler (foto, STEP, DWG, PDF…) + resim önizleme, süreç geçmişi |
| 2. Teknik Analiz | Hammadde, göz sayısı, parça ölçüleri, et kalınlığı, yolluk tipi, sürgü/lifter, yüzey, zorluk, çelik seçimi → **kalıp ölçüsü, çelik ağırlığı, kapama kuvveti, önerilen makine, çekme payı, kaba soğuma süresi** |
| 3. Maliyet | Teknik veriden **otomatik maliyet kalemleri** (çelik, ısıl işlem, standart parça, sıcak yolluk, tasarım ve işçilik saatleri, deneme, doku). Tüm satırlar düzenlenebilir, elle kalem eklenebilir |
| 4. Teklif | Kâr %, indirim, döviz/kur, KDV, termin, ödeme koşulları; **revizyon** (R0, R1…) ve **PDF** çıktısı. Teklif durumu talebin durumunu otomatik günceller |

Ana ekranda durum şeridi (sayaçlı, tıklayınca filtreler), arama ve son teklif fiyatı görünür.

### Temalar
**Tanımlar → Ayarlar → Tema**: Açık, Koyu, Gece mavisi veya Windows ayarını izle. Değişiklik yeniden başlatınca geçerli olur. Koyu temalarda tarih seçici ve sekme şeridinin boş kısmı Windows kısıtı nedeniyle açık renkli kalır.

### Dil seçenekleri
- **Türkçe** (yerleşik) ve **İngilizce** (`Lang\en.json`). Dil, Ayarlar ekranından seçilir.
- **Teklif PDF'inin dili arayüzden bağımsızdır**: Teklif sekmesindeki "PDF dili" kutusundan seçilir (ör. arayüz Türkçe, yurt dışı müşteriye İngilizce teklif).
- Yeni dil eklemek: `Lang\en.json` dosyasını `de.json` gibi kopyalayıp sağ taraftaki metinleri çevirin, `"_name"` değerine dilin adını yazın. Kod değişikliği gerekmez; çevrilmeyen metin Türkçe görünür.
- Veritabanındaki veriler çevrilmez: otomatik üretilen maliyet kalemi açıklamaları, hesap özeti, saat ücreti adları ve CSV başlıkları Türkçe kalır. Sayı ve tarih biçimi her dilde Türkiye biçimidir (1.234,56).

- **Teklif takibi**: her talepte takip tarihi + notu. Teklif "Gönderildi" olunca 7 gün sonrasına otomatik takip önerilir; iş sonuçlanınca temizlenir. Ana listede günü gelen takipler ve **geçerliliği dolan teklifler** kırmızı görünür, "Yalnızca takibi gelenler" filtresi ve durum çubuğunda sayaç vardır.
- **Ek kalemler ve opsiyonlar**: teklife kalıp fiyatı dışında satırlar (yedek maça seti, nakliye, ek numune…) eklenir. "Opsiyon" işaretli olanlar toplama girmez, PDF'te ayrı başlıkta listelenir. Revizyonda kalemler yeni revizyona taşınır.
- **TCMB kuru**: teklif ekranında "TCMB'den al" düğmesi günlük döviz satış kurundan çapraz kuru hesaplar (internet gerekir; elle de girilebilir).
- **Parça başı kalıp payı**: yıllık adet girilmişse teklif ekranında gösterilir.
- **İç maliyet raporu (PDF)**: kategori ara toplamları ve paylarıyla maliyet dökümü + son teklifin kâr özeti. Müşteriye gitmez.
- **Raporlar**: kazanma oranı, teklif toplamı, kazanılan ve cevap bekleyen tutar; müşteri / ay / durum bazında.
- **Talep kopyalama**: benzer parça için teknik veri ve maliyet kalemleriyle yeni talep.
- **Excel'e aktarma (CSV)**: talep listesi ve maliyet kalemleri.
- **Veritabanı geçişi**: Veritabanı ilk açılışta otomatik güncellenir (yeni kolon ve tablo eklenir, veri kaybı olmaz).

## Otomatik hesap nasıl çalışır

- **Göz bloğu** = parça + her yandan et payı (%20 L, 20–60 mm); yükseklik = H + max(25, pay)
- **Yerleşim** kareye yakın ızgara; **kalıp ölçüsü** = bloklar + 20 mm aralık + çerçeve (55 mm, sıcak yollukta 70) + sürgü payı (sürgü başına 70 mm, en çok 2)
- **Kalıp yüksekliği** = bağlama + A/B plakaları + destek + itici boşluğu (+ manifold)
- **Set çeliği ağırlığı** = kalıp hacmi × %62 doluluk − göz blokları
- **Kapama** = izdüşüm alanı × göz × 1,15 (yolluk) × hammadde katsayısı; makine bir üst standart boyut (%10 emniyet)
- **İşçilik saatleri** göz sayısı, parça alanı (dm²), sürgü/lifter sayısı ve zorluk çarpanından (0,8–1,6) türetilir

Bu katsayılar **başlangıç değerleridir**. Birkaç gerçek iş üzerinden karşılaştırıp Ayarlar ekranındaki katsayıları kendi atölyenize göre ayarlayın.
