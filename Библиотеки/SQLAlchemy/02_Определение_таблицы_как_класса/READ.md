# Определяем таблицу как класс
При работе с SQLAlchemy ORM структура таблицы базы данных определяется через Python-класс. Такой класс, именуемый моделью, наследуется от базового класса Base (или DeclarativeBase) и содержит описание таблицы: её название, а также перечень столбцов с указанием их типов.

Чтобы SQLAlchemy корректно распознал ваш класс как представление таблицы в БД, необходимо выполнить три ключевых шага:  
- Сделать класс наследником от Base (который вы создали ранее).  
- Прописать для класса атрибут `__tablename__`, указывающий на реальное имя таблицы в базе данных.  
- Определить все столбцы, используя аннотации типов в сочетании с функцией `mapped_column(...)`.  

Пример:
```python
# импортируем необходимые компоненты
from sqlalchemy import create_engine, String, Integer, JSON, text
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, Session

# создаем движок
engine = create_engine("sqlite:///C:/Users/user/AppData/Roaming/DBeaverData/workspace6/.metadata/sample-database-sqlite-1/Chinook.db")

# базовый класс, от которого наследуются все модели
class Base (DeclarativeBase):
    pass

# Любая таблица, должна быть наследником Base:
class Album(Base):  
    __tablename__ = "Album"

    # mapped_column() — функция для связи аннотации с полем бд
    AlbumId: Mapped[int] = mapped_column(Integer, primary_key=True)
    Title: Mapped[str] = mapped_column(String(160))
    ArtistId: Mapped[int] = mapped_column(Integer)
```