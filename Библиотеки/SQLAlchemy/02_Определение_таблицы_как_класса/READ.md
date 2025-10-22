# Определяем таблицу как класс

При работе с SQLAlchemy ORM структура таблицы базы данных определяется через Python-класс. Такой класс, именуемый моделью, наследуется от базового класса Base (или DeclarativeBase) и содержит описание таблицы: её название, а также перечень столбцов с указанием их типов.

Чтобы SQLAlchemy корректно распознал ваш класс как представление таблицы в БД, необходимо выполнить три ключевых шага:  
- Сделать класс наследником от Base (который вы создали ранее).  
- Прописать для класса атрибут `__tablename__`, указывающий на реальное имя таблицы в базе данных.  
- Определить все столбцы, используя аннотации типов в сочетании с функцией `mapped_column(...)`.  

## Пример:

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

Что здесь происходит:  
- **Album** — это Python-класс, представляющий таблицу Album.  
- **AlbumId** — это целочисленный столбец с флагом primary_key=True, то есть это уникальный идентификатор строки.  
- **Title** — это строковое поле длиной до 160 символов.  
- **ArtistId** — это целочисленный столбец для связи с другими таблицами.  

## Синтаксис

- Аннотация `AlbumId: Mapped[int]` говорит: поле **AlbumId** — в БД это поле типа `int`.  
- Функция `mapped_column(...)` свяжет поле с типом поля в БД (Integer, String, DateTime, и т.д.) и дополнительнйо информацией по полю (`primary_key`, `nullable`, `default` и т.д.).

## Типы полей

| Тип данных | Назначение | Примеры использования |
|------------|------------|----------------------|
| **Integer** | Целочисленные значения | ID записей, возраст, количество, рейтинг |
| **String** | Текстовые строки переменной длины | Имя, email, логин, название |
| **Boolean** | Логические значения (True/False) | Флаги активности, статусы, подтверждения |
| **DateTime** | Дата и время | Время создания, дата события, последнее обновление |