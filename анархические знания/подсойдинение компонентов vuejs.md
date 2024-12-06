в папке components создай файл с разширением `.vue`
и впиши сюда обычный vue код

>components/user.vue
```html
<template>
<div >
    <p>hello</p>
</div>
</template>
```

теперь в App.vue
```html
<script >
import user from "./components/user.vue"
export default{
    data(){
        return {
            elements:[
                {name:"vadim", email:"huev"},
                {name:"vlad", email:"nihuev"},
                {name:"misha", email:"joskiy"},
                {name:"kira", email:"ebaryk"},  
            ]
        }
    },
    components:{ user }
}
  
</script>
```
 тут мы первой стркой (там где `import`) импортируем сам компонент
 а в списке `components` мы этот компонент как бы определяем 

теперь в `template` пишем 
```html
<user v-for = "el, index in elements" :key = "index"/>
```
обычный цикл который проходиться по елементам и подаёт как аргумент в шаблон переменную `el` и у нас получаеться на сайте вот такая штука
![[Pasted image 20240111110027.png]]
что бы на прямую передавать в компонент переменные нужно сделать так:
App.vue
```html
<user :user = "this.elemements"/>
```
далее в components/component.vue
```html
<script>
export default{
    props:{
        user: {
            type: Object, //тут мы указываем что тип у переменной object
            required:true //тут мы говорим что она обязательна
        }
    }
}
</script>
```
в теге `script` нам нужно обьявить `prods` и в `prods` обьявить имя переменной и её настройки
По такой же схеме тут можно передавать `строки`, `функции`, и другие обьекты
[[vuejs]]