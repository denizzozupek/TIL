# Alembic Nedir?

>[!info] Alembic, Python ekosisteminde kullanılan popüler bir migration aracıdır. Genelde SQLAlchemy ORM ile birlikte kullanılır.

Özellikleri:

- Migration dosyalarını otomatik oluşturma (autogenerate) imkanı sunar.
- SQL yerine Python kodu yazarak şema değişikliklerini yönetmeyi sağlar.
- `upgrade` ve `downgrade` komutlarıyla ileri/geri hareket mümkündür.


## Kullanımı

1) Paket kurulumu:

	pip install alembic

2) Projeye Alembic ekleme:

	alembic init alembic

3) `env.py` içinde `target_metadata` ayarı:

Örnek (projenizdeki `Base` tanımına göre uyarlayın):

```python
from myapp.models import Base  # veya proje yolunuza göre

target_metadata = Base.metadata
```

4) `alembic.ini` içinde veritabanı URL'si:

`alembic.ini` dosyasında `sqlalchemy.url` anahtarını kendi veritabanı bağlantı URL'iniz ile değiştirin. Örnek:

```
sqlalchemy.url = postgresql+psycopg2://user:password@localhost:5432/mydatabase
```

5) **Migration oluşturma**:

Aşağıdaki komut, modellerinizdeki değişiklikleri otomatik algılayıp yeni bir revision dosyası oluşturur:

	alembic revision --autogenerate -m "create users table"

Ardından migration'ı uygulamak için:

	alembic upgrade head

6) **Geri alma (downgrade)**

Son yaptığınız migration'ı bir adım geri almak isterseniz:

	alembic downgrade -1

### `head`, `base` ve `-1` nedir?

- **head:** En son migration revision'ını temsil eder. `alembic upgrade head` komutu tüm eksik migration'ları sırasıyla uygular.
- **base:** Migration uygulanmamış (boş) durumdur. `alembic downgrade base` tüm migration'ları geri alır.
- **-1 / -2 / +N:** Sayısal adımlar, revision geçmişinde geriye veya ileriye adım sayısını belirtir. `-1` bir önceki adıma geri döner.

Not: Daha güvenli downgrade/upgrade işlemleri için revision id'lerini kullanmak (`alembic downgrade <rev_id>`) önerilir.