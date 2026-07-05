conftest.py nedir?
─────────────────────────────────────────────
- Pytest'in özel dosyasıdır, ismini değiştiremezsin
- İçindeki fixture'lar klasördeki tüm test
  dosyalarına otomatik inject edilir
- import etmene gerek yoktur, pytest bulur

fixture nedir?
─────────────────────────────────────────────
- Testlerin ihtiyaç duyduğu hazır nesne/veridir
- scope="function" → her test için yeniden çalışır
- yield öncesi = setup (hazırlık)
- yield sonrası = teardown (temizlik)

neden sqlite:///:memory:?
─────────────────────────────────────────────
- Gerçek DB'ye bağlanmak yerine RAM'de çalışır
- Test bitince otomatik yok olur
- Hızlı, temiz, yan etkisiz

### Örnek conftest.py

```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from my_app.models import Base

# 1. Her açılışta bir kere engine çalıştırır.
@pytest.fixture(scope="session")
def engine():
    # TestClient ile aynı process/thread içinde kullanmak için aşağıdaki ayarlar gerekebilir
    from sqlalchemy.pool import StaticPool
    return create_engine(
        "sqlite:///:memory:",
        connect_args={"check_same_thread": False},
        poolclass=StaticPool,
    )

# 2. Her açılışta tabloları kurar, yield ile beklemeye alır, bitince tabloları siler.
@pytest.fixture(scope="session")
def tables(engine):
    Base.metadata.create_all(engine)
    yield
    Base.metadata.drop_all(engine)

# 3. Her test için engine e bağlanır
@pytest.fixture(scope="function")
def db_session(engine, tables):
    connection = engine.connect()
    # , ve bir veritabanı işlemi başlatır ki her testten sonre değişiklikleri geri alabilsin.
    transaction = connection.begin()
    Session = sessionmaker(bind=connection)
    session = Session()

    yield session

    session.close()
    transaction.rollback()
    connection.close()

```

#### Genel Akış

test koşusu başlar
    └── engine oluşur (bir kere)
    └── tablolar oluşur (bir kere)
        └── test_1 başlar
            └── connection aç
            └── transaction başlat
            └── session teslim et
            └── test biter → rollback
        └── test_2 başlar
            └── connection aç
            └── transaction başlat
            └── session teslim et
            └── test biter → rollback
    └── tablolar silinir
test koşusu biter

Not: `TestClient` ile entegrasyon testlerinde `check_same_thread=False` ve `StaticPool` kullanmak, in-memory SQLite bağlantısının farklı thread'lerde de kullanılmasına olanak verir.