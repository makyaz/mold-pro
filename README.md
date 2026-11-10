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

## 0.3 ile gelenler

### Otomatik güncelleme
Uygulama açılışta (ve **Yardım → Güncellemeleri denetle** ile) bir `update.json` dosyasını okur; sürüm yeniyse sorar, zip'i indirir, **SHA-256** özetini doğrular, veritabanını `%AppData%\MoldPro\Yedekler` klasörüne yedekler, dosyaları değiştirip yeniden başlar.

Yayın hazırlamak:

```bat
powershell -ExecutionPolicy Bypass -File tools\release.ps1 -Version 0.3.1 -BaseUrl https://ornek.com/moldpro -Notes "Hata düzeltmeleri"
```

`dist` klasöründe oluşan `MoldPro-0.3.1-win-x64.zip` ve `update.json` dosyalarını o adrese (kendi sunucunuz, GitHub Releases vb.) yükleyin. Uygulamada **Tanımlar → Ayarlar → Güncelleme adresi** alanına `https://ornek.com/moldpro/update.json` yazın; müşterilere dağıtırken adresi `UI/Updater.cs` içindeki `DefaultManifestUrl` sabitine gömebilirsiniz.

- Yalnızca `https` adresleri kabul edilir; özet uyuşmazsa güncelleme iptal olur.
- Uygulama, kullanıcının yazma izni olan bir klasörde durmalıdır (Program Files altında yönetici izni gerekir; bu durumda uyarı verir).
- `update.json` biçimi: `{ "version": "0.3.1", "url": "https://…zip", "sha256": "…", "notes": "…" }`

### Temalar
**Tanımlar → Ayarlar → Tema**: Açık, Koyu, Gece mavisi veya Windows ayarını izle. Değişiklik yeniden başlatınca geçerli olur. Koyu temalarda tarih seçici ve sekme şeridinin boş kısmı Windows kısıtı nedeniyle açık renkli kalır.

### Dil seçenekleri
- **Türkçe** (yerleşik) ve **İngilizce** (`Lang\en.json`). Dil, Ayarlar ekranından seçilir.
- **Teklif PDF'inin dili arayüzden bağımsızdır**: Teklif sekmesindeki "PDF dili" kutusundan seçilir (ör. arayüz Türkçe, yurt dışı müşteriye İngilizce teklif).
- Yeni dil eklemek: `Lang\en.json` dosyasını `de.json` gibi kopyalayıp sağ taraftaki metinleri çevirin, `"_name"` değerine dilin adını yazın. Kod değişikliği gerekmez; çevrilmeyen metin Türkçe görünür.
- Veritabanındaki veriler çevrilmez: otomatik üretilen maliyet kalemi açıklamaları, hesap özeti, saat ücreti adları ve CSV başlıkları Türkçe kalır. Sayı ve tarih biçimi her dilde Türkiye biçimidir (1.234,56).

## 0.2 ile gelenler

- **Teklif takibi**: her talepte takip tarihi + notu. Teklif "Gönderildi" olunca 7 gün sonrasına otomatik takip önerilir; iş sonuçlanınca temizlenir. Ana listede günü gelen takipler ve **geçerliliği dolan teklifler** kırmızı görünür, "Yalnızca takibi gelenler" filtresi ve durum çubuğunda sayaç vardır.
- **Ek kalemler ve opsiyonlar**: teklife kalıp fiyatı dışında satırlar (yedek maça seti, nakliye, ek numune…) eklenir. "Opsiyon" işaretli olanlar toplama girmez, PDF'te ayrı başlıkta listelenir. Revizyonda kalemler yeni revizyona taşınır.
- **TCMB kuru**: teklif ekranında "TCMB'den al" düğmesi günlük döviz satış kurundan çapraz kuru hesaplar (internet gerekir; elle de girilebilir).
- **Parça başı kalıp payı**: yıllık adet girilmişse teklif ekranında gösterilir.
- **İç maliyet raporu (PDF)**: kategori ara toplamları ve paylarıyla maliyet dökümü + son teklifin kâr özeti. Müşteriye gitmez.
- **Raporlar**: kazanma oranı, teklif toplamı, kazanılan ve cevap bekleyen tutar; müşteri / ay / durum bazında.
- **Talep kopyalama**: benzer parça için teknik veri ve maliyet kalemleriyle yeni talep.
- **Excel'e aktarma (CSV)**: talep listesi ve maliyet kalemleri.
- **Veritabanı geçişi**: 0.1 veritabanı ilk açılışta otomatik güncellenir (yeni kolon ve tablo eklenir, veri kaybı olmaz).

## Kurulum / derleme

Gereken: Windows 10/11, **.NET 8 SDK** (veya Visual Studio 2022 17.8+ ".NET masaüstü geliştirme" iş yükü).

```bat
cd MoldPro
dotnet restore
dotnet run --project src\MoldPro
```

Visual Studio ile: `MoldPro.sln` dosyasını açıp F5.

Tek dosya .exe almak için:

```bat
dotnet publish src\MoldPro -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true
```

NuGet paketleri: `Microsoft.Data.Sqlite` (veritabanı), `QuestPDF` (teklif PDF'i; yıllık cirosu 1M USD altındaki firmalar için ücretsiz Community lisansı — `Program.cs` içinde ayarlı).

## Veri

- Veritabanı: `%AppData%\MoldPro\moldpro.db` (ilk açılışta oluşturulur, örnek çelik/hammadde/saat ücretleri yüklenir)
- Ek dosyalar: `%AppData%\MoldPro\Ekler\<TalepNo>\`
- Yedek: **Dosya → Veritabanı yedeği al** (`VACUUM INTO` ile tutarlı kopya). Ek klasörünü ayrıca kopyalayın.

## İlk kullanımda yapılacaklar

1. **Tanımlar → Firma, teklif ve hesap ayarları**: firma bilgisi, logo, ana para birimi, varsayılan kâr/KDV/termin/ödeme koşulları.
2. **Tanımlar → Makine / işçilik saat ücretleri**: kendi saat ücretleriniz (genel gider dahil). Kodlar (TASARIM, CNC, EDM, TEL, TASLAMA, TORNA, POLISAJ, MONTAJ, DENEME) otomatik hesapta kullanılır.
3. **Tanımlar → Kalıp çelikleri**: güncel kg fiyatları.
4. **Tanımlar → Müşteriler**.

## Otomatik hesap nasıl çalışır (`Services/MoldCalculator.cs`)

- **Göz bloğu** = parça + her yandan et payı (%20 L, 20–60 mm); yükseklik = H + max(25, pay)
- **Yerleşim** kareye yakın ızgara; **kalıp ölçüsü** = bloklar + 20 mm aralık + çerçeve (55 mm, sıcak yollukta 70) + sürgü payı (sürgü başına 70 mm, en çok 2)
- **Kalıp yüksekliği** = bağlama + A/B plakaları + destek + itici boşluğu (+ manifold)
- **Set çeliği ağırlığı** = kalıp hacmi × %62 doluluk − göz blokları
- **Kapama** = izdüşüm alanı × göz × 1,15 (yolluk) × hammadde katsayısı; makine bir üst standart boyut (%10 emniyet)
- **İşçilik saatleri** göz sayısı, parça alanı (dm²), sürgü/lifter sayısı ve zorluk çarpanından (0,8–1,6) türetilir

Bu katsayılar **başlangıç değerleridir**. Birkaç gerçek iş üzerinden karşılaştırıp `MoldCalculator.cs` ve Ayarlar ekranındaki katsayıları kendi atölyenize göre ayarlayın.

## Proje yapısı

```
src/MoldPro/
  Program.cs              giriş, kültür (tr-TR), QuestPDF lisansı
  Data/Db.cs              bağımlılıksız SQLite yardımcısı (Query / Execute / Save)
  Data/Schema.cs          tablolar + başlangıç verisi
  Data/Repo.cs            tüm tablo işlemleri, numaralandırma (MP-2026-0001, TKF-2026-0001)
  Models/Models.cs        varlıklar, durum listeleri, ayarlar
  Services/MoldCalculator.cs  kalıp ön boyutlandırma + maliyet kalemleri
  Services/QuotePdf.cs    müşteri teklif PDF'i (maliyet dökümü içermez)
  Services/CostReportPdf.cs   iç kullanım maliyet raporu
  Services/ExchangeRates.cs   TCMB günlük kur
  Services/Csv.cs         Excel uyumlu CSV
  Services/L.cs           çeviri altyapısı (Lang\*.json)
  Lang/en.json            İngilizce çeviri
  UI/Theme.cs             temalar + BaseForm
  UI/Updater.cs           otomatik güncelleme
tools/release.ps1         yayın zip'i + update.json üretir
  UI/                     formlar (kodla kurulur, Designer dosyası yok)
```

## Sonraki adımlar için fikirler

- Parça birim maliyeti hesabı (hammadde + çevrim süresi + makine saati)
- Teklif gönderimi için e-posta entegrasyonu
- Gerçekleşen maliyet takibi (teklif vs. gerçek) ve katsayıların bundan öğrenmesi
- Sac pres / döküm kalıbı modülleri
- Çok kullanıcılı kullanım gerekirse SQL Server Express'e geçiş (Repo katmanı tek yerde)
