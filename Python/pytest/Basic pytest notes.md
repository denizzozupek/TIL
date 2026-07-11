
Pytest fixture'ları, ==testlerin çalıştırılmasından önce veya sonra veritabanı bağlantıları, sahte nesneler (mock) veya test verileri gibi ön koşulları hazırlayan, yeniden kullanılabilir fonksiyonlardır==. `@pytest.fixture` dekoratörü ile tanımlanırlar ve test fonksiyonlarına argüman olarak geçirilerek sürdürülebilir, temiz ve modüler test kodları yazılmasını sağlarlar. [](https://translate.google.com/translate?u=https://docs.pytest.org/en/stable/explanation/fixtures.html&hl=tr&sl=en&tl=tr&client=sge)

**Pytest Fixture Özellikleri ve Kullanım Örnekleri**

- **Ön/Son İşlemler:** Test öncesi kurulum (setup) ve test sonrası temizlik (teardown) işlemlerini otomatikleştirirler.
- **Veri Sağlama:** Testlere sabit veya dinamik veriler iletir.
- **Kapsam (Scope):** Fixture'ların ne sıklıkla çalışacağını belirler (fonksiyon, sınıf, modül veya oturum bazlı).