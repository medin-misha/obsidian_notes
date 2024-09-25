Sentry — это система оперативного мониторинга ошибок с графическим интерфейсом. Для начала нужно создать там аккаунт а потом делать приложение.

 далее установка 
 ```
 pip install sentry-sdk
```
интеграция с flask
```python
import sentry_sdk
from flask import Flask

sentry_sdk.init(
    dsn="https://97c992354e314181aa2f8b70bf3cb3a3@o4506280679243776.ingest.us.sentry.io/4507611931148288",
    # Set traces_sample_rate to 1.0 to capture 100%
    # of transactions for performance monitoring.
    traces_sample_rate=1.0,
    # Set profiles_sample_rate to 1.0 to profile 100%
    # of sampled transactions.
    # We recommend adjusting this value in production.
    profiles_sample_rate=1.0,
)

app = Flask(__name__)
```

тут мы просто интегрируем sentry-sdk в наше приложение.

заходим на сайт и отслеживаем ошибки