В [[NGINX]] как извесно есть конфигурационный файл который рулит всем. Вот что нужно про него знать:

> nginx - состоит из модулей который настраиваются **дерективами** (местные переменные) Дерективы деляться на простые и блочные. Простая деректива состоит из "имени-параметра значения;" А блочная состоит из "имени-параметра {простые дерективы или 'дополнительные инструкции'}" Дерективы которые находятся в блочной дерективе обозначаются как дерективы в контексте.

Примеры блочных деректив 
```
events { ... }
http { ... }
server { ... }
```
пример простой директивы
```
user  nginx;
worker_processes  auto;
error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;
```

Директивы находящиеся вне какой либо блочной директивы находятся в конексте main. Дерективы events & http находяться в конексте main, деректива server в http а деректива location в server.

Соотвецтвенно вот так выглядит базовый шаблон nginx конфигурации:
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
                      '"$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```