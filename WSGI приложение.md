```python
import json
import re

class Router:
    def __init__(self):
        # Инициализация списка маршрутов
        self.routes = []

    def route(self, path):
        # Декоратор для регистрации маршрутов
        def decorator(func):
            # Преобразование пути с параметрами в регулярное выражение
            # Например, '/user/<id>' становится '/user/(?P<id>[^/]+)'
            regex = re.sub(r'<(\w+)>', r'(?P<\1>[^/]+)', path)
            # Добавление скомпилированного регулярного выражения и функции-обработчика в список маршрутов
            self.routes.append((re.compile(f'^{regex}$'), func))
            return func
        return decorator

    def __call__(self, environ, start_response):
        # Получение пути из окружения WSGI
        path = environ.get('PATH_INFO', '')

        # Поиск соответствующего маршрута
        for regex, func in self.routes:
            match = regex.match(path)
            if match:
                # Если найдено соответствие, вызываем функцию-обработчик
                response_body = func(*match.groups())
                status = '200 OK'
                break
        else:
            # Если маршрут не найден, возвращаем ошибку 404
            response_body = json.dumps({"error": "Not Found"})
            status = '404 Not Found'

        # Установка заголовков ответа
        response_headers = [('Content-Type', 'application/json')]
        # Вызов функции start_response для инициализации ответа
        start_response(status, response_headers)
        # Возвращаем тело ответа в байтовом формате
        return [response_body.encode('utf-8')]
```