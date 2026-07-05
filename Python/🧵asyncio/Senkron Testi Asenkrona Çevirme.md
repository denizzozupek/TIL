
**Tarih:** 03.03.2026
**Etiketler:** #asyncio #python #pytest #testing #mocking #AsyncVideoSummarizerProject

> [!info] Temel Kural
> [!info] Temel Kural
> Asenkron testlerde `async def` ile tanımlanan test fonksiyonlarına `@pytest.mark.asyncio` ekleyin veya `pytest-asyncio` plugin'i kullanın.

> [!tip] Asenkron nesneleri taklit etmek (mocking)
> Standart `MagicMock` senkron fonksiyonları taklit eder. Asenkron fonksiyonlar için `unittest.mock.AsyncMock` kullanın.

> [!warning] Await kullanımı
> Asenkron bir fonksiyonu test içinde çağırıyorsanız, sonucu `assert` etmeden önce mutlaka `await` ile bekleyin. Aksi halde coroutine objesi elde edersiniz.

### Örnek kullanım:

```python
import pytest

@pytest.mark.asyncio
async def test_success():
	transcript = "Bu bir test video transkriptidir."
	summary = await youtube_text_summarizer(transcript)
	assert summary == "Özetlenmiş metin"
```

Mock örneği:

```python
from unittest.mock import AsyncMock

async def test_with_mock(monkeypatch):
	mock_summarizer = AsyncMock(return_value="Özetlenmiş metin")
	monkeypatch.setattr("app.summarizer.youtube_text_summarizer", mock_summarizer)
	result = await mock_summarizer("dummy")
	assert result == "Özetlenmiş metin"
```