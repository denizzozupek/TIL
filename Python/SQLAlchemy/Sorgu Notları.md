
1) `notin_` ile belirli değerleri hariç tutarız:

```python
query = select(Book).where(Book.genre.notin_("Çizgi Roman", "Manga"))
```


2) Skip parametresi ile atlanacak sorgu sayısı belirlenir ve query.offset(skip) ile çalıştırırız. skip parametresini ya default olarak fonk içine veri ya da kullanıcıdan alırız.


3)  [[How to Solve N+1 Problem for Query]]


4) **db.refresh() :** 

- **`db.add(db_book)`:** Python, RAM'de bir kitap objesi oluşturur. Bu objenin `title`, `author` gibi bilgileri vardır ama henüz bir **ID'si yoktur**. Çünkü ID'yi (Auto-Increment) sadece veritabanı atayabilir.
    
- **`db.commit()`:** Python, bu veriyi PostgreSQL'e yollar ve "Bunu kaydet" der. PostgreSQL kaydı yapar, ona örneğin `id=38` numarasını verir. **Ancak:** PostgreSQL bu 38 numarasını geri dönüp Python'a söylemez. Python'un elindeki `db_book` objesinin ID'si hala boştur (`None`).
    
- **`db.refresh(db_book)`:** İşte burada Python'a şu emri veriyoruz: _"PostgreSQL'e geri git, az önce kaydettiğimiz kitabın son halini (üzerine basılmış ID damgasıyla birlikte) al ve RAM'deki `db_book` objesini bu yeni verilerle güncelle."_


5) `.label()`, veritabanı sorgusunda bir kolona veya hesaplanmış bir değere **geçici bir takma ad  vermek** için kullanılır.


6) PostgreSQL C diliyle yazılmıştır; tarih tabanlı parçalarda `func.date_part` işe yarar:

```python
func.date_part("month", ReadLog.read_date).label("month")
```


7) Sorgudan dönen tuple verileri, Pydantic’in beklediği response modeline uygun olması için liste içinde sözlüklere dönüştürülür

```python
return [
        {"genre": genre, "average_rating": round(avg_rating, 1)}
        for genre, avg_rating in results
  ]
```


8) SQL sorguları genelde SELECT → FROM/JOIN → WHERE → ORDER BY şeklinde yazılır. Bu, okunabilirliği artırır. Ancak veritabanı motoru sorguyu optimize ederek işlemleri farklı bir sırada gerçekleştirebilir.