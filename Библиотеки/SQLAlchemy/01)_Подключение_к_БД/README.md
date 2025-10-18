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
CREATE TABLE test.clients (
	id uuid DEFAULT gen_random_uuid() NOT NULL,
	name varchar(255) NULL,
	email varchar(100) NULL,
	phone int8 NULL,
	add_phone _int8 NULL,
	worksheet_json json NULL,
	worksheet_jsonb jsonb NULL,
	CONSTRAINT clients_email_key UNIQUE (email),
	CONSTRAINT clients_phone_key UNIQUE (phone),
	CONSTRAINT clients_pkey PRIMARY KEY (id)
);
```

## Импорт необходимых компонентов
```python
# функция создания движка create_engine
# базовые типы данных SQL String, Integer, JSON, text 
from sqlalchemy import create_engine, String, Integer, JSON, text 

# специфичные PostgreSQL типы данных
from sqlalchemy.dialects.postgresql import UUID, ARRAY, JSONB

# ORM компоненты
# DeclarativeBase - базовый класс для декларативного стиля моделей
# Mapped - аннотация типов для полей модели
# Session - менеджер сессий для работы с БД
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, Session

# Стандартная библиотера python
import uuid
```