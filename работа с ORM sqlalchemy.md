	# работа с ORM

ORM - обьектно реляционная отображение. ORM нужно для того что бы представлять обьект базы данных как ООП класс того или иного языка.

[[sqlalchemy|SQLalchemy]] использует паттерн "обьеденение работ" то есть все изменения в базе данных не отпровляются по одиночке а собираются в группу и группой отправляются. Это быстрее и проще чем всё делать отдельными запросами. Обьект сессии отвечает за весь этот функционал в ORM.

Когда мы добавляем в сессию обьект то у него может быть 5 состояний.
- Transient (временный) состояние когда ты ещё не добавил его в сессию. Просто питоновский обьект который никак не связан с базой данных
- Penging (ожидающий) когда ты добавляешь в сессию временный обьект он становиться ожидающим
- Persistent (персистентный) это тот обьект который находиться в сессии и имеет осоциацию с базой данных. 
- Deleted (удалённый) на момент этого состояния обьект в базе данных ещё существует но он помечен как тот который нужно удалить
- Detachet (отсойденённый) обьект отсойденён от сессии и может быть не актуален. 

И так создадим вот такую базу данных
```python
from sqlalchemy import create_engine, String  
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column  
  
# dialect[+driver]://user:password@host/dbname[?key=value..]  
engine = create_engine("sqlite+pysqlite:///:memory:", echo=True)  
  
  
class Model(DeclarativeBase):  
    id: Mapped[int] = mapped_column(primary_key=True, nullable=False, unique=True)  
  
  
class User(Model):  
    __tablename__ = "users"  
    user_name: Mapped[str] = mapped_column(String(50), nullable=False)  
    last_name: Mapped[str] = mapped_column(String(50))  
    age: Mapped[str] = mapped_column(nullable=False)  
  
Model.metadata.create_all(bind=engine)
```

тут мы создаём базовую модель Model от которой будут отталкиваться все остальные модели. И модель User (в базе данных таблица users) которая имеет user_name(str не более 50 символов), last_name(str не более 50 символов), age(int).

чуть ниже я расказывал о состояниях обьекта. Вот как их можно узнать из кода.
```python
from sqlalchemy import inspect
user = User(  
    id=1,  
    user_name="misha",  
    last_name="medinskij",  
    age=16  
)
print("is transient", inspect(user).transient)
print("is penging", inspect(user).pending)
print("is persistent", inspect(user).persistent)
print("is deleted", inspect(user).deleted)

#True
#False
#False
#False
```
## работа с базой
теперь попробуем что то записать в базе:
```python
user = User(  
    id=1,  
    user_name="misha",  
    last_name="medinskij",  
    age=16  
)  
  
session.add(user)  
session.flush()
session.commit()
```
тут мы создаём user-a который добавляется в сессию функцией `.add()` потом сессия перемещает user-a в транзакцию а так же генерятся некоторые автогенерируемые поля (id). Что бы сохранить его окончательно в базе данных нужно вызвать метод `session.commit()`. 

что бы удалить пользователя нужно использовать функцию delete()
```python  
session.delete(user)  
session.commit()
print(session.execute(select(User.id)).fetchall())
```
обязательная часть того удаления елемента будет его сохранение в базе данных то есть нужно сначала его сохранить commit-ом а потом только удалить и закрепить удаление commit-ом.
## session.new
session.new это **множество** новых елементов в сессии. Каждый елемент его уникален. Ты не сможешь добавить в сессию (.add) 2 одинаковых обьекта по тому что эти обьекты сохраняются во множестве.

вот пример как оно может выглядеть:
```python
session.add(user)
print(session.new)
# IdentitySet([<__main__.User object at 0x7fad9528b260>])
```
оно собирает все елементы которые должны быть записаны в базу данных (не важно в какую таблицу)

есть ещё и список не сохранённых обьектов. Обьекты появляются там когда после flush-а ты пытаешься обновить обьект:
```python
session.add(user)  
session.flush()  
user.age = 15  
  
print(session.dirty)
# IdentitySet([<__main__.User object at 0x7f9e1e3a3f80>])
IdentitySet([<__main__.User object at 0x7f9e1e3a3f80>])```

к стати про обновление компонентов
```python
user = User(  
    user_name="misha",  
    last_name="medinskij",  
    age=16  
)
session.add(user)  
user.age = 15  
  
print(session.new)
```
по скольку тут ещё не произошел ни flush ни commit то получилось без SQL запроса заменть переменную age у user-a. Это получилось по тому что .add получает ссылку на обьект в то время как сам обьект у нас. И после commit-а в этом случае user-у будет 15 а не 16.

что бы получить актуальные данные по user-y можно использовать session.execute() аналогично [[работа с данными в Core sqlalchemy|engine.execute в core]] а можно использовать session.get(model, id). Вот так это будет выглядеть:
```python
user = User(  
    user_name="misha",  
    last_name="medinskij",  
    age=16  
)  
session.add(user)  
user.age = 15  
session.commit()  
print(session.get(User, 1)__dict__)
# {'_sa_instance_state': <sqlalchemy.orm.state.InstanceState object at 0x7f473f583f50>, 'user_name': 'misha', 'last_name': 'medinskij', 'age': '15', 'id': 1}
```