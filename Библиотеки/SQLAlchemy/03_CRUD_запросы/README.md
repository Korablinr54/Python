# CRUD
Базовый функционал любого приложения, работающего с базой данных, включает четыре фундаментальные операции, известные под аббревиатурой CRUD.  

CRUD:  
- `Create` — создание новых записей.  
- `Read` — чтение и получение данных.  
- `Update` — модификация существующих записей.  
- `Delete` — удаление записей.  

Библиотека `SQLAlchemy` 2.0 предоставляет современный и эффективный инструментарий для выполнения этих операций. Его ключевые преимущества — это прозрачный синтаксис, строгая типизация для повышения надежности кода и сохранение детального контроля над генерируемыми SQL-запросами.

## Добавление записей (Create)

Операция создания (`Create`) является базовой при работе с ORM. В SQLAlchemy 2.0 этот процесс реализован интуитивно и соответствует современным стандартам читаемости кода.

*Основной механизм*:  
- **Создание экземпляра**: Сначала формируется объект (экземпляр модели), который необходимо сохранить в базе данных.  
- **Добавление в сессию**: Этот объект добавляется в текущую сессию работы с БД с помощью метода add().  
- **Фиксация изменений**: Чтобы запись окончательно сохранилась в базе, выполняется коммит (подтверждение) сессии методом commit(). Этот шаг гарантирует, что транзакция будет завершена.  

У нас есть модель:
```python
class Album(Base):
    __tablename__ = "Album"

    Albumid: Mapped[int] = mapped_column(Integer, primary_key = True)
    Title: Mapped[str] = mapped_column(String(160), nullable = False)
    Artistid: Mapped[int] = mapped_column(Integer, nullable = False)
```

Для внесения новой записи нам нужно:
1) создать объект
2) добавить его в сессию
3) подтвердить изменения

```python
with Session(engine) as session:    
    new_album = Album(Title="Generator of Evil", Artistid=9) 
    session.add(new_album)  
    session.commit()    
```

Разбор примера:  
- `new_album = Album(Title="Generator of Evil", ArtistId=9)` - Создание экземпляра модели.  
- `session.add(new_album)` - Добавление в сессию.  
- `session.commit()` - Фиксация изменений (коммит).  

## Получение данных (Read)

### возврат конкретной записи .first()

```python
with Session(engine) as session:
    stmt = select(Album).where(Album.Title=="Generator of Evil")
    album = session.scalars(stmt).first()
    print(f"id: {album.Albumid}, title: {album.Title}")
```

Разбор примера:    
- `with Session(engine) as session:` — Использование менеджера контекста (with) для автоматического управления сессией (гарантирует корректное закрытие).  
- `stmt = select(Album).where(Album.Title=="Generator of Evil")` — Построение запроса: выбрать все колонки сущности Album, где значение столбца Title строго соответствует заданной строке. 
- `album = session.scalars(stmt).first()` — Ключевая строка. Метод `scalars()` выполняет запрос и возвращает итератор объектов **Album**, а `first()` извлекает из него первую найденную запись (или None, если совпадений нет).  
- `print(album.Title)` — Обращение к атрибуту объекта для вывода значения.  

### Возврат списка значений .all()

```python
with Session(engine) as session:
    stmt = select(Album)
    albums = session.scalars(stmt).all() 
    for album in albums: 
        print(f"id: {album.Albumid}, title: {album.Title}") 
```
Разбор примера:   
- `with Session(engine) as session:` - Создание сессии для работы с БД, которая автоматически закроется после выполнения блока.  
- `stmt = select(Album)` - Формирование SQL-запроса: выбрать все столбцы из таблицы, связанной с моделью Album.  
- `albums = session.scalars(stmt).all()` - Выполнение запроса и получение ВСЕХ результатов в виде списка объектов Album. `.all()` — преобразует итератор в список всех найденных записей.
- `for album in albums:`  
  ____`print(f"id: {album.Albumid}, title: {album.Title}")` - Цикл перебирает каждый объект Album в списке albums. Для каждого объекта происходит обращение к его атрибутам (`.Albumid`, `.Title`).

## Обновление данных (Update)

Обновление — это операция, при которой вы изменяете существующую запись в базе данных: исправляете опечатку, уточняете возраст, меняете статус и т. д.  

В SQLAlchemy 2.0 есть два основных способа обновлять данные:  
- через объект ORM (ручное обновление)  
- через прямое SQL-выражение с помощью session.execute(update(...))  

Оба варианта полезны, но подходят для разных случаев.

### Обновление через ORM

Такой способ подходит для точечных правок. 
```python
# меняем у выбранной строки данные и коммитим (Через ORM)

with Session(engine) as session:
    stmt = select(Album).where(Album.Title == "Generator of Evil")
    row = session.scalars(stmt).first() # выбираем конкретную строку
    row.Artistid = 666 # устанавливаем новое значение атрибута для строки
    session.commit() # фиксируем (коммит)
```

### Обновление через execute

Способ подходит для обновления нескольких строк.  
```python
with Session(engine) as session:
    stmt = (
        update(Album)
        .where(Album.Title == "Generator of Evil")  # Условие выборки
        .values(Artistid = 777)                     # Новые значения
    )
        
    session.execute(stmt)  # Выполнение UPDATE запроса
    session.commit()       # Фиксация изменений в БД
```

## Удаление данных (Delete)