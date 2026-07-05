# postgresql #database

## Verileri toplama

```sql
SELECT SUM(sutun_adi) FROM tablo_adi;
```

## Koşula göre veri sayma

```sql
SELECT COUNT(*) FROM tablo_adi WHERE kosul;
```

# 🛠 SQL Sorun Giderme: Sayaç Senkronizasyonu (setval)

## Sorun: `duplicate key value violates unique constraint`

Eğer `SERIAL` (otomatik artan) bir sütuna sahip tabloya, dışarıdan (CSV veya manuel) `id` değerlerini kendin vererek veri eklersen, PostgreSQL'in arka plandaki sayacı (`sequence`) güncellenmez. Sen tekrar otomatik veri eklemek istediğinde, veritabanı zaten var olan bir ID'yi vermeye çalışır ve hata verir.

## Çözüm: Sayacı En Büyük ID'ye Eşitlemek

Aşağıdaki komut, tablodaki en yüksek ID'yi bulur ve otomatik sayacı o sayıdan başlayacak şekilde günceller.



```SQL
SELECT setval(
    pg_get_serial_sequence('tablo_adi', 'id_sutunu'), 
    (SELECT MAX(id_sutunu) FROM tablo_adi)
);
```


### Örnek (read_log tablosu için):

```sql
SELECT setval(pg_get_serial_sequence('read_log', 'id'), (SELECT COALESCE(MAX(id), 0) FROM read_log));
```

---

**💡 Not:** Bu işlemden sonra yapılacak ilk `INSERT` işlemi, `MAX(id) + 1` değerini alarak sorunsuz devam edecektir.

### Gruplama ve Sıralama

```SQL
SELECT genre, COUNT(*) 
FROM read_log 
WHERE genre != 'Çizgi Roman'
GROUP BY genre 
ORDER BY COUNT(*) DESC;
```

- GROUP BY genreye göre grupladı
- WHERE genre kısmı genrede istenmeyen türü eledi
- ORDER BY COUNT() DESC kısmı azalana göre sıraladı.

## SUBQUERY

```SQL
SELECT title, page_count FROM read_log 
WHERE 
	page_count > (
	SELECT
			AVG(page_count)
		FROM
			read_log
	);
```