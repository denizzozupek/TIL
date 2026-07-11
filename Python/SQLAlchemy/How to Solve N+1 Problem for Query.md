
>[!info] 'selectinload' SQLAlchemy'de ilişkili veriyi optimize edilmiş bir sorguyla yüklemek için kullanılır.

selectinload olmasaydı for log in logs ile tekrar tekrar sorgu atmak zorunda kalırdık.

Örneğin:

```python
    query = (
        select(ReadLog)
        .join(Book)
        .options(selectinload(ReadLog.book))
```
