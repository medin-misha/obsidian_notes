pydantic - это библиотека для валидации данных. 

Вся суть заключается в том что я создаю класс и типизирую его. Далее создаю екземпляр этого класса и закидую в него данные. И это всё автоматически валидируеться. 

разберём как это всё дело работает:
```python
from datetime import datetime  
from pydantic import BaseModel  
from typing import List
```
импорты.
```python
class User(BaseModel):  
    id: float  
    name: str = "username"  
    account_create: datetime | None  
    friends: List["User"] = []
```
тут мы делаем модель `User` которая наследуеться от базовой модели. Это нужно для того что бы к классу User прилогался функционал pydantic-а. Далее я описываю поля:
- id - это поле типа `float` оно обязательное
- name - это поле типа `str` со значением по умолчанию что означает что имя можно и не вводить.
- account_create - значение даты и времени. Не обязательно
- friends - список связных User-ов
```python
users: list = [  
    User(  
        id=index,  
        name=f"user{index}",  
        account_create=datetime.now(),  
    )    for index in range(5)  
]  
  
me: User = users[0]  

  
me.friends = users[1:]  
  
print(me.friends)
```
пример использования. Тут я создаю 5 user-ов беру первого user-а и добавляю в его список friends всех в списке User-ов

для того что бы создавать дополнительные ограничения нужно использовать annotated_types.Annotated вот как это может выглядеть:
```python
from pydantic import BaseModel, EmailStr  
from annotated_types import Annotated, MaxLen, MinLen  

class UserCreate(BaseModel):  
    name: Annotated[str, MaxLen(50), MinLen(2)]  
    age: Annotated[int, MaxLen(120), MinLen(0)]  
    email: EmailStr
```