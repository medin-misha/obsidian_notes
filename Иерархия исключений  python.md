Все-все-все ошибки наследуются от `BaseException` это стандартная ошибка, которую используют другие ошибки говоря нам более конкретно что сломалось.

Есть некая иерархия `BaseException -> Exception -> OSError -> FileNotFound`
и если мы хотим обработать `FileNotFound`и`OSError` то мы должны в сначала проверять более спецефичную ошибку чем более общую. То есть сначала `FileNotFoundError` а потом `OSError` по тому что `OSError` родитель `FileNotFound`


![[Pasted image 20240319161559.png]]
[[python]]