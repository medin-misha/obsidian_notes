Для профилирования Flask есть специальная библиотека: flask_profiler.

Flask_profiler измеряет эндпоинты, количество вызовов, скорость выполнения, и контекст запросов.

## установка
```shell
pip install flask_profiler
```

## настройка
```python
from flask import Flask
import flask_profiler

app = Flask(__name__)

# Конфигурация flask_profiler
app.config["flask_profiler"] = {
    "enabled": app.config["DEBUG"],
    "storage": {
        "engine": "sqlite"
    },
    "basicAuth":{
        "enabled": True,
        "username": "admin",
        "password": "password"
    },
    "ignore": [
        "^/static/.*"
    ]
}

# Инициализация flask_profiler
flask_profiler.init_app(app)
```

заходишь по адресу /flask-profiler