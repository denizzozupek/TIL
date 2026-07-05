
**Tarih:** 09.05.2026
**Etiketler:** #docker #python #container 

Başta Docker'i indirmiş ve bir Git clienti kullanıyor olmalıyız. Bu adımların çoktan geçildiğini var sayıyorum. Ardından Dockerfile yazımına geçiyorum: 

## Dockerfile

#### 1) FROM

Her Dockerfile `FROM` ile başlar. `FROM` bir base image (çalışma ortamını) belirtir; her zaman doğrudan "işletim sistemi" demek doğru olmayabilir (ör. `python:3.12-slim` bir Python çalışma zamanı imajıdır).

#### ==2) WORKDIR==

Çalışma alanını belirler. 

#### 3) COPY vs ADD

`COPY` yerel dosyaları imaj içine kopyalamak için kullanılır. `ADD` ise ekstra özelliklere sahiptir (ör. URL'den indirme veya arşivleri otomatik çıkarma) — çoğu durumda `COPY` daha öngörülebilir ve tercih edilir.

Kullanım örneği: `COPY . /app` (mevcut dizini konteynerde `/app`'e kopyala)

#### 4) ENTRYPOINT ve CMD

`CMD`: Varsayılan komut/parametreleri belirtir; `docker run` ile üzerine yazılabilir.

`ENTRYPOINT`: İmajın her zaman çalıştıracağı komutu kilitler; `CMD` ile birlikte kullanıldığında `CMD` argümanları `ENTRYPOINT`'ın sonuna eklenir.

Örnek: `ENTRYPOINT ["/entrypoint.sh"]` ve `CMD ["uvicorn", "app.main:app"]` kombinasyonu, entrypoint script'in çalışmasını sağlar ve CMD ile varsayılan argümanlar verilir.

#### 5) RUN

İmaj oluşturulurken çalıştırılacak komutlar için kullanılır. Örnek:

```
RUN pip install --no-cache-dir -r requirements.txt
```

Not: Katman (layer) sayısını azaltmak için mümkün olduğunca tek RUN içinde birleştirme yapılabilir.


#### Docker Compose

Servis bağımlılıklarını, ağları ve volümleri tek bir YAML dosyasında tanımlamanızı sağlar. Not: `docker compose` (v2) ve `docker-compose` (v1) komutları arasında küçük farklar olabilir; modern Docker sürümlerinde `docker compose` tercih edilir.

`docker-compose.yml` temel yapı taşları: `services`, `volumes`, `networks`, `version` (isteğe bağlı).


```yaml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    restart: always
    env_file:
      - .env
    volumes:
      - db_data:/var/lib/postgresql/data

  web:
    build: .
    restart: always
    ports:
      - "8000:8000"
    env_file:
      - .env
    volumes:
      - .:/app
    depends_on:
      - db

volumes:

  db_data:
```


Temel komutlar:

- `docker compose up` : Tüm servisleri oluşturur ve başlatır. `-d` arka plan, `--build` imajları yeniden derler.
- `docker compose down` : Servisleri, ağları ve bağlantılı kaynakları durdurur ve isteğe bağlı olarak kaldırır.
- `docker compose start/stop` : Oluşturulmuş servisleri başlatır/durdurur.
- `docker compose restart` : Servisleri yeniden başlatır.

Ek not: Projede gereksiz dosyaların konteyner içine kopyalanmasını önlemek için `.dockerignore` dosyası oluşturun (örn. `venv/`, `.git/`, `__pycache__/`).

# ==Taslaklar==

### Dockerfile örneği: 

```Dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

# Docker Entrypoint.sh

Konteyner başlatıldığında ilk çalışan betiktir. Sorumlulukların ayrılması (separation of concerns) açısından önemlidir.

Örnek kullanım: veritabanı hazır olana kadar bekleme, ortam değişkenlerini ayarlama, migrasyon çalıştırma vb. işleri burada yapabilirsiniz.