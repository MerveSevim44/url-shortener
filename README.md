# 🔗 URL Shortener

Uzun URL'leri kısa ve paylaşılabilir bağlantılara dönüştüren, Docker ile çalışan bir web servisidir.

## 🏗️ Mimari

```
┌─────────────┐     POST /shorten      ┌───────────────────────────────────┐
│   İstemci   │ ─────────────────────► │         Flask API (web)            │
│  (curl/tarayıcı) ◄───────────────── │           port: 5000               │
└─────────────┘    short_url döner     └──────────┬──────────┬─────────────┘
                                                   │          │
                                          yaz/oku  │          │  cache'de yoksa
                                                   ▼          ▼
                                          ┌────────────┐  ┌──────────────┐
                                          │   Redis    │  │  PostgreSQL  │
                                          │  (cache)   │  │    (kalıcı)  │
                                          └────────────┘  └──────────────┘
```

### Servisler

| Servis | Teknoloji | Görev |
|--------|-----------|-------|
| `web`  | Python / Flask | REST API, iş mantığı |
| `db`   | PostgreSQL 16 | URL'lerin kalıcı olarak saklanması |
| `redis`| Redis 7 | Hızlı okuma için önbellekleme |

## ⚙️ Nasıl Çalışır?

### URL Kısaltma (`POST /shorten`)

1. Uzun URL, PostgreSQL'e kaydedilir ve bir `id` alır.
2. `id`, **Base62** algoritmasıyla kısa bir koda dönüştürülür (`0-9`, `a-z`, `A-Z`).
3. `short_code → long_url` eşleşmesi Redis'e cache'lenir.
4. Kısa URL kullanıcıya döndürülür.

### Yönlendirme (`GET /<short_code>`)

1. `short_code` önce **Redis**'te aranır → **Cache HIT** → anında `302 redirect`.
2. Redis'te yoksa **Cache MISS** → `short_code` Base62 decode edilir → PostgreSQL'den `long_url` çekilir → Redis'e cache'lenir → `302 redirect`.

### Rate Limiting

Her IP adresi için 60 saniyelik pencerede maksimum **10 istek** sınırı vardır. Redis pipeline kullanılarak atomik sayaç tutulur.

## 🚀 Kurulum ve Çalıştırma

### Gereksinimler

- [Docker](https://www.docker.com/get-started) ve Docker Compose

### Başlatma

```bash
# Tüm servisleri build edip arka planda başlat
docker compose up --build -d
```

### Durdurma

```bash
docker compose down
```

### Verileri de silerek durdurma

```bash
docker compose down -v
```

## 📡 API Kullanımı

### Sağlık Kontrolü

```bash
curl http://localhost:5000/health
```

```json
{"status": "ok"}
```

### URL Kısaltma

```bash
curl -X POST http://localhost:5000/shorten \
  -H "Content-Type: application/json" \
  -d '{"url": "https://www.example.com/cok/uzun/bir/url"}'
```

```json
{
  "short_url": "http://localhost:5000/1a",
  "short_code": "1a"
}
```

### Kısa URL ile Yönlendirme

```bash
curl -L http://localhost:5000/1a
# → https://www.example.com/cok/uzun/bir/url adresine yönlendirir
```

## 🛠️ Teknoloji Yığını

| Teknoloji | Versiyon | Kullanım Amacı |
|-----------|----------|----------------|
| Python | 3.11 | Ana programlama dili |
| Flask | 3.0.0 | Web framework |
| Gunicorn | 21.2.0 | Production WSGI sunucusu |
| psycopg2 | 2.9.9 | PostgreSQL sürücüsü |
| redis-py | 5.0.1 | Redis istemcisi |
| PostgreSQL | 16-alpine | Kalıcı veri depolama |
| Redis | 7-alpine | Önbellek & rate limiting |

## 🔧 Ortam Değişkenleri

| Değişken | Varsayılan | Açıklama |
|----------|-----------|----------|
| `REDIS_HOST` | `redis` | Redis sunucusu adresi |
| `REDIS_PORT` | `6379` | Redis port numarası |
| `POSTGRES_HOST` | `db` | PostgreSQL sunucusu adresi |
| `POSTGRES_DB` | `urlshortener` | Veritabanı adı |
| `POSTGRES_USER` | `postgres` | Veritabanı kullanıcısı |
| `POSTGRES_PASSWORD` | `postgres` | Veritabanı şifresi |

## 📁 Proje Yapısı

```
url-shortener/
├── docker-compose.yml      # Tüm servislerin orchestration tanımı
└── app/
    ├── Dockerfile          # Flask uygulaması için Docker imajı
    ├── app.py              # Ana uygulama (API endpoint'leri, iş mantığı)
    ├── init.sql            # PostgreSQL tablo şeması (ilk kurulumda çalışır)
    └── requirements.txt    # Python bağımlılıkları
```
