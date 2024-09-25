# multiprocesing.pool

Pool - это програмный шаюлон для автоматического управления списком процессов.

допустим у нас есть маленькая функция которая что то вычисляет:
```python
def task(number):  
    return sum(i ** i for i in range(number))
```
и нам нужно её выполнить 10000 раз и стоит вопрос **как это сдлать еффективно?**
Можно использовать pull процессов 
```python
def pool_practic_map():  
    pool = Pool(processes = cpu_count())  
    result = pool.map_async(task, [1000 for _ in range(10000)]) 
    pool.close()  
    pool.join()  
    logger.debug(result.get())  


```

разберу что тут твориться:
мы создаём pool процессов и говорим ему создать по процессу на каждое ядро процессора, допустим 16 ядер.

далее мы с помощью  map_async ставим на выполнение задачу и выполняем ёё с набором данных, то есть она выполниться столько раз на сколько хватит данных.

далее мы закрываем пул процессов, то есть говорим что мы больше не будем не как трогать эти процессы, и ждём их выполнения
`pool.close()`  
`pool.join()`

функция `result.get()` даёт нам список результатов 10000 результатов 

так же Pool реализует протокол  [[контекстные менеджеры python]] и его можно использовать в `with`
```python
def pool_practic_map():  
    with Pool(processes = cpu_count()) as pool:  
        start = time.time()  
        result = pool.map_async(task, [1000 for _ in range(100)])  
  
        logger.info(result)  
    logger.info(f"time - {time.time() - start}")
```


Так же стоит упомянуть `thread pool` он делает пул тредов
```python
def thread_pool_practic():  
    with ThreadPool(processes = cpu_count()) as pool:  
        start = time.time()  
  
        result = pool.map_async(task, [100 for _ in range(100000)])  
  
        logger.debug(result.get())  
    logger.info(f"time - {time.time() - start}")
```
напомню что потоки (треды) выгодны при IO операциях.

I/O - input/output операции

[[python]]