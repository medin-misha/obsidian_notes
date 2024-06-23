допустим что нам нужно валидировать и проверять то что нам прислал пользователь.

форма:
```python
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
задача: сделать так что бы в поля можно было записывать не что попало

[[WTForms flask]]