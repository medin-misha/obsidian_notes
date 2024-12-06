Fast API это современный, быстрый, асинхронный фреймворк для создания WEB прилоежний. А ещё один его прикол заключаеться в том что у него автоматически генерируеться документация. Так же у него есть фишки с типизацией.

Начнём с установки:
```shell
pip install fastapi[all]
```
тут мы конструкцией `[all]` говорим что бы у нас установился FastAPI все нужные библиотеки которые с ним так же используются.

и создаём приложение:
main.py
```python
from fastapi import FastAPI  
  
app = FastAPI()  
  
@app.get("/")  
def hello():  
    return "hello world"
```
команда для запуска будет такой:
```python
uvicorn main:app --reload
```
тут мы запускаем uvicorn (который установился автоматически изза `[all]`) говорим где файл с приложением и переменная с приложением, --reload означает что по мере изменения кода приложение можно будет автоматически перезапускаться.

## как разносить views по разным файлам
если мы будем писать все эндпоинты в одном файле (main.py) то в какой то момент он может стать невероятно большим и views нужно будет разносить по разным файлам.

Это можно сделать с помощью `fastapi.APIRouter`
main.py
```python
from fastapi import FastAPI  
from views.items import item_router  
  
  
app = FastAPI()  
  
app.include_router(router=item_router)
```
тут всё очень просто мы подключаем через `app.include_router` нужный роутер который находиться в файле views.items.
views/items.py
```python
from fastapi import APIRouter  
from schemes import Item  
  
item_router = APIRouter(prefix="/items", tags=["items"])  
  
  
@item_router.get("/get/all")  
def all_routers():  
    return [1, 2, 3, 4, 5, 6, 7]  
  
  
@item_router.get("/get/<int:pk>")  
def get_by_pk(pk: int):  
    return {"pk": pk}  
  
  
@item_router.post("/create/items")  
def create_item(item: Item):  
    return item
```
APIRouter используется как `app` и в главном модуле сойденяется с app окончательно 