как мы уже знаем почти все приложения в интернете работает на клиент - серверной архитектуре. Любой серьёзный фронтенд отправляет на бэкенд запросы.

Если мы запустим какое то webapp и через консоль скинем ему запрос
```js
fetch(
    "http://127.0.0.1:8080/",
    {method: "GET"}
)
.then(resp => resp.text())
.then(console.log)
{
  "Hello": "User"
}
```
то у нас будет ответ
но если мы попробуем через эту же страницу запросить google.com

```js
fetch(
    "https://google.com",
    {method: "GET"}
)
.then(resp => resp.text())
.then(console.log)

127.0.0.1/:1 Access to fetch at 'https://google.com/' from origin 'http://127.0.0.1:8080' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

```
попытка открыть ресурс Google с другого origin (это можно перевести как источник) localhost была заблокирована. Вот это  CORS в действии. Это нужно для того что бы не допустить каких то плохих действий со стороны сайта по отношению к пользователю. Допустим что бы сайт не смог сделать транзакцию в банке без вашего ведома.

в нашем Flask приложении если мы попросим распечатать headers
```
Host: 127.0.0.1:8080
Connection: keep-alive
Sec-Ch-Ua: "Chromium";v="128", "Not;A=Brand";v="24", "Google Chrome";v="128"
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36
Sec-Ch-Ua-Platform: "Linux"
Accept: */*
Origin: https://www.google.com <- этот
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://www.google.com/
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: ru-RU,ru;q=0.9,en-US;q=0.8,en;q=0.7,uk;q=0.6
```
то увидим заголовок Origin то есть источник запроса. И если мы прямо не укажем что этот источник разрешон то браузер не будет ничего делать для обработки запроса хотя Flask его таки обработает.

в Flask приложении нужно создать вот такую функцию
```python
@app.after_request  
def add_cors(response: Response):  
    response.headers['Access-Control-Allow-Origin'] = 'https://www.google.com'  
    return responses
```
это добавит в ответ сервера дополнительный header который заставит браузер нормально работать с данным источником

	- Access-Control-Allow-Methods — указывает список методов, которые проходят CORS-политику. 
- Access-Control-Allow-Headers — указывает список хедеров, которые разрешены CORS. 
- Access-Control-Max-Age — указывает, сколько времени браузер может кешировать информацию, полученную из двух предыдущих хедеров.