

```SQL
FROM birinci_tablo
JOIN ikinci_tablOB ON birinci_tablo.ortak_sutun = ikinci_tablo.ortak_sutun
```


**1. Çekilecek Veriler (`SELECT`)** Hangi sütunları görmek istiyorsun? İki farklı tabloda çalıştığımız için, sütun isimlerinin başına hangi tablodan geldiklerini belirtmek (isim çakışmalarını önlemek için) standart bir mühendislik pratiğidir. _Kural:_ `SELECT tablo1.sutun_adi, tablo2.sutun_adi`

**2. Ana Tablo (`FROM`)** Sorgunun merkezine hangi tabloyu koyuyorsun? _Kural:_ `FROM tablo1`

**3. Birleştirilecek Tablo (`JOIN`)** Sisteme hangi yeni tabloyu dahil ediyorsun? _Kural:_ `JOIN tablo2`

**4. Kesişim Şartı (`ON`)** Bu iki tablodaki satırlar birbirini hangi sütunlardaki veri eşitliğine bakarak bulacak? (Primary Key = Foreign Key mantığı). _Kural:_ `ON tablo1.anahtar_sutun = tablo2.yabanci_anahtar_sutun`

**5. Filtreleme (`WHERE`)** Birleştirme işlemi bittikten ve ortaya devasa tek bir sanal tablo çıktıktan sonra, hangi satırları eleyeceksin? (Filtreler her zaman birleştirmeden sonra yazılır). _Kural:_ `WHERE tablo2.filtre_sutunu = deger`
