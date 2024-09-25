> Nginx - это инструмент с открытым исходным кодом для создания легкого и мощного вебсервера способного обрабатывать множество запросов

создадим докер файл
```dockerfile
FROM nginx
```
такой вот простой
```
docker build . -t nginx
docker run -ti nginx bash
nginx
```
всё мы запустили NGINX в контейнере [[docker]]

#### сигналы
Для того что бы манипулировать состоянием NGINX-а ему нужно давать сигналы
- `stop` — быстрое завершение
- `quit` — плавное завершение
- `reload` — перезагрузка конфигурационного файла
- `reopen` — переоткрытие лог-файлов
```bash
nginx -s stop
```
#### структура конфигурационного файла
NGINX состоит из модулей которые настраиваються **директивами** указанными в конфигурационном файле. Директивы деляться на простые и блочные:
- простые ключ значение;
- блочные ключ {}
если в блочной дерективе можно задавать другие значения то её можно назвать контекстом ([events](http://nginx.org/ru/docs/ngx_core_module.html#events), [http](http://nginx.org/ru/docs/http/ngx_http_core_module.html#http), [server](http://nginx.org/ru/docs/http/ngx_http_core_module.html#server) и [location](http://nginx.org/ru/docs/http/ngx_http_core_module.html#location)). Дерективы нахлдящиеся вне явного контекста находяться в контексте [main](http://nginx.org/ru/docs/ngx_core_module.html). Директивы events, http находяться в контексте main, директива server в http а location server
#### раздача статики
Одна из важных задач NGINX - это раздача файлов. Изображения, статические HTML. для начала создадим html
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
    <meta charset="UTF-8">  
    <title>Title</title>  
</head>  
<body>  
    <p style="color:orange; width:120%;">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Aliquid distinctio id iure magnam odit officiis provident quas quasi repellendus tenetur! Cum et illo minima nostrum nulla quidem quis repudiandae sit!</p>  
</body>  
</html>
```
далее создадим папку с картинками images/ и положим туда пару любых картинок
```nginx
user nginx;  
worker_processes auto;  
  
error_log /var/log/nginx-error.log info;  
  
events {  
    worker_connections 2048;  
}  
  
  
http {  
    server {  
    listen 8000;  
    root /data/www;  
        location / {  
	        autoindex on; 
	        index index.htm index.html   
        }  
        location /images/ {  
            root /data;  
        }  
	   location ~ \.(mp3|mp4) {
	        root /www/media;
	    }
	}  
}
```
- `lisen 8000`; - это порт который будет слушать nginx
- `root data/www;` это папка по умолчанию для location у которых нет своего root
- `location` URI и настройки пути можно в контексте location поставить root и тогда у него будет свой root
`.(mp3|mp4)` - означает то что при запросе mp3, mp4 использовать этот location
тут мы настраиваем сервер, в location указан порт 8000.
`autoindex` если включить то по пути сама будет искать index.html /path/ если выключено то придёться явно говорить что нужен index.html /path/index.html
`index` говорит autoindex какие файлы искать. возвращает первый найденый


Dockerfile
```dockerfile
FROM nginx  
  
COPY index.html /data/www  
COPY images /data/images  
COPY nginx.conf /etc/nginx
```

```
docker build . -t nginx
```

```
docker run -p 8000:8000 -ti nginx bash
nginx
```
логов не будет. Если зайти на `localhost:8000` то отобразиться текст index.html а если зайти на `localhost:8000/images/last.jpg`