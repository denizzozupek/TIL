#postresql #database 
### Table Basics

**Creating a table:** 

``` SQL
CREATE TABLE my_first_table (
    first_column text,
    second_column integer
);
````

**Inserting value into table:**

To insert data into a table in PostgreSQL, we use the `INSERT INTO` statement.

```SQL
INSERT INTO tablo_adi (sutun1, sutun2, sutun3) 
VALUES (deger1, deger2, deger3);


```

**A column can be assigned a default value.**

``` SQL
CREATE TABLE my_first_table (
    first_column integer DEFAULT 0
);
````


## Tablo Yapısını Değiştirme (ALTER) 🏗️

Mevcut bir tabloyu silip yeniden yapmadan, üzerine yeni özellikler eklemek için kullanılır.

### Yeni Sütun Ekleme


```SQL
ALTER TABLE tablo_adi ADD COLUMN yeni_sutun_adi veri_tipi;
```

- **Örnek:** `ALTER TABLE read_log ADD COLUMN read_year integer;

### 📝 DATA UPDATE: UPDATE


```SQL
UPDATE tablo_adi 
SET sutun_adi = yeni_deger 
WHERE kosul;
```


### Display Table

To check the result we can display the table with this SQL statement:

SELECT * FROM my_first_Table;
### Constrains

- NOT NULL: cannot be empty
- UNIQUE: unique
- CHECK: CHECK (price > 0)
- PRIMARY KEY (PK) : UNIQUE + NOT NULL
- FOREIGN KEY (FK) : Connect two table to each other

**FK Silme/Güncelleme Davranışları (`ON DELETE` / `ON UPDATE`):**

- `RESTRICT`: Bağlı veri varsa silmeyi engeller (Hata fırlatır).
    
- `CASCADE`: Ana veri silinirse, ona bağlı olan tüm alt verileri de otomatik siler (Dominoya benzer).
    
- `SET NULL`: Ana veri silinirse, bağlı tablodaki o alanı boş (`NULL`) bırakır.