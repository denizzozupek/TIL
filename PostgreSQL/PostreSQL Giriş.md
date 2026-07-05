# PostgreSQL Giriş

### Temel Tablo Oluşturma

```sql
CREATE TABLE my_first_table (
    first_column text,
    second_column integer
);
```

### Kayıt Ekleme (INSERT)

```sql
INSERT INTO tablo_adi (sutun1, sutun2, sutun3)
VALUES ('deger1', 2, 'deger3');
```

### Varsayılan Değer (DEFAULT)

```sql
CREATE TABLE my_first_table (
    first_column integer DEFAULT 0
);
```

## Tablo Yapısını Değiştirme (ALTER)

Mevcut bir tabloyu silmeden yeni sütun eklemek veya sütun değiştirmek için `ALTER TABLE` kullanılır.

```sql
ALTER TABLE tablo_adi ADD COLUMN yeni_sutun_adi integer;
```

Örnek:

```sql
ALTER TABLE read_log ADD COLUMN read_year integer;
```

### Veri Güncelleme (UPDATE)

```sql
UPDATE tablo_adi
SET sutun_adi = yeni_deger
WHERE kosul;
```

### Tabloyu Görüntüleme

```sql
SELECT * FROM my_first_table;
```

### Kısıtlar (Constraints)

- `NOT NULL`: Boş bırakılamaz.
- `UNIQUE`: Tekil değer.
- `CHECK`: Özel koşul (`CHECK (price > 0)`).
- `PRIMARY KEY`: Benzersiz ve NOT NULL olan anahtar.
- `FOREIGN KEY`: İki tablo arasında ilişki kurar.

FK silme/güncelleme davranışları (`ON DELETE` / `ON UPDATE`):

- `RESTRICT`: Bağlı veri varsa silmeyi engeller.
- `CASCADE`: Ana veri silinirse bağlı veriler de silinir.
- `SET NULL`: Ana veri silinince bağlı alan `NULL` olur.

Not: Transaction konseptini (BEGIN / COMMIT / ROLLBACK) ve indeksleri (CREATE INDEX) kısa not olarak eklemek faydalıdır.