## теория 
WSGI был описан в [pep333](https://peps.python.org/pep-0333/) за тем дополнен в [pep3333](https://peps.python.org/pep-0333/) это стандарт позволяющий [[python]] взаемодействовать с [[NGINX]], [[Apache]] и другими веб серверами.
Вот как оно должно работать
- Веб-сервер (например, Nginx) получает HTTP-запрос.
- Запрос передается WSGI-серверу (например, Gunicorn).
- WSGI-сервер вызывает ваше Python-приложение, передавая ему информацию о запросе.
- Приложение обрабатывает запрос и возвращает ответ.
- WSGI-сервер передает ответ обратно веб-серверу, который отправляет его клиенту.
как то так: 
Nginx -> 
WSGI-сервер ->
WSGI-приложение (ваша функция) ->
Бизнес-логика
ответ возвращаеться так же только в обратном порядке

По стандарту WSGI-приложение должно удовлетворять следующим критериям: 3
1) должно быть вызываемым объектом в случае с flask это `app=Flask()`; 
2) должно принимать два параметра — словарь с переменными окружения (почти как CGI, даже нейминг переменных похож) и функцию «обработчик запроса»; 
3) должно вызывать внутри себя обработчик запроса и передавать туда код ответа (строкой) и заголовки; 
4) должно вернуть итерируемый объект с телом ответа.

Вся магия веб-фреймворков включается именно в этот момент. 
- за нас парсится словарь переменных окружения(headers)
- мы понимаем, какой http-метод вызвал нас, 
- какой ресурс запрашивают(url-path),
- какие параметры в него передали и т. д. 

Всё это передаётся в функцию, которая обрабатывает запросы на конкретном endpoint. А в конце то, что возвращает эта функция, отдаётся обратно в нужном виде.

Тут возникает логичный вопрос: кто-то же должен вызывать эту wsgi-функцию — обработчик — так кто?
[[uWSGI]] или [[Gunicorn]].

## практика
![[Pasted image 20240628195834.png]]
будем делать вот такое приложение.

#### создадим /src/app.py
```python
from flask import Flask  
  
app = Flask(__name__)  
  
@app.routes("/")  
def hello_view():  
    return {  
        "name":"misha"  
    }
```
обрати внимание на то что нет if __name__ == "__main__": app.run()

#### теперь создадим Dockerfile
```dockerfile
FROM python:3.8  
  
RUN apt-get update && apt-get install -y python3-dev nginx supervisor\  
 && rm -rf /var/lib/apt/lists/*  
  
COPY requirements.txt /app/  
RUN pip install -r /app/requirements.txt  
  
COPY src/ /app/  
COPY nginx.conf /etc/nginx/nginx.conf  
COPY uwsgi.ini /etc/uwsgi/uwsgi.ini  
COPY supervisor.ini /etc/supervisor/conf.d/

WORKDIR /app

CMD [
"/usr/bin/supervisord",
"-c",
"/etc/supervisor/conf.d/supervisord.ini"
]
```
тут мы конфигурируем Dockerfile. В целом тут всё понятно но я разберу то что мне не понятно.
```dockerfile
RUN apt-get update && apt-get install -y python3-dev nginx \  
 && rm -rf /var/lib/apt/lists/*  
```
- Обновление пакетов: `apt-get update` обновляет список доступных пакетов и их версий.
- Установка программ: `apt-get install -y` устанавливает следующие пакеты:
	- `python3-dev`: Инструменты разработки для Python 3
	- `nginx`: Веб-сервер и прокси-сервер
- чистка кэша: `rm -rf /var/lib/apt/lists/*` удаляет списки пакетов, скачанные apt-get, чтобы уменьшить размер образа.
```dockerfile
CMD [
"/usr/bin/supervisord",
"-c",
"/etc/supervisor/conf.d/supervisord.ini"
]
```
Команда запускает Supervisord:

- `/usr/bin/supervisord` - путь к исполняемому файлу Supervisord
- `-c /etc/supervisor/conf.d/supervisord.ini` - указывает путь к конфигурационному файлу Supervisord#### 
#### создаём nginx.conf
```nginx.conf
user  www-data;  
worker_processes auto;  
error_log /var/log/nginx/error.log notice;  
pid /var/run/nginx.pid;  
events {  
    worker_connections 1024;  
}  
http {  
         include /etc/nginx/mime.types;  
         default_type application/octet-stream;  
  
         log_format main '$remote_addr - $remote_user [$time_local] "$request" '  
         '$status $body_bytes_sent "$http_referer" '  
         '"$http_user_agent" "$http_x_forwarded_for"';  
  
         access_log /var/log/nginx/access.log main;  
         sendfile on;  
         keepalive_timeout 65;  
  
         server {  
             listen 8000;  
             location / {  
             include uwsgi_params;  
             uwsgi_pass unix:/run/uwsgi.sock;  
           }  
     }  
}  
daemon off;
```
1. Основные настройки:
	- `user www-data;` - Nginx будет работать от имени пользователя nginx.
	- `worker_processes auto;` - Количество рабочих процессов будет определяться автоматически.
	- `error_log` и `pid` - пути к файлу журнала ошибок и PID-файлу.
2. Блок `events`:
	- `worker_connections 1024;` - максимальное количество одновременных соединений для каждого рабочего процесса.
3. Блок `http`:
	- Включает файл с MIME-типами и устанавливает формат логирования.
	- Настраивает доступ к лог-файлу.
	- Включает `sendfile` для более эффективной отправки файлов.
	- Устанавливает таймаут для keep-alive соединений.
4. Блок `server`:
    - `listen 80;` - сервер слушает порт 80 (HTTP).
    - Блок `location /` "/" - это путь по которому можно подключиться к этому ресурсу:
        - `include uwsgi_params`Включает параметры для работы с uWSGI.
        - `uwsgi_pass unix:/run/uwsgi.sock;` - передает запросы на Unix-сокет uWSGI.
5. `daemon off;` - Nginx будет работать в режиме переднего плана, а не как демон.
по сути тут всё как в дефолтной конфигурации только `server` отличается

#### uwsgi.ini
```ini
[uwsgi]  
wsgi-file = /app/routes.py  
socket = /run/uwsgi.sock  
chown-socket = www-data:www-data  
chmod-socket = 664  
show-config = true  
callable = app  
module = routes  
uid = www-data  
gid = www-data  
master = true
processes = 4
```
`wsgi-file` путь к файлу приложения
`socket` Создает Unix-сокет для связи с веб-сервером (например, Nginx).
`chown-socket` устанавливает влажельца
`chmod-socket` права доступа для сокета
`show-config` вывод конфигурации при запуске
`callable` это обьект приложения (app=Flask())
`module` модуль в котором лежит callable
`uid` пользователь от имени которого запускается приложение
`gid` группа 
`master` включает режим главного процесса это нужно включать только когда у тебя несколько процессов подключено
`processes` указываем количество процессовNG
#### supervisor.ini
```ini
[supervisord]  
nodaemon=true  
  
[program:uwsgi]  
command=/usr/local/bin/uwsgi --ini /etc/uwsgi/uwsgi.ini  
stdout_logfile=/dev/stdout  
stdout_logfile_maxbytes=0  
stderr_logfile=/dev/stderr  
stderr_logfile_maxbytes=0  
  
[program:nginx]  
command=/usr/sbin/nginx  
stdout_logfile=/dev/stdout  
stdout_logfile_maxbytes=0  
stderr_logfile=/dev/stderr  
stderr_logfile_maxbytes=0  
stopsignal=QUIT
```
1. `[supervisord]`
	- nodeamon=true
2. `[program:uwsgi]`
	- `command=..` Команда для запуска uWSGI с указанным конфигурационным файлом.
	- `stdout_logfile=..` перенаправляет stdout в указаный файл
	- `stderr_logfile=..` перенаправляет stderr в указаный файл
	- `stdout_logfile_maxbytes=..` отключает ротацию логов stdout что позволяет не удалять старые логи
	- `stderr_logfile_maxbytes=..` отключает ротацию логов stderr что позволяет не удалять старые логи
3. `[program:nginx]`
	- `command /usr/sbin/nginx` путь кужа устанавливаеться nginx
	- `stopsignal=QUIT` Указывает Supervisord использовать сигнал QUIT для graceful остановки Nginx.
#### requirements.txt
в консоли
```
pip freeze > requirements.txt
```
