**Tarih:** 24.06.2026
**Etiketler:** #embedding #vectors #wordembedding #vectorembedding

When dealing with vectors, a common way to calculate a similarity score is [cosine_similarity](https://en.wikipedia.org/wiki/Cosine_similarity)

Okumalarımı yaparken kullandığım kaynaklar:  https://jalammar.github.io/illustrated-word2vec/

Vector embedding, insanlar tarafından algılanan anlamsal benzerliği, bir vektör uzayındaki yakınlık olarak çevirmeyi mümkün kılar.

Vector embedding hakkında en çok fayda sağlayacağım kaynak: https://developers.openai.com/api/docs/guides/embeddings?lang=python

text-embedding-3-small ile vektöre çevirip numpy ile kosinüs benzerliği hesaplamak için örnek kod: 

```python
from openai import OpenAI
import numpy as np
import os
from dotenv import load_dotenv
  
load_dotenv()

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
 

def get_embedding(text: str, model: str= "text-embedding-3-small") -> np.ndarray:
    response = client.embeddings.create(
        model=model,
        input=text
    )

    return np.array(response.data[0].embedding)

def cosine_similarity(vec1: np.ndarray, vec2: np.ndarray) -> float:

  
    dot_product = np.dot(vec1, vec2)
    norm_vec1 = np.linalg.norm(vec1)
    norm_vec2 = np.linalg.norm(vec2)

    return dot_product / (norm_vec1 * norm_vec2)

cümleler = {

    "Kitap": "Cal Newport'un Pürdikkat kitabı odaklanma ve derin çalışma üzerine önemli bilgiler sunuyor.",

    "Kitap 2": "Mark Manson'un Ustalık Gerektiren Kafaya Takmama Sanatı kitabı, odaklanma ve zihinsel rahatlama konularında rehberlik ediyor.",

    "Kodlama": "RAG (Retrieval-Augmented Generation) yaklaşımı, bilgiye dayalı içerik üretiminde etkili bir yöntem olarak öne çıkıyor.",

    "Tarih": "Julius Caesar, Roma İmparatorluğu'nun önemli bir lideri olarak tarihe damgasını vurmuştur.",

    "Alakasız": "Bugün Brezilya - İskoçya futbol maçı var ve hava çok güzel."

}  

print("Vectors are ready to be used for similarity calculations....")
vectors = {key: get_embedding(value) for key, value in cümleler.items()}

print("Similarity Scores:")

benzerlik_kitap = cosine_similarity(vectors["Kitap"], vectors["Kitap 2"])
benzerlik_kodlama = cosine_similarity(vectors["Kitap"], vectors["Kodlama"])
benzerlik_tarih = cosine_similarity(vectors["Kitap"], vectors["Tarih"])
benzerlik_alakasız = cosine_similarity(vectors["Kitap"], vectors["Alakasız"])
  

print(f"Kitap ve Kitap 2 arasındaki benzerlik: {benzerlik_kitap:.4f}")
print(f"Kitap ve Kodlama arasındaki benzerlik: {benzerlik_kodlama:.4f}")
print(f"Kitap ve Tarih arasındaki benzerlik: {benzerlik_tarih:.4f}")
print(f"Kitap ve Alakasız arasındaki benzerlik: {benzerlik_alakasız:.4f}")
```