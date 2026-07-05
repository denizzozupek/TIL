# pydantic # fastapi # api # database

## UX ve DB çatışmasını çözmek

Veritabanı modelleri (`models`) ile API'den gelen Pydantic şemaları (`schemas`) birebir aynı olmak zorunda değildir. Örneğin API'den `read_month` ve `read_year` alıp arka planda `read_date = date(read_year, read_month, 1)` şeklinde DB'ye kaydetmek UX açısından daha kolay olabilir.

Örnek akış:

- API → `read_month=5`, `read_year=2025`
- DB  → `read_date=2025-05-01`

Bu dönüşüm service layer içinde yapılmalıdır; schema değil, iş mantığı bu dönüşümü gerçekleştirmeli.

## Dinamik sınırlar ve doğrulamalar

`Field(...)` içindeki `...` değeri alanı zorunlu yapar. Statik sabitler yerine dinamik sınırlar tercih edin (ör. `le=date.today().year`) fakat import-time değerlendirme farkına dikkat edin — runtime validator genelde daha güvenlidir.

## Opsiyonel alanlar ve tip hataları

`read_year: int | None = Field(ge=1900)` gibi bir tanım `None` geldiğinde karşılaştırma hatası üretebilir. Bu durumda ya `Optional[int]` için custom validator yazın, ya da `default=None` bırakıp validator ile koşulu kontrol edin.

## Get-or-Create mantığı

Birden fazla API çağrısı yerine tek bir geniş JSON alıp sunucu tarafında gerekli varlıkları (ör. kitap) bulup veya yaratmak genelde daha pratiktir. Bu işlemleri transaction içinde, atomic olarak yapmak önemlidir.

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
	read_year: int | None = Field(default=None)

	# read_year doğrulaması için validator kullanılabilir
```

## `model_config = {"from_attributes": True}` (sürüm farkı)

- Pydantic v1: ORM objelerini okumak için `class Config: orm_mode = True` kullanılır.
- Pydantic v2: `model_config = {"from_attributes": True}` veya ilgili ayarlar kullanılır.

Eğer direkt ORM nesnesi (SQLAlchemy objesi) veriliyorsa ilgili `orm_mode`/`from_attributes` ayarlarını açmak gerekir.
