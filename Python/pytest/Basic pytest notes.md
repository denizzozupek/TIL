
Pytest'te `fixture`lar, testlerden önce veya sonra çalışacak hazırlık/temizlik (setup/teardown) kodlarını paylaşmak için kullanılır. `@pytest.fixture` ile tanımlanır ve test fonksiyonlarına argüman olarak verilebilir.

Basit örnek:

```python
import pytest

@pytest.fixture
def sample_data():
	# setup
	data = {"a": 1, "b": 2}
	yield data
	# teardown (cleanup) - örneğin veritabanı temizleme

def test_sum(sample_data):
	assert sample_data["a"] + sample_data["b"] == 3
```

Özellikler:

- **Setup/Teardown:** Fixture içinde `yield` kullandığınızda `yield` öncesi setup, sonrasında teardown kodu çalışır.
- **Veri sağlama:** Testlere sabit veya dinamik veri gönderirler.
- **Scope:** `scope` parametresiyle `function`, `class`, `module` veya `session` gibi çalıştırma sıklığı belirlenebilir.

Detaylı dokümantasyon için pytest fixtures sayfasına bakılabilir; not olarak doğrudan uzun URL koymak yerine kısa örnekler faydalıdır.