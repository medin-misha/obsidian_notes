
## string related field
для начала покажу метод с использованием метода `__str__`. В модели создаём соответствующий метод:
```python
class ProductImage(models.Model):
    src = models.ImageField(upload_to=save_product_image)
    def __str__(self):
        return self.src.url
```
В сериализаторе делаем пишем такой код
```python
class serializer1(ModelSerializer):
	image = serializer.StringRelatedField(many = True)
	class Meta:
		model = ProductImage
		fields = ["image"]
```
всё теперь на основе метода `__str__` мы отобарзили всё

## Primary Key Related Field
это отображение на основе первичного ключа, то есть если у нас есть несколько связных моделей то будет показан список первичных ключей связных моделей,
записываеться это так:
```python
class ProductImage(models.Model):
   src = models.ImageField(upload_to=save_product_image)
    def __str__(self):
        return self.src.url
```
В сериализаторе делаем пишем такой код
```python
class serializer1(ModelSerializer):
	images = serializers.PrimaryKeyRelatedField(many = True, read_only = True)
	class Meta:
		model = ProductImage
		fields = ["images"]
```
вот ещё список оргументов этого класса
- `queryset` - Набор запросов, используемый для поиска экземпляров модели при проверке введенного поля. Отношения должны либо задавать набор запросов явно, либо set `read_only=True`.
- `many` - Если применяется ко многим отношениям, вы должны установить для этого аргумента значение `True`.
- `allow_null` - Если установлено значение `True`, поле будет принимать значения `None` или пустую строку для отношений с нулевым значением. По умолчанию используется значение `False`.
- `pk_field` - Устанавливается в поле для управления сериализацией / десериализацией значения первичного ключа. Например, `pk_field=UUIDField(format='hex')` сериализует первичный ключ UUID в его компактное шестнадцатеричное представление.

## Вложеные отношения
просто создай 2 сериализатора и сделай так
```python
class ProductImageSerializer(serializers.ModelSerializer):
    class Meta:
        model = ProductImage
        fields = "__all__"
```
```python
class ProductSerializer(ModelSerializer):
    images = ProductImageSerializer(many = True)
    class Meta:
	    model = Model
	    fields = ["images"]
```
[[DRF]]