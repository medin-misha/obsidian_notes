## генераторы
[[python|Генераторы]] - это функции в которых есть слово **yield**. Как только генератор видит это слово в функцию он переводит её в байткод **не как обычную функцию** а как генератор.

```python
from typing import Iterator

def generator() -> Iterator[int] :
	for i in range(10**10):
		yield i ** 2
  
print(f"тип функции {type(generator)}\n
тип вывода функции {type(generator())}")
```
Это простой генератор. К слову генераторы аннотируются с помощью модуля typing `Iterator[return_type]`. Стоит обратить внимание на то что вывод у кода будет такой:
```shell
тип функции <class 'function'>
тип вывода функции <class 'generator'>
```
сама функция - всё ещё функция а вывод у неё уже **генератор**.

## StopIteration
Обьект являеться итератором когда он реализует 2 магических метода 
- __iter__ - позволяет использовать обьект в цикле for и конструкциях которые используют итерато
- __next__ - позволяет получить следуйщий елемент итератора
```python
from typing import Iterator

def generator() -> Iterator[int] :
	for i in range(2):
		yield i ** 2
  
gen = generator()
for _ in range(3):
	print(gen.__next__())
```
у этого кода вывод будет вот такой:
```shell
0
1
Traceback (most recent call last):
  File "/home/misha/projects/my/experimentation/py/2/gen.py", line 11, in <module>
    print(gen.__next__())
          ^^^^^^^^^^^^^^
StopIteration
```
то есть когда генератор исчерпывает себя он выкидывает ошибку `StopIteration` и через эту ошибку мы понимаем что там больше ничего нет.
## ленивые вычисления
Генераторы выгодны тем что позволяют делать ленивые вычисление. Когда компьютер вычисляет ровно столько сколько нужно.
```python
from typing import Iterator
  
def generator() -> Iterator[int] :
	for i in range(10**100):
		yield i ** 2
  
gen = generator()
for i in range(10):
	print(next(gen))
```
Тут генератор делает вычесления 10 ** 100 раз но выполниться код быстро по тому что сам цикл попросит функцию выполниться только 10 раз. Акцентирую внимание на том что генераторы сохраняют своё состояние каждый next вызов.

Получаеться что функция - это то от чего возможно получить обьект генератора. А генератор - это то у чего мы можем вызвать `next()`.

## состояния генератора
У каждого генератора есть 4 состояния.
- **GEN_CREATED** — мы только что создали объект генератора, вызвав функцию генератора. 
- **GEN_RUNNING** — мы вызвали метод next, и генератор в данный момент исполняется интерпретатором. 
- **GEN_SUSPENDED** — генератор вернул что-то после yield и уснул. 
- **GEN_CLOSED** — генератор закончил выполнение (исчерпал себя) и уснул.


![[Pasted image 20240720140639.png]]
## методы генератора
у обьекта генератора есть 3 спец метода:
- send - генератор может не только возвращать что то но и получать через **yield**. Принцып работы чуть ниже. Send можно использовать только тогда когда генератор находиться в состоянии **GEN_SUSPENDED**
- throw - 
- close - переводит генератор в состояние **GEN_CLOSED**

send
```python
def generator():  
    while True:  
        item = yield  
        print(item)  
  
g = generator()  
  
next(g)  
g.send("123")  
g.send(5234)
```
## yield from
Эта конструкция создана для упрощения работы с генераторами внутри генератора.
```python
def sub_gen():  
    yield from range(5)
# аналогично 
def sub_gen():
	for i in range(5):
		yield i
```
так же можно более лёгким образом передавать в дочерний генератор значения.
```python
def sub_gen():  
    while True:  
        x = yield  
        yield f"я получил значение {x}"  
  
def main_gen():  
    sub = sub_gen()  
    yield from sub  
  
gen = main_gen()  
  
next(gen)  
r1 = gen.send(123)  
next(gen)  
r2 = gen.send(42)

#анологично
def sub_gen():  
    while True:  
        x = yield  
        yield f"я получил значение {x}"  
  
def main_gen():  
    sub = sub_gen()  
    next(sub)  
    for i in [123, 42]:  
        result =sub.send(i)  
        next(sub)  
        yield result  
  
gen = main_gen()  
  
for i in gen:  
    print(i)
```