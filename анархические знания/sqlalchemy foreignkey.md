В ORM [[sqlalchemy]] есть 2 варианта создания [[Foreign key]] 

один из них создать обьект **ForeignKey** как атрибут обьект Column
```python
class Book(Base):
   book_id = Column("book_id", ForeignKey("books.id"))
```
Минус этого способа в том, что создать составной ключ не получиться, по тому что если мы укажим даже несколько полей ForeignKey то sqlalchemy не поймёт как ими пользоваться.

 второй способ для создания составного ForeignKey
 ```python
 class Child(Base):
   __tablename__ = 'child' 
   id = Column(Integer, primary_key=True) 
   parent_id = Column(Integer) 
   name = Column(String) 
   __table_args__ = ( 
   ForeignKeyConstraint(['parent_id'], ['parent.id']  
   )
```
то есть задавать мы будем не внутри колонки а внутри описания таблицы

# 1 ко многим
один ко многим определяеться через колонку таблицы родителя
```python
from sqlalchemy.orm import relationship
class Child(Base):  
    __tablename__ = 'child'  
    id = Column(Integer, primary_key=True)  
    parent_id = Column(Integer, ForeignKey('parent.id'))  
    parent = relationship("Parent", back_populates="children"))
```
в родительской таблицы связь опимываеться с помощью функции `relationship`
```python
class Parent(Base):  
    __tablename__ = 'parent'  
    id = Column(Integer, primary_key=True)  
    children = relationship("Child", back_populates="parent")
```
эта функция является ссылкой на дочерние обьекты (класс Child) 
>Для `Parent`, `children = relationship("Child", back_populates="parent")` указывает на связь один-ко-многим с классом `Child`. Это означает, что у одного объекта `Parent` может быть несколько объектов `Child`.

>Для `Child`, `parent = relationship("Parent", back_populates="children")` указывает на связь с `Parent`, что позволяет объекту `Child` ссылаться на его родителя.

говоря проще relationship это определение ссылки на связанный обьект

# 1 к 1
```python
class Parent(Base):  
    __tablename__ = 'parent'  
    id = Column(Integer, primary_key=True)  
    child = relationship("Child", back_populates="parent", uselist=False)  
  
  
class Child(Base):  
    __tablename__ = 'child'  
    id = Column(Integer, primary_key=True)  
    parent_id = Column(Integer, ForeignKey('parent.id'), unique=True)  
    # parent = relationship("Parent", back_populates="children")
```
один к одному это как бы соглашение для связи 1 к 1 нужно использовать атрибут `uselist=Flase` это укажет что в случае вызова вернёться только 1 сущность, так же во второй сущности (`Child`) нужно  указать `unique=True` это условие которое говорт что связь уникальна

- на уровне `Parent` нельзя привязать детей изза `uselist=Fasle`
- на уровне `Child` мы указываем что запись должна быть уникальной `unique=True`
# много к многим
Для реализации связи многие ко многим нужна дополнительная таблица которая держала бы все связи в себе.
 ```python
integration_table = Table(  
    'integrations', Base.metadata,  
    Column(  
        'parent_id',  
        ForeignKey('parent.id'),  
        ),  
  
    Column(  
        'child_id',  
        ForeignKey('child.id'),  
        )  
)
```
эта таблица нужна для того, что бы хранить в себе связи таблиц
```python
class Parent(Base):  
    __tablename__ = 'parent'  
    id = Column(Integer, primary_key=True)  
    children = relationship(  
        "Child",  
        secondary=integration_table,  
    )
class Child(Base):  
    __tablename__ = 'child'  
    id = Column(Integer, primary_key=True)  
  
    # двунаправленная связь  
    parents = relationship(  
        "Parent",  
        secondary=integration_table,  
    )
```
и тут мы определяем 2 модели которые обращяются к друг другу через integration_table которая их соеденяет

# from sqlalchemy.orm  relationship
это функция которая указывает с каким классом связана модель 
```python
class Students(Base):  
    __tablename__ = "students"  
    id = Column("id", Integer, primary_key=True)  
    name = Column("name", String(100), nullable=False)  
	receiving_books = relationship("ReceivingBooks")
```
при связи M2M сюда нужно передать аргумент `secondary=tablename` нужен он для указания таблицы в которой будут храниться связи
```python

students_book = Table(  
    "students_book", Base.metadata,  
    Column("student_id", Integer, ForeignKey("students.id")),  
    Column("receiving_books_id", Integer, ForeignKey("receiving_books.id"))  
)

class Students(Base):  
    __tablename__ = "students"  
    id = Column("id", Integer, primary_key=True)  
    name = Column("name", String(100), nullable=False)  
	receiving_books = relationship("ReceivingBooks", secondary="students_book",  back_populates="student")
```
так же про аргумент `back_populates="receiving_books"` он говорит что обратиться в классе ReceivingBooks можно по имени **receiving_books** но название переменной в классе ReceivingBooks должно быть идентично **receiving_books**