Тут всё ну пряяям максимально просто. В  конфиг [[NGINX]]'а нужно добавить всего пару строк. А точнее в дериктиву server.

Но для начала нужно запустить приложение которое будет работать допустим приложение [[Fast API]]:

```python
from fastapi import FastAPI
import uvicorn
app = FastAPI()

@app.get("/hello")
async def hello_router():
    return {"hello":"world"}

if __name__ == "__main__":
    uvicorn.run(
        "main:app", port=8080, host="192.168.5.174"
    )
```
теперь отредактируем nginx.conf вот таким образом
```conf
http {
    server {
        location / {
            proxy_pass http://192.168.5.174:8080;
        }       
    }
}
```
Тут мы указываем что при запросе на / отправить запрос на http://192.168.5.174:8080 что являеться адресом fastapi приложки. По сути всё