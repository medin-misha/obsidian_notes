Частая боль и проблемма которая охватывает почти всех бэкендов это то как можно что то запустить/загрузить/установить/остановить до или после загрузки приложения. Тут я покажу как это делаеться на примере [[python]] [[Fast API|fastapi]]

мы создадим специальную **асинхронную функцию** первая часть которой будет выполняться до начала обработки запросов, а вторая часть будет выполняться при выключении приложения.

```python
from fastapi import FastAPI  
from contextlib import asynccontextmanager

@asynccontextmanager  
async def lifespan(app: FastAPI) -> None:  
    print(1)  
    yield  
    print(2)

app = FastAPI(lifespan=lifespan)
```
эта функция асинхронно выполняеться во время выполнения программы. Её прикол в том что до запуска и начала работы веб приложения выполняеться **только верхняя часть** эта та которая выше `yield` а перед завершением работы выполняеться то что ниже `yield` вот примерно вот так будет выглядить вывод
```python
INFO:     Will watch for changes in these directories: ['/home/misha/projects/skillbox/python_advanced-master/module_26_fastapi/homework']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [16437] using StatReload
INFO:     Started server process [16439]
INFO:     Waiting for application startup.
1 <-# вывод наших функций
INFO:     Application startup complete.
^CINFO:     Shutting down
INFO:     Waiting for application shutdown.
2 <-# вывод наших функций
INFO:     Application shutdown complete.
INFO:     Finished server process [16439]
INFO:     Stopping reloader process [16437]
```