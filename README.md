# Zanka Haber — n8n iş akışı
# Zanka Haber | n8n X Otomatik Haber Botu

RSS haberlerini, piyasa verilerini ve YouTube içeriklerini X üzerinde paylaşmak için hazırlanan **49 düğümlü n8n otomasyonu**.

Haber havuzundan güncel ve görselli içerikleri önceliklendirir, Google Gemini ile kısa haber metinleri hazırlar, PostPulse üzerinden yayınlar ve paylaşımları Google Sheets'e kaydeder. Aynı içeriğin tekrar paylaşılmasını azaltmak için geçmiş kayıtları, benzerlik kontrollerini ve geçici paylaşım kilitlerini birlikte kullanır.

> Bu depo, v35 iş akışının kişisel hesap bağlantıları temizlenmiş paylaşım sürümüdür. Çalıştırmak için kendi n8n ortamınızı ve servis bağlantılarınızı yapılandırmanız gerekir. GitHub dosyaları barındırır; otomasyon n8n üzerinde çalışır.

## Özellikler

- **Çok kaynaklı RSS takibi:** Farklı haber kaynaklarını ortak havuzda toplar.
- **Akıllı görsel seçimi:** RSS görsellerini ve haber sayfasındaki görsel metaverilerini değerlendirir; logo, reklam ve takip görsellerini elemeye çalışır.
- **Gemini ile haber metni:** Kısa başlık ve detaylardan oluşan, 260 karakteri aşmaması istenen X gönderileri üretir.
- **Tekrar paylaşım kontrolü:** Bağlantı, içerik özeti, başlık benzerliği ve Google Sheets geçmişini karşılaştırır.
- **Geçici paylaşım kilidi:** Seçilen haberin kısa süre içinde yeniden seçilmesini azaltır.
- **Görselli veya yalnızca metin paylaşımı:** Görsel varsa medya yükleme akışını, yoksa metin paylaşımı akışını kullanır. Yüklenen medya hazır değilse paylaşımı atlar.
- **YouTube takibi:** Kanalın video, canlı yayın ve RSS kaynaklarını kontrol eder. En güncel içerik daha önce paylaşılmışsa eski videolara dönmez.
- **Piyasa özeti:** Dolar/TL, gram altın, gram gümüş ve Bitcoin verilerinden gönderi hazırlar.
- **Kayıt tutma:** Paylaşılan içerikleri Google Sheets'e yazar.
- **Zamanlama:** Haber akışına 0–10 dakika rastgele gecikme ve gece saatleri için paylaşım sınırlaması uygular.

## Nasıl çalışır?

```text
RSS kaynakları → Haber havuzu → Güncellik / tekrar / görsel kontrolü
    → Haber sayfasından görsel tamamlama → Gemini ile metin hazırlama
    → Görsel varsa yükleme ve durum kontrolü → PostPulse → X → Google Sheets

YouTube kaynakları → En güncel içeriği seçme → Tekrar kontrolü
    → PostPulse → X → Google Sheets

Piyasa API'leri → Verileri birleştirme → Piyasa özeti
    → PostPulse → X → Google Sheets
```

## Gereksinimler

| Bileşen | Kullanım |
| --- | --- |
| n8n | İş akışını çalıştırma |
| `@postpulse/n8n-nodes-postpulse` | PostPulse community node paketi |
| PostPulse ve bağlı X hesabı | Medya yükleme ve gönderi yayınlama |
| Google Gemini API bağlantısı | Haber metni oluşturma |
| Google Sheets OAuth2 bağlantısı | Paylaşım geçmişini okuma ve kaydetme |
| Google Sheets belgesi | Geçmiş kayıtlarını saklama |

RSS haberleri, piyasa verileri ve YouTube içerikleri için hazırlanan v35 iş akışının paylaşım kopyasıdır.
İş akışında n8n LangChain Agent ve Google Gemini düğümleri de kullanılır. İçe aktardıktan sonra eksik veya tanınmayan düğüm bulunmadığını kontrol edin.


1. `zanka-haber-v35-public.json` dosyasını n8n içine aktarın.
2. `@postpulse/n8n-nodes-postpulse` community node paketini kendi ortamınızda kurun.
3. PostPulse, Google Sheets OAuth2 ve Google Gemini kimlik bilgilerini n8n içinde kendiniz oluşturup ilgili düğümlere bağlayın. Bu dosya kimlik bilgisi içermez.
4. Google Sheets kullanan 5 düğümde `YOUR_GOOGLE_SHEET_ID` ve `YOUR_SHEET_NAME` alanlarını doldurun. Tablo sütunları: Hash, Baslik, Link, Kategori, Trend Skoru, Tweet Metni, Gorsel URL, Yayinlanma Tarihi.
5. Dört yayın düğümündeki `YOUR_POSTPULSE_SOCIAL_ACCOUNT_ID` alanını kendi PostPulse sosyal hesap kimliğinizle değiştirin.
6. YouTube kaynaklarında `YOUR_YOUTUBE_HANDLE` ve `YOUR_YOUTUBE_CHANNEL_ID` alanlarını doldurun.
7. Kurulumu kontrol ettikten sonra iş akışını etkinleştirin. Paylaşım kopyası pasif durumdadır.
### 1. İş akışını içe aktarın

Bu depodaki [zanka-haber-v35-public.json](./zanka-haber-v35-public.json) dosyasını indirin ve n8n'in dosyadan içe aktarma seçeneğiyle açın. PostPulse community node paketini ortamınıza kurun.

İş akışı paylaşım dosyasında **pasif** durumdadır.

### 2. Servis bağlantılarınızı ekleyin

n8n içinde kendi kimlik bilgilerinizi oluşturun ve ilgili düğümlerde seçin:

- **PostPulse OAuth2:** Altı PostPulse düğümünde.
- **Google Sheets OAuth2:** Beş Google Sheets düğümünde.
- **Google Gemini:** `Google Gemini Chat Model1` düğümünde.

Gemini düğümündeki model seçimini hesabınızda erişebildiğiniz modele göre kontrol edin.

### 3. Google Sheets tablosunu hazırlayın

Bir elektronik tablo ve çalışma sayfası oluşturun. İlk satıra aşağıdaki başlıkları, yazımlarını koruyarak ekleyin:

```text
Hash | Baslik | Link | Kategori | Trend Skoru | Tweet Metni | Gorsel URL | Yayinlanma Tarihi
```

Bu başlıkların her biri ayrı sütunda olmalıdır. Ardından aşağıdaki beş düğümde aynı belgeyi ve çalışma sayfasını seçin:

- `Log Market Post in sheet`
- `Get published rows for YouTube`
- `Log YouTube Post in sheet`
- `Log published in sheet`
- `Get row(s) in sheet`

### 4. Örnek değerleri değiştirin

| Örnek değer | Yerine yazılacak bilgi |
| --- | --- |
| `YOUR_GOOGLE_SHEET_ID` | Kendi Google Sheets belge kimliğiniz |
| `YOUR_SHEET_NAME` | Çalışma sayfanızın adı |
