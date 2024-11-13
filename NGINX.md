NGINX это веб сервер, обратный прокси сервер (сервер перенаправляющий запросы на приложения) с поддержкой кеширования и балансировки нагрузки.

У NGINX есть 1 главный и несколько рабочих процессов как правило 1 + количество ядер CPU. Основная цель главного процесса управление рабочими процессами в соотвецтвии с конфигурацией, а рабочие обработку запросов. 

Файл конфигурации NGINX находиться где то там:
- /etc/nginx
- /usr/local/etc/nginx
- /usr/local/nginx/conf
у меня он находился в /etc/nginx.
Для большего упрощения будем работать в [[docker|докер]] контейнере. Создадим [[Dockerfile]]
```Dockerfile
FROM nginx
COPY nginx.conf /etc/nginx/
```
черезвычайно простой. Теперь обьясним жалкой машине что такое `nginx.conf`
```conf
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```
это базовая конфигурация nginx. C которой он запускаеться.
Далее не забывая включить docker нужно построить контейнер (в папке где Dockerfile и nginx.conf)
```shell
docker build -t nginx .
```
теперь включим докер контейнер и зайдём в него
```shell
sudo docker run -p 8000:80 -ti nginx bash
```
тут всё также (и макака справиться) просто запускаем контейнер сойденяем порты компьютера 8000 и порты докера 80 запускаем созданый контейнер и заходим в него из терминала.

В контейнере пишем `nginx` и заходим на http://localhost:8000. Вот и всё ты смог включить NGINX.

В Nginx есть ещё так называемые сигнал которые дают знать NGINX что нужно сделать для удовлетворения апетитов пользователя. От имени пользователя который запустил NGINX пишем
- `nginx -s stop` и nginx остановиться
- `nginx -s quit` плавное завершение (с ожиданием окончания всех процессов)
- `nginx -s reload` перезагрузка
- `nginx -s logopen` переоткрытие логов