Если пользователь скинул нам не корректный POST запрос то нужно отобразить ошибку. Можно использовать валидаторы, можно уже в коде отлавливать ошибки и говорить пользователю "ты чё мудак?" отправить данные на последуйщее переделывание

КАК ЭТО СДЕЛАТЬ?
для начала напишу эндпоинт с потенциально опасным действием.
```python
@app.route("/endpoint/", methods=["POST"])
def divide():
	data = request.get_data(as_text=True)
	data = json.loads(data)
	return f"{data["num1"] / data["num2"]}", 200
```

тут в теории может выпасть ошибка `ZeroDivisionError` означающяя что на ноль делить нельзя
Отлавливать мы её будем по следуйщей схеме:
```PYTHON
@app.errorhandler(ZeroDivisionError)
def handle_exception(error:ZeroDivisionError):

	return "деление на ноль"
```

тут мы создаём функцию которая получает ошибку и возвращяет ответ пользователю что это за ошибка. В логах мы не увидим огромную ошибку, и ничего при правильном применении посыпатся не должно.


Однако не всегда получиться угадать с ошибкой так что мы можем перехватывать вообще все ошибки
```python
@app.errorhandler(Exception)
	def handle_exception(error:Exception):
		return "ошибошка"
```
просто поставив класс `Exception` в `errorhandler`

[[flask]]