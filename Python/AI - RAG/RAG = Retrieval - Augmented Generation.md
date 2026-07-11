
RAG, LLM'lerin dışarıdan verilen verilerden bilgi çekerek daha doğru ve güvenilir yanıtlar üretmesini sağlayan bir tekniktir. Bu sayede güncelliği yitirilmiş halüsinasyon ihtimali yüksek bilgileri, fine-tuning gibi büyük ölçekli bir işlem yerine, direkt modele verilerek halüsinasyon ihtimali azaltılır. 

### Nasıl Çalışır?

Süreç temel olarak dört adımdan oluşur:

1. **Veri Hazırlama:** Şirket içi PDF'ler, Word dosyaları veya web siteleri gibi veriler küçük metin parçalarına (chunk) bölünür.

2. **Vektörleştirme ([[Vector embedding]]):** Bu metin parçaları matematiksel sayılara dönüştürülerek bir vektör veritabanına kaydedilir.

3. **Bilgi Çekme (Retrieval):** Kullanıcı bir soru sorduğunda, model veritabanına gider ve en alakalı bilgi parçalarını arar.

4. **Yanıt Üretimi:** Yapay zeka, kullanıcının sorusunu ve veritabanından çektiği özel bilgileri birleştirerek tamamen gerçeğe dayalı bir cevap üretir.

![[Pasted image 20260705191134.jpg]]


RAG başarısı, contextin doğru olmasına, chunk kalitesine ve llm in contexti anlama becerisine bağlıdır. 