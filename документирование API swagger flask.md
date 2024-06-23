### импорты
```python
from apispec.ext.marshmallow import MarshmallowPlugin  
from apispec_webframeworks.flask import FlaskPlugin  
from flasgger import APISpec, Swagger  
from flask import Flask, request  
from flask_restful import Api, Resource  
from marshmallow import ValidationError  
  
from models import (  
    DATA,  
    get_all_books,  
    init_db,  
    add_book,  
)  
from schemas import BookSchema
```
 тут нужно установить дополнительно несколько библиотек:
 - `pip install flasgger`
 - `pip install apispec`
 - `pip install apispec_webframeworks`
они нужны для плагинов и создания сваггера
### инициализация
```python
app = Flask(__name__)  
api = Api(app)  
spec = APISpec(  
    title="bookList",  
    version="1.1.1",  
    openapi_version='2.0',  
    plugins=[  
        FlaskPlugin(),  
        MarshmallowPlugin()  
    ]  
)
```
тут в целом база, мы создаём веб-приложение и API, так же мы создаём speс: APISpec

#### APISpec
нужен для создания спецификации(настройки) документации в моём приложении в 
APISpec передаються аргументы (
- `title` название документации
- `version` версия документации (в целом может быть любой строй хоть 'хуй-пиздец 2.0')
- `openapi_version` версия OpenAPI который мы используем тут уже важно какая версия стандартно можно поставить `2.0` но есть ещё и 3.x.x 
- `plugins` это **список** плагинов в моём случае (по скольку я использую Flask и marshmallow) я использую FlaskPlugin(), MarshmallowPlugin() нужно обратить внимание что классы мы **инициализируем** и передаём а не просто передаём  
)
[[REST API]][[flask]]
### схемы
нужно изменить немного код схем, по сути нужно просто изменить импорты
```python
from flasgger import Schema, fields, ValidationError  
from marshmallow import validates, post_load  
  
from models import get_book_by_title, Book  
  
  
class BookSchema(Schema):  
    id = fields.Int(dump_only=True)  
    title = fields.Str(required=True)  
    author = fields.Str(required=True)  
  
    @validates('title')  
    def validate_title(self, title: str) -> None:  
        if get_book_by_title(title) is not None:  
            raise ValidationError(  
                'Book with title "{title}" already exists, '  
                'please use a different title.'.format(title=title)  
            )  
  
    @post_load  
    def create_book(self, data: dict) -> Book:  
        return Book(**data)
```
тут мы просто импортировали 
- `Schema`
- `fields`
- `ValidatorError`
из flasgger а не из marshmallow
### спецификация
создадим дополнительные переменные
```python
template = spec.to_flasgger(  
    app, definitions=[BookSchema]  
)
swagger = Swagger(app, template=template)
```
эта template говорит swagger как должен выглядеть сваггер и помогает ему нормально работать свойство definitions подсказывает какие есть схемы в нашем API что бозволяет более информативно генерировать документацию
### анотации в документации функций
в нашем классе
```python
class BookList(Resource):  
    def get(self) -> tuple[list[dict], int]:  
        schema = BookSchema()  
        return schema.dump(get_all_books(), many=True), 200  
  
    def post(self) -> tuple[dict, int]:  
        data = request.json  
        schema = BookSchema()  
        try:  
            book = schema.load(data)  
        except ValidationError as exc:  
            return exc.messages, 400  
  
        book = add_book(book)  
        return schema.dump(book), 201
```
нужно сделать докстринги в методе get это будет
![[Pasted image 20240424180704.png]]
1. `---`: Это разделитель между заголовком docstring и его телом. Всегда используется в docstrings Flask.
2. `tags`: Это свойство, определяющее теги, к которым относится эндпоинт. Теги помогают организовать эндпоинты по группам и категориям.
3.  `responses`: Это свойство, определяющее возможные ответы от эндпоинта. Ваш код определяет только один ответ с кодом состояния 200. в целом можно и добавить другие ответы типа 400 401
4.  `description`: Это краткое описание ответа.
5. `content`: Это свойство, определяющее форматы контента, наш сервис поддерживает только формат `application/json`.
6.  `schema`: Это свойство, определяющее схему данных, возвращаемых сервером. Ваш код использует схему, определенную в формате OpenAPI (Swagger). В вашем случае это массив объектов типа `Book`, определенный в файле с именем "definitions/Book". почему не `BookSchema`? по тому что.
а в post
![[Pasted image 20240424180723.png]]
1. `---`: Это разделитель между заголовком docstring и его телом. Всегда используется в docstrings Flask.
2. `tags`: Это свойство, определяющее теги, к которым относится эндпоинт. Теги помогают организовать эндпоинты по группам и категориям.
3. `parameters`: Параметры запроса, которые ожидает ваш метод. В вашем случае, это параметр `body`, который содержит данные новой книги.
    - `in`: Определяет местоположение параметра. Здесь он указывает, что параметр находится в теле запроса.
    - `name`: Название параметра.
    - `schema`: Описывает структуру данных параметра, в данном случае, ссылка на определение схемы книги.
4. `responses`: Описывает возможные ответы от вашего метода.
    - `description`: Описание ответа.
    - `schema`: Описывает структуру данных ответа, в данном случае, ссылка на определение схемы книги.



это формат [[yaml]]

