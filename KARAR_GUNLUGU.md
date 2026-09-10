# Karar Günlüğü

**Proje:** Odoo ERP + AI Voice Agent — İş Değerlendirme Case Çalışması
**Sorumlu:** Ömer Yasir Önal
**Başlangıç:** 9 Eylül 2026 · **Hedef teslim:** 15 Eylül 2026

## Bu dosya ne işe yarıyor?

Bu bir iş günlüğü ("bugün şunu yaptım") değil, bir **karar günlüğü**. Amacı, projedeki
her anlamlı seçimin arkasındaki düşünceyi kalıcı hale getirmek: hangi seçenekler vardı,
neden bu seçildi, neyin bilerek dışarıda bırakıldığı ve nerede takılındığı.

Format, klasik proje yönetimi karar günlüğü (decision log) şablonundan uyarlandı.
Şablondaki **"Onay imzası"** alanı çıkarıldı — tek kişilik bir çalışmada onay makamı
yok, doldurulsa tören olurdu. **"Katkıda bulunanlar"** alanı ise korundu ve kararda
kullanılan AI araçlarını beyan etmek için kullanılıyor (aşağıda K-003).

---

## K-001 — Odoo 19.0 Community, Docker Compose ile ayağa kaldırma

| Alan | İçerik |
|---|---|
| **Kimlik** | K-001 |
| **Tarih** | 9 Eylül 2026 |
| **Alan** | Altyapı / geliştirme ortamı |
| **Katkıda bulunanlar** | Ömer Yasir Önal |

**Açıklama**
Odoo'nun en güncel major sürümü olan **19.0 Community**, resmi `odoo:19.0` Docker image'ı
ve ayrı bir `postgres` container'ı ile, `docker compose` üzerinden çalıştırılıyor.
Custom addon'lar host'taki `./addons` klasöründen container içindeki `/mnt/extra-addons`
yoluna bağlanıyor (bind mount).

Çalışan sürüm doğrulandı: `Odoo Server 19.0-20260908`.

**Gerekçe**
- Case, "Odoo'nun **güncel** Community sürümünü Docker ile ayağa kaldır" diyor; 19.0
  bulunabilen en yeni major sürüm.
- Docker, kaynaktan kuruluma göre tekrarlanabilir: değerlendiren kişi `docker compose up`
  ile aynı ortamı elde ediyor. Bu, README'nin "nasıl ayağa kalkar" bölümünü tek komuta
  indiriyor.
- Bind mount (`./addons`) sayesinde addon kodu host'ta düzenlenip container yeniden
  başlatılarak yükleniyor; kod repoda, çalışma ortamı container'da kalıyor.
- Image mimarisi kontrol edildi: `arm64/linux`. Apple Silicon makinede **emülasyonsuz**
  çalışıyor, dolayısıyla Rosetta/QEMU kaynaklı performans veya uyumluluk riski yok.

**Alternatifler**
- *Kaynaktan (source) kurulum:* Python sürümü, sistem kütüphaneleri ve `wkhtmltopdf`
  bağımlılıklarını elle yönetmek gerekirdi. Öğretici olurdu ama case'in verdiği 15-20
  saatlik bütçede yeri yok ve değerlendiren kişinin ortamında tekrarlanabilir değil.
- *Odoo.sh / Odoo Online:* Custom addon yükleme ve Docker gereksinimi ile uyumsuz;
  case açıkça "Docker ile" diyor.
- *Odoo 18.0:* Daha oturmuş ve daha çok OCA modülü hazır. Ama case "güncel sürüm" ve
  ayrıca "güncel sürümdeki 2-3 değişikliğe kendi yorumun" istiyor — 18 seçmek o soruyu
  cevapsız bırakırdı.

**Beklenen etki**
Geliştirme ortamı tek komutla ayağa kalkıyor. Buna karşılık Odoo 19 yeni olduğu için
üçüncü parti/OCA modül desteği daha dar; bu, addon'u tamamen kendimiz yazmamız gerektiği
anlamına geliyor (zaten case'in istediği de bu).

**Kaynak**
- Resmi doküman: https://www.odoo.com/documentation/19.0/administration/on_premise/source.html
- Resmi Docker image: https://hub.docker.com/_/odoo
- Doğrulama: `docker exec <container> odoo --version` → `Odoo Server 19.0-20260908`

---

## K-002 — Veritabanı: PostgreSQL 16

| Alan | İçerik |
|---|---|
| **Kimlik** | K-002 |
| **Tarih** | 9 Eylül 2026 (10 Eylül'de kaynak doğrulaması ile revize edildi) |
| **Alan** | Altyapı / veritabanı |
| **Katkıda bulunanlar** | Ömer Yasir Önal, Claude (kaynak doğrulama) — bkz. K-003 |

**Açıklama**
`postgres:16` image'ı kullanılıyor. Çalışan sürüm doğrulandı:
`PostgreSQL 16.15 (Debian 16.15-1.pgdg13+2) on aarch64-unknown-linux-gnu`.

**Gerekçe**

Karar üç veriyi kesiştirerek verildi:

**1) Odoo tarafında sınır ne?**
Odoo 19.0 resmi dokümanı: *"Use a package manager to download and install PostgreSQL
**(supported versions: 13.0 or above)**"* ve *"Minimum requirement updated from
PostgreSQL 12 to PostgreSQL 13."*
Yani Odoo'nun beyanı **13 ve üzeri**. Bir **üst sınır belirtilmiyor**, "16 önerilir"
gibi bir ifade de resmi dokümanda **yok**.

**2) PostgreSQL tarafında sınır ne?**
PostgreSQL projesi her major sürümü ilk çıkışından itibaren **5 yıl** destekliyor.
Eylül 2026 itibarıyla:

| Sürüm | Çıkış | EOL | Durum |
|---|---|---|---|
| 18 | Eyl 2025 | Kas 2030 | Destekli |
| 17 | Eyl 2024 | Kas 2029 | Destekli |
| **16** | Eyl 2023 | **Kas 2028** | Destekli |
| 15 | Eki 2022 | Kas 2027 | Destekli |
| 14 | Eyl 2021 | **12 Kas 2026** | Destekli, ama ~2 ay kaldı |
| ≤13 | — | — | EOL |

**3) Kesişim ve eleme**
- Odoo "13+", PostgreSQL "14+ hâlâ destekli" → aday küme: **14, 15, 16, 17, 18**.
- **14 elendi:** EOL'e 2 ay kaldı. Bir case ortamı için sorun değil ama savunulabilir
  bir varsayılan değil.
- **15 elendi (zayıf eleme):** Teknik bir sorunu yok; sadece destek penceresi 16'dan
  bir yıl kısa (Kas 2027 vs Kas 2028).
- **17 / 18 elendi (risk gerekçesiyle):** Odoo çekirdeğinin bunlarda çalışmayacağına dair
  bir kanıt bulamadım — böyle bir iddiayı doğrulayamadığım için gerekçe olarak da
  kullanmıyorum. Eleme sebebi çevre ekosistem: OCA modülleri, yedekleme araçları,
  PgBouncer sürümleri ve hosting sağlayıcıları yeni major sürümlerde daha az saha
  süresine sahip. Bu bir ölçülmüş risk değil, ihtiyatlı bir tercih.
- **16 seçildi:** Destekli sürümler içinde en uzun kalan destek penceresine sahip olgun
  seçenek (Kas 2028), 3 yıllık saha süresi var ve Debian/Ubuntu 24.04 LTS depolarında
  varsayılan olarak geldiği için Odoo kurulumlarında en yaygın karşılaşılan sürüm.

**4) Ek doğrulama: Odoo bu kuralı kodda zorluyor mu?**
Çalışan container içinde Odoo 19 kaynak kodu tarandı. Minimum PostgreSQL sürümünü
kontrol eden bir sabit veya guard **bulunamadı**; bulunan tek sürüm kontrolleri çok eski
uyumluluk dalları:
```
odoo/addons/base/models/ir_sequence.py:76  → if self.env.cr._cnx.server_version < 100000:
odoo/service/db.py:222                     → 'pid' if cr._cnx.server_version >= 90200 else 'procpid'
```
Sonuç: **"desteklenen sürüm" bir test/politika beyanı, çalışma zamanı kapısı değil.**
Bu, sürüm seçimini bir uyumluluk sorusu olmaktan çıkarıp risk yönetimi sorusuna
dönüştürüyor — desteklenmeyen bir sürümde sistem büyük ihtimalle çalışır, ama sorun
çıktığında Odoo'nun destek kapsamı ve topluluğun tecrübesi yanınızda olmaz.

**5) pgvector / AI özellikleri parantezi**
Odoo 19 dokümanı, AI özellikleri için `pg-vector` eklentisinin gerektiğini ve bunun
"PostgreSQL 15 ve üzeri" için mevcut olduğunu söylüyor. Bu, sürüm seçiminde 15+ lehine
ek bir argüman gibi görünüyor — **ama bu case'i etkilemiyor**, çünkü:
- Community image'daki 692 core addon içinde `ai*` desenine uyan bir modül yok.
- Odoo veritabanı oluştururken yalnızca `pg_trgm` ve (ayar açıksa) `unaccent`
  eklentilerini kuruyor (`odoo/service/db.py:154`). `pgvector` hiç geçmiyor.

Bu gözlem doğrudan Case 1a'nın "Community ile Enterprise arasındaki fonksiyonel fark"
sorusuna somut bir örnek: **AI özellikleri Community dağıtımında yok.**

**Alternatifler**
| Seçenek | Neden seçilmedi |
|---|---|
| PostgreSQL 14 | 12 Kasım 2026'da EOL — 2 ay sonra desteksiz kalacak |
| PostgreSQL 15 | Sorunsuz, ama destek penceresi 16'dan 1 yıl kısa |
| PostgreSQL 17 | Odoo tarafında engel yok; çevre ekosistemde saha süresi daha az |
| PostgreSQL 18 | Aynı gerekçe, daha da yeni (Eylül 2025) |

**Beklenen etki**
- Case ortamı için pratik bir etkisi yok; her aday sürüm çalışırdı.
- Gerçek bir kurulumda etkisi şu: Kasım 2028'e kadar major upgrade baskısı yok.
- Bilinmesi gereken kısıt: `pg_dump` ile alınan bir yedek **daha eski** bir sunucuya geri
  yüklenemez. Yani prod 16 ise, test ortamını 17'ye çekmek yedek akışını tek yönlü hale
  getirir. (Bu, bazı kaynaklarda iddia edildiği gibi "PG 17'ye özgü bir format değişikliği"
  değil; PostgreSQL'in genel, yön bağımlı kuralı.)

**Kaynak**
| Tür | Kaynak |
|---|---|
| Birincil — ürün dokümantasyonu | https://www.odoo.com/documentation/19.0/administration/on_premise/source.html |
| Birincil — upstream proje politikası | https://www.postgresql.org/support/versioning/ |
| Birincil — kaynak kodu | Odoo 19.0 kaynak kodu (container içi doğrulama, yukarıdaki grep sonuçları) |
| İkincil — satıcı blogu | https://oec.sh/blog/odoo-server-requirements-2026 |
| İkincil — satıcı blogu | https://deploymonkey.com/blog/odoo-postgresql-version-requirements |
| İkincil — satıcı blogu | https://www.ksolves.com/blog/big-data/postgresql-and-odoo-compatibility |

**Doğrulanamayan / çürütülen iddialar**
Bu kararı verirken karşılaştığım ve **kullanmamaya karar verdiğim** iddialar:

1. *"Odoo için PostgreSQL 12-17 destekleniyor, 16 önerilir."* — Bu cümle bir satıcı
   blogundan (oec.sh) geliyor, birincil kaynak göstermiyor ve resmi Odoo 19 dokümanıyla
   uyuşmuyor (doküman "13.0 or above" diyor, üst sınır ve öneri belirtmiyor).
2. *"PostgreSQL 13 ve 14 güvenlik yaması almıyor."* — Yanlış. PostgreSQL'in resmi
   politika sayfasına göre 14, 12 Kasım 2026'ya kadar destekli; 15/16/17/18 de destekli.
   EOL olan yalnızca 13 ve altı.
3. *"ksolves.com PostgreSQL 16'yı öneriyor."* — Sayfayı okuduğumda hiçbir sürüm
   önermiyor; "resmi dokümana ve sürüm notlarına bakın" diyor.
4. `anriztech.com/blog/odoo-19-migration-guide` — sayfa HTTP 403 döndürüyor, içeriği
   doğrulanamadı. Doğrulayamadığım bir kaynağı gerekçe olarak kullanmadım.

> **Yöntem notu:** Odoo ekosisteminde partner/hosting firmalarının SEO amaçlı blog
> içeriği çok yoğun ve büyük ölçüde birbirini tekrarlıyor. Bu kararda izlenen kural:
> **iddiayı ancak birincil kaynakta (resmi doküman, upstream proje sayfası veya kaynak
> kodu) doğrulayabiliyorsam gerekçe olarak kullan.**

---

## K-003 — AI araçlarının kullanımı ve beyanı

| Alan | İçerik |
|---|---|
| **Kimlik** | K-003 |
| **Tarih** | 10 Eylül 2026 |
| **Alan** | Çalışma yöntemi |
| **Katkıda bulunanlar** | Ömer Yasir Önal |

**Açıklama**
Bu projede ChatGPT ve Claude (Claude Code) kullanılıyor. Kullanım alanları: literatür
taraması ve kaynak doğrulama, kod yazımında eşlik, doküman taslakları. Her kararın
"Katkıda bulunanlar" satırında AI katkısı varsa belirtiliyor.

**Gerekçe**
- Case AI kullanımını serbest bırakmakla kalmıyor, açıkça bekliyor.
- Aynı case şu kuralı da koyuyor: *"sunumda getirdiğin her şeyi savunman ve canlı olarak
  değiştirebilmen istenecek. Anlamadığın bir şeyi getirme."* Bu, AI çıktısının doğrudan
  teslime girmesini değil, **anlaşıldıktan sonra** girmesini gerektiriyor.
- Dolayısıyla benimsenen çalışma kuralı: AI'ın ürettiği hiçbir iddia birincil kaynakta
  doğrulanmadan karar gerekçesi olmuyor; hiçbir kod satırı ne yaptığı anlaşılmadan
  repoya girmiyor. K-002 bunun somut örneği — AI'ın ürettiği ilk özet, birincil
  kaynaklarla karşılaştırıldığında dört noktada düzeltildi.

**Alternatifler**
- *AI kullanmamak:* Case'in beklentisine aykırı ve 15-20 saatlik bütçede gerçekçi değil.
- *Kullanıp beyan etmemek:* Şeffaflık kaybı; ayrıca soru-cevap bölümünde savunulamayan
  bir içerik kalırsa maliyeti daha yüksek olur.

**Beklenen etki**
Hız kazancı, ama doğrulama adımının ek maliyeti var. K-002'de bu maliyet yaklaşık
30 dakika ek araştırma olarak gerçekleşti ve dört hatalı iddiayı teslimden çıkardı.

**Kaynak**
- Case dokümanı, "Genel kurallar" bölümü.

---

## Açık Kararlar (henüz verilmedi)

| Kimlik | Konu | Ne zaman karara bağlanacak |
|---|---|---|
| K-004 | Repo klasör yapısı: iki case tek repoda mı, ayrı klasörlerde mi? | Case 2'ye başlarken |
| K-005 | `odoo.conf` dosyasının repoya alınması ve mount edilmesi | Custom addon'a başlarken |
| K-006 | Garanti süresi alanı `product.template`'te mi `product.product`'ta mı? | Addon tasarımında |
| K-007 | Seri numarası `Char` mı, `stock.lot` ilişkisi mi? | Addon tasarımında |
| K-008 | Garanti kaydı ne zaman oluşacak: sipariş onayı mı, sevkiyat doğrulaması mı? | Addon tasarımında |
| K-009 | Case 2'de hangi seviyeye çıkılacak (metin / ses / telefon)? | Case 1 bittikten sonra, kalan süreye göre |

---

## Nerede Takıldım

**10 Eylül — Kaynak güvenilirliği (PostgreSQL sürüm araştırması)**
Bu araştırmada "takılma" teknik değil, epistemikti. AI ile hazırlanan ilk özet, üç kaynağa
atıfla net ve ikna edici bir tablo sunuyordu. Kaynaklar tek tek açıldığında: biri iddiayı
hiç içermiyordu (ksolves), biri erişilemiyordu (anriztech, HTTP 403), biri ise birincil
kaynak göstermeyen bir satıcı blogu olduğu halde resmi dokümanla çelişiyordu (oec.sh).
Kaybedilen süre yaklaşık 30 dakika; kazanılan şey, teslimin içine dört yanlış iddianın
girmemesi. Bundan sonraki tüm araştırma adımlarında birincil kaynak zorunlu tutuluyor.

**9 Eylül — (kayda değer teknik takılma yaşanmadı)**
Docker Compose ortamı ilk denemede ayağa kalktı.

---

## Bilerek Kapsam Dışı Bırakılanlar

Bu bölüm proje ilerledikçe doldurulacak. Case'in kendi ifadesiyle: *"bitmeyen kısımları
'yapmadım, çünkü…' diye yazman bizim için tamamlamış olmandan daha kıymetli."*

- **Production sertleştirmesi** (reverse proxy, TLS, `pgbouncer`, workers/`--workers`
  ayarı, yedekleme stratejisi): Case bir geliştirme ortamı istiyor. `compose.yaml`
  içindeki `odoo/odoo` kimlik bilgileri yalnızca yerel geliştirme içindir.
