
>[!info] `selectinload` SQLAlchemy'de ilişkili veriyi optimize şekilde yüklemek için kullanılır.

`selectinload` kullanılmazsa, ilişki erişildiğinde (ör. `for log in logs: print(log.book)`) ayrı ayrı sorgular tetiklenerek N+1 problemi oluşur.

Basit, doğru örnek:

```python
from sqlalchemy import select
from sqlalchemy.orm import selectinload

stmt = select(ReadLog).options(selectinload(ReadLog.book))
logs = session.execute(stmt).scalars().all()

for log in logs:
	# log.book ilişkisi önceden yüklenmiş olur — ekstra sorgu atılmaz
	print(log.book.title)
```

Notlar / farklar:

- `selectinload`: İlişkili nesnuları ayrı, ama tek seferde yükleyen (IN query) stratejidir; büyük koleksiyonlarda genelde daha iyi bellek kullanımı ve performans sunar.
- `joinedload`: Tek bir JOIN ile ilişkili veriyi getirir; küçük sonuç kümeleri veya filtrelenmiş sorgularda daha uygundur, ama JOIN sonucu satır tekrarına (row duplication) yol açabilir.
- `subqueryload`: `selectinload` ile benzer ama alt sorgu (SUBQUERY) kullanır; kullanım durumuna göre fark gösterebilir.

Eğer `.join(Book)` ile ayrıca bir JOIN kullandıysanız, `selectinload` ile birlikte kullanmak anlamlı olmayabilir; amacınıza göre ya sadece `selectinload` kullanın ya da sorguda JOIN + `joinedload` tercih edin.
