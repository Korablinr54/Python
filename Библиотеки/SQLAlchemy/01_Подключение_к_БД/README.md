# Подключение в БД

📚 Ключевые термины

- **Engine** — основной компонент для установки и управления подключениями к СУБД
- **Session** — рабочая область, где происходит оперативное взаимодействие с базой данных
- **flush()** — синхронизирует изменения с БД без фиксации транзакции
- **commit()** — окончательно сохраняет все изменения в базе данных
- **rollback()** — отменяет текущую транзакцию и все её изменения

## Создание движка

Перед тем как начать работать с базой с ней нужно установить соединение. Для этого используется объект `engine`, для создания которого используется функция `create_engine`. Базовый синтаксис:  
```python
engine = create_engine("dialect+driver://user:password@host:port/dbname")
```  

**SQLAlchemy** предоставляет унифицированный API, но для работы с конкретными СУБД требуются соответствующие драйверы.  

Например, создадим подключение к локальной БД Postgres:  
```python
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg2://postgres:postgres@localhost/postgres")
```

Еще примеры:
```python
# SQLite (файл в текущей папке):
create_engine("sqlite:///example.db")
# SQLite (абсолютный путь):
create_engine("sqlite:////home/user/example.db")
                  
# PostgreSQL:
create_engine("postgresql+psycopg2://postgres:1234@localhost/mydb")
                  
# MySQL:
create_engine("mysql+pymysql://user:pass@localhost:3306/mydb")
```

# Сессия

**Сессия** - это объект, через который мы взаимодествуем в базе. она как переводчик между Python и SQL.  

Сессии являются центральным элементом взаимодействия с БД в **SQLAlchemy**, обеспечивая безопасное и эффективное управление соединениями и транзакциями.  

Рекомендуемым синтаксисом является подход with-блок:
```python
from sqlalchemy.orm import Session

with Session(engine) as session:
    # Здесь выполняются действия с базой
    ...
```

**Плюсы такого подхода**:  
- **Автоматическое закрытие**: Как только работа в блоке завершается, сессия сразу же закрывается без вашего участия.  
- **Безопасность при ошибках**: В случае любой ошибки SQLAlchemy автоматически откатывает все изменения, чтобы сохранить целостность базы данных.  
- **Читаемость кода**: Чётко видно границы работы с базой — от начала до конца, что повышает надёжность и удобство чтения кода.  

# Базовые операции

Базовыми операциями являются операции **CRUD** (Create, Update, Read, Delete).  

# Практический пример
Разберем пример для таблицы:
```sql
CREATE TABLE [Album]
(
    [AlbumId] INTEGER  NOT NULL,
    [Title] NVARCHAR(160)  NOT NULL,
    [ArtistId] INTEGER  NOT NULL,
    CONSTRAINT [PK_Album] PRIMARY KEY  ([AlbumId]),
    FOREIGN KEY ([ArtistId]) REFERENCES [Artist] ([ArtistId]) 
		ON DELETE NO ACTION ON UPDATE NO ACTION
);
```

## Импорт необходимых компонентов
```python
from sqlalchemy import create_engine, String, Integer, JSON, text
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, Session
```

## Создание подключения к БД
После импорта необходимых компонентов нужно настроить подключение в базе.  
```python
# создаем движок
engine = create_engine("sqlite:///C:/Users/user/AppData/Roaming/DBeaverData/workspace6/.metadata/sample-database-sqlite-1/Chinook.db")
```

## Создание базового класса
Что дает наследование от **Base**:  
- **Общие метаданные** - все таблицы в одном месте  
- **Авторегистрация** - модели автоматически регистрируются  
- **Единая точка управления** - можно добавить общую логику  

```python
# базовый класс, от которого наследуются все модели
class Base (DeclarativeBase):
    pass
```

## Объявление класса + метаданные
```python
# Любая таблица, должна быть наследником Base:
class Album(Base):  
    __tablename__ = "Album"

    # mapped_column() — функция для связи аннотации с полем бд
    AlbumId: Mapped[int] = mapped_column(Integer, primary_key=True)
    Title: Mapped[str] = mapped_column(String(160))
    ArtistId: Mapped[int] = mapped_column(Integer)
```

## Выполняем запрос
```python
# работа с сессией через контекстный менеджер
with Session(engine) as session:
    # логика работы с базой
    album = session.get(Album, 5)
 
    if album:
        print(f"Album ID: {album.AlbumId}, Artist ID: {album.ArtistId}, Title: {album.Title}")
    else:
        print("Альбом не найден")
```