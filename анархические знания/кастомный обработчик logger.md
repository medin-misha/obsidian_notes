


для начала созадим сам хендлер
```python
class HandlerFile(logging.Handler):  

    def __init__(self, filename:str, mode:str) -> None:  
        super().__init__()  
        self.filename = filename  
        self.mode = mode  
    def emit(self, record:logging.LogRecord) -> None:  
        message = self.format(record)  
        with open(self.filename, self.mode) as file:  
            file.write(message)
```
в методе __init__ обязательно нужно вызвать super по тому что без этого работать не будет

сама обработка происходит в методе emit, переменная message получает уже отформатированное сообщение и далее с ним нам нужно что то сделать, в этом случае я записываю его в файл


[[логирование python]]