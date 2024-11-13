запуск postgres из командной строки в docker
```
docker run --name skillbox-postgres --rm -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e PGDATA=/var/lib/postgresql/data/pgdata -v /tmp:/var/lib/postgresql/data -p 5432:5432 -it postgres 
```
эта команда создаст и запустит postgres.
Теперь подключимся из другого терминала к контейнеру
```python
docker exec -ti skillbox-postgres /bin/sh
psql -U postgres
```
команда psql позволяет подключиться к postgres внутри контейнера

```yml
version: '3.2'  
services:  
  postgres:  
    image: postgres  
    restart: always  
    environment:  
      - POSTGRES_USER=postgres  
      - POSTGRES_PASSWORD=postgres  
    ports:  
      - '5432:5432'  
    volumes:  
      - ./db/:/var/lib/postgresql/data
```
вот так нужно подключаться из под уcompose

В дериктории где ты запускал docker-compose.yml будет создана папка с файлами postgres. Для pycharm и других IDE она может быть пустой по тому что у IDE нет прав для доступа к этой папке. То есть для того что бы узнать что там находиться нужно дать права или зайти от админа.

 Как подключиться из кода к postgres