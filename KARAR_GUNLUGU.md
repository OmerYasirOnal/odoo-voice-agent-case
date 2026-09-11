# Karar Günlüğü

**Proje:** Odoo ERP + AI Voice Agent — Case Çalışması
**Sorumlu:** Ömer Yasir Önal
**Tarih aralığı:** 9-15 Eylül 2026

Bu bir iş günlüğü değil, karar günlüğü. Her kayıt: ne seçildi, neden, hangi alternatifler
elendi, sonucu ne olacak, kanıt nerede.

**Yöntem kuralı:** Bir iddia ancak birincil kaynakta (resmi doküman, upstream proje
sayfası veya kaynak kodu) doğrulanabiliyorsa gerekçe olarak kullanılır. Satıcı blogları
ikincil kaynak sayılır ve tek başına gerekçe olmaz.

---

## K-001 — Geliştirme ortamının kurgusu

**Tarih:** 9 Eylül 2026 · **Alan:** Altyapı

Case, Odoo'nun güncel Community sürümünü Docker ile çalıştırmayı **şart koşuyor**.
Dolayısıyla "Odoo 19" ve "Docker" birer karar değil, verilen kısıt. Karar gerektiren
noktalar şunlardı:

**Açıklama**

| Seçim | Ne yapıldı |
|---|---|
| Veritabanı | Ayrı `postgres` container'ı (tek imaja gömmek yerine) |
| Custom addon yolu | Host'taki `./addons` → container'da `/mnt/extra-addons` (bind mount) |
| Compose proje adı | `compose.yaml` içinde `name:` ile sabitlendi |

**Gerekçe**
- Ayrı DB container'ı, Odoo ve PostgreSQL sürümlerini birbirinden bağımsız değiştirmeyi
  sağlıyor. K-002'yi yeniden değerlendirmek tek satırlık bir değişiklik oluyor.
- Bind mount sayesinde addon kodu host'ta düzenlenip container yeniden başlatılarak
  yükleniyor. Kod repoda, çalışma ortamı container'da kalıyor.
- Compose proje adı varsayılan olarak **klasör adından** türer. Sabitlenmezse, repoyu
  farklı bir klasör adına klonlayan kişi farklı volume'lar oluşturur ve önceki veriye
  erişemez. `name:` bunu klasör adından bağımsız hale getiriyor.

**Alternatifler**
- *Kaynaktan kurulum:* Python, sistem kütüphaneleri ve `wkhtmltopdf` bağımlılıkları elle
  yönetilirdi; değerlendiren kişinin ortamında tekrarlanabilir olmazdı.
- *Compose `healthcheck` eklemek:* Eklenmedi. Resmi Odoo image'ının entrypoint'i
  `/usr/local/bin/wait-for-psql.py` ile veritabanının hazır olmasını zaten bekliyor
  (container içinde doğrulandı). Gereksiz katman eklenmedi.

**Beklenen etki**
`docker compose up -d` tek komutla çalışan bir ortam veriyor. Buna karşılık Odoo 19 yeni
olduğu için üçüncü parti/OCA modül desteği dar — addon'un tamamen kendimiz tarafından
yazılması gerekiyor.

**Kaynak**
- Resmi image: https://hub.docker.com/_/odoo
- Doğrulama: `docker exec … odoo --version` → `Odoo Server 19.0-20260908`; image mimarisi
  `arm64/linux` (Apple Silicon'da emülasyonsuz).

---

## K-002 — Veritabanı: PostgreSQL 16

**Tarih:** 9 Eylül 2026 (10 Eylül'de birincil kaynaklarla yeniden gerekçelendirildi)
· **Alan:** Altyapı

**Açıklama**
`postgres:16` kullanılıyor. Çalışan sürüm: `PostgreSQL 16.15 (Debian) on aarch64`.

**Gerekçe**

*1) Odoo'nun beyan ettiği sınır.* Odoo 19 dokümanı: *"supported versions: 13.0 or above"*
ve *"Minimum requirement updated from PostgreSQL 12 to PostgreSQL 13."* Üst sınır yok.

*2) PostgreSQL'in kendi yaşam döngüsü.* Her major sürüm 5 yıl destekleniyor. Eylül 2026
itibarıyla destekli: 14 (EOL Kas 2026), 15 (Kas 2027), 16 (Kas 2028), 17 (Kas 2029),
18 (Kas 2030). 13 ve altı EOL.

*3) Odoo'nun kendi tercihi — belirleyici kanıt.* Odoo 19.0 branch'inde Windows kurulum
betiği şu paketi indiriyor:

```
setup/win32/setup.nsi:
StrCpy $postgresql_exe_filename "postgresql-16.14-1-windows-x64.exe"
```

Bu değişikliğin gerekçesi Odoo'nun kendi PR açıklamasında yazıyor (Mayıs 2026):
*"The Windows installer installs PostgreSQL 12. … version 12 is no longer supported,
so it's **time to bump to version 16**."*

Yani Odoo, 2026'da kendi ürünüyle **hangi PostgreSQL'i dağıtacağını** seçerken 16'yı
seçmiş. Bu, bir blog önerisi değil; şirketin kendi paketleme kararı.

*4) 17 / 18 neden değil.* Odoo çekirdeğinin 17'de çalışmadığına dair kanıt bulamadım;
bu yüzden 17'yi "çalışmaz" diye elemiyorum. Eleme sebebi şu: Odoo kendi dağıtımında 16'yı
seçmişken daha yenisine gitmenin somut bir kazancı yok, ama saha süresi daha az.

18 için ise **somut bir tehlike kaydı var.** Odoo çekirdek geliştiricisinin commit'i
(30 Eylül 2025, `ad47259b`):

> "pg18 promoted `NOT NULL` to 'real' named constraints … trying to migrate a database to
> pg18 (either upgrading a cluster from 17 to 18 or restoring a db on a pg18) the
> restoration fails with `duplicate key value violates unique constraint
> "pg_constraint_conrelid_contypid_conname_index"` … **AFAIK Odoo does not generally drop
> constraints so I don't think this will fix existing databases**, but it at least makes
> future databases compatible with pg18."

Yani PG18'de yeni veritabanı açmak çalışıyor, ama **mevcut bir Odoo veritabanını PG18'e
taşımak kırılıyor** ve düzeltme geçmiş veritabanlarını kurtarmıyor. Bir ERP'de yedekten
geri dönebilmek pazarlık konusu değil.

*5) Sonuç.* PostgreSQL 16: Eylül 2023'te çıktı, yani **3 yıllık saha tecrübesi** var;
desteği **Kasım 2028'e kadar**, yani bugünden itibaren **~2 yıl 2 ay** sürüyor. Odoo'nun
kendi dağıtım tercihi de bu. Case ortamı için de, gerçek bir kurulum için de savunulabilir.

**Alternatifler**

| Sürüm | Neden seçilmedi |
|---|---|
| 14 | 12 Kasım 2026'da EOL — 2 ay kaldı |
| 15 | Sorunsuz; resmi Docker örneğinde de bu kullanılıyor. Ama Odoo'nun güncel tercihi 16 ve destek penceresi 1 yıl kısa |
| 17 | Bilinen engel yok, ama Odoo kendi dağıtımında 16'yı seçmişken somut kazancı yok |
| 18 | Mevcut Odoo veritabanlarının PG18'e taşınması kırılıyor (yukarıdaki commit) |

**Beklenen etki**
Kasım 2028'e kadar major upgrade baskısı yok. Bilinmesi gereken kısıt: `pg_dump` yedeği
**daha eski** bir sunucuya geri yüklenemez; test ortamı prod'dan yeni bir sürüme çekilirse
yedek akışı tek yönlü hale gelir.

**Kaynak**

| Tür | Kaynak |
|---|---|
| Birincil — kaynak kodu | `odoo/odoo` @ `19.0` → `setup/win32/setup.nsi` |
| Birincil — kaynak kodu | Commit `ad47259b` (pg18 NOT NULL constraint sorunu) |
| Birincil — issue tracker | https://github.com/odoo/odoo/issues/234457 |
| Birincil — ürün dokümanı | https://www.odoo.com/documentation/19.0/administration/on_premise/source.html |
| Birincil — upstream politika | https://www.postgresql.org/support/versioning/ |
| İkincil — resmi image dokümanı | https://hub.docker.com/_/odoo (örnekte `postgres:15`) |

**Çürütülen iddialar**
Araştırma sırasında karşılaşılan ve kullanılmayan iddialar:

1. *"Odoo için PG 12-17 destekleniyor, 16 önerilir."* — Bir satıcı blogundan (oec.sh);
   birincil kaynak göstermiyor ve resmi dokümanla çelişiyor ("13.0 or above", öneri yok).
2. *"PG 13 ve 14 güvenlik yaması almıyor."* — Yanlış; 14, Kasım 2026'ya kadar destekli.
3. *"ksolves.com PG 16'yı öneriyor."* — Sayfa hiçbir sürüm önermiyor.
4. *"anriztech.com PG 16 için partitioning avantajını gösteriyor."* — Sayfanın PostgreSQL
   ile ilgili tek somut cümlesi "Odoo 19 is built to leverage PostgreSQL 16" ve bunun için
   kaynak vermiyor. Doğru sonuca farklı yoldan varmış olabilir, ama kanıt sunmadığı için
   gerekçe olarak kullanılmadı.

---

## K-003 — AI araçlarının kullanımı

**Tarih:** 10 Eylül 2026 · **Alan:** Çalışma yöntemi

**Açıklama**
Projede ChatGPT ve Claude kullanıldı: kaynak taraması, kod yazımında eşlik, doküman
taslakları. Çıktıların doğruluğu ve teslime girip girmemesi tamamen bana ait.

**Gerekçe**
Case AI kullanımını bekliyor, ama aynı zamanda *"getirdiğin her şeyi savunman"* şartını
koyuyor. Bu ikisini bağlayan kural: AI çıktısı, birincil kaynakta doğrulanmadan gerekçe
olmuyor; anlaşılmadan kod repoya girmiyor.

K-002 bunun ölçülebilir örneği: AI ile hazırlanan ilk özet dört noktada yanlıştı ve
gösterdiği üç kaynaktan biri iddiayı hiç içermiyordu. Doğrulama ~30 dakika sürdü.

**Beklenen etki**
Araştırma hızlanıyor, doğrulama maliyeti ekleniyor. Net etki pozitif — ama yalnızca
doğrulama adımı atlanmazsa.

---

## K-004 — Case senaryosundaki şirket için edition önerisi: Enterprise

**Tarih:** 11 Eylül 2026 · **Alan:** Araştırma (1a)

**Açıklama**
Araştırmanın 3. sorusunda, senaryodaki grup şirketi için **Odoo Enterprise** önerildi.
OCA modülleri buna ek olarak kullanılacak (ikisi birbirini dışlamıyor).

**Gerekçe**
- Community kurulumunda `mrp_workorder` yok (kendi kurulumumda doğrulandı). Beyaz eşya
  üretimi iş istasyonu bazlı bir süreç; OCA bu katmanı doldurmuyor.
- `account_accountant` Community'de yok. Grup yapısında çoklu şirket muhasebesi gerekiyor.
- Odoo'da major sürüm ömrü 3 yıl. Bu ölçekte yükseltme sürekli bir sorumluluk;
  Enterprise aboneliği veritabanı yükseltme servisini içeriyor.

**Alternatifler**
- *Community + OCA:* Şirketin **iç Odoo geliştirme ekibi varsa** bu tercih edilirdi.
  Enterprise'ın maliyeti kullanıcı başına ve doğrusal; üretim + depo + fulfillment
  profilinde operasyonel kullanıcı sayısı hızla üç haneye çıkar. Enterprise'ın asıl
  sattığı şey yazılım değil risk transferi (yükseltme, destek, sorumluluk); bu riski
  taşıyabilen bir ekip varsa transfer için ödeme yapmanın anlamı azalır.

**Beklenen etki**
Enterprise seçilmesi OCA'ya olan ihtiyacı ortadan kaldırmıyor: `queue_job`, `report_xlsx`,
`stock_inventory` gibi modüllerin karşılığı hiçbir edition'da yok. Ayrıca Türk kargo ve
pazaryeri entegrasyonları ile garanti takibi her iki yolda da custom yazılacak.

**Kaynak**
- `ARASTIRMA.md` → Bölüm 1.3 (Community'de bulunmayan modüllerin ölçümü) ve Bölüm 3.

---

## Açık kararlar

| Kimlik | Konu | Ne zaman |
|---|---|---|
| K-010 | Repo klasör yapısı: iki case tek repoda nasıl ayrılacak? | Case 2'ye başlarken |
| K-005 | `odoo.conf` repoya alınıp mount edilecek mi? | Addon'a başlarken |
| K-006 | Garanti süresi `product.template`'te mi `product.product`'ta mı? | Addon tasarımı |
| K-007 | Seri numarası `Char` mı, `stock.lot` ilişkisi mi? | Addon tasarımı |
| K-008 | Garanti kaydı sipariş onayında mı, sevkiyat doğrulamasında mı oluşacak? | Addon tasarımı |
| K-009 | Case 2'de hangi seviyeye çıkılacak? | Case 1 bittikten sonra |

---

## Nerede takıldım

**10 Eylül — Kaynak güvenilirliği (PostgreSQL sürüm araştırması)**
İlk araştırma üç kaynağa atıfla ikna edici bir tablo üretti. Kaynaklar tek tek
açıldığında: biri iddiayı hiç içermiyordu, biri HTTP 403 veriyordu, biri birincil kaynak
göstermeyen bir satıcı blogıydı ve resmi dokümanla çelişiyordu.

Çözüm, doğru kaynağı bulmaktı: Odoo'nun kendi deposunda `setup/win32/setup.nsi` dosyası,
Odoo'nun 19.0 sürümünde hangi PostgreSQL'i dağıttığını doğrudan gösteriyor. Ders: Odoo
ekosisteminde blog aramak yerine **odoo/odoo deposunda arama yapmak** çok daha hızlı
sonuç veriyor.

Kayıp: ~30 dakika. Kazanç: dört yanlış iddia teslime girmedi.

---

## Bilerek kapsam dışı

- **Production sertleştirmesi** (reverse proxy, TLS, PgBouncer, `--workers`, yedekleme
  stratejisi): Case bir geliştirme ortamı istiyor. `compose.yaml` içindeki `odoo/odoo`
  kimlik bilgileri yalnızca yereldir.
- **Odoo veritabanı yükseltmesi (OpenUpgrade vb.):** Sıfırdan kurulan bir ortam var,
  taşınacak veri yok.
