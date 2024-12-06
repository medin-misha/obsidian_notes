Для создания графика в [[python]] я буду использовать matplotlib.
```
pip install matplotlib
```
теперь создадим данные:
```python
from datetime import datetime, timedelta  
start_date = datetime.now()  

dates_analis = [(start_date + timedelta(days=i)).strftime('%Y-%m-%d') for i in range(10)]  
values_analis = [1, 3, 3, 4, 2, 5, 3, 2, 4, 4]
```
это будет типо дата здачи анализа и его значение.

теперь в этом же файле пропишем код:
```python
import matplotlib.pyplot as plt 

# это размеры в дюймах
weight = 15  
height = 10  

# тут мы прописываем ширину и высоту самого графика
plt.figure(figsize=(weight, height))

# это настройки графика. его подпись, название осей x (горизонтальной) y (вертикальной)
# grid - это сетка
# xticks отвечает за то как будут выглядеть подпись значений на оси x
plt.title("analis")  
plt.xlabel("date")  
plt.ylabel("value")  
plt.grid(True)  
plt.xticks(rotation=45)

  
plt.plot(dates_analis, values_analis, "-d", color="#94007b")  
plt.savefig("image.png")
```
запуская этот код мы запускаем базовый график.

теперь попробуем более заморочится и сделать адаптивный под данные график. Для начала нужно будет заменить исходные данные графика.
```python
import random
import matplotlib.pyplot as plt
from datetime import datetime, timedelta

start_date = datetime.now()
dates_analis = [
				(start_date + timedelta(days=i)).strftime('%Y-%m-%d') 
				for i in range(10)
]

values_analis = [random.random() for i in range(10)]
```
это создание данных, 10 рандомных значений и 10 дат. По задумке количество данных будет на прямую влиять на размеры и параметры графиков. Чем больше параметров тем больше график.
```python
import random  
  
import matplotlib.pyplot as plt  
from datetime import datetime, timedelta  
  
  
  
  
def create_grafic(date: list, value: list) -> None:  
    '''  
    Я зделал очень интересную конфигурацию графика, таким образом что бы она зависила от количества данных.  
    переменные width и height отвечают за ширину и высоту картинки соотвецтвенно.    :param date:    :param value:    :return:  
    '''  
    weight = len(date) * 5  
    height = weight * 0.7  
    # это выставление размемра картинки в дюймах  
    plt.figure(figsize=(weight, height))  
    # заголовок окна  
    plt.title("analis", fontsize=weight * 2)  
    # настройка сетки  
    plt.grid(True, linewidth=weight / 16.6)  
  
    #подпись осей x и y  
    plt.xlabel("date", fontsize=weight * 1.5)  
    plt.ylabel("value", fontsize=weight * 1.5)  
    # настройка подпись значений графика  
    plt.xticks(rotation=30, fontsize=weight * 0.9)  
    plt.yticks(fontsize=weight * 1.2)  
  
    # разстановка значений в каждый елемент графика  
    for x, y in zip(date, value):  
        plt.text(x, y, str(round(y, 3)), ha="left", fontsize=weight * 0.9)  
  
    # создание графика  
    plt.plot(  
        date,  
        value,  
        "-o",  
        color="#94007b",  
        linewidth=weight / 10,  
        markersize=weight * 0.66,  
    )  
    # сохранение в файл  
    plt.savefig("image1.png")  
  
  
if __name__ == '__main__':  
    start_date = datetime.now()  
    dates_analis = [  
        (start_date + timedelta(days=i)).strftime("%Y-%m-%d") for i in range(10)  
    ]    values_analis = [random.random() for i in range(10)]  
  
    create_grafic(date=dates_analis, value=values_analis)
```
вот как то так и выходит. Создасца файл image1.png и это и будет наш график