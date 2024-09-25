
[[sqlalchemy|для начала создадим вот такую базу данных]]
```python
from sqlalchemy import create_engine, MetaData, Table, Column, ForeignKey, String, Integer  
# dialect[+driver]://user:password@host/dbname[?key=value..]  
engine = create_engine("sqlite+pysqlite:///:memory:", echo=True)  
metadata = MetaData()  
  
user_table = Table(  
    "users", metadata,  
    Column("id", Integer, primary_key=True, unique=True),  
    Column("name", String(50), nullable=True),  
    Column("last_name", String(50), nullable=False)  
)
  
address_table = Table(  
    "addresses", metadata,  
    Column("id", Integer, primary_key=True, unique=True),  
    Column("email", String, unique=True),  
    Column("user_id", ForeignKey("users.id"))  
)  
  
metadata.create_all(bind=engine)
```
тут мы создаём таблицы:
1. users:
	- id число, первичный ключ, уникальное
	- name строка, не null
	- last_name строка которая может быть null
1. asresses:
	- id число, первичный ключ, уникальное
	- email уникальная строка
	- user_id привязка к модели user по user.id
ну и далее `metadata.create_all(bind=engine)` это создание таблиц в базе данных.
## вставка insert
теперь вставим в таблицу значение
```python
from sqlalchemy.dialects.sqlite import insert
# from sqlalchemy import insert
stmt = insert(user_table).values(  
    name="misha"  
)  
with engine.begin() as conn:  # type: Connection  
    result = conn.execute(stmt)
```
функция insert отвечает за вставку. Если ты работаешь с конкретной базой данных то лучше всего импортировать `insert` из `sqlalchemy.dialects.<database>` а если ты работаешь без конкретной базы то из `sqlalchemy`

так же упомяну что .values может быть необязательным
```python
stmt = insert(user_table)  
  
with engine.begin() as conn:  # type: Connection  
    result = conn.execute(stmt, {"name": "misha"})
```
значение можно закидывать прямо в `execute` словарём, это не на что не влияет просто дело удобства.

так же можно создавать несколько сущностей
```python
stmt = insert(user_table)  
  
with engine.begin() as conn:  # type: Connection  
    result = conn.execute(  
        stmt,  
        [ 
            {"name": "misha"},  
            {"name": "anton"},  
            {"name": "dimas"}
        ]  
    )
```
замечу то что это не одновременная вставка данных, а вставка по одному то есть сначала misha одной операцией, потом anton второй операцией, и dimas третей операцией.
## обновление update
Для того что бы мне каким то образом изменять сущности нужно использовать `sqlalchemy.update`
```python
with engine.begin() as conn: #type: Connection
    ​￼update = conn.execute(
        update(user_table).where(user_table.c.id == 1).values(name="mishana")
    )
```
и всё. Мы тут просто указываем `update()` на таблицу елемент которой нужно изменить далее указываем `.where()` какой конкретно елемент нужно заменить и `.values` указываем поле и новое значение.

так же тут можно сделать массовое обновление елементов.
```python
#
smtp = update(user_table).where(  
    user_table.c.name == bindparam("old")  
).values(name=bindparam("new"))
```
 тут мы с помощью where ищем елемент и далее с помощью bindparam указываем ключ словаря который будет отвечать за это значение. После в values мы делаем тоже самое с ключом.
 ```python
 update = conn.execute(
        smtp,
        [
            {"old": "misha", "new": "mishana"},
            {"old": "daniil", "new": "danya"},
            {"old": "ilya", "new": "ilyha"}
        ]
    )
```
тут циклически все `old` вставляются в `.where()` а `new` в `.values`
## удаление delete
для того что бы удалить сущность модели нужно использовать `sqlalchemy.delete`
```python
with engine.begin() as conn:  #type: Connection  
    conn.execute(  
        delete(user_table).where(user_table.c.id == 1)  
    )
```
Указываем в `delete()` таблицу из которой нужно удалить елемент и через `where()` казываем каким параметрам должен соотвецтвовать елемент который мы удаляем.

так же иногда нам нужно будет возвращать то что мы удалили в последний раз.
```python
result = conn.execute(  
    delete(user_table).where(user_table.c.id == 1).returning(user_table.c.name)  
)  
print(result.scalars().all())
```
для этого используем функцию returning()
## выборка данных
у нас есть стандартная функция `select` которая будет возвращать **все елементы таблицы** но если нам нужно будет более точно сказать какие нам данные из таблицы нужны то нам понадабятся доп функции и доп синтаксис.\
### where
where позволяет ставить условия по которым мы будем искать данные. Допустим нам нужун user у которого name будет misha
```python
with engine.begin() as conn:  
    result = conn.execute(  
        select(user_table).where(user_table.c.name == "misha")  
    )  
    print(result.mappings().all())
#[{'id': 1, 'name': 'misha', 'last_name': 'medinskij'}]
```
так же мы можем ставить несколько условий
```python
with engine.begin() as conn:  
    result = conn.execute(  
        select(user_table).where(  
            user_table.c.name.contains("misha"),
            user_table.c.last_name.startswith("me")  
        )  
    )
```
тут мы сказали что нужна сущность user в name которой содержится "misha" а last_name которой начинается с "me".

Кстати у колонок можно вызывать почти такие же методы как и у стандартных типов. 

тут между условиями как бы не явно стоит оператор END то есть обязательное выполнение обоих условий. Но что если нам нужно сделать OR а не END? 

```python
# from sqlalchemy import or_
select(user_table).where(  
    or_(user_table.c.name.contains("ilya"), 
    user_table.c.last_name.s￼…￼Python￼tartswith("me"))  
# {'id': 1, 'name': 'misha', 'last_name': 'medinskij'}, 
# {'id': 3, 'name': 'dimas', 'last_name': 'medinskij'},
# {'id': 4, 'name': 'ilya', 'last_name': 'unmedinskij'},
# {'id': 5, 'name': 'daniil', 'last_name': 'medinskij'}
)
```
тут мы вывели всех с last_name начинающигося с "me" и всех у с name содержит "ilya".

Ещё есть оператор in_ которой указывает что нужно поле которое соотвецтвует хотя бы одному елементу из списка.
```python
select(user_table.c.id).where(  
    user_table.c.id.in_([1,2,4])  
)
# [{'id': 1}, {'id': 2}, {'id': 4}]
```

так же есть возможность указать какие конкретно столбцы мне нужно достать из таблицы:
```python
select(user_table.c.id).where(...)
# вывод будет вот такой
#[{'id': 1}, {'id': 3}, {'id': 4}, {'id': 5}]
```

ещё можно сойденять столбцы в месте обычной конкотынацией
```python
select(  
    (user_table.c.name + ' ' + user_table.c.last_name).label("full_name")  
).where()
for elem in result.fetchall():  
    print(elem.full_name)
#misha medinskij
#anton unmedinskij
#dimas medinskij
#ilya unmedinskij
#daniil medinskij
```
возвращается список [[namedtuple|именнованных кортежей]]
### сортировка order_by
Сортировать можно по какому угодно типу. Вот как это делаеться.

```python
select(  
    (user_table.c.name + ' ' + user_table.c.last_name).label("full_name")  
).order_by(user_table.c.name)

for elem in result.fetchall():  
    print(elem.full_name)
#anton unmedinskij
#daniil medinskij
#dimas medinskij
#ilya unmedinskij
#misha medinskij
```
просто в order_by указываешь нужное поле и сортируешь.
что бы отсортировать в обратном порядке нужно использовать функцию `desc`
```python
# from sqlalchemy import desc
select(  
    (user_table.c.name + ' ' + user_table.c.last_name).label("full_name")  
).order_by(desc(user_table.c.name))

for elem in result.fetchall():  
    print(elem.full_name)
#misha medinskij
#ilya unmedinskij
#dimas medinskij
#daniil medinskij
#anton unmedinskij
```
### группировка group_by
```python
select(  
    user_table.c.last_name  
).group_by("last_name")
for elem in result.scalars().all():  
    print(elem)
#medinskij
#unmedinskij
```
вот такая группировка. Можно группировать по всем полям.