**Tarih:** 14.07.2026
**Etiketler:** #RAG #numpy #optimization #psycopg 

---

1) psycopg veritabanına extension kuracaksak cursor açmamıza gerek yok, conn.execute ile halledilebilir. Eğer geriye bir veri dönecekse cursor açılır.
2) Eğer embeddingle matematiksel işlem yapılacaksa numpy ile array olarak embedding yapılır.
3) nparrayda dtype = float32 yapmak veri sayısı arttıkça sistemin darboğaz yapmasını engeller. LLM float64 neredeyse hiç kullanmaz.
4) 
