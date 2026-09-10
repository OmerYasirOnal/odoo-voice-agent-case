# Odoo ERP + AI Voice Agent — Case Çalışması

Bu repo iki case çalışmasını içerir:

- **Case 1 — Odoo:** Araştırma + Docker ile ayağa kaldırılmış Odoo 19 Community ve
  bir custom addon (ürün garanti takibi).
- **Case 2 — AI Voice Agent:** Satış sonrası hizmet hattı için mimari doküman ve
  çalışan bir dilim.

Alınan tüm teknik kararların gerekçesi, değerlendirilen alternatifler ve bilerek kapsam
dışı bırakılanlar → **[KARAR_GUNLUGU.md](KARAR_GUNLUGU.md)**

---

## Durum

| Bölüm | Durum |
|---|---|
| Case 1b — Docker ortamı | ✅ Çalışıyor |
| Case 1a — Araştırma | 🚧 Devam ediyor |
| Case 1b — Custom addon | ⬜ Başlanmadı |
| Case 2a — Mimari | ⬜ Başlanmadı |
| Case 2b — Uygulama | ⬜ Başlanmadı |

---

## Ortamı ayağa kaldırma

**Gereken:** Docker ve Docker Compose. (Apple Silicon dahil; `odoo:19.0` image'ı
`arm64` mimarisini native destekliyor, emülasyon gerekmiyor.)

```bash
git clone <repo-url>
cd odoo-voice-agent-case
docker compose up -d
```

Ardından tarayıcıdan **http://localhost:8069** adresine gidin.

### İlk kurulum

İlk açılışta Odoo veritabanı oluşturma ekranı gelir:

1. **Master Password:** `admin`
2. **Database Name:** `dev` (serbest)
3. **Email / Password:** giriş bilgileriniz
4. **Demo data:** işaretleyin — denemek için hazır müşteri, ürün ve sipariş kayıtları gelir.

Veritabanı oluştuktan sonra **Apps** menüsünden **Sales** ve **Inventory**
uygulamalarını kurun.

### Faydalı komutlar

```bash
docker compose logs -f odoo      # Odoo loglarını izle
docker compose restart odoo      # Addon değişikliğinden sonra yeniden başlat
docker compose down              # Durdur (veri korunur)
docker compose down -v           # Durdur ve veritabanını tamamen sil
```

---

## Ortam hakkında

| Bileşen | Sürüm | Not |
|---|---|---|
| Odoo | 19.0 Community | `Odoo Server 19.0-20260908` |
| PostgreSQL | 16 | Sürüm seçimi gerekçesi: KARAR_GUNLUGU.md → K-002 |
| Custom addon yolu | `./addons` | Container içinde `/mnt/extra-addons` |

> ⚠️ `compose.yaml` içindeki `odoo/odoo` kimlik bilgileri **yalnızca yerel geliştirme
> içindir**. Bu dosya production'da olduğu gibi kullanılmamalıdır.
