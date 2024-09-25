### prometeus 
Переведено с английского языка.-Prometheus — это бесплатное программное приложение, используемое для мониторинга и оповещения о событиях. Он записывает метрики в базу данных временных рядов, созданную с использованием модели запроса HTTP, с гибкими запросами и оповещениями в реальном времени

prometeus собирает метрики по HTTP вызовам по определённым в конфигурации конечным точка
![[Pasted image 20240717103045.png]]

стоит отметить что не программа которая присойденена к promiteus отдаёт данные а promiteus сам запрашивает данные у приложения как бы встраиваясь в него.

### grafana
Grafana — свободная программная система визуализации данных, ориентированная на данные систем ИТ-мониторинга. 

# flask & promiteus
Promiteus очень легко интегрировать с flask просто нужно установить библиотеку `pip install prometheus-flask-exporter` и создать метрики
импорты
```python
import time  
import random  
  
from flask import Flask  
from prometheus_flask_exporter import PrometheusMetrics  
```
инициализация
```python
app = Flask(__name__) 
metrics = PrometheusMetrics(app)
```
роуты
```python
@app.route('/one')  
def first_route():  
    time.sleep(random.random() * 0.2)  
    return 'ok'  
  
  
@app.route('/two')  
def the_second():  
    time.sleep(random.random() * 0.4)  
    return 'ok'  
  
  
@app.route('/three')  
def test_3rd():  
    time.sleep(random.random() * 0.6)  
    return 'ok'  
  
  
@app.route('/four')  
def fourth_one():  
    time.sleep(random.random() * 0.8)  
    return 'ok'  
  
  
@app.route('/error')  
def oops():  
    return ':(', 500
```
main
```python
if __name__ == '__main__':  
    app.run('0.0.0.0', 5000, threaded=True)
```

promiteus встраиваеться в flask и от туда же создаёт route который будет чекать метрики автоматически.

Теперь у нас есть /metrics/ который будет отображать метрики просто с верху вниз
![[Pasted image 20240717130702.png]]для того что бы в более удобном формате читать эти метрики мы будем использовать GRAFANA

