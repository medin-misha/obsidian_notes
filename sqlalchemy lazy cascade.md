Каскадное поведение - определяет что должно происходить с дочерним обьектом в случае изменения родительского обьекта

**простейший пример**
> при удалении родительского обьекта будут удалены все дети

Lazy - [[sqlalchemy]] предоставляет управлением загрузки связанных обьектов оператором lazy.
1. `select` (по умолчанию)**: Загрузка выполняется лениво. Связанные объекты загружаются по мере необходимости, при первом обращении к атрибуту.
```python
relationship('RelatedClass', lazy='select')
```
2. **`immediate`**: Связанные объекты загружаются сразу после загрузки основного объекта, используя отдельные SQL-запросы.
```python
relationship('RelatedClass', lazy='immediate')
```
3. **`joined`**: Жадная подгрузка с использованием соединения (JOIN). Связанные объекты загружаются вместе с основным объектом в одном SQL-запросе.
```python
relationship('RelatedClass', lazy='joined')
```
4. **`subquery`**: Жадная подгрузка с использованием подзапросов. Связанные объекты загружаются вместе с основным объектом, но через подзапрос.
```python
relationship('RelatedClass', lazy='subquery')
```
всё выше указанное используеться для `relationship`
5. **`selectin`**: Жадная подгрузка с использованием IN-запросов. Связанные объекты загружаются в отдельных SQL-запросах, но с использованием WHERE IN (ключи основного объекта).
```python
relationship('RelatedClass', lazy='selectin')
```
7. **`noload`**: Связанные объекты не загружаются автоматически. Этот метод может быть полезен, если вам нужно отключить подгрузку по умолчанию и управлять ею вручную.
```python
relationship('RelatedClass', lazy='noload')
```
8. **`raise`**: Исключение будет выброшено при попытке доступиться к связанным объектам, если они не загружены. Используется для отладки.
```python
relationship('RelatedClass', lazy='raise')
```
9. **`raise_on_sql`**: Исключение будет выброшено, если связанным объектам нужно выполнить SQL-запрос для загрузки. Полезно для отладки проблем, связанных с неожиданной ленивой загрузкой.
```python
relationship('RelatedClass', lazy='raise_on_sql')
```