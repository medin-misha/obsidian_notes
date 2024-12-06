Poetry - это система контроля зависимости как venv.

Для начала работы нужно установить poetry 
```shell
pip install poetry
```
далее создать проэкт
```python
poetry new </dir>
cd </dir>

poetry add packege1 packege2 pachege3
```
в папке будет создан `.toml` файл в котором будут настройки проэкта. Выглядить он будет как то так:
```python
[tool.poetry]  
name = "fastapicource"  
version = "0.1.0"  
description = ""  
authors = ["Your Name <you@example.com>"]  
readme = "README.md"  
  
[tool.poetry.dependencies]  
python = "^3.12"  
fastapi = "^0.112.1"  
  
  
[build-system]  
requires = ["poetry-core"]  
build-backend = "poetry.core.masonry.api"
```
 в `tool.poetry`:
 - name - название проэкта
 - version - версия проэкта
 - description - описание проэкта
 - authors - авторы
 - readme - путь к README.md
`[tool.poetry.dependencies]  ` - версия пакетов и питона
`[build-system]` - конфигурация системы

файл `.toml` можно обновлять в ручную. Но при работе с poetry будет создан `poetry.lock` файл и его трогать не нужно.

для того что бы обновить версию пакета нужно использовать команду 
```python
poetry update <package>
```

## как установить все пакеты 
Представим ситуацию что мы в хотим закинуть наш проект в докер и нужно как то устанавливать пакеты. В pyproject.toml находиться информация о проекте и пакетах которые должны быть в проекте. Для того что бы установить пакеты нужно ввести команду poetry install