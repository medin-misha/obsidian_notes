
статические файлы это файлы проекта которые не меняються с експлоатацией самого проекта. [[html]], [[css]], [[javascript]] файлы
В [[django]] есть специальный помощник который помогает (кто бы мог подумать) работать со статикой


# практика
## settings.py

>В `INSTALLED_APPS` должно быть подключено приложение >**django.contrib.staticfiles**
```python
INSTALLED_APPS = [
    'django.contrib.staticfiles',
    ...
]
```
в приложении должно быть определена 
`STATIC_URL = '/static/'`
эта переменная которая говорит приложению по какому адресу доставать статику

так же определим переменную `STATIC_URL` и `STATIC_ROOT`
```python
STATIC_URL = 'static/'
STATIC_ROOT = BASE_DIR / 'static'
```
>эта паременная отвечает за пути к статическим файлам

## BASE_DIR / APP
тут нам нужно создать папку с указанным в настройках названием
`static` и там уже создавать нужные файлы, допустим тут структура типа:
```python
static{
	   images[],
	   css[]
	   js[]
}
```
## как это юзать?
откроем шаблон base.html
```django
{% load static%} - импортируем статику

<!DOCTYPE html>

<html lang="en">

<head>
   <link rel="stylesheet" href="{% static 'css/base/style.css' %}">
   тут указываем через тег путь в папке статики
</head>
<body>
</body>

</html>
```
