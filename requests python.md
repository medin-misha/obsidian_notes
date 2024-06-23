## запросы
- get метод позволяющий получать данные вот пример кода
```python
import requests
product_list = "http://localhost:8000/api/list/product/"
response = requests.get(product_list, params = {"pk":1})

print(response.text)
```
	
- post позволяет создавать новые сущности, отправлять данные на сервер пример кода
```python
import request
product_create = "http://localhost:8000/api/create/product/"

data = {"name":"product 2", "descriprion":"description", "price":123}
requests.post(product_create, json=data)
```
- put позволяет обновлять существующие ресурсы или сущности  пример кода
```python
product_update = "http://localhost:8000/api/update/product/3"
data_updated = {"name":"product 31", "description":"descrip13tion", "price":112323}
requests.put(product_update, json=data)

```
- delete позволяет удалять объекты пример кода
```python 
product_delete = "http://localhost:8000/api/destroy/product/3"

requests.delete(product_delete)
```
## нюансы
При работы с пост запросами нужно использовать json аргумет в методе  `request.post`
```python
data = {"name":"product 2", "descriprion":"description", "price":123}
requests.post(product_create, json=data)
```
так же есть аргументы 
- `params` - параметры адреса  
`http://localhost:8000/api/list/product/?name=misha&age=15`
- data - В библиотеке `requests` в Python аргумент `data` используется для передачи данных в теле запроса при выполнении HTTP-запросов методами `POST`, `PUT` и `PATCH`.
[[python]]