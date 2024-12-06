**namedtuple** - это именованый кортеж. Может стать даже как замена определения класса.

вот так выглядит синтаксис
```python
from collections import namedtuple  
  
Car = namedtuple(  
    "Car",  
    [  
        "color",  
        "glass"  
    ]  
)  
  
car = Car(  
    color="red", glass="tinted"  
)  
print(f"color: {car.color} \nglass: {car.glass}")
```
обрати внимание на синтаксис **namedtuple** сначала мы передаём typename (имя класса) за тем field_tyoes (поля класса). Поля класса можно определить и как строка "color glass" тогда матод вызовет .split и в итоге получиться всё равно список.

при это сохраняеться все преймущества кортежа.