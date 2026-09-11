# Case 1a — Odoo Araştırması

**Hazırlayan:** Ömer Yasir Önal · **Tarih:** 11 Eylül 2026
**Test ortamı:** Odoo 19.0 Community (`19.0-20260908`), Docker, PostgreSQL 16

## Yöntem notu

Bu araştırmada bir kural uyguladım: **bir iddiayı ancak birincil kaynakta
doğrulayabiliyorsam yazdım.** Birincil kaynak derken resmi dokümantasyon, upstream proje
sayfaları ve kaynak kodunun kendisini kastediyorum.

Bu kuralı koymamın sebebi, araştırmanın ilk turunda yaşadığım bir olay: PostgreSQL sürüm
seçimi için baktığım üç satıcı blogundan biri iddiayı hiç içermiyordu, biri erişilemiyordu,
biri de resmi dokümanla çelişiyordu. Odoo ekosisteminde partner/hosting firmalarının SEO
içeriği çok yoğun ve büyük ölçüde birbirini kopyalıyor. Ayrıntısı `KARAR_GUNLUGU.md` →
K-002 → "Çürütülen iddialar" bölümünde.

Aşağıdaki bulguların bir kısmını **kendim ölçtüm** — çalışan Odoo kurulumunun içine ve
Odoo'nun kaynak deposuna bakarak. Bu ölçümlerin nasıl tekrarlanacağı en sonda, Ek A'da.

---

# 1. Community, Enterprise ve OCA nedir, nasıl ayrılır?

## 1.1 Önce: bu üçü aynı türden şey değil

Karşılaştırmaya başlamadan önce kurulması gereken ayrım bu:

| | Ne? | Sahibi |
|---|---|---|
| **Odoo Community** | Bir **ürün** — Odoo'nun açık kaynak çekirdeği | Odoo S.A. |
| **Odoo Enterprise** | Bir **ürün + abonelik** — Community üzerine eklenen kapalı kaynak modüller ve hizmetler | Odoo S.A. |
| **OCA** | Bir **dernek** — kâr amacı gütmeyen, İsviçre merkezli topluluk kuruluşu | Üyeleri |

Odoo'nun kendi ifadesiyle: *"Odoo Community is the core upon which Odoo Enterprise is
built."* Enterprise ayrı bir yazılım değil, Community'nin üzerine kurulan bir katman.

OCA ise Odoo S.A.'nın bir parçası **değil**, bağımsız bir dernek. Kendi tanımladığı
misyonun bir maddesi şu: *"Ensure that Odoo remains a viable open source ERP regardless of
what Odoo SA decides to do in the future."* Yani varlık sebebinin bir kısmı, Odoo S.A.'nın
gelecekteki kararlarına karşı bir sigorta olmak.

Bu yüzden OCA'yı "Enterprise'ın ücretsiz alternatifi" diye tanımlamak yanlış olur. Kesişim
var, ama OCA'nın amacı Enterprise'ı taklit etmek değil; topluluk modüllerine ortak bir
kalite, inceleme ve bakım standardı getirmek. Bunu somut olarak Bölüm 2'de göstereceğim —
OCA'nın en güçlü olduğu yerlerden biri, Enterprise'ın hiç ilgilenmediği bir alan.

## 1.2 Lisans

Lisans metinlerini doğrudan kaynak depolarından okudum:

| | Lisans | Kaynak |
|---|---|---|
| **Odoo Community** | **LGPLv3** | `odoo/odoo` kök `LICENSE` dosyası |
| **OCA modülleri** | **AGPLv3** | OCA depolarının kök `LICENSE` dosyası + modül manifest'leri |
| **Odoo Enterprise** | Tescilli (proprietary), kullanıcı başı abonelik | Odoo Enterprise lisans sözleşmesi |

Odoo'nun `LICENSE` dosyasındaki ifade:
> *"Odoo is published under the GNU LESSER GENERAL PUBLIC LICENSE, Version 3 (LGPLv3)"*

OCA tarafında hem deponun kök `LICENSE` dosyası AGPLv3, hem de tek tek modül
manifest'lerinde `# License AGPL-3.0 or later` satırı var. Bunu dört ayrı OCA deposunda
(`account-financial-reporting`, `manufacture`, `queue`, `delivery-carrier`) kontrol ettim,
hepsi aynı.

### Bu farkın pratik anlamı

LGPL ile AGPL arasındaki fark bu karşılaştırmanın en az konuşulan ama en somut sonuçlu
kısmı:

- **LGPL (Community):** Community'nin üzerine kendi modülünü yazabilir, o modülü **kapalı
  kaynak tutarak** dağıtabilirsin. Ticari Odoo eklentisi satan firmaların iş modelinin
  dayanağı bu.
- **AGPL (OCA):** GPL'in "yazılımı ağ üzerinden hizmet olarak sunmak da dağıtım sayılır"
  diyen versiyonu. Bir AGPL modülünü değiştirip kullanıcılara web üzerinden sunuyorsan,
  değiştirdiğin kaynağı paylaşman beklenir.

Yani **"Community kullanmak" ile "OCA modülü kullanmak" aynı serbestlikte değil.** Bir OCA
modülünü alıp değiştirerek kendi kapalı ürününe katmayı planlayan bir şirket için bu
operasyonel bir karardır, teknik bir detay değil.

> Bunun kesin hukuki yorumunu yapacak konumda değilim; bir şirket için bu değerlendirme
> hukuk danışmanıyla yapılmalı. Burada amacım ayrımın var olduğunu ve sonuç doğurduğunu
> göstermek. OCA ile Odoo S.A. bu iki lisansın bir arada yaşayabilmesi konusunda ayrıca
> anlaşmaya varmışlar (LGPLv3 ve AGPLv3 uyumlu lisanslar).

## 1.3 Fonksiyonel kapsam

Bu bölümü Odoo'nun karşılaştırma sayfasına dayandırmak istemedim, çünkü o sayfa bir
pazarlama materyali ve okuduğumda bazı modülleri Community'de gösteriyordu. Bunun yerine
**kendi Community kurulumumu kontrol ettim.** 692 çekirdek modülün içinde şunlar **yok**:

| Alan | Community'de bulunmayan modül |
|---|---|
| Muhasebe | `account_accountant` (tam muhasebe uygulaması) |
| İK | `hr_payroll` (bordro), `timesheet_grid` |
| Üretim | `mrp_workorder` (iş emri/shopfloor), `mrp_plm` (ürün yaşam döngüsü), `quality_control` (kalite kontrol) |
| Hizmet | `helpdesk` (destek masası), `industry_fsm` (saha servisi), `appointment`, `project_forecast` |
| Doküman | `documents`, `sign` (e-imza) |
| Pazarlama | `marketing_automation`, `social` |
| İletişim | `voip`, `whatsapp` |
| Platform | `web_studio` (kodsuz özelleştirme) |

Community'de bunların yerine çekirdek karşılıkları var: `account` (faturalama ve temel
muhasebe), `mrp` (temel üretim), `hr` (personel kayıtları), `project`, `stock`, `sale`,
`purchase`, `website`, `pos` vb.

**Bu listeden çıkan iki gözlem:**

1. **Üretim yapan bir şirket için `mrp_workorder` ve `quality_control`'ün yokluğu ciddi.**
   Community'deki `mrp` üretim emri oluşturur, ama iş istasyonu bazlı iş emri takibi ve
   kalite kontrol akışı yok. (OCA bu ikinciyi dolduruyor — Bölüm 2.)
2. **Satış sonrası servis senaryosu için `helpdesk` ve `industry_fsm` yok.** Bu, bu
   case'in ikinci bölümündeki AI voice agent senaryosunu doğrudan etkiliyor: agent'ın
   "servis kaydı açması" isteniyor, ama Community'de hazır bir destek masası modülü yok.
   Yani o kayıt ya `project`/`crm` üzerine kurulacak, ya OCA'dan bir modülle, ya da custom
   yazılacak. Bu bir mimari karar ve Community seçildiği anda ortaya çıkıyor.

### Odoo 19'un vitrin özelliği Community'de yok

Odoo 19'un sürüm notlarında AI özellikleri uzun bir liste: doğal dilde veritabanı
sorgulama, "Ask AI search", AI ile alan doldurma, e-posta şablonu promptları, ses
transkripti ve özetleme, prompt'tan web sayfası üretme.

**Community kurulumunda bunların hiçbiri yok.** Üç ayrı yerden doğruladım:
- 692 çekirdek modül içinde `ai`, `llm`, `gpt`, `gemini` desenine uyan tek modül yok
- `odoo/odoo` deposunun 19.0 dalındaki modül listesinde de yok
- AI özellikleri için gereken `pgvector` eklentisi `odoo/odoo` deposunda **0 kez** geçiyor;
  Odoo veritabanı oluştururken yalnızca `pg_trgm` ve `unaccent` kuruyor
  (`odoo/service/db.py:154`)

**Yorumum:** Odoo 19'un pazarlama anlatısının merkezinde AI var, ama Community kullanıcısı
bu anlatının hiçbir parçasını almıyor. Community tarafında 19'a geçmenin gerekçesi başka
yerlerde aranmalı (Bölüm 4). Bu ayrımı görebilmek için sürüm notlarını okumak yetmiyor,
kurulumun içine bakmak gerekiyor.

## 1.4 Bakım ve sürüm politikası

Odoo'nun resmi dokümanından:

> *"Odoo provides **standard support** for all major versions for **three years**. It
> includes helpdesk support, bug fixing, and security updates."*
>
> *"Beyond those three years, **extended support** is subject to a **mandatory additional
> fee** and includes helpdesk support and bug fixes (depending on feasibility)."*

| Sürüm | Çıkış | Standart desteğin sonu |
|---|---|---|
| **19.0** | Eylül 2025 | Eylül 2028 (planlanan) |
| 18.0 | Ekim 2024 | Eylül 2027 |
| 17.0 | Kasım 2023 | Eylül 2026 |
| 16.0 | Ekim 2022 | Eylül 2025 (bitti) |

Yani Odoo'da bir major sürümün ömrü **3 yıl**, ve her yıl yeni bir major sürüm çıkıyor.
Pratikte bu, aynı anda üç sürümün desteklendiği ve dördüncü yılda ya yükselttiğiniz ya da
ek ücret ödediğiniz anlamına geliyor.

**Kritik nokta:** Bu destek Odoo S.A.'nın verdiği destektir ve **helpdesk support kısmı
Enterprise aboneliğine bağlıdır.** Community kullanıcısı güvenlik yamalarını açık depodan
alır, ama arayabileceği bir destek hattı yoktur. Sürüm yükseltmesi de Community'de tamamen
kullanıcının problemidir; Enterprise'da Odoo'nun kendi veritabanı yükseltme servisi
sürecin parçasıdır.

**OCA tarafında bakım nasıl işliyor?** Merkezi bir destek taahhüdü yok. Her depo gönüllü
bakımcılara bağlı, modüller yapılandırılmış bir kod inceleme sürecinden geçiyor ve her
major sürüm için ayrı bir dal (`17.0`, `18.0`, `19.0`) tutuluyor. Ama bir modülün yeni
sürüme taşınacağının garantisi yok — bu tamamen o modüle ihtiyaç duyan birinin ortaya
çıkmasına bağlı.

**Bunu ölçülebilir hale getirdim** ve Bölüm 2'de bir tablo olarak sunuyorum. OCA ile
ilerlemenin gizli maliyeti tartışmasının çekirdeği burası: bakım vaadi değil, bakım
istatistiği.

---

# 2. OCA hangi boşlukları dolduruyor?

OCA'nın 266 public deposu var. Case'deki şirketin profiline (üretim + fulfillment +
Türkiye) uyan beş alanı seçtim. Aşağıdaki modül adlarının tamamının **19.0 dalında
gerçekten var olduğunu** tek tek kontrol ettim.

## 2.1 Kalite kontrol — Enterprise'daki boşluğun en net örneği

`quality_control` modülü **Community'de yok** (Bölüm 1.3). OCA bunu dolduruyor:

**`OCA/manufacture` → `quality_control_oca`**

Bu, "OCA Enterprise'daki bir boşluğu doldurur" ifadesinin en temiz örneği: Enterprise'da
para ödeyerek aldığınız bir uygulamanın açık kaynak karşılığı.

## 2.2 Üretim planlama — `OCA/manufacture` (19 modül)

| Modül | Ne yapıyor |
|---|---|
| **`mrp_multi_level`** | Çok seviyeli malzeme ihtiyaç planlaması (MRP II) |
| `mrp_multi_level_estimate` | Talep tahminine dayalı planlama |
| `mrp_bom_tracking` | Ürün ağacı değişiklik takibi |
| `mrp_repair_order` | Tamir/servis emri |
| `mrp_subcontracting_purchase_link` | Fason üretim–satınalma bağı |
| `mrp_warehouse_calendar` | Depo takvimi ile üretim planlama |

`mrp_multi_level` özellikle önemli: Odoo'nun çekirdek MRP'si tek seviyeli çalışır. Çok
kademeli ürün ağacı olan bir beyaz eşya üreticisi için bu doğrudan bir boşluktur ve
Enterprise'ın MPS'i de aynı problemi çözmez.

## 2.3 Stok ve envanter — `OCA/stock-logistics-warehouse` (15 modül)

| Modül | Ne yapıyor |
|---|---|
| **`stock_inventory`** | Envanter sayım belgesi akışı |
| `stock_inventory_discrepancy` | Sayım farkı eşikleri ve onay mekanizması |
| `stock_inventory_lockdown` | Sayım sırasında stok hareketlerini kilitleme |
| `stock_demand_estimate` | Talep tahmini |
| `stock_secondary_unit` | İkincil ölçü birimi (koli/adet) |

**Bu alan OCA'nın rolünü anlatmak için en iyi örnek.** Odoo, 15.0 sürümünde envanter sayım
belgesi akışını kaldırdı ve doğrudan miktar düzeltmesine geçti. Denetim izi isteyen bir
şirket için bu bir kayıptı. `stock_inventory` o akışı geri getiriyor.

Yani OCA burada **Enterprise'da para ile satılan bir şeyi bedavaya yapmıyor** — Odoo'nun
bilinçli olarak çıkardığı bir özelliği geri koyuyor. OCA'nın "Odoo S.A.'nın kararlarına
karşı sigorta" misyonunun somutlaşmış hâli bu.

## 2.4 Kargo ve fulfillment — `OCA/delivery-carrier` (24 modül)

Case'deki fulfillment iştiraki için doğrudan ilgili:

| Modül | Ne yapıyor |
|---|---|
| `partner_delivery_zone` / `_calendar` | Teslimat bölgesi ve bölge takvimi |
| `delivery_multi_destination` | Tek siparişte birden fazla teslimat adresi |
| `delivery_carrier_agency` | Kargo acentesi yönetimi |
| `delivery_driver` / `_stock_picking_batch` | Sürücü ataması ve toplu sevkiyat |
| `delivery_package_number` | Koli numaralandırma |

**Ama önemli bir sınır var:** Bu modüller altyapı sağlıyor. Türk kargo firmalarının
(Yurtiçi, Aras, MNG, Sürat) API entegrasyonları OCA'da **yok**. Bu doğrudan Bölüm 3'teki
"custom yazılacaklar" listesine gidiyor.

## 2.5 Raporlama — `OCA/reporting-engine` (21 modül)

| Modül | Ne yapıyor |
|---|---|
| **`report_xlsx`** + `report_xlsx_helper` | Excel çıktısı |
| `bi_sql_editor` | Arayüzden SQL yazarak rapor üretme |
| `kpi` | KPI tanımlama ve takip |
| `report_async` | Büyük raporları arka planda üretme |

`report_xlsx` dikkat çekici: Excel'e rapor çıkarmak neredeyse her şirketin günlük
ihtiyacı, ama Odoo çekirdeğinde yok. Enterprise'daki Studio ve gelişmiş pivot görünümleri
bu ihtiyacın bir kısmını karşılıyor; `reporting-engine` farklı bir açıdan yaklaşıyor.

## 2.6 Teknik altyapı — `OCA/queue` (8 modül)

| Modül | Ne yapıyor |
|---|---|
| **`queue_job`** | Asenkron iş kuyruğu |

`queue_job`'ın özel bir yeri var: bunun karşılığı **ne Community'de ne Enterprise'da**
var. Yoğun entegrasyon yapan bir sistemde (pazaryeri senkronizasyonu, kargo etiketi
üretimi, e-fatura gönderimi) bu tür işleri istek-yanıt döngüsünün dışına çıkarmak
zorunludur. Fulfillment iştiraki için altyapısal bir gereklilik.

Yani OCA sadece "Enterprise'ın ücretsiz karşılığı" değil; Odoo'nun hiç ele almadığı
mühendislik problemlerini de çözüyor.

## 2.7 Zayıf bulduğum alan: `OCA/l10n-turkey`

OCA depolarının 19.0 dalındaki gerçek durumunu ölçtüm (11 Eylül 2026):

| OCA deposu | 19.0'daki modül sayısı | Son push | Açık issue |
|---|---|---|---|
| `delivery-carrier` | 24 | 2026-09-10 | 73 |
| `manufacture` | 19 | 2026-09-09 | 96 |
| `reporting-engine` | 21 | 2026-09-04 | 41 |
| `stock-logistics-warehouse` | 15 | 2026-09-09 | 82 |
| `queue` | 8 | 2026-09-06 | 49 |
| `account-financial-reporting` | 4 | 2026-09-09 | 46 |
| **`l10n-turkey`** | **0** | **2025-09-30** | 4 |

**`OCA/l10n-turkey` deposu 14.0'dan 19.0'a kadar her dalda boş.** İçinde yalnızca depo
iskeleti var (`.github/`, `LICENSE`, `README.md`, `setup/`), tek bir modül yok. Son commit
bir yıl öncesine ait, diğer depolar ise günlük olarak güncelleniyor.

Türkiye'de faaliyet gösteren bir şirket açısından bakınca bu ilk bakışta ciddi bir boşluk
gibi duruyor. Ama devamına bakınca tablo değişiyor.

### Ama bu bir boşluk yaratmıyor — ve asıl ilginç olan bu

Odoo Community **çekirdeğinde** Türkiye lokalizasyonu var ve her sürümde büyüyor:

| Odoo | Modüller |
|---|---|
| 17.0 | `l10n_tr`, `l10n_tr_nilvera`, `l10n_tr_nilvera_edispatch`, `l10n_tr_nilvera_einvoice` |
| 18.0 | + `l10n_tr_nilvera_einvoice_extended` |
| 19.0 | + `l10n_tr_nilvera_base_vat` |

`l10n_tr` modülü hesap planı, vergiler ve KDV beyanını kuruyor; yazarı *"Odoo S.A.,
Drysharks Consulting and Trading Ltd."*. Nilvera ise bir Türk e-belge entegratörü — yani
**e-Fatura ve e-İrsaliye Community'de mevcut.**

**Değerlendirmem:** OCA'nın Türkiye lokalizasyonu boş, ama bu bir boşluk değil bir
gereksizlik göstergesi. Odoo bu işi çekirdeğe almış ve OCA'nın oraya girmesine gerek
kalmamış.

**Asıl sorulması gereken soru şu:** Odoo'nun çözümü tek bir entegratöre (Nilvera) bağlı.
Şirketin mevcut e-belge entegratörü farklıysa, ne Odoo çekirdeğinden ne OCA'dan yardım
gelir — o entegrasyon custom yazılacaktır. Bu, Bölüm 3'ün doğrudan girdisi.

---

# 3. Bu şirket için karar

**Şirket profili:** Türkiye'de üretim yapan, kendi markalarıyla beyaz eşya/ısıtma ürünleri
satan bir grup şirketi; ayrıca e-ticaret siparişlerini toplayan, paketleyen ve kargolayan
bir fulfillment iştiraki var.

## 3.1 Profilden çıkan ihtiyaçlar

| Profil özelliği | Doğurduğu ihtiyaç |
|---|---|
| Üretim (beyaz eşya, çok parçalı) | Çok seviyeli ürün ağacı, iş emri takibi, kalite kontrol |
| Kendi markaları, dayanıklı tüketim | Seri numarası takibi, garanti yönetimi, satış sonrası servis |
| Fulfillment iştiraki | WMS, kargo entegrasyonu, pazaryeri entegrasyonu, yüksek işlem hacmi |
| Grup şirketi | Çoklu şirket, şirketler arası işlemler, konsolidasyon |
| Türkiye | e-Fatura, e-Arşiv, e-İrsaliye, KDV beyanı, bordro |

## 3.2 Kararım: Enterprise

**Bu şirket için Enterprise seçerdim.** Üç gerekçeyle:

**1. Üretim tarafındaki boşluk kapatılamıyor.** Community'de `mrp_workorder` yok. Beyaz
eşya üretimi iş istasyonu bazlı bir süreç: montaj hattı, iş emri, operatör takibi. OCA
bunu doldurmuyor — `OCA/manufacture` planlama ve ürün ağacı tarafını güçlendiriyor, ama
shopfloor katmanının karşılığı yok. Kalite kontrolü OCA (`quality_control_oca`) ile
çözebilirsiniz; iş emri yönetimini çözemezsiniz.

**2. Grup şirketi muhasebesi.** `account_accountant` Community'de yok. Grup yapısında
çoklu şirket muhasebesi, şirketler arası işlemler ve konsolidasyon var. Bunu Community
üzerinde OCA modülleriyle kurmak mümkün ama ciddi bir entegrasyon ve bakım yükü demek.

**3. Sürüm yükseltme riski.** Odoo yılda bir major sürüm çıkarıyor ve destek 3 yıl. Bu
büyüklükte bir kurulumda yükseltme sürekli bir sorumluluk. Enterprise aboneliği Odoo'nun
veritabanı yükseltme servisini içeriyor; Community'de bu tamamen sizin probleminiz.

## 3.3 Fikrimi değiştirecek koşul

**Şirketin iç Odoo geliştirme ekibi varsa Community + OCA'yı seçerdim.**

Sebep şu: Enterprise'ın maliyeti kullanıcı başına ve doğrusal büyüyor. Üretim + depo +
fulfillment demek çok sayıda operasyonel kullanıcı demek — üretim operatörleri, depo
personeli, paketleme ekibi. Bu profilde kullanıcı sayısı hızla üç haneye çıkar ve lisans
kalemi ciddi bir rakama ulaşır.

Enterprise'ın asıl sattığı şey yazılım değil, **risk transferi**: yükseltme, destek,
sorumluluk. İç ekibi olan bir şirket o riski zaten taşıyabiliyorsa, transfer için ödeme
yapmasının anlamı azalır. İç ekip yoksa, Community'nin görünmeyen maliyeti Enterprise
lisansından yüksek olur.

## 3.4 Gizli maliyetler — iki yolun dürüst karşılaştırması

**Community + OCA yolunun gizli maliyetleri:**

| Maliyet | Somut karşılığı |
|---|---|
| Sürüm yükseltmede modül uyumu | Kullandığınız her OCA modülünün yeni sürüm dalının hazır olmasını beklersiniz. Hazır değilse ya siz taşırsınız ya beklersiniz. |
| Bakımın gönüllülüğe bağlı olması | `l10n-turkey` örneği: depo var, içi boş, bir yıldır dokunulmamış. Bu bir istisna değil, modelin doğal sonucu. |
| Modüller arası çakışma | Aynı modeli genişleten iki OCA modülü çakışabilir; entegrasyon testi sizin sorumluluğunuzda. |
| Açık issue yükünü okuma | `OCA/server-tools`: 286 açık issue. Kuracağınız modülün issue'larını okumak sizin işiniz. |
| İç uzmanlık zorunluluğu | Sorun çıktığında arayacak kimse yok. Python ve Odoo ORM'i okuyabilen biri şart. |

**Enterprise yolunun maliyetleri:**

| Maliyet | Somut karşılığı |
|---|---|
| Kullanıcı başı abonelik | Operasyonel kullanıcı sayısıyla doğrusal büyür |
| 3 yıl sonra uzatılmış destek | Dokümanın ifadesi: *"mandatory additional fee"* |
| Custom ihtiyacı yine devam eder | Enterprise şirkete özgü iş kurallarını çözmez |
| Kapalı kaynak bağımlılığı | Enterprise modülünde bir hata varsa kendiniz düzeltemezsiniz |
| OCA'ya yine ihtiyaç duyulması | `queue_job`, `report_xlsx`, `stock_inventory` Enterprise'da da yok |

Son satır önemli: **Enterprise seçmek OCA'dan vazgeçmek anlamına gelmiyor.** İkisi bir
arada kullanılabilir ve bu şirkette kullanılması gerekir.

## 3.5 Ekleyeceğim OCA modülleri

Enterprise seçmiş olsam bile:

| Modül | Neden |
|---|---|
| `queue_job` | Pazaryeri/kargo/e-fatura entegrasyonlarını asenkron çalıştırmak için. Karşılığı hiçbir edition'da yok. |
| `mrp_multi_level` | Çok seviyeli ürün ağacı planlaması |
| `stock_inventory` + `stock_inventory_discrepancy` | Denetim izi bırakan envanter sayımı |
| `report_xlsx` | Excel raporlama — günlük operasyonel ihtiyaç |
| `partner_delivery_zone`, `delivery_multi_destination` | Fulfillment teslimat bölgesi yönetimi |
| `stock_secondary_unit` | Koli/adet ikili birim takibi |

## 3.6 Custom yazmak zorunda kalacaklarım

Ne Enterprise'da ne OCA'da olan, şirkete özgü işler:

1. **Türk kargo firması entegrasyonları** (Yurtiçi, Aras, MNG, Sürat). `OCA/delivery-carrier`
   altyapıyı veriyor, API entegrasyonlarını vermiyor.
2. **Türk pazaryeri entegrasyonları** (Trendyol, Hepsiburada, N11). Odoo çekirdeğinde
   Amazon ve eBay var, Türk pazaryerleri yok.
3. **Farklı e-belge entegratörü**, eğer şirket Nilvera kullanmıyorsa.
4. **Garanti takibi** — ürün bazlı garanti süresi, seri numarasına bağlı garanti kaydı,
   garanti bitiş tarihi takibi. Bu case'in 1b bölümünde yazacağım addon tam olarak bu
   kategorinin örneği: standart bir ERP fonksiyonu değil, dayanıklı tüketim malı üreten bir
   şirkete özgü bir iş kuralı.
5. **Satış sonrası servis kaydı akışı.** Community'de `helpdesk` ve `industry_fsm` yok;
   Enterprise seçilirse bu kısım hazır gelir. Bu, Enterprise kararını destekleyen ek bir
   argüman — ve bu case'in ikinci bölümündeki voice agent senaryosunun bağlanacağı yer.

---

# 4. Odoo 19'da fark yaratan değişiklikler

Sürüm notu özetlemek yerine **ölçtüm**: Odoo 18.0 ile 19.0'ın modül listelerini
karşılaştırdım.

```
18.0 → 627 modül
19.0 → 638 modül

57 modül eklendi, 46 modül kaldırıldı
```

Bana eklenenlerden çok **kaldırılanlar** bir şey anlattı. Çünkü Odoo bir modülü
kaldırdığında bu genelde bir mimari kararın sonucudur ve o modüle bağımlı olan herkesi
etkiler.

## 4.1 `web_editor` kaldırıldı, yerine `html_builder` geldi

```
Kaldırılan: web_editor
Eklenen:    html_builder
```

Kendi kurulumumda doğruladım: `web_editor` klasörü **yok**, `html_builder` **var**.
`html_builder`'ın manifest'indeki tanım: *"generic html builder … designed to be used by
the website builder and mass mailing editor."*

**Yorumum:** Odoo, web sitesi düzenleyicisi ve e-posta editörünün tamamını yeni bir temel
üzerine taşımış. Sürüm notlarında bu "geliştirilmiş zengin metin editörü" diye tek satır
geçiyor. Ama pratikte anlamı şu: **`web_editor`'a bağımlı her custom modül 19'da kırılır.**

Bu benim için 19'un en önemli değişikliği, çünkü bir sürüm yükseltmesinin maliyetini
belirleyen şey yeni özellikler değil, bu tür sessiz mimari değişiklikler. Web sitesi
tarafında özelleştirme yapmış bir şirket için 18 → 19 geçişi bir yükseltme değil, bir
yeniden yazma projesidir.

## 4.2 `hr_contract` kaldırıldı, sözleşme "versiyon" oldu

```
Kaldırılan: hr_contract, hr_holidays_contract, hr_work_entry_contract, test_hr_contract_calendar
```

`hr` modülünün model dosyalarına baktım: `hr_contract.py` yok, yerine **`hr_version.py`**
var.

**Yorumum:** Çalışan sözleşmesi ayrı bir varlık olmaktan çıkıp, çalışan kaydının zaman
içindeki bir **versiyonu** hâline gelmiş. Bu bir özellik eklemesi değil, bir **veri modeli
değişikliği**. Bordro Enterprise'da olduğu için Community kullanıcısını doğrudan az
etkiliyor; ama İK verisiyle entegrasyonu olan (bordro yazılımına veri gönderen, personel
maliyetini üretime yansıtan) her sistem etkilenir.

Bu ikisini bir arada koyunca çıkan sonuç: **Odoo 19, kullanıcıya "yeni özellikler" olarak
sunulan ama geliştiriciye "veri modeli değişikliği" olarak yansıyan bir sürüm.** Yükseltme
planlaması yaparken bakılacak yer sürüm notları değil, modül listesi farkı.

## 4.3 `rpc` ve `api_doc` modülleri eklendi

```
Eklenen: rpc      → "Standard Odoo RPC endpoints to models"
Eklenen: api_doc  → "Odoo Dynamic API Documentation"
```

**Yorumum:** Odoo'nun dış sistemlere açılan yüzeyi (XML-RPC / JSON-RPC) bugüne kadar
vardı, ama çekirdeğe gömülüydü ve dokümantasyonu statikti. 19'da bu yüzey ayrı modüllere
çıkarılmış ve API dokümantasyonu dinamik hâle gelmiş.

**Bu şirket için neden önemli:** Fulfillment iştiraki doğası gereği entegrasyon yoğun
çalışıyor — pazaryerleri, kargo firmaları, muhtemelen bir WMS. Entegrasyon yüzeyinin ayrı
bir modül olarak tanımlanmış olması bu entegrasyonların bakımını ve sürüm geçişlerini
öngörülebilir kılar. Bir önceki iki maddede "kırılıyor" dediğim yerlerin aksine, burası
işi kolaylaştıran bir değişiklik.

## 4.4 Not: AI, Community'de yok

Bölüm 1.3'te ayrıntısını verdim. Kısaca: Odoo 19'un pazarlama anlatısının merkezindeki AI
özellikleri Community kurulumunda mevcut değil. Community tarafında değerlendirme yapan bir
şirket için 19'a geçme gerekçesi başka yerlerde aranmalı — ve yukarıdaki üç maddede
anlattığım gibi, o gerekçelerin bir kısmı aslında maliyet kalemi.

---

# 5. Kaynaklar

Case, Odoo'nun pazarlama sayfaları dışında en az 3 farklı **tür** kaynak bekliyordu.
Kullandığım kaynak türleri:

| # | Tür | Kaynak | Ne için kullandım |
|---|---|---|---|
| 1 | Resmi dokümantasyon | [Standard and extended support](https://www.odoo.com/documentation/19.0/administration/standard_extended_support.html) · [Source install](https://www.odoo.com/documentation/19.0/administration/on_premise/source.html) | Destek politikası, PostgreSQL gereksinimi |
| 2 | Kaynak kodu | [github.com/odoo/odoo](https://github.com/odoo/odoo) — `LICENSE`, `addons/`, `setup/win32/setup.nsi`, `odoo/service/db.py` | Lisans, modül envanteri, 18→19 farkı, PostgreSQL kararı |
| 3 | Çalışan kurulumun kendisi | Odoo 19.0 Community Docker container | Enterprise/Community modül farkı, AI modüllerinin yokluğu |
| 4 | Issue tracker / commit geçmişi | [odoo/odoo#234457](https://github.com/odoo/odoo/issues/234457) · commit `ad47259b` | PostgreSQL 18 uyumluluk riski |
| 5 | Topluluk deposu | [github.com/OCA](https://github.com/OCA) — 266 depo | OCA modül envanteri, bakım aktifliği ölçümü |
| 6 | Topluluk kuruluşu | [odoo-community.org](https://www.odoo-community.org/about) · [FAQ](https://www.odoo-community.org/resources/faq) · [lisans anlaşması](https://www.odoo-community.org/blog/news-updates-1/oca-odoo-meeting-on-licenses-21) | OCA'nın misyonu, yapısı, lisans politikası |
| 7 | Forum / topluluk tartışması | [OCA Discussions #181 — "Questioning the Preference: Community vs. Enterprise"](https://github.com/orgs/OCA/discussions/181) | OCA bakımcılarının Community/Enterprise tercihine dair argümanları |
| 8 | Video / konferans | [OCA YouTube kanalı](https://www.youtube.com/@OdooCommunity) — OCA Days 2025 (Liège) sunumları | OCA'nın çalışma biçimi ve öncelikleri |
| 9 | Upstream proje | [postgresql.org/support/versioning](https://www.postgresql.org/support/versioning/) | PostgreSQL sürüm yaşam döngüsü |
| 10 | Sürüm notları | [Odoo 19 release notes](https://www.odoo.com/odoo-19-release-notes) | Duyurulan özelliklerin envanteri (kritik okumayla) |
| 11 | Satıcı blogları | oec.sh, ksolves.com, anriztech.com | **Çürütülen iddia örneği olarak** — aşağıya bakınız |

## 5.1 Kullanmadığım kaynaklar ve nedeni

Bu bölümü bilerek ekliyorum, çünkü kaynak toplamak ile kaynak değerlendirmek farklı şeyler.

PostgreSQL sürüm seçimi için üç satıcı blogu inceledim ve **üçünü de gerekçe olarak
kullanmadım**:

- **ksolves.com** — PostgreSQL 16'yı önerdiği söyleniyordu; okuduğumda hiçbir sürüm
  önermiyor, "resmi dokümana ve sürüm notlarına bakın" diyor.
- **anriztech.com** — PostgreSQL ile ilgili tek somut cümlesi *"Odoo 19 is built to
  leverage PostgreSQL 16"* ve bunun için kaynak vermiyor.
- **oec.sh** — *"Version 16 recommended. Versions 12-17 all supported."* diyor. Bu ifade
  hiçbir birincil kaynak göstermiyor ve resmi Odoo 19 dokümanıyla çelişiyor (doküman
  "13.0 or above" diyor, üst sınır veya öneri belirtmiyor).

Kararı bunların yerine Odoo'nun **kendi Windows kurulum betiğine** dayandırdım: `odoo/odoo`
deposunun 19.0 dalındaki `setup/win32/setup.nsi` dosyası `postgresql-16.14-1-windows-x64.exe`
indiriyor. İlgili PR'ın gerekçesi de Odoo'nun kendi ifadesiyle: *"version 12 is no longer
supported, so it's time to bump to version 16."*

Ayrıntısı `KARAR_GUNLUGU.md` → K-002'de.

---

# Ek A — Bulguların tekrar üretilmesi

Bu araştırmadaki ölçümler tekrarlanabilir. Kullandığım komutlar:

```bash
# Community'de hangi modüller yok? (Enterprise/Community farkı)
docker exec <odoo-container> ls /usr/lib/python3/dist-packages/odoo/addons/ | grep -x hr_payroll

# AI modüllerinin yokluğu
docker exec <odoo-container> ls /usr/lib/python3/dist-packages/odoo/addons/ \
  | grep -iE "^ai|ai_|llm|gpt|gemini"

# Odoo 18 → 19 modül farkı
diff <(gh api "repos/odoo/odoo/contents/addons?ref=18.0" --jq '.[].name' | sort) \
     <(gh api "repos/odoo/odoo/contents/addons?ref=19.0" --jq '.[].name' | sort)

# Bir OCA deposunun gerçek durumu
gh api "repos/OCA/l10n-turkey/contents?ref=19.0" --jq '.[] | select(.type=="dir") | .name'
gh api "repos/OCA/l10n-turkey" --jq '"son push: \(.pushed_at)"'

# Lisans doğrulaması
gh api repos/odoo/odoo/contents/LICENSE --jq '.content' | base64 -d | head -8
gh api "repos/OCA/manufacture/contents/LICENSE?ref=19.0" --jq '.content' | base64 -d | head -3
```
