### реализация модели

создадим модель Task:
```python
@dataclass  
class Task:  
    """класс Task означает обьект таблицы tasks_table"""  
    id: Union[int, None]  
    title: str  
    description: str  
    done: bool
```
тут я использую [[dataclass]] он позволяет мне не писать __init__ что немного сокращает мой код

далее создадим методы get_all_tasks  get_task_by_id  create_task  delete_task
```python
def get_all_tasks() -> list:  
    with sqlite3.connect(database) as conn:  
        cursor: sqlite3.Cursor = conn.cursor()  
        cursor.execute(  
            f"""  
            select * from {tasks_table}  
            """  
        )  
        data: list = [  
            Task(id=item[0], title=item[1], description=item[2], done=item[3])  
            for item in cursor.fetchall()  
        ]  
        return data  
  
  
def get_task_by_id(task_id: int = -1) -> list:  
    if task_id != -1:  
        with sqlite3.connect(database) as conn:  
            cursor: sqlite3.Cursor = conn.cursor()  
            cursor.execute(  
                f"""  
                SELECT * FROM {tasks_table}  
                WHERE id = ?                """, (task_id,)  
            )  
        data: list = [  
            Task(id=item[0], title=item[1], description=item[2], done=item[3])  
            for item in cursor.fetchall()  
        ]  
        return data  
    return get_all_tasks()  
  
  
def create_task(task: Task) -> list:  
    with sqlite3.connect(database) as conn:  
        cursor: sqlite3.Cursor = conn.cursor()  
        cursor.execute(  
            f"""  
            insert into {tasks_table} (title, description, done)            values (?,?,?)            """, (task.title, task.description, task.done)  
        )  
        task.id = cursor.lastrowid  
        data: list = [  
            task  
        ]  
        return data  
  
  
def delete_task(task_id: int) -> None:  
    with sqlite3.connect(database) as conn:  
        cursor: sqlite3.Cursor = conn.cursor()  
        cursor.execute(  
            f"""  
            DELETE FROM {tasks_table}  
            WHERE id = {task_id}  
            """  
        )
```
		
### flask restfull
это библиотека которая позволяет с большим удовольствием делать REST API
вот как её использовать:
импорты
```python
from flask import Flask, request  
from flask_restful import Api, Resource  
from models import get_all_tasks, create_task, Task, delete_task
```
создание переменных
```python
app = Flask(__name__)  
api = Api(app)
```
переменная API будет принимать на себя запросы

Создадим класс TaskResource
```python
class TaskResource(Resource):  
    def get(self, task_id: int = -1):  
        if task_id != -1:  
            pass  
        return str(get_all_tasks())  
  
    def post(self):  
        data: dict = dict(request.json)  
  
        task = Task(  
            id=-1,  
            title=data["title"],  
            description=data["description"],  
            done=data["done"]  
        )  
        return str(create_task(task=task)), 201  
  
    def delete(self, task_id: int):  
        return delete_task(task_id), 204
```
class TaskResource реализует ресурс этот класс может обрабатывать методы  `get post delete` и имеет возможность реализовать обработку других HTTP методов.
Что бы принимать аргументы типа `/task/1` нужно прописать их как Аргументы функции

### URLы
теперь определим пути
```python
api.add_resource(TaskResource, "/tasks/", "/tasks/<int:task_id>")  
  
if __name__ == '__main__':  
    app.run(debug=True)
```
важно заметить что `<int:task_id>` должен соотвецтвовать названию функции то есть нельзя сделать `<int:ebat>`



[[REST API]]
[[flask]]

### JSON + валидация
Окей мы реализовали базовый функцилнал API  но ещё требуется затронуть тему JSON и валидации данных

создадим схему c помощью библиотеки marshmallow
```python
from marshmallow import Schema, fields  
  
class TaskSchema(Schema):  
    id = fields.Int(dump_only=True)  
    title = fields.Str(required=True)  
    description = fields.Str(required=False)  
    done = fields.Bool(default=False)
```
очень похоже на наш dataclass да?

теперь в классе TaskResource обьявим переменную schema и используем её
```python
class TaskResource(Resource):  
    schema = TaskSchema()
```
```python
	def get(self, task_id: int = -1):  
	    if task_id != -1:  
	        return self.schema.dump(get_task_by_id(task_id), many=True), 200  
	    return self.schema.dump(get_all_tasks(), many=True), 200
```
тут мы используем схему для сериализации данных из функций, функции фозвращяют списки с обьектами класса Task а наша схема конвертирует их в json, если возвращяеться список то `many=True`
```python
	def post(self):  
  
	    try:  
	        data = self.schema.load(request.json)  
	    except ValidationError as exc:  
	        return exc.messages, 400  
	  
	    data["id"] = -1  
	    task = Task(**data)  
	    return self.schema.dump(create_task(task=task), many=True), 201
```
тут мы получаем данные из reqest.json закидываем в схему (на валидацию) если валидация пройдена то добавляем id в данные закидываем данные в Task и создаём новый Task а функция `create_task` возвращает данные созданые схемой в виде списка

если валидация не пройдена то возвращяеться ошибка и мы возвращяем error пользователю что бы он что то делал со своими говно-данными