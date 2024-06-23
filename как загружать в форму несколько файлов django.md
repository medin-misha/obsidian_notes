создадим класс CreateView и [[html шаблон django]]
```python 
class CreateProductView(LoginRequiredMixin, CreateView):  
    form_class = ProductModelForm  
    model = Product  
    template_name = "shop/create-product/index.html"  
    success_url = reverse_lazy("shop:main")
      
    @transaction.atomic  
    def form_valid(self, form):  
        response = super().form_valid(form)  
          
        for img in form.files.getlist("images"):  
            ProductImages.objects.create(  
                product=self.object,  
                image=img  
            )  
  
        return response
```
>тут типичный [[class based view django]] Create View у неё есть аргументы 
>	1. form_class = [[model form]]
>	2. model = [[django model]] 
>	3. template_name = путь к шаблону
>	4. success_url = перенаправление после создания сущностей модели

Теперь шаблон или template
```html
{% extends "shop/base/base.html"%}  
  
{% block title %} create new product {% endblock %}  
  
{% block body %}  
<form method="post" enctype="multipart/form-data"> 

    {% csrf_token%}  
    {{form.as_p}}  
    <label for="images">products images</label>  
    <input name = "images" id = "images" type="file" multiple>  
    <button type="submit">create</button>  
  
</form>  
  
{% endblock %}
```
>тут у нас форма **POST** для возможности отправления нескольких файлов
>`enctype="multipart/form-data"`
>далее идёт стандартный `{% csrf_token%}` и `{{form.as_p}} `
>после нам нужно ручками создать поле формы
```html
	<label for="images">products images</label>  
    <input name = "images" id = "images" type="file" multiple> 
```
>`label` нужен для того что бы подписать наше поле 
>`for="ссылка на name поля"`
>`input` это само поле тут мы его именуем `name` даём `id` `type`
>и говорим что в этот `input` можно загружать несколько  файлов `multiple`



[[django]] 
