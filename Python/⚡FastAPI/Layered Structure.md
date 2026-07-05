
> Explicit is better than implicit.

Sorumlulukların ayrılması prensibine göre, proje büyüdükçe kontrolün azalmasını önlemek için katmanlı (layered) mimari kullanılır.

# FastAPI + SQLAlchemy Proje Yapısı

```text
READING_LOGS/                  # Projenin kök dizini
│
├── app/                       # Çekirdek uygulama
│   ├── __init__.py            # 'app' klasörünü bir paket yapar
│   ├── main.py                # HTTP isteklerini alır, router'ları ve bağımlılık enjeksiyonunu başlatır
│   ├── database.py            # Veritabanı engine ve session bağımlılıkları
│   ├── models.py              # SQLAlchemy modelleri (DB tabloları)
│   ├── schemas.py             # Pydantic şemaları (DTO / API input-output)
│   └── crud.py                # Veritabanı işlemleri / repository fonksiyonları
│
├── alembic/                   # Database migration dosyaları
├── alembic.ini                # Alembic yapılandırması
│
├── venv/                      # Sanal ortam (gitignore'da tutulur)
│
├── .env                       # Ortam değişkenleri (gizli bilgileri burada saklama, .gitignore)
└── .gitignore
```

Kısa açıklamalar:

- `models.py`: Veri katmanı — veritabanı tabloları tanımlanır.
- `schemas.py`: API ve kullanıcı arayüzü için veri aktarımlarını tanımlar (Pydantic).
- `crud.py`: Veri erişim katmanı (repository) — SQL/ORM mantığını izole eder.
- `main.py`: Router/Controller'lar; HTTP isteklerini uygun servislere yönlendirir.

Not: Veritabanı session'ını bağımlılık (`dependency`) olarak tanımlayıp request-scoped kullanmak, transaction yönetimini ve test edilebilirliği artırır.
