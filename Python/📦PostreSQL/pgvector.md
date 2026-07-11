**Tarih:** 14.05.2026
**Etiketler:** #pgvector

### Kurulum

Uzantıyı çalıştırıyoruz.

```SQL
CREATE EXTENSION vector IF NOT EXISTS vector;
```

3 boyutta vektör column ile bir tablo oluşturuyoruz.

```SQL
CREATE TABLE items(id bigserial PRIMARY KEY, embedding vector(3))
```

Tabloya vektör ekliyoruz.

 ```SQL
 INSERT INTO items(embedding) values ('[1,2,3]'), ('[4,5,6]')
 ```

Mesafeyi (Distance) Kullanarak sorgu yapıyoruz.

```SQL 
SELECT * FROM items ORDER BY embedding <=> '[3,1,2]' LIMIT 5;
```

Şimdi aralarındaki mesafeyi soruyoruz.

```SQL
SELECT embedding <=> '[3,1,2]' AS distance FROM items
```

> [!info]
>  <=> Kosinüs mesafe operatörü anlamına gelir. 
> <#> Dot product mesafe operatörü anlamına gelir.

HSNW hızlı vektör aramaları için kullanılabilir.