Экземпляр класса Engine предостовляет нам сойденение с базой данных, что бы подключиться к базе.

>Единственная цель [`Engine`](https://docs.sqlalchemy.org/en/20/core/connections.html#sqlalchemy.engine.Engine "sqlalchemy.engine.Движок") объекта с точки зрения пользователя - обеспечить единицу подключения к базе данных, называемую [`Connection`](https://docs.sqlalchemy.org/en/20/core/connections.html#sqlalchemy.engine.Connection "sqlalchemy.engine.Подключение"). При непосредственной работе с Ядром [`Connection`](https://docs.sqlalchemy.org/en/20/core/connections.html#sqlalchemy.engine.Connection "sqlalchemy.engine.Подключение") объектом является то, как осуществляется все взаимодействие с базой данных. Поскольку [`Connection`](https://docs.sqlalchemy.org/en/20/core/connections.html#sqlalchemy.engine.Connection "sqlalchemy.engine.Подключение") представляет собой открытый ресурс для базы данных, мы хотим всегда ограничивать область использования этого объекта определенным контекстом, и лучший способ сделать это - использовать форму диспетчера контекста Python, также известную как [оператор with](https://docs.python.org/3/reference/compound_stmts.html#with).