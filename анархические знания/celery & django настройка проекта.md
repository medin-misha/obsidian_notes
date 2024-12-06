мы научились использовать [[Celery]] отдельно от фреймворков, пора бы научиться использовать этот фреймворк с [[django]]


## инициализация в django

app/ 
├── __init__.py 
├── asgi.py 
├── settings.py 
├── urls.py 
└── wsgi.py
manage.py
	
что бы нам добавить в неё Celery нужно создать в app/ celery.py
app/ 
├── __init__.py 
├── celery.py 
├── asgi.py 
├── settings.py 
├── urls.py 
└── wsgi.py
в нём записать такой код
```python
from celery import Celery  
import os  
  
os.environ.setdefault(  
    "DJANGO_SETTINGS_MODULE", "app.settings"  
)  
celery_app = Celery(  
    "django_celery",  
)  
celery_app.config_from_object("django.conf:settings", namespace="CELERY"),  
celery_app.autodiscover_tasks()
```

естественно для того что бы всё получилось нужно запустить Redis
- `os.environ.setdefault` нужно что бы убедиться что settings доступен через ключ
- `celery_app = Celery` создаём экземпляр Celery и указываем backend, brocker, имя
- `celery_app.config_from_object` тут мы говорим что в `settings.py` любые параметры celery будут начинаться с **CELERY_** допустим **CELERY_BROKER_URL** или **CELERY_WORKER_CONCURRENCY**. Теперь мы можем настраивать celery с помощью settings.py.
- `celery_app.autodiscover_tasks` Далее, обычной практикой для повторно используемых приложений является определение всех задач в отдельном модуле tasks.py и у Celery есть способ автоматического обнаружения этих модулей.
	- app1/
	    - tasks.py этот модуль
	    - models.py

далее в settings.py
```python
...
  
CELERY_BROCKER_URL = "redis://localhost:6379/0"  
CELERY_RESULT_BACKEND = "redis://localhost:6379/0"
```

в init.py
```python
from .celery import celery_app  
  
__all__ = ("celery_app",)
```
Откройте файл в текстовом редакторе. В проекте Django по умолчанию в каждой папке приложения есть `__init__.py` файл, который помогает пометить его как модуль. По умолчанию файл пуст, но вы можете добавить код, влияющий на поведение импорта.

Чтобы убедиться, что ваше приложение Celery загружается при запуске Django, вы должны добавить его в [__all__](https://docs.python.org/3/tutorial/modules.html#importing-from-a-package):



теперь попробуем запусть всё это дело, должно быть запущенно django, redis, celery worker
`python manage.py runserver` - запуск django
`docker run -p 6379:6379 --name my_redis -d redis` запуск redis
`celery -A app worker` - запуск celery


и так далее в приложении мы создаём tasks.py
webapp/ 
│ 
├── __init__.py 
├── admin.py 
├── apps.py 
├── forms.py 
├── models.py 
├── tasks.py < вот это 
├── tests.py 
├── urls.py 
└── views.py


