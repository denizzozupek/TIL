```sql
SELECT *
FROM birinci_tablo
JOIN ikinci_tablo ON birinci_tablo.ortak_sutun = ikinci_tablo.ortak_sutun;
```

1. Çekilecek veriler (`SELECT`): Hangi sütunları görmek istiyorsun? İki farklı tabloda çalışırken sütun isimlerinin önüne tablo adını yazmak isim çakışmasını önler. Örnek: `SELECT tablo1.sutun_adi, tablo2.sutun_adi`.

2. Ana tablo (`FROM`): Sorgunun merkezine hangi tabloyu koyduğunuz.

3. Birleştirilecek tablo (`JOIN`): Hangi tabloyu eklediğiniz.

4. Kesişim şartı (`ON`): İki tablodaki satırların eşleşme koşulu (genelde PK = FK).

5. Filtreleme (`WHERE`): Birleştirmeden sonra hangi satırları eleyeceğiniz. Örnek: `WHERE tablo2.filtre_sutunu = deger`.
