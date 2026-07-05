**Tarih:** 14.05.2026
**Etiketler:** #embedding #vectors #wordembedding

## Neden kelimeleri vektöre çeviriyoruz?

Bilgisayarlar doğrudan insan dilini anlayamaz; bu yüzden dili nümerik forma (vektör) dönüştürürüz. Vektör uzayında benzer anlamlı kelimeler birbirine yakın konumlanır (ör. `kedi` ile `köpek` yakın, `masa` daha uzak). Bu uzaklık/benzerlik ölçüsü sayesinde arama, kümeleme veya semantic similarity gibi işlemler yapılabilir.

Kelimeleri vektöre dönüştüren modeller: Word2Vec, GloVe, FastText, Transformer tabanlı embedding modelleri (OpenAI, Hugging Face vb.). Farklı modeller aynı kelime için farklı vektörler üretebilir — model ve eğitim verisi çıktıyı etkiler.

Kısa not:
- Embedding kullanırken normalize etme, cosine similarity ve vektör boyutu gibi konular performansı etkiler.