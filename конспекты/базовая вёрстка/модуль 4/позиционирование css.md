Поток - это базовые стили по умолчанию в html документе. Поток всегда стремиться переместить елементы с лева на право и с верху в низ. 

А позиционирование - это то что заставит елементы вести себя так как будто они вне потока.

варианты позиционирования:
#### static
Это когда елемент по потоку
#### absolute
полностью вырывает елемент из потока (если елемент вложен в relative то он вырывает из потока вложеного елемента). Елемент позиционируется относительно всей страницы если он не вложен и относительно всего блока если вложен.
#### relative
Относительное позиционирование в потоке. То есть елемент подчиняется потоку но тем не менее мы можем создавать для него отступ в top, right, bottom, right:
<div class="block relative"></div>
```html
<div class="block relative"></div>
<div class="block"></div>
```
```css
.block {
    background: black;
    width: 100px;
    height: 100px;
    margin: 10px;
}
.relative {
    position: relative;
    top: 20px;
    left: 20px;
    background-color: blue;
}
```
блок .relative будет смещён на 20 пикселей в низ и в право при этом второй блок .block на это реагировать не будет. При этом top, right, bottom, right не будут работать с `position: static`
#### fixed
относительно ВСЕГО экрана. Даже если мы будем там чё то скролить то fixed будет как бы приклеен к видимых экранов