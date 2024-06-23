# process threads
обьясню на практике:
для начала импорты:
```python
import logging  
import time  
import threading  
import multiprocessing
```
у нас есть функция которая что то вычисляет
```python
def calculations():  
    result = sum([i ** i for i in range(1, 1000)])  
    return result
```
и у нас есть задача провертеть её 1000 раз настолько быстро на сколько это в принцепе возможно

Можно это сделать по обычному
```python
def calculations_default(count:int = 1000):  
    time_start = time.time()  
    for i in range(count):  
        calculations()  
    logger.info(f"default calculations {time.time() - time_start}")
```
Можно сделать с помощью тредов (потоков):
```python
def calculations_threads(count:int = 1000):  
    time_start = time.time()  
    calc = []  
    for i in range(count):  
        thread = threading.Thread(target = calculations)  
        thread.start()  
        calc.append(thread)  
    for obj in calc:  
        obj.join()  
    logger.info(f"threads calculations {time.time() - time_start}")
```
можно сделать с помощью отделтных процессов:
```python
def calculations_process(count:int = 1000):  
    time_start = time.time()  
    calc = []  
    for i in range(count):  
        proc = multiprocessing.Process(target=calculations)  
        proc.start()  
        calc.append(proc)  
    for obj in calc:  
        obj.join()  
    logger.info(f"process calculations {time.time() - time_start}")
```

время всех етих методов:
- INFO:__main__:default calculations 6.1
- INFO:__main__:threads calculations 15.7
- INFO:__main__:process calculations 1.6
Вопрос: мы делали одно и тоже, но почему тогда время разное?

Тут будет еффективнее создание под каждую функцию отдельного процесса, по тому что если мы выполняем потоками то у нас появляеться такая штука как GIL (Global Interpreter Lock) которая не позволит нам выполнять всё и сразу, а обычный способ подразумевает выполнение последывательно, то есть никакого параллельного вычисления.

Треды, (потоки) будут еффективны для операций ввода\вывода I\O пока мы будем ожидать пока нам дадут ответ мы закинем ещё один запрос, то есть треды будут выгоднее когда мы будем заниматься с вебом

[[python]]
[[процесс]]


# threading.lock
Если у нас есть 2 потока которые работают с одной переменной процесса, допустим один поток её увеличивает на 1 второй уменьшает циклично
```python
def worker_one() -> None:  
    global COUNTER  
    while COUNTER < 1000:  
        COUNTER += 1  
        logger.info(f'Worker one incremented counter to {COUNTER}')
```
```python
def worker_two() -> None:  
    global COUNTER  
    while COUNTER > -1000:  
        COUNTER -= 1  
        logger.info(f'Worker two decremented counter to {COUNTER}')
```
и функция которая разбрасывает это по потокам
```python
def main():
	start: float = time.time()  
	thread_1 = threading.Thread(target=worker_one)  
	thread_2 = threading.Thread(target=worker_two)  
	thread_1.start()  
	thread_2.start()  
	thread_1.join()  
	thread_2.join()
```
и в итоге получаеться что поток 1 за 1 единицу времени добавляет а поток 2 отнимает и в итоге ничего не происходит 

какое решение будет самое лучшее?
Конечно же не выебываться и без потоков вызвать эти функции, тогда они выполняться последывательно и всё сработает быстро.
Но есть и другой способ! Можно просто вызвать функцию lock
```python
from threading import Lock
LOCK = Lock()
def worker_one() -> None:  
    global COUNTER  
    with LOCK:  
        while COUNTER < 1000:  
            COUNTER += 1  
            logger.info(f'Worker one incremented counter to {COUNTER}')  
  
  
def worker_two() -> None:  
    global COUNTER  
    with LOCK  
        while COUNTER > -1000:  
            COUNTER -= 1  
            logger.info(f'Worker two decremented counter to {COUNTER}')
```
теперь во время время выполнения одной функции вторая будет ничего не делать, то есть ждать, во время ожидания (по скольку мы используем треды) может происходить ещё какая нибуть полезная робота, которая не связана с COUNTER(глобальной переменной с которой работают функции),

**после выполнения кода в контекстном менеджере lock перестаёт действовать**


# threading.Semaphore
Семафор, это такая штука которая контролируемо выдаёт доступ определённому количеству тредов, к допустим глобальной переменной, работает оно просто:
```python
semaphore = threading.Semaphore(1)
```
число которая передаёться в класс `Semaphore` означает то сколько тредов он допустит к нашей глобальной переменной
```python
def worker_one() -> None:  
    global COUNTER  
    with semaphore:  
        while COUNTER < 1000:  
            COUNTER += 1  
            logger.info(f'Worker one incremented counter to {COUNTER}')  

def worker_two() -> None:  
    global COUNTER  
    with semaphore:  
        while COUNTER > -1000:  
            COUNTER -= 1  
            logger.info(f'Worker two decremented counter to {COUNTER
```
по сути семафор выполняет функцию `Lock` которая блокирует доступ остальным потокам, однако он может немного приспускать блок, допустим если нужно что бы 2 потока одновременно пользовались переменной мы ставим при иницилизации 2 и теперь наши воркеры будут драться за переменную, или работать с ней, смотря какой код в функциях