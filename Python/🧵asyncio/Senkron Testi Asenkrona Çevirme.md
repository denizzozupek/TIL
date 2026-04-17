
**Tarih:** 03.03.2026
**Etiketler:** #asyncio #python #pytest #testing #mocking #AsyncVideoSummarizerProject

> [!info] Temel Kural
>1. async def ile asenkron yapılan fonksiyonun üstüne hata vermemesi için @pytest.mark.asyncio eklenir. 

>[!tip] Asenkron Nesneleri Taklit Etmek (Mocking) > 
Standart `MagicMock` sadece senkron fonksiyonları taklit eder. Eğer asenkron çalışan (başında `await` ile çağrılan) bir nesneyi veya fonksiyonu test ortamında taklit (mock) etmek istiyorsak, `unittest.mock` kütüphanesinden **`AsyncMock()`** modülünü kullanmalıyız.

> [!warning] Await Kullanımı > Testin içinde asenkron bir fonksiyonu çağırıyorsak, sonucunu `assert` (doğrulama) ile test etmeden önce o fonksiyonu mutlaka `await` ile bekleyip gerçek değerini bir değişkene çıkarmalıyız. (Aksi halde elimizde gerçek veri değil, bir "coroutine/bekleme bileti" olur ve test yanlışlıkla başarısız olur.)
### 💻 Örnek Kullanım:
```python
    @pytest.mark.asyncio

    async def test_success(self , mock_env):

        """Checks if we get a summary back when everything goes right."""

        transcript = "Bu bir test video transkriptidir. İçeriği özetlenecektir."

        summary = await youtube_text_summarizer(transcript)

        assert summary == "Özetlenmiş metin"