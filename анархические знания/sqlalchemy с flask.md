ORM нужен для того что бы помогать разработчику создавать хорошие приложения с базами данных, тут будет описано как совмещать ORM [[sqlalchemy]] и фреймворк [[flask]].

# настройка бд
```python
from sqlalchemy import create_engine, Column, Integer, String, update  
from sqlalchemy.orm import sessionmaker, declarative_base

engine = create_engine("sqlite:///database.db")  
Session = sessionmaker(engine)  
session = Session()  
Base = declarative_base()

class Product(Base):  
    __tablename__ = "product"  
  
    id = Column("id", Integer, primary_key=True)  
    name = Column("name", String(50))  
    price = Column("price", Integer)  
    count = Column("count", Integer)  
  
    def to_json(self):  
        print(self.__table__.columns)  
        return {c.name: getattr(self, c.name) for c in self.__table__.columns}  
  
    @classmethod  
    def get_all_products(cls):  
        return session.query(Product).all()  
  
    @classmethod  
    def get_product_by_id(cls, id: int):  
        return session.query(Product).filter(Product.id == id).one_or_none()  
  
    @classmethod  
    def delete_product_by_id(cls, id: int):  
        session.query(Product).filter(Product.id == id).delete()  
        session.commit()
```
тут мы указываем путь к базе данных и создаём модель `Product`:
- id - Integer pk
- name String
- price Integer
- count Integer
далее метод который представляет конкретную запись в json, `to_json`:
> принцип его работы заключаеться в том что он генерирует словарь, `self.__table__.columns` из таблицы достаёт все имена колонок, а `c.name: getattr(self, c.name)` создаёт пару "key":value если  у `self` нет атрибута c.name то он возвращает None но это не возможно по тому что в таблице есть этот атрибут
@classmethodы 
- get_all_products возвращает все продукты
- get_product_by_id возвращает продукт по его id
- delete_product_by_id удаляет продукты по id
# настройка приложения
```python
import json  
from flask import Flask, request  
from models import Base, engine, Product, session, update  
  
app = Flask(__name__)  
  
  
@app.before_request  
def before_request():  
    Base.metadata.create_all(bind=engine)  
  
  
@app.route("/")  
def get_all_products():  
    product_list: list = []  
    products = Product.get_all_products()  
    for product in products:  
        product_list.append(product.to_json())  
    return product_list  
  
  
@app.route("/<int:id>")  
def get_product(id: int):  
    product = Product.get_product_by_id(id=id)  
    if product is None:  
        return "error", 404  
    return product.to_json()  
  
  
@app.route("/", methods=["POST"])  
def create_product():  
    data: dict = json.loads(request.get_data(as_text=True))  
    product = Product(  
        name=data.get("name"),  
        price=data.get("price"),  
        count=data.get("count")  
    )  
    session.add(product)  
    session.commit()  
    return data  
  
  
@app.route("/<int:id>", methods=["DELETE"])  
def delete_product(id: int):  
    product = Product.delete_product_by_id(id=id)  
    return "успех", 200  
  
  
@app.route("/<int:id>", methods=["PUT"])  
def update_product(id: int):  
    data: dict = json.loads(request.get_data(as_text=True))  
    query = update(Product).where(Product.id == id).values(  
        name=data.get("name"),  
        count=data.get("count"),  
        price=data.get("count")  
    )  
    session.execute(query)  
    return "ok", 200  
  
  
if __name__ == '__main__':  
    app.run(debug=True)
```
тут мы создаём небольшое CRUD API которое работает с базой данных через ранее созданную модель.

before_request эта функция обёрнута в декоратор `@before_request` для того что бы инициализировать базу данных с первого запроса

get_all_products возвращает список с продуктами, берёт список с продуктами функцией `get_all_products` и создаёт свой отдельный список, и циклически добавляет продукты в свой список, перед этим конвертируя их в json фроман

get_product функция которая принимает id продукта и функцией `get_product_by_id` получает обьект продукта, или не получает если его нет, далее конвертирует его в json и возвращает пользователю

create_product функция POST которая создаёт продукт добавляет в базу данных и сохраняет его, в неё передаёться словарь типа:
```python
{
    "count": 220,
    "name": "myproduct4",
    "price": 430
}
```

delete_product функция DELETE которая принимает id продукта и удаляет его

update_product функция PUT принимает словарь типа
```python
{
    "count": 220,
    "name": "myproduct4",
    "price": 430
}
```
обновляет сущность в базе данных с помощью `sqlalchemy.update`
