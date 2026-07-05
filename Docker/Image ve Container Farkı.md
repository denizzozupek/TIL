
>[!info] Docker image, bir uygulamanın çalışması için gerekli kod, kütüphane ve ayarları içerenread-only bir şablon iken; container, bu image'ın çalışan, aktif ve değiştirilebilir örneğidir
[!info] Docker image, bir uygulamanın çalışması için gerekli kod, kütüphane ve ayarları içeren read-only bir şablon iken; container, bu image'ın çalışan, aktif ve değiştirilebilir örneğidir.

**Docker Image ve Container Arasındaki Temel Farklar**

- **Yapı:** Image'lar katmanlı (layered) ve statik dosyalardır; container'lar ise bu katmanların üzerinde oluşturulan yazılabilir (writable) bir katmana sahip aktif süreçlerdir.

- **Durum:** Image'lar değiştirilemez (immutable), container'lar ise anlık olarak durumunu değiştirebilir.

- **İlişki:** Bir image'dan (şablon) binlerce farklı container (örnek) türetilebilir.

- **Kullanım:** Image'lar `docker build` ile oluşturulur ve Docker Hub gibi yerlerde saklanır; container'lar ise `docker run` ile çalıştırılır.

- **Ömür:** Container silindiğinde, içerisindeki geçici veriler kaybolur ancak onu oluşturan image var kalmaya devam eder.

### Bind mounts

Geliştirme sırasında kullanılır; yerel bir klasörü konteynerin içine bağlar (`-v /local/path:/app`), böylece kod değişiklikleri anında konteynerde görünür.

### Docker volumes

Veri kalıcılığı için kullanılır (örn. PostgreSQL verileri). Container silinse bile volume içindeki veriler korunur ve başka bir container tarafından yeniden kullanılabilir.

Kısa notlar:

- Production'da bind mount yerine image build + CI/CD tercih edin.
- `docker-compose` ile volume tanımları kolayca yönetilebilir.