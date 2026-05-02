# Alembic Nedir?

>[!info] Alembic, Python ekosisteminde kullanılan en popüler migration aracıdır. Genelde SQAlchemy ORM ile birlikte çalışır.

Özellikleri:

- Migration dosyalarını otomatik oluşturur.
- SQL yerine Python kodu yazarak şema değişikliklerini yönetirsin.
- upgrade ve downgrade komutlarıyla ileri/geri hareket mümkündür.


## Kullanımı

1) İlk olarak paket indirilir ve kurulur. 
	pip install alembic

2) Ardından alembic başlatılır
	alembic init alembic

3) alembic dosyasında açılan env.py dosyasına database import edilir ardından target_metadata = Base.metadata olarak değiştirilir

4) alembic.ini dosyasının içindeki sqlalchemy.orm urlsi bizim kendi database urlmiz ile değiştirilir.

5)  **Migration Oluşturma**

Aşağıdaki komutla o an kodlarla oluşturulan veri tabanı şema bilgisi migration olarak kaydedilir.

	alembic revision --autogenerate -m "create users table"

6) **Geri Alma(Downgrade)**

Son yaptığın değişiklik eğer yanlışsa ve geri almak istiyorsan:

	alembic downgrade -1

### head, base ve -1 nedir?

- **head:** En son migration dosyasını temsil eder. alembic upgrade head dendiğinde tüm migrationları sırasıyla çalıştırır ve en güncele(head) ulaşır.
- **base:** En başlangıç noktasıdır. Yani migration uygulanmamış halidir.

alembic downgrade base