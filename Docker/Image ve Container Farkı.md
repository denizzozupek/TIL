
>[!info] Docker image, bir uygulamanın çalışması için gerekli kod, kütüphane ve ayarları içerenread-only bir şablon iken; container, bu image'ın çalışan, aktif ve değiştirilebilir örneğidir

**Docker Image ve Container Arasındaki Temel Farklar**

- **Yapı:** Image'lar katmanlı (layered) ve statik dosyalardır; container'lar ise bu katmanların üzerinde oluşturulan yazılabilir (writable) bir katmana sahip aktif süreçlerdir.

- **Durum:** Image'lar değiştirilemez (immutable), container'lar ise anlık olarak durumunu değiştirebilir.

- **İlişki:** Bir image'dan (şablon) binlerce farklı container (örnek) türetilebilir.

- **Kullanım:** Image'lar `docker build` ile oluşturulur ve Docker Hub gibi yerlerde saklanır; container'lar ise `docker run` ile çalıştırılır.

- **Ömür:** Container silindiğinde, içerisindeki geçici veriler kaybolur ancak onu oluşturan image var kalmaya devam eder.

### Bind Mounts

Sadece Developement aşamasında kullanılır. Copy'yi es geçip kopyalamadan spesifik bir klasörü direkt Docker'a bağlar.

### Docker Volume

PostgreSQL gibi veritabanlarının verilerini, docker tarafından sağlanan, container silinse bile verileri saklar. Ayrıca birden fazla container'ın veriyi kullanabilmesini sağlar.