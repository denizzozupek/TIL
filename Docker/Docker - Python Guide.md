
**Tarih:** 09.05.2026
**Etiketler:** #docker #python #container 

Başta Docker'i indirmiş ve bir Git clienti kullanıyor olmalıyız. Bu adımların çoktan geçildiğini var sayıyorum. Ardından Dockerfile yazımına geçiyorum: 

## Dockerfile

#### ==1) FROM==

Her Dockerfile ilk olarak FROM ile başlar. FROM işletim sistemini ve çalıştırılacağı dili söyler.

#### ==2) WORKDIR==

Çalışma alanını belirler. 

#### ==3)COPY==

Yerel cihazımızdaki dosyaları Docker container'inin içine kopyalamak için kullanılır.

**Kullanım:** Genellikle `COPY . /app` (mevcut dizindeki her şeyi konteynerdeki `/app` klasörüne kopyala) şeklinde kullanılır.

#### ==4) CMD==

Dockerfile'daki `CMD` komutu, bir Docker konteyneri başlatıldığında çalıştırılacak **varsayılan komutu veya parametreleri** tanımlar. İmaj çalıştırılırken (docker run) ekstra bir komut verilmezse `CMD` devreye girer, ancak çalışma zamanında kolayca ezilebilir (geçersiz kılınabilir). 


==**CMD Komutunun Temel Özellikleri:**==

- **Varsayılan İşlem:** Konteyner ayağa kalktığında otomatik olarak çalışmasını istediğiniz uygulamayı veya betiği belirtir.
- **Ezilebilir Yapı:** `docker run` komutunun sonuna yeni bir komut eklenirse, Dockerfile'daki `CMD` komutu devre dışı kalır.
- **Sınırlama:** Bir Dockerfile içinde yalnızca **bir adet** `CMD` bulunmalıdır. Birden fazla varsa, yalnızca en sonuncusu çalışır.
- **Kullanım Formatı:** Genellikle JSON dizisi formatında (`CMD ["executable","param1"]`) kullanılır.

#### ==4) RUN==

Docker imajı oluşturulurken komutlar çalıştırmak için işe yarar.

 **Kullanımı**: Genellikle RUN pip install -r requirement.txt 


#### ==Docker Compose== 

Tek tek terminal komutları yazmak yerine, tüm sistemin mimarisini tek bir YAML dosyasına yazarak, tek komutla (`docker-compose up`) hem Python uygulamanı hem de PostgreSQL veritabanını aynı anda, birbirine bağlı şekilde ayağa kaldırmanı sağlayan orkestrasyon aracı.

İşin temelinde `docker-compose.yml` dosyası yatar. Bu dosya üç ana yapı taşından oluşur: **Services**, **Volumes** ve **Networks**. (+ Version)


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


==Temel Yönetim Komutları==

- **docker compose up: Yapılandırma dosyasındaki tüm servisleri oluşturur ve başlatır.

    - `-d`: Servisleri arka planda (detached mod) çalıştırır.
    - `--build`: Başlatmadan önce imajları yeniden derler.

- **docker compose down: Tüm konteynerleri, ağları ve (isteğe bağlı olarak) disk birimlerini durdurur ve tamamen kaldırır.

- **docker compose start/ stop**: Halihazırda oluşturulmuş olan servisleri başlatır veya durdurur (konteynerleri silmez).

- **docker compose restart**: Tüm servisleri veya belirli bir servisi yeniden başlatır. 

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

Konteyner başlatıldığında ilk çalışan komut dosyası. Sorumlulukların ayrılması "Seperation of Concerns" konusu açısından çok önemlidir.

Konteynerin ayağa kalkması, veritabanı bağlantılarının kurulması, ortam değişkenlerinin ayarlanması veya ana uygulamanın başlatılması gibi hazırlık işlemlerini yönetir.