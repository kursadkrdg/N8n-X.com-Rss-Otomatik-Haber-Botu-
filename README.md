# n8n X Otomatik Haber Botu

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

İş akışında n8n LangChain Agent ve Google Gemini düğümleri de kullanılır. İçe aktardıktan sonra eksik veya tanınmayan düğüm bulunmadığını kontrol edin.

## Kurulum

### 1. İş akışını içe aktarın

Bu depodaki [haber-botu-v35-public.json](./haber-botu-v35-public.json) dosyasını indirin ve n8n'in dosyadan içe aktarma seçeneğiyle açın. PostPulse community node paketini ortamınıza kurun.

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
| `YOUR_POSTPULSE_SOCIAL_ACCOUNT_ID` | PostPulse içindeki bağlı X hesabınızın kimliği |
| `YOUR_YOUTUBE_HANDLE` | YouTube kanal kullanıcı adınız; başında `@` olmadan |
| `YOUR_YOUTUBE_CHANNEL_ID` | YouTube kanal kimliğiniz |

PostPulse sosyal hesap kimliğini şu dört yayın düğümünde doldurun:

- `Publish Market Post`
- `Publish YouTube Post`
- `Publish Post1`
- `Publish Post (text only)1`

YouTube bilgilerini `YouTube Videos Page`, `YouTube Streams Page` ve `YouTube Channel Feed` düğümlerinde değiştirin.

### 5. Metinleri ve zamanlamayı düzenleyin

- `AI Agent1` içindeki editör talimatlarını kendi yayın dilinize ve markanıza uyarlayın.
- `YouTube Duplicate Filter` kodundaki duyuru metinlerini kendi kanal adınıza göre değiştirin.
- Kullanmak istediğiniz RSS kaynaklarını seçin; bazı kaynak düğümleri dosyada devre dışıdır.
- `Google Trends TR RSS` düğümü dosyada devre dışıdır. Trend kaynağını kullanmak isterseniz adresini kontrol edip etkinleştirin.
- İş akışının saat dilimini `Europe/Istanbul` olarak ayarlayın ve iki zamanlayıcının gerçek ayarlarını kontrol edin.

**Zamanlama notu:** Piyasa düğümünün adında `08:00` yazmasına rağmen JSON içindeki `triggerAtHour` değeri `1` olarak kayıtlıdır. Haber zamanlayıcısı ise saat bazlıdır; düğüm notundaki 40 dakika ifadesiyle örtüşmez. İstediğiniz saat ve aralığı zamanlayıcıların ayarlarından belirleyin.

### 6. Kontrol edip etkinleştirin

Önce RSS okuma, Gemini çıktısı ve Sheets erişimini kontrol edin. Yayın düğümlerini çalıştırdığınızda bağlı X hesabına gerçek gönderi gönderilir. Çıktıları ve yayın hesabını kontrol ettikten sonra zamanlanmış çalışmayı etkinleştirin.

## Özelleştirilebilir filtreler

`Trend + Duplicate Filter` düğümündeki bazı başlangıç değerleri:

| Ayar | Değer | Açıklama |
| --- | --- | --- |
| `MAX_NEWS_AGE_HOURS` | `24` | Haber yaşı üst sınırı, saat |
| `MAX_SHEET_ROWS_TO_SCAN` | `350` | Geçmiş karşılaştırmasında taranan satır sınırı |
| `NIGHT_START_HOUR` | `3` | Gece aralığının başlangıcı |
| `NIGHT_END_HOUR` | `7` | Gece aralığının bitişi |
| `NIGHT_MAX_NON_CRITICAL_POSTS` | `2` | Gece paylaşım filtresinin eşik değeri |
| `IN_FLIGHT_LOCK_TTL_HOURS` | `12` | Geçici haber kilidinin geçerlilik süresi, saat |

Kritik haber sınıflandırması kod içindeki anahtar kelime kurallarına dayanır. Bu kuralları, reklam filtrelerini ve benzerlik eşiklerini yayın politikanıza göre düzenleyebilirsiniz.

## Paylaşım sürümü ve doğrulama

Paylaşım dosyasından hesap bağlantılarının adları ve kimlikleri, özel Sheets adresleri, sosyal hesap kimlikleri, YouTube kanal bilgileri ve n8n ortam kimlikleri çıkarılmıştır. Kurulum alanlarında örnek değerler bulunur; herkese açık RSS ve piyasa API adresleri korunmuştur.

JSON yapısı, 49 düğümün korunduğu, düğümler arasındaki bağlantıların ve JavaScript kodlarının değişmediği kontrol edilmiştir. Temizlenmiş kopya canlı servislerle n8n üzerinde çalıştırılmamıştır; kendi bağlantılarınızı ekledikten sonra ortamınızda doğrulayın.
