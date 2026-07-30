RAG stands for Retrieval-Augmented Generation. It’s a way to build LLM applications that can answer questions using external data – like product docs, support content, or internal knowledge bases – instead of relying only on what the model saw during training.

RAG works in two steps:

- First, the system retrieves relevant information – that’s the “R.”
- Then, it uses that information to generate the answer – that’s the “G.”

[source](https://www.evidentlyai.com/llm-guide/rag-evaluation#what-is-rag-evaluation)

### Nasıl Çalışır?

Süreç temel olarak dört adımdan oluşur:

1. **Veri Hazırlama:** Şirket içi PDF'ler, Word dosyaları veya web siteleri gibi veriler küçük metin parçalarına (chunk) bölünür.

2. **Vektörleştirme ([[Vector embedding]]):** Bu metin parçaları matematiksel sayılara dönüştürülerek bir vektör veritabanına kaydedilir.

3. **Bilgi Çekme (Retrieval):** Kullanıcı bir soru sorduğunda, model veritabanına gider ve en alakalı bilgi parçalarını arar.

4. **Yanıt Üretimi:** Yapay zeka, kullanıcının sorusunu ve veritabanından çektiği özel bilgileri birleştirerek tamamen gerçeğe dayalı bir cevap üretir.

Görselleştirme için : https://gradientflow.com/techniques-challenges-and-future-of-augmented-language-models/

RAG başarısı, contextin doğru olmasına, chunk kalitesine ve llm in contexti anlama becerisine bağlıdır. 

Retrieval optimizasyonu olarak vektör indexleme kullanılır. Vektör indexleme embeddingleri verimli bir şekilde optimize ederek arama doğruluğunu minimum kayıpla hızlandırmayı amaçlar. 
1) Hnsw İndex: Doğruluğu yüksek fakat veri arttıkça RAM kullanımı da artar. Bellek kullanımını düşürmek için [vektör niceleme](https://qdrant.tech/course/essentials/day-2/what-is-hnsw/) (scalar veya binary quantization) yöntemleri uygulanabilir
2) Flat Search: İndexsiz standart aramadır. Ram kullanımı sıfırdır fakat veri büyüdükçe arama hızı azalır.
3) IVFFlat: Sorgu hızı ve doğruluğu hnswye göre düşüktür.  Ram kullanımı da hnswye göre düşüktür. Veri güncelleme esnekliği de zayıftır. 