requests - это библиотека которая позволяет мне легко и просто работать с API. Requests позволяет мне делать всё то что может сделать браузер и даже больше
```python
import requests
response = requests.get("http://127.0.0.1:5000/api/books")
```
тут мы делаем в целом стандартный запрос, по скольку я использую API то мне вернулся json и его я могу получить вот так
```python
resonse.json()
# {data:[...]}
```
если я вдруг захочу получить картинку или файл есть замечательный метод 
```python
response.content()
```
он возвращяет строку байтов


Если вам нужно передать JSON данные, вы можете использовать метод `requests.post()` с параметром `json`, который автоматически преобразует словарь в JSON формат:

```python
data = {"key": "value"}
response = requests.post("http://127.0.0.1:5000/api/post_endpoint", json=data)
```

Если вам нужно отправить файл на сервер, вы можете использовать метод `requests.post()` с параметром `files`, который принимает словарь, в котором ключи - это имена файлов, а значения - это объекты файлов. Например:

```python
files = {'file': open('example.txt', 'rb')}
response = requests.post("http://127.0.0.1:5000/api/upload_endpoint", files=files)
```

Когда вы отправляете запрос на сервер, иногда может потребоваться установить некоторые заголовки. Вы можете сделать это, передав словарь с заголовками в параметре `headers` метода `requests.get()` или `requests.post()`. Например:

```python
headers = {'Content-Type': 'application/json'}
response = requests.get("http://127.0.0.1:5000/api/endpoint", headers=headers)
```

И помните, перед отправкой запросов на живые серверы, убедитесь, что вы обрабатываете исключения, которые могут возникнуть, такие как `requests.exceptions.RequestException`, чтобы ваше приложение не останавливалось из-за ошибок сети или сервера.

в целом суть уловили
### сессии
использование сессий позволяет невероятно увеличить производительность скрипта если к одному API мы обращяемся несколько раз по скольку оставляет TCP сойденения.

так это работает
```python
import requests # Создание сессии
session = requests.Session() # Отправка запросов через сессию 
response1 = session.get('http://api.example.com/endpoint1') 
response2 = session.get('http://api.example.com/endpoint2')
```
ровно так же как и с обычным `requests.get` только через Session()

[[python]]