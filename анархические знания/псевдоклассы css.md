

продолжая тему [[css свойства]] есть такая штука как ***псевдоклассы***

## :first-child/:last-child
псевдоклассы которые означают что
- first-child первый елемент этого класса будет такой-то
- last-child последний елемент этого класса будет такой-то
свойства у них такие:
```css
.block:first-child{
    background-color:brown
}
.block {
        background-color:rgb(255, 255, 255);
        width: 100px;
        height: 100px;
        border-radius: 10px;
        margin: 10px;
}
```
это означает то что первый елемент класса `block` будет коричневым

# :nth-child(X/Xn)
этот класса принимает аргумент и ставит стили только x елементу
так же же можно использовать каждый n-ный елемент

```css
.block {
        background-color:rgb(255, 255, 255);
        width: 100px;
        height: 100px;
        border-radius: 10px;
        margin: 10px;
}

.block:nth-child(3){
    background-color:rgb(255, 118, 118)
}

%% каждый n-ный %%
.block:nth-child(3n){
    background-color:rgb(255, 118, 118)
}
```
# :not()
принимает в себя другой псевдокласс и отменят его дизайн
```css
.block:nth-child(2n):not(:nth-child(2)){
    background-color:rgb(255, 118, 118)
}
```
