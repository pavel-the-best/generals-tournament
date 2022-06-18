# generals-tournament

## Disclaimer

Проще понять правила сыграв ```~10``` игр по ссылке [generals.io](https://generals.io), мы почти не меняли правила, ниже формальное описание нашей версии игры.

Мы оставляем за собой право изменять любые константы)

# Правила

## Описание игры

### Условия победы
Игрок выигрывает, когда все остальные проиграли)))

### Карта игры

Игровое поле — табличка из клеток размером ```n * m```, ```1 <= n, m <= 50```.

#### Типы клеток

1. Горы — заблокированная клетка, недоступна для захвата.
2. Пустая клетка — клетка доступная к захвату, раз в ```50``` ходов количество юнитов в каждой вашей клетке увеличивается на ```1```.
3. Город — изначально в ней находится ```35—55``` нейтральных юнитов, раз в ```2``` хода количество юнитов в каждом вашем городе увеличивается на ```1``` (в нейтральных городах количество юнитов не изменяется).
4. Столица — изначально единственная ваша клетка, раз в ```2``` хода количество юнитов в столице увеличивается на ```1```. Игрок проигрывает, когда кто-то захватывает его столицу.

Область видимости — на ```1``` клетку во все стороны (по стороне и углу) от каждой вашей клетки.

```k``` игроков играют на поле. Зачётный турнир будет проводиться ```1 vs 1```, но так же будет проведено какое-то количество произвольных игр, в которых будут участвовать больше двух программ.

Ходы делаются по очереди, очередность ходов определяется случайным образом и фиксируется в начале партии. 

### Варианты действий во время хода
Во время своего хода игрок может сделать одно из следующих действий.
1. Ничего не делать.
2. Передвинуть войска из любой своей клетки ```A — (i, j)``` в произвольную доступную соседнюю по стороне клетку ```B — (i', j')```, ```|i — i'| + |j — j'| = 1```. Если в клетке находится армия размера ```x```, можно передвинуть ```y = x — 1```, или ```x / 2``` округленное вниз юнитов. Если клетка ```B``` принадлежит вам, войска просто перемещаются — пусть вы двигали армию размера ```y```, тогда из количества юнитов в клетке, откуда совершается ход уменьшится на ```y```, количество юнитов в клетке, куда совершается ход увеличится на ```y```. Если же клетка ```B``` вам не принадлежит, пусть вы перемещаете армию размера ```y```, в клетке, в которую вы перемещаете армию ```u >= 0``` юнитов. Из ```y``` и ```u``` вычитается ```min(y, u)```, если остаток ```y > 0```, клетка переходит вам, остаток юнитов перемещается в нее.

При захвате столицы противника, все его клетки переходят вам (его столица превращается в обычный город), количество юнитов во всех новых клетках, кроме столицы уменьшается в два раза (если было нечетное число юнитов в клетке ```x```, вам переходят ```x / 2``` округленные вверх). Игрок, столицу которого захватили — выбывает из игры.

Игра будет ограничена каким-то разумным числом ходов (```1000```, возможно больше), в случае, если игра за это количество ходов не заканчивается, победителем объявляется игрок с наибольшей армией, если таких несколько, с наибольшим количеством городов, если таких тоже несколько, с наибольшим количеством клеток во владении, если и таких несколько, объявляется ничья.

## Протокол взаимодействия:
По сути для вас это будет обычная интерактивка, как только программа жюри готова, она отдаёт видимое вам поле (формат описан ниже), и ждёт от вас ход (TL на ход уточняется и так по кругу пока вы не проиграли/выиграли. 

Да, не забывайте буфер обмена сбрасывать).

### Формат входных данных
Перед тем, как начать играть в единственной строке вводятся четыре числа через пробел ```n, m, k, id```, ```(1 <= n, m <= 50, 1 <= id <= k)``` – размеры поля и количество игроков в начале игры, ваш номер игрока.

Затем, перед каждым вашим ходом вам передаётся информация о текущем положении дел.

В первой строке идёт число, обозначающее, проиграли/выиграли ли вы на прошлом ходу, или нет (```0```, если игра для вас окончена, ```1```, если всё ещё играете). 
Если на вход поступил ```0``` – НЕМЕДЛЕННО завершите работу программы (вызовите ```exit(0)```, или его аналог).

Если на вход поступил ```0```, больше ввода нет, иначе:

В следующих ```k``` строках вводится информация об игроках. 
В ```i-й``` строке вводятся 2 числа через пробел ```army, size```, количество войск у ```i-го``` игрока и суммарное количество захваченных им клеток. Если игрок успел проиграть, оба числа равны нулю.

В последующих ```n * m``` строках вводится информация о клетках. В ```i * m + j``` строке идёт информация о клетке ```(i, j)```.

В начале каждой такой строки идёт число ```v```,  ```(0 <= v <= 1)``` — доступна ли вам информация об этой клетке (```0``` – не видно, ```1``` – видно). 
Через пробел после него идёт число ```t``` ```1 <= t <= 2 + 2 * v``` – информация о типе клетки. 
В случае если вы видите клетку, есть следующие варианты: 1 – пустая клетка, 2 – город (или уже захваченная кем-то столица), 3 – столица, 4 – гора.
Если же вы не видите клетку, есть следующие варианты: 1 – пустая клетка, или столица другого игрока, 2 – всё остальное. 
В случае, если вы видите клетку, и эта клетка не горы ```(v == 1, t != 4)``` далее через пробел записаны 2 числа ```owner, units```,  ```(0 <= owner <= k, 0 <= units)```, номер игрока, которому клетка принадлежит, ```0```, если клетка не принадлежит ни одному из игроков, и размер армии в этой клетке.

Фух ввод вроде описан

### Формат выходных данных

1. Если вы хотите пропустить ход выведите единственное число ```—1``` и перевод строки после него.
2. Иначе выведите число ```option```, ```(1 <= option <= 2)```, ```1```, если вы хотите переместить все войска, кроме ```1``` единицы из клетки, ```2```, если вы хотите переместить половину, затем ```4``` числа ```(i, j)``` клетки, откуда совершается ход, ```(i', j')``` клетки, в которую совершается ход. ```|i — i'| + |j — j'| = 1```. ```1 <= i, i' <= n```, ```1 <= j, j' <= m```.

Учтите, ЛЮБОЙ невалидный ход приравнивается к проигрышу. Все клетки под вашим контролем превращаются в ничейные с нейтральными юнитами (их количество не изменяется), столица превращается в нейтральный город.

Ура вывод тоже описал

### TL

Изначально у вас есть лимит ```2``` секунды на все ходы, к этому лимиту прибавляется по ```0.001``` секунде за каждый сделанный ход. Превышение вашего лимита приравнивается к проигрышу (происходит действие аналогичное невалидному ходу).

# Тестирование

Мы постарались сделать достаточное число тестовых примеров, чтобы было понятно что к чему, есть 3 простых стратегии (отдых, рандом и жадник), готовые примеры лога и конфигов для двух и нескольки игроков. Если вдруг чего-то не хватает — пишите, постараемся добавить.

Если вы гений windows/macos, поставьте, запустите, и расскажите @ElderlyPassionFruit, как вы это сделали. По идее всё написано кроссплатформенно, так что в каком-то виде это должно быть возможно сделать.

### Установка

0. Предустановите библиотеки
    
    Для linux это делается так
    ```
    sudo apt update
    sudo apt upgrade
    sudo apt install make
    sudo apt install cmake
    sudo apt install qt5-default
    ```
    Если вдруг не поставилась последняя библиотека, попробуйте вот так.
    ```
    sudo apt install qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools
    ```
    Если всё ещё не работет, подойдите к Васе, он поможет.

    Для macos это делается так
    ```
    brew install make
    brew install cmake
    brew install qt
    brew install qt@5
    ```
    Если у вас windows — молитесь

1. Откройте терминал, перейдите в директорию, в которой хотите получить этот репозиторий, и скачайте его.
    ```
    git clone https://github.com/ElderlyPassionFruit/generals-tournament.git --recursive
    ```
2. Соберите репозиторий

    Под linux/macos это делается так
    ```
    cd generals-tournament && mkdir build && cd build && cmake .. && make
    ```
    Под windows
    ```
    Перекреститься
    ```
3. В папке ```generals-tournament/bin``` собрались бинарники стратегий, прописанные в ```generals-tournament/examples/CMakeLists.txt```, бинарник ```generals-tournament```, нужный для запуска стратегий и генератор карт ```map-generator```.

4. Осталось собрать визуализатор, для этого перейдите в директорию ```generals-tournament/generals_visual```, и вызовите
    ```
    qmake generals_visual.pro -o Makefile && make
    ```
5. Вы великолепны, в папке ```generals-tournament/generals_visual``` появился бинарник ```generals-visual```.

### Запуск

Чтобы всё это работало, вам нужно иметь 1, или больше, программ — стратегий.

1. Для генерации карты запустите ```generals-tournament/bin/map-generator```, с параметрами командной строки ```n, m, k``` — размеры поля и количество игроков.
Пример запуска:
    ```
    cd generals-tournament/bin && ./map-generator 5 10 3 > ../examples/map.txt
    ```
    Вызов такой команды создаст карту из 5 строк и 10 столбцов для трёх игроков и положит её в файл ```generals-tournament/examples/map.txt```.

2. Для генерации лога вам нужен ```config``` файл, его формат максимально простой: 

    В первой строке, путь к файлу в который нужно положить лог игры
о второй строке, путь к файлу с картой от директории ```generals-tournament```
    
    Во второй строке количество игроков (оно должно быть равно количеству столиц на карте)

    Далее ```k``` строк, в ```i```-й строке путь к файлу с бинарником стратегии для ```i```-го игрока. Посмотрите пример в файле ```generals-tournament/examples/config_example.txt```.

3. Для запуска процесса нужно из директории ```generals-tournament``` запустить скрипт ```game_runner.py``` с единственным параметром командной строки — путём к файлу с конфигом. 

    Пример:
    ```
    python3 game_runner.py examples/config_example.txt
    ```
    После этого, если ваш (и мой) код работает корректно, в файле, в который вы хотели записать лог запишется лог.

4. Чтобы визуализировать лог, перейдите в директорию ```generals-tournament/generals_visual``` и запустите файл ```generals_visual```.
    ```
    ./generals_visual
    ```
    Так же это можно сделать через проводник.
    В открывшемся окне выберите файл с логом. 
5. Наслаждайтесь.
