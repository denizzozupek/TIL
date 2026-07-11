#pydantic #fastapi #api #database 

## **UX ve DB Çatışmasını Çözmek (Abstraction):**

Veritabanındaki sütunlarla (models), API'den gelen JSON (schemas) birebir aynı olmak zorunda değil. Kullanıcının hayatını kolaylaştırmak için API'den sadece `read_month` ve `read_year` (5 ve 2025) alıp, arka planda bunu Python ile birleştirerek PostgreSQL'in sevdiği `Date` objesine (`2025-05-01`) çevirerek kaydedebilirim. Bu, UX'i basitleştirirken veritabanının analitik gücünü korur.

Örnek:

- API → `read_month=5`, `read_year=2025`
- DB → `read_date=2025-05-01`

 Bu dönüşüm:

- UX’i basitleştirir
- DB’de filtreleme / analiz gücünü korur

Bu katman genelde:  
**service layer** içinde yapılmalı (schema değil)
## **Dinamik Sınırlar:

** `Field(...)` içindeki üç nokta o verinin zorunlu olduğunu belirtir. Sınır çizerken kodun birkaç yıl sonra eskimemesi (technical debt yaratmaması) için statik rakamlar yerine `le=date.today().year` gibi dinamik fonksiyonlar kullanmak daha güvenli.

## **Sinsi Tip Hatası (Type Error):** 

Bir değişkene `int | None` (opsiyonel) deyip aynı zamanda `ge=1900` gibi bir matematiksel koşul eklersem; API'ye `null` geldiğinde sistem `None >= 1900` kıyaslaması yapmaya çalışıp patlar (500 Internal Server Error). Matematiksel büyüklük sınırı olan şeyler opsiyonel olabilir ama oluyorsa da field validator ile korunmalı. 

## **Get-or-Create Mantığı:** 
Kullanıcıya önce kitap ekletip sonra okuma kaydı ekletmek (iki ayrı API isteği atmak) kötü bir tasarımdır. API tek bir geniş JSON almalı (`BookAndLogCreate`), arka plandaki iş mantığı kitabı veritabanında arayıp bulmalı (veya yaratmalı) ve ID'leri kendi kendine eşleştirmelidir.

```python
from pydantic import BaseModel, Field
from datetime import date

class BookAndLogCreate(BaseModel):

    title: str
    author: str
    genre: str
    page_count: int = Field(gt=0)

    rating: int = Field(ge=1, le=10)
    read_month: int = Field(ge=1, le=12)
    read_year: int | None = Field(ge=1900, le=date.today().year)
```


### model_config = {"from_attributes": True}

>[!important] Veritabanı Tablosunun kendisini yolluyorsak kullanmak şarttır.

Pydantic gelen veriyi JSON/sözlük formatında görmezse hata verir. Örneğin eğer veri obje ise from_attributes=True bunları okumaya yarar. 

İç içe verilerde özellikle object de geldiği için bu ayar sadece ana objeyi değil içindeki objeyi tanımayı ve onu da otomatik olarak JSON şemasına dönüştürmeyi sağlar.
