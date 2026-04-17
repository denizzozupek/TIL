**Tarih:** 06.03.2026
**Etiketler:** #python #sqlite3 #database #AsyncVideoSummarizerProject

## 📌 Nedir?
Python'un içine gömülü (built-in) olarak gelen, hafif, sunucusuz (serverless) ve kendi kendine yeten (self-contained) bir veritabanı modülüdür. Veritabanı tek bir dosya (`.db`) veya doğrudan RAM üzerinde (`:memory:`) yaşayabilir.

**Kullanım Alanları:**
- Uygulama verilerini yerel olarak saklamak.
- Hızlıca SQL sorguları prototiplemek.
- Kurulum gerektirmeyen, taşınabilir veri depolama çözümleri üretmek.

---

## 🔑 Temel Sınıflar ve Fonksiyonlar

| Sınıf / Fonksiyon       | Ne İşe Yarar?                                                                      |
| :---------------------- | :--------------------------------------------------------------------------------- |
| `sqlite3.connect()`     | Veritabanına bağlanır (veya yoksa yeni dosya oluşturur).                           |
| `sqlite3.Connection`    | Veritabanı bağlantısını temsil eder. (Deponun kapısı)                              |
| `sqlite3.Cursor`        | Sorguları çalıştırır ve sonuçları getirir. (Depodaki kurye)                        |
| `sqlite3.Row`           | Sorgu sonuçlarına sözlük (dictionary) gibi erişmeyi sağlar.                        |
| `sqlite3.Error`         | Tüm SQLite hatalarını yakalamak için temel hata (exception) sınıfı.                |
| `sqlite3.executemany()` | Aynı SQL komutunu bir liste/demet içindeki çoklu verilerle tek seferde çalıştırır. |

---

## 🛠️ Hızlı Kopya Kağıdı (Cheatsheet)

### 1. Bağlantı ve Tablo Oluşturma
	`with` bloğu kullanmak (Context Manager), işlem bitince bağlantının otomatik olarak kapanmasını ve kaynakların temizlenmesini sağlar.

```python
import sqlite3

# with bloğu ile güvenli bağlantı (dosya yoksa yaratılır)
with sqlite3.connect("app.db") as connection:
    cursor = connection.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY, 
            name TEXT
        )
    """)
    connection.commit() # Değişiklikleri kaydet
````

### 2. Veri Ekleme (SQL Injection'dan Korunarak!)

> [!WARNING] Kritik Güvenlik Kuralı
> 
> Asla f-string veya string birleştirme ile SQL yazma! Her zaman `?` parametresini (placeholder) kullan.



```python
with sqlite3.connect("app.db") as connection:
    cursor = connection.cursor()
    # Tekli veri ekleme
    cursor.execute("INSERT INTO users (name) VALUES (?)", ("Alice",))
    
    # Çoklu veri ekleme (executemany)
    users_list = [("Bob",), ("Carol",), ("Dave",)]
    cursor.executemany("INSERT INTO users (name) VALUES (?)", users_list)
    
    connection.commit()
```

### 3. Veri Okuma (SELECT)


```python
with sqlite3.connect("app.db") as connection:
    cursor = connection.cursor()
    cursor.execute("SELECT * FROM users")
    
    sonuclar = cursor.fetchall()
    print(sonuclar) # Çıktı: [(1, 'Alice'), (2, 'Bob'), ...]
```

### 4. Hata Yönetimi (Rollback)

Eğer bir işlem sırasında hata çıkarsa, veritabanının bozuk kalmaması için işlemi geri almak (`rollback`) gerekir.


```python
try:
    connection = sqlite3.connect("app.db")
    cursor = connection.cursor()
    # Kasıtlı hatalı bir işlem
    cursor.execute("INSERT INTO users (name) VALUES (?)", (None,))
    connection.commit()
except sqlite3.Error as e:
    print(f"Bir hata oluştu: {e}")
    connection.rollback() # Yapılan tüm işlemleri geri al
finally:
    connection.close() # with kullanılmadığı için manuel kapat

````
---

## 🧠 Senior Mimari Notları (Gerçek Dünya Senaryoları)

### 5. Okuma (Read) vs Yazma (Write) Kuralları
Veritabanı işlemleri ikiye ayrılır. İşlem tipine göre `cursor` ve `commit` kullanımı değişir:

* **Okuma (SELECT):** * Depodan paket almak gibidir. Mecburen kuryeyi ismen atamalısın (`cursor = conn.cursor()`).
  * Gelen paketi havada yakalamak için `fetchone()` (ilk kaydı getirir) veya `fetchall()` (tüm kayıtları liste döner) kullanılır.
  * 🚨 **Kural:** Okuma işlemlerinde `conn.commit()` **YAPILMAZ** (Çünkü depoda bir şey değiştirmedik).

* **Yazma / Değiştirme (INSERT, UPDATE, DELETE):**
  * Depoya emir vermek gibidir. Kuryeyi ismen atamana (`cursor = ...`) gerek yoktur. Doğrudan `conn.execute(...)` ile kestirmeden vurup geçebilirsin.
  * 🚨 **Kural:** İşlem sonunda değişiklikleri kalıcı yapmak için mecburen `conn.commit()` **YAPILMALIDIR**. Aksi halde veriler uçup gider.

```python
# 🟢 OKUMA (SELECT) ÖRNEĞİ: Cursor şart, Commit yok.
cursor = conn.cursor()
cursor.execute("SELECT status FROM summaries WHERE video_id = ?", (v_id,))
kayit = cursor.fetchone() 

# 🔴 YAZMA (UPDATE) ÖRNEĞİ: Cursor'a gerek yok, Commit şart!
conn.execute("UPDATE summaries SET status = 'completed' WHERE video_id = ?", (v_id,))
conn.commit()
````

### 6. Asenkron Mimaride "Vur-Kaç" Taktiği (Database Locking Önlemi)

Asenkron (aynı anda çok iş yapan) bir FastAPI veya arkaplan (worker) sisteminde, veritabanı bağlantısını gereksiz yere uzun süre açık tutmak en büyük mimari hatadır. Eğer bağlantı açıkken uzun süren bir API isteği (örn: Gemini/OpenAI) yaparsan, veritabanı "rehin alınır" ve dışarıdan gelen diğer isteklere şu hatayı fırlatır:

> `sqlite3.OperationalError: database is locked`

**Doğru Mimari (Vur-Kaç):**

1. Bağlantıyı Aç (`with sqlite3.connect...`)
    
2. Hızlıca kaydı yap/güncelle.
    
3. Bağlantıyı hemen Kapat!
    
4. Uzun süren API / Yapay Zeka işlemini **veritabanı kapalıyken** (dışarıda) yap.
    
5. Cevap gelince tekrar Bağlan, yaz ve Kapat.
    

``` Python
# YANLIŞ MİMARİ (Sistemi Kilitler):
with sqlite3.connect("app.db") as conn:
    conn.execute("...") # Kayıt açıldı
    # ❌ Veritabanı açıkken 15 saniye LLM cevabı bekleniyor! Sistem kilitlendi!
    summary = await get_llm_summary() 
    conn.execute("...", (summary,))

# DOĞRU MİMARİ (Vur-Kaç):
with sqlite3.connect("app.db") as conn:
    conn.execute("...") # Durumu işleniyor yap
    conn.commit() # Ve hemen KAPAN!

# ✅ Veritabanı şu an boşta, diğer müşteriler girebilir. LLM rahatça beklenebilir.
summary = await get_llm_summary()

with sqlite3.connect("app.db") as conn:
    conn.execute("...", (summary,)) # Tekrar gir, kaydet
    conn.commit() # Ve KAPAN!
```
