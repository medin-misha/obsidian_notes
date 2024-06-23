


работать тут я буду с бд Sqlite по тому что это очень лёгкая бд
Допустим у нас есть файл .db и нам нужно подключиться и достать какие то данные
й
Вот как это  можно сделать
```python
import sqlite
with sqlite3.connect("sample_database.db") as connection:  
    cusor = connection.cursor()  
  
    cusor.execute(  
        "SELECT * FROM `table_people`"  
    )  
    result = cusor.fetchall()  
  
    print(result)
```

тут мы с помощью контекстного менеджера конектимся к БД  и запрашиваем какие то данные с помощью [[SQL]] и выводим их в консоль



[[python]]