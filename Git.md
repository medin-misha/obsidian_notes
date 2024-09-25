Git это система контроля версий. Git умеет в командную работу. Система контроля версий нужна для контроля над версиями проэкта и обеспечения командной работы.

Git - это самая крутая система контроля версий с открытым исходным кодом + он компактен и быстро-работающий + система веток для отдельных задач + можно копировать ВЕСЬ репозиторий и синхронизировать изменения.

#  как выглядит работа с git
для начала нам нужен репозиторий(проэкт) с которым мы будем работать в git. Он будет таким https://github.com/sortedmap/git-basics.

Что бы скопировать сам проэкт нужно использовать команду clone
```git
git clone https://github.com/sortedmap/git-basics
```
и в той директории в которой мы находимся создаёться папка с вот такими фалами
![[Pasted image 20240825201950.png]]
всё прошло успешно. Если написать команду `git status` мы сможем увидеть инфу о текущей ветке и какой ветке она соотвецтвует. Так же информацию о коммитах.

Если мы что то изменим в index.html или в любом другом файле этого репозитория и введём `git status` то мы увидим что изменён какой то файл и указка на сам файл
![[Pasted image 20240825202505.png]]
теперь если мы напишем 
```
git add .
git commit -m "описание изменений"
```
то при вводе `git status` нам скажут что наша ветка опережает orgigin/master на 1 коммит
# работа с git в командной строке
Если мы зайдём в рандомную папку и напишем там `git status` то нас вежливо проинформируют ФАТАЛЬНОЙ ОШИБКОЙ что git репозиторий не найден. Что бы его создать нужно написать `git init`
теперь у нас инициализирован пустой репозиторий, а в `.git` директории. Это директория которая содержит в себе всю историю изменений в проэкте. 

Если мы добавим директорию в которой есть папка .git что нибуть и напишем `git status` то нам скажут что найден новый файл.

на этом моменте нужно сохраниться командами
```
git add .
git commit -m "создал файлик"
```
`add` добавляет в сохранение указанные файлы в данном случае все которые находяться в деректории где выполняеться команда
`commit` это как кнопка `сохранить` `-m "создал файлик"` это имя сохраниения.

Для того что бы быстро глянуть на изменения существует команда `git show` что бы двигаться по git show нужно использовать стрелочки, а что бы выйти из git show нужно нажать `Q`.

Если мы изменим какой нибуть файлик Git это сразу заметит `git status` и нам покажеться информация о том что что то изменилось.

снова сохранимся
```
git add .
git commit -m "что то изменил"
```

теперь глянем как на историю коммитов `git log`
![[Pasted image 20240825204718.png]]
# жизненный цикл файлов в репозитории
В гит репозитории файлы могут находиться в нескольких состояниях
- закоммичен (сохранён)
- изменён добавлен в index гита то есть подготовлен к коммиту
- удалённый
добавить в индекс файл можно командой `git add path`. `git status` говорит нам в каком состоянии файл.

# игнор файлов
Для того что бы игнорировать файлы которые не должны войти в итоговый репозиторий а это прежде всего
- логи
- файлы которые не влияют на работу проэкта
- файлы с паролями
- бызы данных
- виртуальные окружения
- конфиг редактора кода
- пользовательские файлы
нужно использовать .gitignore
создадим файл .gitignore
```.gitignore
*.log  
logs/  
*.db  
database/
```
и папку `logs[err.log, info.logs]` а так же `database[database.db]`
теперь `git status`
![[Pasted image 20240825211957.png]]
 тут найден git ignore но файлы и папки которые создали не видно и в репозиторий они не попадут.

Если понадобиться исключить все файлы кроме опеределённого то мы используем игнорирование паттернов:

```
!logs/err.log
```
тут мы !отрицаем что мы игнорируем файл. При этом нельзя если использовать отрицание игнорить папку в которой есть файл который нужен.
# создание SSH ключа
Secure Shell (SSH) - способ общения с уалённым сервером или репозиторием. Его разница от HTTPS в том что в HTTPS придёться вводить пароль каждый раз а в SSH не придётся. То есть настроил и погнал.
## алгоритм созлания SSH ключа
1. в терминале вводим `ssh-keygen -t ed25519 -C "skillbox"`. Аргумент -t говорит о том какой будет тип у ключа. Параметр -С комментарий к SSH-ключу типо имя.
2. нужно ответить на вопросы в терминале 
	1. `Enter file in which to save the key (/home/mihsa/.ssh/id_ed25519):` тут у нас спрашиваеться в какой директории будет находиться ключ. Если нажать Enter то она поставиться по умолчанию
	2. `Enter passphrase (empty for no passphrase):` это ключивая фраза ключа то есть как бы его пароль. Если нажать Enter то она не будет поставлена
	3. `Enter same passphrase again` попросит ввести тот же пароль ещё раз но изза того что ранее мы поставили Enter то и сейчас Enter
	если вывелась вот такая штука то ты всё сделал правильно
	```python
	Your identification has been saved in /home/mihsa/.ssh/id_ed25519
	Your public key has been saved in /home/mihsa/.ssh/id_ed25519.pub
	The key fingerprint is:
	SHA256:EWcQZI4wzXKS+zKorVH9/eSNHK023zG73iMQ6yOoYyA skilbox
	The key's randomart image is:
	+--[ED25519 256]--+
	|    o+ .*o o     |
	|   +o++ +        |
	|     =. o        |
	|   ..    . .     |
	|  ....  S    o   |
	| E..o...+        |
	|.o. .o. o +o  o  |
	|...    o.*oB o * |
	|..    ..o.Bo+.*.o|
	+----[SHA256]-----+
	```
3. теперь введя команду cat `~/.ssh/id_ed25519.pub` мы выведем этот ключ
4. заходим на github profile>settigs>SSH and GPG keys>mew ssh key> и в форму копируем `~/.ssh/id_ed25519.pub`
5. клонируем созданый мной репозиторий `git clone git@github.com:medin-misha/learn_git.git` если будет вопрс на него отвечаем yes
готово ты скопировал репозиторий

## как явно указать какой SSH ключ нужно использовать для конкретной странички
В папке `~/.ssh/` создай файл config 
```shell
touch ~/.ssh/config
```
и напиши в этом файле следуйщее
```python
Host github-medin-misha
        HostName github.com
        User git
        IdentityFile ~/.ssh/id_ed25519                                         
```
# подключение к удалённому репозиторию