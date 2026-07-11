**Tarih:** 14.05.2026
**Etiketler:** #pgvector 

<=>: cosine similarity
<->: euclidean distance (L2)

>[!info] Cosine similarity is calculated as 1 – cosine_distance

```python
SELECT 1 - (embedding <=> '[3,1,2]') AS cosine_similarity FROM items;
```
