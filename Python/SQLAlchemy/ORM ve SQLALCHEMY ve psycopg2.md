#ORM #sqlalchemy #sqlalchemy #psycopg2 #database #Alembic #migration
# ORM

>[!info] **ORM (Object-Relational Mapping - Nesne-İlişkisel Eşleme);** nesne yönelimli programlama (OOP) sistemlerindeki veri modellerini, ilişkisel veritabanlarının (RDBMS) tablo mimarisine otomatik olarak çeviren ve iki sistem arasındaki veri senkronizasyonunu sağlayan bir programlama tekniğidir.
>Bu teknik, kaputun altında şu eşleşmeyi kurala bağlar:
>- **Class (Sınıf)** $\rightarrow$ **Table (Tablo)**
>
>- **Object/Instance (Nesne)** $\rightarrow$ **Row/Record (Satır)**
>    
>- **Attribute (Nitelik/Özellik)** $\rightarrow$ **Column (Sütun)**

**Teknik Olarak Ne Çözer?** Amacı sadece iki dili konuşturmak değildir. RAM üzerinde yaşayan "hiyerarşik, birbirine referans veren ve metotları olan" nesneler ile disk üzerinde yaşayan "matematiksel, düz ve ilişkisel" veri setleri arasındaki **yapısal paradigma çatışmasını (Object-Relational Impedance Mismatch)** yazılımcıdan soyutlamaktır.

Bu sayede yazılımcı, sistemin veri katmanında saf SQL sözdizimi yazmak yerine, projenin kendi nesneleri ve fonksiyonları üzerinden (örneğin; `book.save()`, `session.delete(user)`) veritabanı operasyonlarını (CRUD) yürütebilir. ORM, bu nesne komutlarını arka planda uygun SQL sorgularına derleyip veritabanına iletir.
# SQLAlchemy

>[!info] SQLAlchemy’nin ORM katmanı, SQLite veritabanı ile Python programı arasında bir köprü görevi görür. Veritabanından gelen verileri Python nesnelerine dönüştürür ve tersini de yapar. Böylece geliştirici, verilerle doğrudan SQL yerine nesneler üzerinden çalışabilir; aynı zamanda veritabanının güçlü özelliklerinden de yararlanmaya devam eder.

```python
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey
from sqlalchemy.orm import declarative_base, sessionmaker, relationship

Base = declarative_base()

class Book(Base):
    __tablename__ = 'books'
    id = Column(Integer, primary_key=True)
    title = Column(String, nullable=False)
    author = Column(String, nullable=False)
    genre = Column(String, nullable=False)
    page_count = Column(Integer, nullable=False)

    logs = relationship('ReadLog', back_populates='book')

    def __repr__(self): #def __repr__ metodu, bir nesnenin temsilini tanımlamak için kullanılır. Bu durumda, Book sınıfının bir örneği ekrana yazdırıldığında, bu metodun döndürdüğü string gösterilir. Bu, nesnenin içeriğini daha okunabilir ve anlaşılır hale getirir.
        return f"<Book(title='{self.title}', author='{self.author}', genre='{self.genre}', page_count={self.page_count})>"

class ReadLog(Base):
    __tablename__ = 'readlogs'
    id = Column(Integer, primary_key=True)
    book_id = Column(Integer, ForeignKey('books.id'), nullable=False)
    status = Column(String, nullable=False)
    rating = Column(Integer, nullable=False)
    read_date = Column(String, nullable=False)
    read_pages = Column(Integer, nullable=False)

    # ReadLog sınıfında book_id sütunu, books tablosundaki id sütununa bir yabancı anahtar (foreign key) olarak tanımlanır. Bu, her ReadLog kaydının hangi Book kaydına ait olduğunu belirtir. relationship fonksiyonu ise SQLAlchemy'ye bu iki tablo arasında bir ilişki olduğunu söyler ve bu sayede Book nesneleri üzerinden ilgili ReadLog nesnelerine kolayca erişilebilir hale gelir.
    book = relationship('Book', back_populates='logs')

# SQLAlchemy'ye bağlanmak istediğimiz veritabanının türünü, kullanıcı adını, şifresini, sunucu adresini ve veritabanı adını belirtir. Bu örnekte, PostgreSQL veritabanına bağlanmak için gerekli bilgileri içerir.
engine = create_engine('postgresql+psycopg2://postgres:dnzking60@localhost:5432/postgres')
Base.metadata.create_all(engine)

# sessionmaker, SQLAlchemy'de veritabanı oturumları oluşturmak için kullanılan bir sınıftır. sessionmaker, bir veritabanı bağlantısı (engine) ile yapılandırılır ve daha sonra bu yapılandırmayı kullanarak yeni oturumlar (sessions) oluşturabilir. Bu oturumlar, veritabanı işlemlerini gerçekleştirmek için kullanılır. sessionmaker'ı kullanarak oluşturulan oturumlar, veritabanına sorgular göndermek, veri eklemek, güncellemek veya silmek gibi işlemleri yapmanıza olanak tanır.

session = sessionmaker()
session.configure(bind=engine)
```

# Psycopg2 
Python'un PostgreSQL veritabanı ile doğrudan iletişim kurmasını sağlayan, C diliyle yazılmış endüstri standardı bir adaptördür (driver).

### Session (Oturum) Nedir?

Motor (`engine`) senin PostgreSQL'e giden fiziksel kablondur. Ancak o kablodan rastgele veri gönderemezsin. Verileri güvenli bir şekilde paketleyip, hata çıkarsa geri alabilecek bir "Çalışma Alanı"na ihtiyacın vardır. İşte buna `Session` denir.

# C (Create)

```python
ornek_kitap = Book(title="The Great Gatsby", author="F. Scott Fitzgerald", genre="Novel", page_count=180)

with session() as s:
    s.add(ornek_kitap)
    s.commit()
```

# R (Read)

### Sorgu Hazırlamak: `select()`

Eğer sınıfa "__ repr__" methodu eklersek, from sqlalchemy import select modulü ile print result yaparak direkt okuyabiliriz. repr methodu olmasa print(result.author) diye tek tek belirtmemiz gerekecekti.

SQLAlchemy 2.0 mimarisinde sorgular doğrudan çalıştırılmaz, önce inşa edilir. `select(Book)` kodu veritabanına gitmez; sadece Python hafızasında saf bir SQL metni (Örn: `SELECT * FROM books`) oluşturur. Bu bir taslaktır. Bu taslağın içine `where()`, `order_by()` gibi filtreler eklenerek sorgu zenginleştirilir.

**Veriyi Çekmek: `execute()` vs `scalar()` vs `scalars()`** Oluşturulan `select()` taslağını veritabanına göndermek ve dönen cevabı Python'a almak için oturumun (session) çalıştırıcı fonksiyonları kullanılır.

- **`s.execute(...)`:** Ham (raw) veritabanı satırları döndürür. Okuması ve yönetmesi zordur.
    
- **`s.scalars(...)`:** Gelen ham satırları alır, onları senin belirlediğin nesnelere (örneğin `Book` sınıfına) dönüştürür ve bir nesne listesi (iterable) olarak sana sunar. Birden fazla sonuç bekliyorsan (örneğin tüm kitaplar) `s.scalars(...).all()` kullanırsın.
    
- **`s.scalar(...)`:** "Bana gelen listenin sadece **ilk satırındaki ilk nesneyi** ver, gerisini çöpe at" demektir. Yalnızca tek bir kaydı (örneğin ID'si 1 olan kitap veya "Dune" isimli spesifik bir kitap) bulmak istediğimizde nokta atışı yapmak için kullanılır.
	
- **where:** Sorguya arama filtreleri eklememizi sağlar
  
```python
result = s.scalar(select(Book).where(Book.title == "The Great Gatsby"))
print(result)
```

	s.scalars(query).all() s.execute.scalar().all 'a göre daha modern bir yazımdır.'

# U (Update)

ORM (Nesne-İlişkisel Eşleme) felsefesinde güncelleme işlemi için SQL'deki gibi `UPDATE ... SET` komutları yazılmaz. Veritabanıyla değil, RAM'deki Python nesneleriyle konuşulur. SQLAlchemy'nin **Unit of Work (İş Birimi)** mekanizması RAM'e çekilen nesneleri izler ve bir değişim gördüğünde SQL komutunu arka planda kendisi oluşturur.

```Python
with session() as s:
    # 1. Nesneyi veritabanından çekip RAM'e al (SQLAlchemy izlemeye başlar)
    guncellenecek_kitap = s.scalar(select(Book).where(Book.title == "The Great Gatsby"))
    
    # 2. Özelliği standart bir Python değişkeni gibi değiştir
    guncellenecek_kitap.page_count = 200
    
    # 3. Onayla. SQLAlchemy değişimi fark edip UPDATE sorgusunu yollar.
    s.commit()
```

# D (Delete)

Silme işleminde en büyük performans tuzağı, silinecek veriyi önce RAM'e çekip (select) sonra silmektir. Endüstri standardı olan **Toplu Silme (Bulk Delete)** yönteminde, veriler RAM'e hiç çekilmeden doğrudan veritabanına `execute` ile silme emri yollanır.

_(Not: `delete` fonksiyonunu kullanmak için kodun başına `from sqlalchemy import delete` eklenmelidir.)_

```Python
with session() as s:
    # Veriyi RAM'e çekmeden doğrudan SQL'e "şu şarta uyanları sil" emri gönder
    s.execute(delete(Book).where(Book.title == "The Great Gatsby"))
    s.commit()

# TEHLİKE BÖLGESİ: Tüm Tabloyu Boşaltmak
# Eğer where() filtresi konulmazsa, tablodaki tüm veriler tek hamlede silinir:
# s.execute(delete(Book)) 
```


# **İlişkiler (Relationships) ve JOIN Mantığı** 

İlişkisel veritabanlarında tablolar `FOREIGN KEY` ile birbirine bağlanır. Ancak standart SQL'de iki tablonun verisini aynı anda görmek için karmaşık `JOIN` komutları yazmak gerekir. SQLAlchemy'deki `relationship()` modülü, bu `JOIN` ameleliğini ortadan kaldırır.

- Python sınıflarının (örneğin `Book` ve `ReadLog`) içine yerleştirilen çift yönlü bir köprüdür.
    
- **`back_populates`**: Köprünün iki ucunu şeffaf bir şekilde birbirine bağlar.
    
- Bu sayede sen bir kitabı `scalar()` ile RAM'e çektiğinde, o kitabın okuma geçmişi (`kitap.logs`) otomatik olarak Python listesi şeklinde o nesnenin içine gömülü olarak gelir. Tekrar tekrar SQL sorgusu yazmana gerek kalmaz.
 
 logs = relationship('ReadLog', back_populates='book')


# Sorgu Tipleri ve Karmaşıklık Analizi

İlişkili verilerin (örneğin kitabın loglarının) ne zaman çekileceğini belirler:

| **Yöntem**                    | **Mantık**                                                                                                                           | **SQL Etkisi**                                                                 | **Big O**               |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------ | ----------------------- |
| **Lazy Load** (Varsayılan) 😴 | Sadece ihtiyaç duyulduğunda (nokta operatörü ile çağrılınca) veriyi çeker.                                                           | N+1 sorgu problemi yaratabilir.                                                | $O(1)$ veya $O(\log n)$ |
| **Joined Load** 🚀            | Ana tablo ile ilişkiyi `LEFT OUTER JOIN` yaparak tek seferde çeker.                                                                  | Tek sorguyla her şeyi bitirir.                                                 | $O(n)$                  |
| **Subquery Load** 📑          | Önce ana tabloyu çeker, sonra tüm ilişkili verileri ikinci bir sorguyla (IN operatörü kullanarak) topluca çeker.                     | Çok büyük veri setlerinde JOIN'den daha performanslı olabilir.                 | $O(n + m)$              |
| **Select In Load** 📥         | Subquery load'a benzer ama daha modern bir yöntemdir. İlişkili ID'leri toplayıp tek bir `SELECT ... WHERE id IN (...)` sorgusu atar. | SQLAlchemy'nin modern sürümlerinde koleksiyonlar için en çok önerilen hızlı yö | $O(n)$                  |

### Joinedload ile sorgu için ufak bir örnek:

```Python
with session() as s:
    find_rating_with_book = (
        select(ReadLog)
        .where(ReadLog.rating > 7)
        .order_by(ReadLog.rating.desc())
        .limit(3)
        ).options(joinedload(ReadLog.book))
        
    result = s.scalars(find_rating_with_book).all()

    for log in result:
        print(f"Okuma Tarihi: {log.read_date}, Puan: {log.rating}, Durum: {log.status}")
        print(f"Kitap: {log.book.title} - {log.book.author} ({log.book.genre})")
```
#  Migration Nedir?

Veritabanı şeması değişikliklerini (tablo ekleme, kolon silme, index oluşturma, foreign key ekleme vb.) versiyonlanabilir hale getirmektir. Bu sayede takımda herkes aynı veritabanı yapısını kullanır, değişiklikler kolayca ileri(upgrade) ya da geri(downgrade) alınabilir. Manuel olarak SQL dosyaları ile uğraşmaya gerek kalmaz.

#  [[Alembic]] 


## Sorgu Örnekleri

Kitapların türüne göre kaç sayfa okuduğunu ve ortalama kaç rating verdiğini gösteren bir sorgu örneği:

```Python 
with SessionLocal() as s:
    query = (
        select(Book.genre, func.avg(ReadLog.rating), func.sum(ReadLog.read_pages))
        .join(ReadLog)
        .group_by(Book.genre)
        .order_by(func.avg(ReadLog.rating).desc())
    )
    
    results = s.execute(query)
    for genre, avg_rating, total_pages in results:
        print(
            f"Genre: {genre}, Average Rating: {int(avg_rating)}, Total Pages Read: {total_pages}"
        )
```


Bütün kitapların ortalamasını alıp ortalamadan büyük olan kitapları veren bir sorgu örneği:

```Python
with SessionLocal() as s:
    query = select(func.avg(Book.page_count)).scalar_subquery()
    result = s.execute(select(Book).where(Book.page_count > query)).scalars().all()
    
    for book in result:
        print(f"{book.title} - {book.page_count} pages")
```


