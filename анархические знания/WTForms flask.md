представим ситуацию что нам нужно сделать бекенд для регистрации пользователя.

Для этого мы используем формы в [[flask]] для этого нужно установить библиотеку Flask-wtf

`pip install Flask-WTF`
после пишем такой код
```python
from wtforms import IntegerField, StringField, Form
class RegistrationForm(Form):

"""
форма для регистрации пользователя
"""
	email = StringField()
	phone = IntegerField()
	name = StringField()
	address = StringField()
	comment = StringField()
```
это будет наша форма
```python
@app.route("/registration", methods=["POST"])
def registration():
	#получение данных с пост запроса в формате dict
	data = request.get_data(as_text=True)
	data = json.loads(data)
	# использование данных и валидация
	form = RegistrationForm(data=data)
	if form.validate():
		return f"{form.data}"
	return "400", 400
```
тут мы получаем данные из формы (данные писылаються в raw формате (json)) валидируем их и выводим, пользователю. [[flask запросы]]