виртуальная транзакция создана для того что бы на уровне ORM инициировать уже настоящую транзакцию.

```python
from sqlalchemy.orm import Session
session = Session(engine)
with session.begin():
	session.add(object)
	# код..
	session.add(object2)
```
 



[[sqlalchemy]]