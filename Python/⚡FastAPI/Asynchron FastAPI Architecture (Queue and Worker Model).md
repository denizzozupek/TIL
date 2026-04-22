
**Tarih:** 03.03.2026
**Etiketler:** #fastapi #asyncio #architecture #background-tasks #queue #AsyncVideoSummarizerProject
## 🎯 Problem
Kullanıcıdan gelen URL'nin transkriptini çıkarıp AI ile özetlemek zaman harcar. Eğer bunu doğrudan yaparsak sunucu kilitlenir, yığılma olur.
**Çözüm:** URL'yi al, Kuyruğa (Queue) at, kullanıcıya anında "işleme alındı" mesajı dön. Arka planda bir Worker çalıştır. Bu Worker kuyruktaki işleri sırayla alıp yapsın.

---

## 1. Worker
Sunucu yaşadığı sürece devam eden sonsuz bir döngüdür. Kuyruktaki işleri alır, işler ve döngünün başına döner.

> [!WARNING] Sonsuz Döngü Kuralı
> İşçinin içinde asla `return` veya `break` kullanılmaz (yoksa işçi istifa eder ve dükkanı kapatır). Hata olsa bile `continue` ile sıradaki işe geçmesi sağlanır. Dönüş tipi (Type Hint) `-> None` olmalıdır.

> [!ERROR] Kapanış Sinyali (CancelledError)
> Sunucu kapanırken işçiyi zorla durdurduğumuzda sistem `asyncio.CancelledError` fırlatır. Bunu yakalayıp `raise` ile geri fırlatmalıyız ki işçi resmi olarak paydos edebilsin.

---

## 2. Lifespan
> [!WARNING] Uygulama (app) ayağa kalkarken kuyruğu oluşturduğumuz ve işçiyi işe aldığımız yerdir. FastAPI'nin bu yapıyı anlaması için `@asynccontextmanager` dekoratörü şarttır. (Python `yield` içeren fonksiyonları bir generator olarak algıladığı için bu dekoratör zorunludur).

> [!TIP] Yield 
> `yield` kelimesinin üstü sunucu açılırken, altı ise sunucu kapanırken çalışır. İşçiyi iptal etmek (`cancel()`) sunucu kapanırken yapılacak en önemli temizlik (teardown) işlemidir.

---

### 💻 Full Architecture:

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
import asyncio
import logging

# 1. İŞÇİ FONKSİYONU
async def worker(queue: asyncio.Queue[str]) -> None:
    while True:
        try:
            video_url = await queue.get() # Kuyruktan iş bekle
            # ... asenkron işlemler (transkript, özet vb.) ...
            
        except asyncio.CancelledError:
            logger.info("Sunucu kapanıyor, işçi paydos etti.")
            raise
        except Exception as e:
            logger.error(f"Hata: {e}")
            continue # Hata olsa da diğer işe geç
        finally:
            queue.task_done() # İşin bittiğini kuyruğa bildir

# 2. YAŞAM DÖNGÜSÜ (LIFESPAN)
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Sunucu Başlarken: Kuyruğu ve İşçiyi yarat
    app.state.queue = asyncio.Queue()
    app.state.worker_task = asyncio.create_task(worker(app.state.queue))
    
    yield # Sunucu çalışıyor... (Burada bekler)
    
    # Sunucu Kapanırken: İşçiyi durdur ve temizle
    app.state.worker_task.cancel()
    try:
        await app.state.worker_task
    except asyncio.CancelledError:
        pass

# 3. UYGULAMA VE UÇ NOKTA
app = FastAPI(lifespan=lifespan)

@app.post("/summarize")
async def add_queue(video_url: str):
    # İşi kuyruğa atıp anında cevap dönüyoruz (Asıl sihir burası!)
    await app.state.queue.put(video_url)
    return {"message": "Video işleme alındı, arka planda özetleniyor."}
```