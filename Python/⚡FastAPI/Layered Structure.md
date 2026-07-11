
>  Explicit is better than implicit. 

Sorumlulukların ayrılması prensibine göre, dosya büyüdükçe kontrolün azalmasını önlemek için katmanlı mimari kullanılır.

# FastAPI + SQLAlchemy Proje Yapısı

```python
READING_LOGS/                  # Projenin Kök Dizini (Root Directory)
│
├── app/                       # Core
│   ├── __init__.py            # Python'a "app" klasörünün bir paket olduğunu söyler.
│   ├── main.py                # Recevies HTTP requests, hands them over worker(crud.py)
│   ├── database.py            # Database engine and session.
│   ├── models.py              # Database Tables.
│   ├── schemas.py             # View Model - Data Transfer Object (pydantic etc.)
│   └── crud.py                # SQL queries are written here. Seperates SQL/ORM complexity from main.py.
│
├── alembic/                   # Database Migrations
├── alembic.ini                # Alembic conf.
│
├── venv/                      # Virtual Environment
│
├── .env                       # Passwords - API key etc. for .gitignore
└── .gitignore                 
```

models.py : Data Layer
schemas.py : Data Transfer Object
crud.py : Data Access Layer 
main.py : Controllers / Routers
