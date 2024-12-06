# работа со связями sqlalchemy
![[Pasted image 20240817213816.png]]
у нас есть три вида связей 
- one to one
- many to many
- one to many
с 1 и с 3 проблем не будет а вот с many to many придёться чуть чуть заморочиться.

итак база данных у нас будет выглядить вот так включая все импорты:
```python
from sqlalchemy import create_engine, String, select, ForeignKey  
from sqlalchemy.orm import Session, DeclarativeBase, Mapped, mapped_column, relationship  
  
# dialect[+driver]://user:password@host/dbname[?key=value..]  
engine = create_engine("sqlite+pysqlite:///:memory:", echo=True)  
session = Session(bind=engine, expire_on_commit=True)  
  
  
class Model(DeclarativeBase):  
    id: Mapped[int] = mapped_column(primary_key=True, nullable=False, unique=True)
```
### one to one
Тип связи ONE TO ONE достаточно простой в реализации вот как он выглядит
```python
class User(Model):  
    __tablename__ = "users"  
    user_name: Mapped[str] = mapped_column(String(50), nullable=False)  
    last_name: Mapped[str] = mapped_column(String(50), nullable=True)  
    age: Mapped[str] = mapped_column(nullable=False)  
    email: Mapped["Email"] = relationship(back_populates="user", uselist=False)  
  
    def __repr__(self):  
        return f"{self.user_name=}:{self.age=}:{self.id=}:{self.email.email=}"  
  
  
class Email(Model):  
    __tablename__ = "emails"  
    email: Mapped[str] = mapped_column(primary_key=True)  
    user: Mapped["User"] = relationship(back_populates="email", uselist=False)  
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))  
  
  
Model.metadata.create_all(bind=engine)
```
связь one-to-one реализуется за счёт relationship и ForeignKey в моделях. Главный её елемент являеться Email.user_id по тому что в этом поле храниться id user-a а relationship-ы указывают настройки к ниму и как можно связаться со связной моделью.

- back_populates указывает на поле связной модели в котором будет храниться значение этой модели. Оно указано в обоих relationship по тому что доступ к связной модели нужен у обоих моделях.
- uselist без этого параметра [[sqlalchemy]] будет думать что связь one-to-one

### one to many
one to many тоже достаточно простой почти такой же как one to one.
```python
class User(Model):  
    __tablename__ = "users"  
    user_name: Mapped[str] = mapped_column(String(50), nullable=False)  
    last_name: Mapped[str] = mapped_column(String(50), nullable=True)  
    age: Mapped[str] = mapped_column(nullable=False)  
    emails: Mapped[list["Email"]] = relationship(back_populates="user", uselist=True)  
  
    def __repr__(self):  
        return f"{self.user_name=}:{self.age=}:{self.id=}:{self.emails=}"  
  
  
class Email(Model):  
    __tablename__ = "emails"  
    email: Mapped[str] = mapped_column(primary_key=True)  
    user: Mapped["User"] = relationship(back_populates="emails", uselist=True)  
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
```
тут замечу что структура ровно такая же как у one to one отличаются лишь режимы relationship и названия.
- uselist=True (у User и у Email)
- название поля с email-ами у User emails аннотированная как Mapped[list["Email"]].
обращаться к emails нужно как к списку а не как к одиночному обьекту.
```python
user = User(  
    user_name="misha",  
    last_name="medinskij",  
    age=16  
)  
  
emails: list = [  
    Email(id=1, email='medinskij@ru.ru'),  
    Email(id=2, email="medinskij@gmail.com")  
]  
  
for email in emails:  
    user.emails.append(  
        email  
    )  
  
session.add(user)  
session.commit()  
  
for email in session.execute(select(User)).scalar_one().emails:  
    print(email.email)
```
тут мы не присваеваем обьект Email к emails а добавляем его в список emails/
### many to many
Тут нам нужно будет создать дополнительную таблицу которая будет отвечать за связи между таблицами.
```python
class UserAdress(Model):  
    __tablename__ = "user_adress"  
    user_fk = mapped_column(ForeignKey("users.id"))  
    adress_fk = mapped_column(ForeignKey("emails.email"))
```
эта таблица которая будет хранить в себе связи
```python
class User(Model):  
    __tablename__ = "users"  
    user_name: Mapped[str] = ...
    last_name: Mapped[str] = ...
    age: Mapped[str] = ...
    
    emails: Mapped[list["Email"]] = relationship(back_populates="users", uselist=True, secondary="user_adress")  
  
```
эта таблица User в emails имеет тип `Mapped[list["Email"]]` это означает что это поле нужно использовать как список и работает оно как список. В relationship есть новый для нас аргумент `secondary="user_adress"` это ссылка на таблицу UserAdress в которой храняться связи. 
```python
class Email(Model):  
    __tablename__ = "emails"  
    email: Mapped[str] = ...
    
    users: Mapped[list["User"]] = relationship(back_populates="emails", uselist=True, secondary="user_adress")
```
тут точно так же как у Users. relationship у users в аргументе secondary создаёт связь между таблицей UserAdress и Email

общая картина выглядит как то так:
![[Pasted image 20240818134723.png]]
вот пример добавление всех User-ов ко всем Email.
```python
  
user = User(  
    user_name="misha",  
    last_name="medinskij",  
    age=16  
)  
user2 = User(  
    user_name="danil",  
    last_name="medinskij",  
    age=22  
)  
  
  
emails: list = [  
    Email(id=1, email='medinskij@ru.ru'),  
    Email(id=2, email="medinskij@gmail.com"),  
    Email(id=3, email='danilmedinskij@ru.ru'),  
    Email(id=4, email="danilmedinskij@gmail.com")  
  
]  
  
for email in emails:  
    user.emails.append(  
        email  
    )  
    user2.emails.append(  
        email  
    )  
  
session.add(user)  
session.add(user2)  
session.commit()  
  
for user in session.execute(select(User)).scalars().all():  
    print([email.email for email in user.emails])
```