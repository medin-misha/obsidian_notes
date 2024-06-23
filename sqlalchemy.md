SQLAlchemy строиться на двух различных API, **core** & **orm**.

- **core** являеться набором инструментов для работы с базой данных, и работает на более низом уровне
- **orm** строиться на core и предостовляет возможность обьектно реляционного отображени

## подключение к базе данных
```python
from sqlalchemy import create_engine, text
  
engine = create_engine("sqlite+pysqlite:///memory:", echo=True)
```
тут мы подключаемся к базе данных **sqlite** с sqlдиалектом **pysqlite** у нас в **memory** эта база данных доступна у нас в оперативной памяти, так же атрибут `echo` включает логирование.

## взаемодействие с базой данных
```python
with engine.connect() as conn:  
    command = text(  
        """  
            select "result"
        """
    )  
    result = conn.execute(command)  
    print(result.all())
```
тут очень похожая схема с sqlite3 подключаемся через `connect` и вводим команды через `execute` единственное мы вводим команды через функцию `text`

теперь попробуем создать таблицу и установить туда какие то данные
```python
with engine.connect() as conn:  
    create_table = """  
        create table my_table (x integer primary key,y integer)"""    
    insert = """  
        insert into my_table (x, y) values (:x, :y)"""
    select = """select * from my_table"""  
    
    conn.execute(text(create_table))  
    conn.execute(text(insert), [{"x": 1, "y": 2}, {"x": 3, "y": 4}])  
	conn.commit()
	
	result = conn.execute(text(select))  
    print(result.all())
    
```
тут мы создаём таблицу **my_table** и закидываем в неё данные, так же нужно зафиксировать изменение с помощью `conn.commit`

## создание таблиц
```python
from sqlalchemy.orm import declarative_base
Base = declarative_base()  
  
class User(Base):  
    __tablename__ = "user"  
  
    id = Column(Integer, primary_key=True)  
    name = Column(String(100), nullable=False)  
    email = Column(String(100), primary_key=True)  
    login = Column(String(100), nullable=False)
```
тут мы описываем таблицу на основе ООП, в классе можно создать переменные:
- **__tablename__** - имя таблицы
- **__table_args__** параметры таблицы
так же в классе можно создавать функции вот одна из них
```python
@classmethod  
def get_all_users(cls):  
    return session.query(User).all()
```
эта функция возвращает вообще всех `User-ов` он обёрнут в декоратор `@classmethod` по тому что мы возвращаем вообще все записи а не конкретного `User-a`
```python
@classmethod  
def get_user_by_email(cls, email: str):  
    try:  
        return session.query(User).filter(User.email == email).one()  
    except:  
        return None
```
вот ещё одна функция которая ищет `User-a` по его email если фильтер не находит нужной записи то он возвращает ошибку **NoResultFound** если у нас 2 одинаковых записей то тоже возвращаеться ошибка