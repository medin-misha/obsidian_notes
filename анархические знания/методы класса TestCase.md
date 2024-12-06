
# assertы

|                           |                      |
| ------------------------- | -------------------- |
| **Метод**                 | **Проверяет**        |
| assertEqual(a, b)         | a == b               |
| assertNotEqual(a, b)      | a != b               |
| assertTrue(x)             | bool(x) is True      |
| assertFalse(x)            | bool(x) is False     |
| assertIs(a, b)            | a is b               |
| assertIsNot(a, b)         | a is not b           |
| assertIsNone(x)           | x is None            |
| assertIsNotNone(x)        | x is not None        |
| assertIn(a, b)            | a in b               |
| assertNotIn(a, b)         | a not in b           |
| assertIsInstance(a, b)    | isinstance(a, b)     |
| assertNotIsInstance(a, b) | not isinstance(a, b) |
|                           |                      |
# SetUp
метод который запускаеться перед запуском любого теста если допустим я тестирую веб приложения то было бы не плохо запускать это приложение для каждого теста что бы тесты не ебли друг друга
```python
def setUp(self):
	app.config['TESTING'] = True
	app.config['DEBUG'] = False
	self.app = app.test_client()
	self.base_url = '/hello-world/'
```


# tearDown
метод позволяющий прибратса за тестом, он запускаеться после каждого теста, его задача сделать так что бы тесты после выполнения не ебли друг друга

Может использоаться допустим для отчистки базы данных от данных которых создал тест и так далее

#  setUpClass
метод который запускаеться перед выполнением всех тестов (один раз перед процедурой), нужен для подгрузки общих данных которые нужны всем тестам и которые по мере тестирования не как не меняються
```python
@classmethod  
    def setUpClass(cls):  
        app.config['TESTING'] = True  
        app.config['DEBUG'] = False  
        cls.app = app.test_client()  
        cls.base_url: str = '/hello-world/'
```


# tearDownClass
тоже самое что и *setUpClass* только запускаеться после выполнения тестов



[[тестирование]]
