**Tarih:** 14.05.2026
**Etiketler:** #pgvector

### Kurulum

Uzantıyı etkinleştirin:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

Örnek: 3 boyutlu embedding kolonu ile tablo oluşturma

```sql
CREATE TABLE items (
	id bigserial PRIMARY KEY,
	embedding vector(3)
);
```

Kayıt ekleme:

```sql
INSERT INTO items(embedding) VALUES ('[1,2,3]'), ('[4,5,6]');
```

Benzerlik sıralaması (en yakın 5):

```sql
SELECT * FROM items ORDER BY embedding <=> '[3,1,2]' LIMIT 5;
```

Mesafe hesaplama:

```sql
SELECT embedding <=> '[3,1,2]' AS distance FROM items;
```

Notlar:
- `<=>` operatörü genelde cosine veya L2 bazlı mesafe sağlar; sürüme göre davranış değişebilir — dokümantasyonu kontrol edin.
- HNSW (pgvector içinde `ivfflat`/`hnsw` gibi metodlar) büyük veri setlerinde hızlı nearest-neighbor aramaları sağlar.