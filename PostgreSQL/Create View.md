
```sql
CREATE VIEW kitap_gecmisim_2025 AS
SELECT books.title, read_log.rating
FROM books
JOIN read_log ON books.id = read_log.book_id
WHERE read_log.read_year = 2025;
```

Sık kullanılan sorguları `VIEW` içine alıp daha sonra `SELECT * FROM kitap_gecmisim_2025;` ile çağırabilirsiniz.

```sql
SELECT * FROM kitap_gecmisim_2025;
```
