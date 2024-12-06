zя уже изучил работу с [[flask]] отдельно и с [[Celery]] отдельно, настало время разобраться как их совокупить так что бы из этого получилось что то очень интересное. Так же использует [[группы задач]]

### utils.py
```python
from celery import Celery  
  
import time  
import random  
  
celery_app = Celery(  
    "tasks",  
    broker="redis://localhost:6379/0",  
    backend="redis://localhost:6379/0"  
)  
  
  
@celery_app.task  
def process_image(image_id):  
    # В реальной ситуации здесь может быть обработка изображения  
    # В данном примере просто делаем задержку для демонстрации    
    print("started")  
    time.sleep(random.randint(1, 3))  
    print("end")  
    return f'Image {image_id} processed'
```
тут мы видим стандартное использование [[Celery]] ничего такого необычного функция которое останавливает время на 1-3 сек что эмулирует обработку фото
### app.py
```python
from flask import Flask, request, jsonify  
from celery import group  
import json  
from utils import process_image, celery_app  
  
app = Flask(__name__)  
  
  
@app.route('/process_images', methods=['POST'])  
def process_images():  
    images = json.loads(request.get_data(as_text=True))["images"]  
    if images and isinstance(images, list):  
        task_group = group(  
            process_image.s(image_id)  
            for image_id in images  
        )  
  
        result = task_group.apply_async()  
        result.save()  
  
        return jsonify({'group_id': result.id}), 202  
    else:  
        return jsonify({'error': 'Missing or invalid images parameter'}), 400  
  
  
@app.route('/status/<group_id>', methods=['GET'])  
def get_group_status(group_id):  
    result = celery_app.GroupResult.restore(group_id)  
  
    if result:  
        status = result.completed_count() / len(result)  
        return jsonify({'status': status}), 200  
    else:  
        return jsonify({'error': 'Invalid group_id'}), 404  
  
  
if __name__ == '__main__':  
    app.run(debug=True)
```
тут у нас 2 flask эндпоинта `/process_images` и `/status/group_id` 
`process_images` POST получает [[json]] вида:
```json
{
    "images":[
            "<<image content 1>>",
            "<<image content 2>>",
            "<<image content 3>>",
            "<<image content 4>>",
            "<<image content 5>>",
            "<<image content 6>>"
        ]
}
```
валидирует что бы images было вида list и запускает группу задач заключая в неё каждый елемент каждый елемент и сохраняет процессы в backend что был доступ даже к не выплненому скрипту. И возвращает Id этой группы задач.

`/status/group_id` GET этдпоинт он получает Id группы задач и возвращает статус выполнения задачи в виде
```python
{
 "status": 1.0/0.0
}
```
1.0 - выполнено
0.0 - не выполнено

обратим внимание на `result = celery_app.GroupResult.restore(group_id)` 
`GroupResult` позволяет вам отслеживать состояние и получать результаты всех задач в группе. `.restore(group_id)` берёт из бэкенда группу задач по её id.

Далее функция работает с группой узнавая её статус и выводя пользователю