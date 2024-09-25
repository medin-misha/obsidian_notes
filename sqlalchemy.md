SQLAlchemy — программная библиотека на языке [[Python]] для работы с реляционными СУБД с применением технологии ORM. Служит для синхронизации объектов Python и записей реляционной базы данных. SQLAlchemy позволяет описывать структуры баз данных и способы взаимодействия с ними на языке Python без использования [[SQL]].

# engine 
**обьект engine** - Этот обьект представляет собой центральный обьект который сойденяется с Базой данных

```python
from sqlalchemy import create_engine  
  
# dialect[+driver]://user:password@host/dbname[?key=value..]  
engine = create_engine("sqlite+pysqlite:///memory", echo=True)
```
`dialect[+driver]://user:password@host/dbname[?key=value..]  ` - это шаблон пути по которому мы будем подключаться к базе данных.

тут мы создаём обьект engine. Само сойденение сейчас не создано по тому что это будет ленивое сойденение, то есть сойденение создаёться по мере необходимости.

Самый базовый и простой способ построить сойденение через [[контекстные менеджеры python|контекстный менеджер]] 
```python
with engine.connect() as connection:  
    result = connection.execute(text("select 'hello'"))  
    print(result)
```
тут мы создаём подключение через engine и просим базу данных отдать нам строку `"hello"`. Важно что бы мы не передавали на прямую в `execute` текст нужно передать его через функцию `text()`. Это конструктор SQL выражений.

вот что выводит наш код:
```bash
2024-08-12 21:12:18,863 INFO sqlalchemy.engine.Engine BEGIN (implicit)
2024-08-12 21:12:18,863 INFO sqlalchemy.engine.Engine select 'hello'
2024-08-12 21:12:18,863 INFO sqlalchemy.engine.Engine [generated in 0.00018s] ()
<sqlalchemy.engine.cursor.CursorResult object at 0x7f641d3b6a50>
2024-08-12 21:12:18,863 INFO sqlalchemy.engine.Engine ROLLBACK

```
в первую очередь (первый лог) отправляеться команда BEGIN. SQLAlchemy работает с базой данных транзокционально то есть через [[транзакции sqlalchemy|транзакции]]. BEGIN это начало транзакции (первый лог) а ROLLBACK это конец транзакции (последний лог).

разберём второй и третий логи. Второй лог говорит нам что мы отправляем команду `select 'hello'` а третий лог говорит что ответ генерился 0.00018 секунд и список оргументов при запросе пустой `()`

`<sqlalchemy.engine.cursor.CursorResult object at 0x7f641d3b6a50>` это обьект курсора в котором содержится результат запроса. Для необработаного прямого вывода можно использовать функцию `.all()`
```python
print(result.all())
# [('hello',)]
```
выведиться тупо список кортежей. Также можно использовать функции
- `.scalar()` функция просто берёт первый елемент в ответе и выводит возвращает его (выведет просто строку `hello`).
- `.scalars()` это более гибкая версия scalar, она работает с несколькими елементами.

разница между ними в том что `scalar` берёт первый елемент и закрывает сойденение. А `scalars` работает со всеми елементами и тоже закрывает сойденение. То есть на одной переменной не получиться 2 раза использовать `scalar/scalars` функцию

# метаданные
метаданные - это описание того как данные находяться в таблице. Метаданные хранят в себе информациб **о таблицах**
Создадим метадату:
```python
from sqlalchemy import (create_engine, text,  
                        MetaData, Table, Column, Integer, String)  
  
  
# dialect[+driver]://user:password@host/dbname[?key=value..]  
engine = create_engine("sqlite+pysqlite:///:memory:", echo=True)  
metadata = MetaData()  
  
user_table = Table(  
    "users", metadata,  
    Column("id", Integer, primary_key=True),  
    Column("username", String(20), nullable=False),  
    Column("email", String(40), unique=True)  
)  
  
print(user_table.c.keys())  
  
metadata.create_all(engine)
```
тут мы создаём сначала создаём обект metadata которы отвечает за описание таблиц. Далее создаём обьект таблицы c полями
- id - первычный ключ модели
- username - имя пользователя в рамках 20 текстовых символов, поле не должно быть пустым
- email - уникальное на всю таблицу поле которое не должно повторяться (до 40 символов)
если запустить такой код то получиться что мы создаём таблицу user_table то мы в логах получим сообщение которое говорит нам что сначала  SQLAlchemy проверяет, существует ли таблица а потом уже начинает её создавать.

## обьявление метаданных с помощью ORM
```python
from sqlalchemy import (create_engine, Column, Integer, String)  
from sqlalchemy.orm import declarative_base  
  
  
  
# dialect[+driver]://user:password@host/dbname[?key=value..]  
engine = create_engine("sqlite+pysqlite:///:memory:", echo=True)  
Base = declarative_base()  
  
class User(Base):  
    __tablename__ = "users"  
    id = Column(Integer, primary_key=True)  
    name = Column(String(20))  
  
  
class Product(Base):  
    __tablename__ = "products"  
    id = Column(Integer, primary_key=True)  
    product_name = Column(String(50), unique=True)  
    description = Column(String(500))
```
это самый популярный способ, через `declarative_dase` он простой и понятный. Мы обьявляем переменную Base и от Base наследуем уже наши модели. В обязательно нужно сделать `__tablename__` это имя нашей таблицы.

```python
from sqlalchemy import (create_engine, BigInteger, String)  
from sqlalchemy.orm import declarative_base, as_declarative, mapped_column, Mapped  
  
# dialect[+driver]://user:password@host/dbname[?key=value..]  
engine = create_engine("sqlite+pysqlite:///:memory:", echo=True)  
Base = declarative_base()  
  
@as_declarative()  
class Model:  
    id: Mapped[int] = mapped_column(BigInteger, primary_key=True)  
  
  
class User(Model):  
    __tablename__ = "users"  
    name: Mapped[str] = mapped_column()  
  
  
class Product(Model):  
    __tablename__ = "products"  
    product_name: Mapped[str] = mapped_column(String(50))  
	description: Mapped[str] = mapped_column(String(500))
```
Этот способ более понтовый за счёт того что тут мы сделали основную модель от которой наследуеться каждая из созданых моделей и по умолчанию у всех есть свой `id`

По поводу `mapped_column` теперь мы так обьявляем поля модели, однако стоит заметить что если нужен какой то специфичный тип то всё таки придёться переопределять с помощью SQLalchemy типов а не с помощью типизации. `Mapped` нужен для того что бы были подсказки типов.

по скольку иногда нам нужно к модели обращаться как к обьекту Table нужно использовать атрибут `__table__`.

```python
print(User.__table__.c.keys())  
print(Product.__table__.c.keys())
```

вот такой будет вывод
```
['id', 'name']
['id', 'product_name', 'description']
```

# работа с самой базой данных
мы оставим вот такую версию кода
```python
from sqlalchemy import (create_engine, BigInteger, String)  
from sqlalchemy.orm import declarative_base, as_declarative, mapped_column, Mapped, Session  
  
# dialect[+driver]://user:password@host/dbname[?key=value..]  
engine = create_engine("sqlite+pysqlite:///:memory:", echo=True)  
Base = declarative_base()  
  
@as_declarative()  
class Model:  
    id: Mapped[int] = mapped_column(BigInteger, primary_key=True, autoincrement=True)  
  
  
class User(Model):  
    __tablename__ = "users"  
    name: Mapped[str] = mapped_column()  
  
  
class Product(Model):  
    __tablename__ = "products"  
    product_name: Mapped[str] = mapped_column(String(50))  
    description: Mapped[str] = mapped_column(String(500))
```

теперь допустим нам нужно добавить одну сущность User и одну сущность Product
```python
#...
#создание таблиц
Model.metadata.create_all(engine)

from sqlalchemy.orm import Session
session = Session(engine)

  
with session.begin():  
    user = User(name="misha", id=123)  
    product = Product(product_name="помидор", description="реально вкусный помидор")  
    session.add(user, product)
```
тут мы создаём пустые таблицы в базе данных. И сессию по которой будем обращаться к базе данных. Далее через контекстный менеджер обращаемся к `session.begin()` таким образом мы обьявляем транзакцию к базе данных, далее создаём user и product и добавляем их в базу методом `session.add()` обращу внимание что в `with session.begin` не нужно делать `as session` иначе выдадет ошибку по тому что `begin` выдаёт `SessionTransaction` но не `Session`.

Как же я харош, базу данных сделал, модели создал, сессию создал, теперь пора получить данные.

```python
from sqlalchemy import select
with session.begin():  
    result = session.execute(select(User).where(User.id == 123))  
    print(result.scalar())
# <__main__.User object at 0x7f5450447530> 
```
тут мы поиском `where` взяли конкретного user-а а для того что бы взять все сущности нужно просто вызвать обычный select
```python
with session.begin():  
    result = session.execute(select(User))  
    print(result.scalars().all())
# [<__main__.User object at 0x7f5450447530>]
```

дополнительно обьясню что `select` получает таблицу на вход которую нужно взять, и по умолчанию `select` заберает все елементы.


# схемы и миграции sqlalchemy

# стратегии загрузки sqlalchemy
основные стратегии загрузки это
- lazy mode загрузка по необходимости (по умолчанию)

- select mode

- joined mode загружает все сущности голавной модели и связной с помощью одного запроса с join-ом. В целом самым быстрым будет этот

- subquery mode этот будет самым быстрым за 100000 загруженных обьектов. По тому что оно групперует запросы по N загружаемых обьектов.

- selectin mode загружает отдельным запросом связанные сущности. Нужен когда много одинаковых обьектов.

- noload mode
стратегия загрузки указываеться в relationship как аргумент lazy в строке.