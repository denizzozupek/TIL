
```SQL
CREATE VIEW kitap_gecmisim_2025 AS
SELECT books.title, read_log.rating
FROM books 
JOIN read_log  ON books.id = read_log.book_id
WHERE read_log.read_year = 2025;
```

Bu şekilde sık kullanılan sorguları bir `VIEW` içine alıp, daha sonra basit `SELECT * FROM kitap_gecmisim_2025` ile sorgulayabilirsiniz.

```sql
SELECT * FROM kitap_gecmisim_2025;
```
