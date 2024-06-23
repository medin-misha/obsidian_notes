Для начала установим правило:
- **тесты находяться в папке tests**
- **название папки с тестом начинаеться  _test-name.py_**
- **название функции-теста будет начинаться с test**

у нас есть небольшая папка с файлами для работы
main.py
tests/test_age.py

в main.py
```python
def get_social_status(age):
	if not isinstance(age, (float, int)):
		raise ValueError('Please provide a number')
	  
	if age < 0:
		raise ValueError("Check age")
	elif 0 <= age < 13:
		return "ребенок"
	elif 13 <= age < 18:
		return "подросток"
	elif 18 <= age < 50:
		return "взрослый"
	elif 50 <= age < 65:
		return "пожилой"
	else:
		return "пенсионер"
```
эта функция принимает возраст и возвращяет то кто есть человек,
	- взрослый
	- ребёнок
	- подросток
	- пенсионер
	- пожилой
в tests/test.py тесты к этой функции
```python
import unittest

from main import get_social_status

class TestSocialAge(unittest.TestCase):
	def test_can_get_child_age(self):
		age = 8
		expected_res = 'ребенок'
		function_res = get_social_status(age)
		self.assertEqual(expected_res, function_res)
  
	def test_cannot_pass_str_as_age(self):
		age = "old"
		with self.assertRaises(ValueError):
			get_social_status(age)
```
этот класс тестирует нашу функцию:
	1. **test_can_get_child_age** функцией self.assertEqual(*ожидаемое значение*, *значение после выполнения*) проверяет, правильно ли работает функция и корректен ли вывод
	2. **test_cannot_pass_str_as_age** проверяет что при не правильном выводе выводиться исключение *ValueError*


как только я написал тесты из деректории я в консоли пишу
`python -m unittest` важно запускать из дериктории с проектом там где main.py
test/test_age.py

[[тестирование]]