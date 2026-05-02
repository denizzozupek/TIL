
**Kısa Özet:** Tarayıcıların (Chrome, Safari vb.) güvenlik mekanizmasıdır. API'mizin (Backend) bulunduğu adres ile, web sitemizin (Frontend) bulunduğu adres birbirinden farklıysa, tarayıcı araya girip veri alışverişini güvenlik gerekçesiyle engeller. CORS Middleware, API'mizin tarayıcıya "ben bu adresi tanıyorum, geçişine izin ver" dediği vize belgesidir.


```python
app.add_middleware(
    CORSMiddleware,
    # 1. KİMLER GELEBİLİR? (allow_origins)
    # Sınır kapısından hangi ülkelerin (domainlerin) geçebileceğini belirtiriz.
    # ["*"] demek: "Dünyadaki tüm web siteleri benim API'me istek atabilir." (Geliştirme aşamasında kullanılır, canlıda ASLA kullanılmaz!)
    # Canlıda (Production) doğrusu: ["https://benim-react-sitem.com"] olmalıdır.
    allow_origins=["*"],

    # 2. KİMLİK KARTI/PASAPORT KONTROLÜ (allow_credentials)
    # Gelen isteklerin içinde çerezler (cookies) veya giriş yapmış kullanıcının yetki token'ları (kimlikleri) taşınabilir mi?
    # True diyerek "Evet, yetkilendirme belgelerini getirebilirler" diyoruz.
    allow_credentials=True,

    # 3. HANGİ İŞLEMLERİ YAPABİLİRLER? (allow_methods)
    # Gümrükten geçen kişinin içeride ne yapacağına izin veriyoruz? Sadece bakıp çıkacak mı (GET), yoksa yeni mal mı indirecek (POST/DELETE)?
    # ["*"] demek: "GET, POST, PUT, DELETE... Aklına gelen tüm HTTP komutlarını kullanabilir."
    allow_methods=["*"],

    # 4. YANLARINDA HANGİ PAKETLERİ GETİREBİLİRLER? (allow_headers)
    # İstek atılırken HTTP başlıklarında (Headers) ekstra bilgiler (Örn: Content-Type, Authorization, özel keyler) gönderilir.
    # ["*"] demek: "İstedikleri tüm başlık bilgilerini kargoya koyup gönderebilirler, kısıtlama yok."
    allow_headers=["*"],
)
```

