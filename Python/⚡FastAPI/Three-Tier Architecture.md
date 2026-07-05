Presentation Layer --> Business Layer --> Data Access Layer

Kısa açıklama:

- **Presentation Layer (Sunum Katmanı):** Kullanıcı arayüzü veya API router'larının bulunduğu katman.
- **Business Layer (İş Mantığı / Service):** Uygulama kurallarının ve dönüşümlerin yapıldığı katman (ör. servisler, validator'lar).
- **Data Access Layer (Repository / DAL):** Veritabanı ile direkt konuşan katman (`crud.py`, SQLAlchemy oturumları).

Not: Bu ayrım kodun test edilebilirliğini, bakımını ve sorumlulukların netliğini artırır.