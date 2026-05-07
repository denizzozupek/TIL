
**Tarih:** 06.05.2026
**Etiketler:** #pytest #sqlite3 #fastapi #sqlalchemy #testing
## Ön Hazırlık

```python
from fastapi.testclient import TestClient
from app.main import app
from app.database import get_db

@pytest.fixture
def client(db_session):
    def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db
    with TestClient(app) as c:
        yield c
```

Testler genelde Hazırlık Eylem ve Assert olarak üçe ayrılır. Eylem kısmında crud kullanılmaz onun yerine client.delete client.post yani TestClient Kullanılır.


## 🧪 `conftest.py` Neden Kırılıyordu?

>[!info] Sorunun kaynağı test kodunda değil, test veritabanı kurulumundaydı.
`sqlite:///:memory:` ile oluşturulan in-memory SQLite bağlantısı, `TestClient` istekleri başka bir thread’de çalışınca aynı bağlantıyı tekrar kullanamıyordu. Bu yüzden şu hata çıkıyordu:

`SQLite objects created in a thread can only be used in that same thread`

## ✅ Ne Değiştirdim?

- `create_engine()` içine `connect_args={"check_same_thread": False}` ekledim.
- `poolclass=StaticPool` kullandım.

## Neden?

- `check_same_thread=False` SQLite’ın bağlantıyı farklı thread’lerde kullanmasına izin veriyor.
- `StaticPool` ise aynı in-memory veritabanı bağlantısının test boyunca korunmasını sağlıyor.

## Test Tarafındaki Ek Düzeltme

- `client.post(..., json=model)` yerine `json=model.model_dump()` kullandım.
- Böylece Pydantic modelini JSON’a çevirmeden gönderme hatası çözüldü.
- Response verisini de sözlük gibi okudum: `data["book"]["title"]`

## Sonuç

- `conftest.py` tarafındaki thread hatası çözüldü.
- Test verisi gönderimi düzeldi.
- `tests/test_main.py` tamamen geçti.