# Написание Пакмана

Игру Пакман можно функционально разделить на несколько частей:
•	Поле игры – Лабиринт;
•	Фишки, разбросанные по полю игры, собирая которые можно выиграть.
•	Главный герой – Пакман; 
•	Противники и логика их работы – Приведения;

## 1. Инициализация Pygame и переменных
В самом начале мы должны инициализировать библиотеки и переменные размера окна и счетчика событий, а также константы, которые мы будем использовать во всем проекте (коды кнопок WASD и RGD коды цветов).
```python
import pygame
import random

BOARD_WIDTH, BOARD_HEIGHT = 19, 22 # размер в клетках
CELL_SIZE = 25

WINDOW_SIZE = WIDTH, HEIGHT = BOARD_WIDTH * CELL_SIZE + 100, BOARD_HEIGHT * CELL_SIZE # добавляем 100 для счетчика очков
TICK = pygame.USEREVENT + 1  # событие, нужно для отсчета одного момента
PACMAN_MOTION = pygame.USEREVENT + 1  # событие для отсчета смены кадра

BLACK = (0, 0, 0)
BLUE = (0, 0, 128)
PINK = (252, 15, 192)

RIGHT_KEY = 100
UP_KEY = 119
LEFT_KEY = 97
DOWN_KEY = 115
KEYS = [RIGHT_KEY, UP_KEY, LEFT_KEY, DOWN_KEY]

pygame.init()
```

## 2. Доска и Обработчик Событий
На данном этапе стоит ввести поле игры, пакман довольно удобен в своем проектировании, поэтому мы можем создать список с вложенными списками и ориентироваться по полю с помощью `board[i][j]`, где 
`i` – координата по строке, 
`j` – координата по столбцу 
Код ниже является примером такой доски

```
board = [[1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
         [1, 0, 0, 0, 3, 0, 0, 0, 3, 1],
         [1, 0, 1, 1, 0, 1, 1, 1, 0, 1],
         [1, 3, 0, 0, 3, 0, 0, 0, 3, 1],
         [1, 0, 1, 1, 0, 1, 1, 1, 3, 1],
         [1, 3, 3, 1, 3, 3, 3, 0, 0, 1],
         [1, 1, 0, 1, 0, 1, 0, 1, 1, 1],
         [1, 3, 3, 0, 3, 1, 3, 0, 3, 1],
         [1, 0, 1, 1, 1, 1, 1, 1, 0, 1],
         [1, 0, 0, 0, 0, 0, 0, 0, 3, 1],
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1]]
```

Еще одной отличительной способностью такого подхода является гибкость самих клеток, так например на нашем поле есть разделения клеток:

`0` - клетка, по которой можно ходить

`1` – стена

`2` - стенка выхода привидений

`3` - узел, расхождение путей

Далее мы вводим класс Board, который рендерит поле игры
```python
class Board:
    def __init__(self, screen):
        self.screen = screen
        BOARD_WIDTH = 19  # ширина поля
        BOARD_HEIGHT = 22  # высота поля
        CELL_SIZE = 25  # размер клетки в пикселях

        self.render()

    def _fill_cell(self, x, y, color):
        pygame.draw.rect(
            self.screen, 
            color, 
            (
                x,
                y,
                CELL_SIZE, 
                CELL_SIZE
            ), 
            width=0)

    def render(self):
        # рендер поля
        for x in range(BOARD_WIDTH):
            for y in range(BOARD_HEIGHT):
                if board_list[y][x] == 0 or board_list[y][x] == 3:
                    self._fill_cell(x * CELL_SIZE, y * CELL_SIZE, BLACK)
                if board_list[y][x] == 1:
                    self._fill_cell(x * CELL_SIZE, y * CELL_SIZE, BLUE)
                if board_list[y][x] == 2:
                    self._fill_cell(x * CELL_SIZE, y * CELL_SIZE, PINK)
	
А также вводим класс Game, который инициализирует класс Board и отвечает за обработку событий
class Game:
    def __init__(self):
        self.screen = pygame.display.set_mode(WINDOW_SIZE)

        board = Board(self.screen)

        pygame.time.set_timer(TICK, 15)
        pygame.time.set_timer(PACMAN_MOTION, 10)

        pygame.display.flip()

        self.run()

    def run(self):
        running = True
        PacmanCurrentKey = ''

        while running:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
            pygame.display.flip()
        pygame.quit()
```

На этом этапе очень важно рассмотреть таймеры.

```python
pygame.time.set_timer(TICK, 15)
pygame.time.set_timer(PACMAN_MOTION, 10)
```

В данный момент я ввожу 2 таймера:
1. Tick – Данный таймер будет отвечать за обработку нажатий, а также столкновения объектов
2. Pacman_Motion – Данный таймер отвечает за перемещения всех объектов в игре (Призраков и Пакмана).

Важно, что числа после названия таймеров являются периодичностью срабатывания таймера (каждые 15 миллисекунд и каждые 10 миллисекунд соответственно). 

Если мы на данном этапе запустим игру, то увидим пустой лабиринт:

<p align="center">
  <img src="/docs/imgs/доска.png" alt="coding" width="50%"/>
</p>

## 3.	Пакман
Вводим главного персонажа игры – Пакмана. У него есть определенный функционал:

1.	Ходить по лабиринту и не проходить сквозь стены
2.	Собирать фишки (которые будут добавлены позже)
3.	Изменять направление своего взгляда в зависимости от направления движения

Для начала определим класс Pacman:

```python
class Pacman(Board):
    def __init__(self, screen):
        super().__init__(screen)
        self.screen = screen
        self.retset = set()

        self.y = 12
        self.x = 9
        self.PacmanCurrentPos = (225, 300)

        self._load_sprite()
        self.pacman_sprites = {
            "R": ['data/pcmn_circ.png', 'data/pcmn_right_2.png', 'data/pcmn_right_3.png', 'data/pcmn_right_2.png'],
            "U": ['data/pcmn_circ.png', 'data/pcmn_up_2.png', 'data/pcmn_up_3.png', 'data/pcmn_up_2.png'],
            "L": ['data/pcmn_circ.png', 'data/pcmn_left_2.png', 'data/pcmn_left_3.png', 'data/pcmn_left_2.png'],
            "D": ['data/pcmn_circ.png', 'data/pcmn_down_2.png', 'data/pcmn_down_3.png', 'data/pcmn_down_2.png'],
        }
        
        self.sprite_direction = "L"
        
        self.currentkey = 0
        self.count = 0
    def _load_sprite(self):
        self.all_sprites = pygame.sprite.Group()
        self.main_pacman_sprite = pygame.sprite.Sprite()
        self.main_pacman_sprite.image = pygame.image.load(
            'data/pcmn_left_3.png'
        )
        self.main_pacman_sprite.rect = self.main_pacman_sprite.image.get_rect()
        self.main_pacman_sprite.add(self.all_sprites)
```

Разберем переменные класса и их использование:
•	self.screen = screen – определяем screen для возможности отображения спрайтов и обновления поля игры в классе Pacman;
•	self.retset = set() – множество, которое используется для определения возможных путей, когда Пакман своим спрайтом заслоняет ровно одну клетку, и ему нужно идти дальше;
•	self.y, self.x = 12, 9 – координаты Пакмана во вложенном списке, в дальнейшем изменяются от его местоположения;
•	self.PacmanCurrentPos = (225, 300) – координаты Пакмана в пикселях на самом игровом поле;
•	self._load_sprite() – вызывает функцию, которая загружает спрайт Пакмана;
•	self.pacman_sprites = {
    "R": [...],
    "U": [...],
    "L": [...],
    "D": [...],
} – Пути в файловой системе до спрайтов Пакмана в различных положения и в различном раскрытии рта;
•	self.sprite_direction = "L" – сохраняет направление головы при повороте пакмана, связан со списком self.pacman_sprites для большего удобства;
•	self.currentkey = 0 – сохраняет код последней нажатой кнопки, для попиксельного смещения пакмана на поле. Хранит одно значения до момента, пока пакман не свернет или не упрется в стену;
•	self.count = 0 – счетчик для смены спрайтов пакмана, нужен для анимации, иттерирует списки в self.pacman_sprites;

### 3.1. Реализация движения
Любое передвижения объектов в играх построено на изменении координат объектов, однако мы имеем некоторое преимущество, так как пакман движется в лабиринте. Схема передвижения пакмана выглядит вот так:

<p align="center">
  <img src="/docs/imgs/scheme.png" alt="coding" width="50%"/>
</p>

Данная схема проверок реализована в двух функциях класса Pacman, которые вызываются из цикла событий:

```python
def pacman_movement(self, key, cy, cx):
    
        y = cy // CELL_SIZE
        x = cx // CELL_SIZE
        self.retset = set()

        horkeycheck = (key == LEFT_KEY or key == RIGHT_KEY)
        verkeycheck = (key == UP_KEY or key == DOWN_KEY)

        ycellcheck = cy % CELL_SIZE == 0
        xcellcheck = cx % CELL_SIZE == 0

        if (horkeycheck and ycellcheck) or (verkeycheck and xcellcheck):
            if board_list[y][(cx + 26) // CELL_SIZE] not in [1, 2]:
                self.retset.add(RIGHT_KEY)
            if board_list[(cy + 26) // CELL_SIZE][x] not in [1, 2]:
                self.retset.add(DOWN_KEY)
            if board_list[y][(cx - 1) // CELL_SIZE] not in [1, 2]:
                self.retset.add(LEFT_KEY)
            if board_list[(cy - 1) // CELL_SIZE][x] not in [1, 2]:
                self.retset.add(UP_KEY)

            if key in self.retset:
                self.currentkey = key
                self.pacman_move(key)
        else:
            self.pacman_move(self.currentkey)
```

Данная функция вызывается из цикла событий и производит проверку – “стоит ли Пакман на ровной клетке?”, то есть совпадает ли положение спрайта пакмана с клеткой на которой он находится, так как спрайты пакмана и клетки равны 25 пикселям, мы можем легко проверить данное условие. 

Если условие выполняется, то мы записываем клавишу на которую нажал игрок в отдельную переменную в цикле событий, чтобы потом поменять направление пакмана в нужную сторону.

Следующая функция непосредственно перемещает пакмана по полю в соответствии с нынешним направлением, которое записано в self.currentkey.

```python
def pacman_move(self, key):
        if key == LEFT_KEY:
            x = (self.PacmanCurrentPos[0] - 1) // CELL_SIZE
            y = (self.PacmanCurrentPos[1]) // CELL_SIZE
            if board_list[y][x] not in [1, 2]:
                self._fill_cell(self.PacmanCurrentPos[0], self.PacmanCurrentPos[1], BLACK)
                self.PacmanCurrentPos = (self.PacmanCurrentPos[0] - 1, self.PacmanCurrentPos[1])
                
                self.sprite_direction = "L"
        elif key == DOWN_KEY:
            x = (self.PacmanCurrentPos[0]) // CELL_SIZE
            y = (self.PacmanCurrentPos[1] + 1) // CELL_SIZE
            if board_list[y][x] not in [1, 2]:
                self._fill_cell(self.PacmanCurrentPos[0], self.PacmanCurrentPos[1], BLACK)
                self.PacmanCurrentPos = (self.PacmanCurrentPos[0], self.PacmanCurrentPos[1] + 1)
                
                self.sprite_direction = "D"
        elif key == RIGHT_KEY:
            x = (self.PacmanCurrentPos[0] + 1) // CELL_SIZE
            y = (self.PacmanCurrentPos[1]) // CELL_SIZE
            if board_list[y][x] not in [1, 2]:
                self._fill_cell(self.PacmanCurrentPos[0], self.PacmanCurrentPos[1], BLACK)
                self.PacmanCurrentPos = (self.PacmanCurrentPos[0] + 1, self.PacmanCurrentPos[1])
                
                self.sprite_direction = "R"
        elif key == UP_KEY:
            x = (self.PacmanCurrentPos[0]) // CELL_SIZE
            y = (self.PacmanCurrentPos[1] - 1) // CELL_SIZE
            if board_list[y][x] not in [1, 2]:
                self._fill_cell(self.PacmanCurrentPos[0], self.PacmanCurrentPos[1], BLACK)
                self.PacmanCurrentPos = (self.PacmanCurrentPos[0], self.PacmanCurrentPos[1] - 1)
                
                self.sprite_direction = "U"
        self.main_pacman_sprite.rect.x = self.PacmanCurrentPos[0]
        self.main_pacman_sprite.rect.y = self.PacmanCurrentPos[1]
```

Чтобы функционал движения заработал, нужно добавить выполнение функций в цикл событий, на данном этапе цикл событий выглядит вот так 

```python
running = True
PacmanCurrentKey = ''

while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN:  # проверка по кнопкам ASDW
            if event.key == LEFT_KEY:  # проверяем A
                PacmanCurrentKey = LEFT_KEY
                self.pacman.pacman_movement(
                    LEFT_KEY, 
                    self.pacman.PacmanCurrentPos[1], 
                    self.pacman.PacmanCurrentPos[0]
                )
            if event.key == DOWN_KEY:  # проверяем S
                PacmanCurrentKey = DOWN_KEY
                self.pacman.pacman_movement(
                    DOWN_KEY, 
                    self.pacman.PacmanCurrentPos[1], 
                    self.pacman.PacmanCurrentPos[0]
                )
            if event.key == RIGHT_KEY:  # проверяем D
                PacmanCurrentKey = RIGHT_KEY
                self.pacman.pacman_movement(
                    RIGHT_KEY, 
                    self.pacman.PacmanCurrentPos[1], 
                    self.pacman.PacmanCurrentPos[0]
                )
            if event.key == UP_KEY:  # проверяем W
                PacmanCurrentKey = UP_KEY
                self.pacman.pacman_movement(
                    UP_KEY, 
                    self.pacman.PacmanCurrentPos[1], 
                    self.pacman.PacmanCurrentPos[0]
                )
        if event.type == TICK:
            self.pacman.pacman_movement(
                PacmanCurrentKey, 
                self.pacman.PacmanCurrentPos[1], 
                self.pacman.PacmanCurrentPos[0]
            )
        if event.type == PACMAN_MOTION:
            self.pacman.motion_counting()
    pygame.display.flip()
pygame.quit()
```

### 3.2.	Реализация анимации

Анимации в игре являются важной частью отклика игры на действия пользователя. Чтобы добавить анимацию в Pygame нужно последовательно сменять спрайты на объекте. Данный функционал реализован с помощью функции motion_counting.

```python
def motion_counting(self):
    # счетчик для смены спрайта пакмана
    self.count = (self.count + 1) % 4

    self.main_pacman_sprite.image = pygame.image.load(
        self.pacman_sprites[self.sprite_direction][self.count]
    )
    self.all_sprites.draw(self.screen)
    pygame.display.flip()
```
	
В данной функции мы добавляем к счетчику единицу и получаем остаток от деления на 4, чтобы не хранить в оперативной памяти миллионные значения счетчика для анимации. Вызов данной функции происходит каждый тик таймера PACMAN_MOTION.

Смена спрайтов исходит из self.count, который принимает значения: 0, 1, 2, 3. Соответственно удобнее будет, если список, содержащий пути к картинкам спрайтов будет иметь длину 4. Поэтому мы вводим словарь self.pacman_sprites, который содержит 4 ключа (соответствующие 4-м направлениям взора пакмана), а в каждом списке содержится 4 пути в таком порядке.

<p align="center">
  <img src="/docs/imgs/спрайты пакмана.png" alt="coding" width="50%"/>
</p>


## 4.	Фишки

Выигрыш немаловажная часть каждой игры, нам нужно добавить фишки, которые при их съедении пакманом будут прибавлять очки победы.

Весь функционал фишек, а также их столкновения с пакманом прописан в классе Dots

```python
class Dots(Pacman, pygame.sprite.Sprite):
    # класс точечек, которые ест пакман
    def __init__(self, screen):
        super().__init__(screen)
        self.screen = screen
        self.dots = pygame.sprite.Group()

        self.score = 0
``` 

В переменной score будет храниться количество съеденных пакманом точек.

Чтобы отрисовать точки, добавляем функцию render_dots

```python
def render_dots(self):
    # рендер точечек
    for y in range(len(board_list)):
        for x in range(len(board_list[y])):
            if board_list[y][x] == 0 or board_list[y][x] == 3:
                self.dot = pygame.sprite.Sprite()
                self.dot.image = pygame.image.load('data/dot.png')
                # спрайт точечек
                self.dot.rect = self.dot.image.get_rect()
                self.dot.add(self.dots)

                self.dot.rect.x = x * CELL_SIZE
                self.dot.rect.y = y * CELL_SIZE
                self.dots.draw(self.screen)
                pygame.display.flip()
``` 

Данная функция будет отрисовывать точки во всех свободных клетках на поле, а также добавит все точки в группу спрайтов для более удобного вычисления столкновений с пакманом.

Нам также нужно вычислять счет, поэтому добавляем еще несколько функций:

```python
def update_dots(self):
    self.dots.draw(self.screen)
    pygame.display.flip()

def update(self, pos):
    self.main_pacman_sprite.rect.x = pos[0]
    self.main_pacman_sprite.rect.y = pos[1]

    if pygame.sprite.spritecollideany(self.main_pacman_sprite, self.dots):
        self.score += 1
        self.score_calc()
    pygame.sprite.spritecollide(
        self.main_pacman_sprite, 
        self.dots, 
        True
    )

def score_calc(self):
    # очки игрока
    font = pygame.font.Font(None, 25)
    text = font.render(f"score {self.score * 10}", True, (255, 255, 0))
    place = text.get_rect(
        center=(525, 20))
    text_w = text.get_width()
    text_h = text.get_height()
    pygame.draw.rect(self.screen, (0, 0, 0), (place[0], place[1],
                                            text_w, text_h), 0)
    self.screen.blit(text, place)

def score_update(self):
    # для главного игрового цикла
    return self.score * 10
```

Каждая функция отвечает за свою логику:

•	update – рассчитывает столкновения пакмана с точками каждый тик и в случае столкновения прибавляет к счету 10 очков.
•	update_dots – обновляет точки на поле, ведь если точка съедена, на поле она отображаться больше не должна.
•	score_calc – выводит на экран счетчик очков, который также обновляется каждый тик.
•	score_update – возвращает значение счетчика count умноженное на 10, ведь выводим мы не количество съеденных точек, а число очков.

Теперь поле выглядит вот так:

<p align="center">
  <img src="/docs/imgs/board with dots.png" alt="coding" width="50%"/>
</p>
 
## 5 Призраки
Собрать фишки чтобы выиграть это хорошо, но нет никакого смысла в игре без противников – добавляем призраков.

Весь функционал призраков будет прописан в классе Ghosts

```python
class Ghosts(Pacman, pygame.sprite.Sprite):
    # класс привидений, где прописана вся их логика
    def __init__(self, screen):
        super().__init__(screen)
        self.screen = screen
        self.CianCurrentPos = (200, 250)
        self.RedCurrentPos = (225, 200)
        self.YellCurrentPos = (225, 250)
        self.PinkCurrentPos = (250, 250)
        # начальное положение привидений
        self.ghosts = pygame.sprite.Group()

        self.pink_cell = [(250, 225), (225, 225), (200, 225)]

        self.ghostpos = []
        self.ghostsmoves = {}
```

Всего призраков будет 4, начальное положение каждого задано соответствующим кортежем для удобства. В переменной ghosts мы будем хранить спрайты всех привидений. В переменной ghotspos хранятся объекты спрайтов призраков, которые в свою очередь содержат информацию о расположении призраков на поле. В переменной ghostmoves содержится направление движения каждого призрака до момента следующей развилки.

Для инициализации привидений мы используем функцию render_ghosts

```python
def render_ghosts(self):
        ghosts_colors = [
            'data/ghostcian.png',
            'data/ghostred.png',
            'data/ghostyellow.png',
            'data/ghostpink.png'
        ]

        for g_color in ghosts_colors:
            self.ghost = pygame.sprite.Sprite()
            self.ghost.image = pygame.image.load(g_color)
            self.ghost.rect = self.ghost.image.get_rect()
            self.ghost.rect.x = self.CianCurrentPos[0]
            self.ghost.rect.y = self.CianCurrentPos[1]
            self.ghost.add(self.ghosts)

            self.ghostsmoves[self.ghost] = ''
            self.ghostpos.append(self.ghost)

        self.ghosts.draw(self.screen)
        pygame.display.flip()
```

Создание спрайтов происходит в цикле, так как с точки зрения объектов они ничем не отличаются, кроме самой картинки спрайта.

```python
for g in self.ghostpos:
    x = g.rect.x 
    y = g.rect.y  
    cx = x // CELL_SIZE  
    cy = y // CELL_SIZE  

    if x % CELL_SIZE == 0 and y % CELL_SIZE == 0:
        oklist = {2, 3, 4}  
        notokey = {1}
        goodmoves = []
        
        wflag = True
        aflag = True
        sflag = True
        dflag = True
        
        
        if board_list[cy][cx] == 3 or board_list[cy][cx] == 4:  
            if board_list[cy + 1][cx] == 2:
                oklist.remove(2)
                notokey.add(2)
            for i in range(1, 9):  
                if (cx + i) <= 18 and (cx - i) >= 0:
                    if board_list[cy][cx + i] in oklist and dflag:
                        goodmoves.append('d')  
                        dflag = False
                    elif board_list[cy][cx + i] in notokey:
                        dflag = False
                    if board_list[cy][cx - i] in oklist and aflag:
                        goodmoves.append('a')  
                        aflag = False
                    elif board_list[cy][cx - i] in notokey:
                        aflag = False

                if (cy + i) <= 21 and (cy - i) >= 0:
                    if board_list[cy + i][cx] in oklist and sflag:
                        goodmoves.append('s')  
                        sflag = False
                    elif board_list[cy + i][cx] in notokey:
                        sflag = False
                    if board_list[cy - i][cx] in oklist and wflag:
                        goodmoves.append('w')  
                        wflag = False
                    elif board_list[cy - i][cx] in notokey:
                        wflag = False

        if len(goodmoves) >= 1:
            move = random.randint(0, len(goodmoves) - 1)  
            self.ghostsmoves[g] = goodmoves[move]
        self.ghost_move(g)  
    else:  #
        self.ghost_move(g)  
    self.ghosts.draw(self.screen)
    pygame.display.update()
```

Движение призраков производится в коде выше. Чтобы призрак сходил мы должны рассчитать возможность того или иного хода, посредством изменения флагов возможности ходов в различных направлениях, чтобы в последствии случайно выбрать тот или иной ход. В конце выбора хода выполняется функция ghost_move.

```python
def ghost_move(self, g):
    # перемещение привидений
    pygame.draw.rect(self.screen, (0, 0, 0), (g.rect.x, g.rect.y,
                                                CELL_SIZE, CELL_SIZE), width=0)
    if self.ghostsmoves[g] == 'w':
        g.rect.y = g.rect.y - 1
    if self.ghostsmoves[g] == 'a':
        g.rect.x = g.rect.x - 1
    if self.ghostsmoves[g] == 's':
        g.rect.y = g.rect.y + 1
    if self.ghostsmoves[g] == 'd':
        g.rect.x = g.rect.x + 1
    self.ghosts.draw(self.screen)
    pygame.display.flip()
``` 

Эта функция выполняет движение приведений посредством попиксельного изменения расположения призраков на поле, в соответствии с выбранным направлением.

На данном этапе призраки и пакман существуют на поле и передвигаются, но все еще не могут сталкиваться друг с другом, а мы не можем проверять столкнулись ли они. Эту проблему решает код ниже:

```python
def collide_pacman(self, pos):  # просчитываем столкновения с памананом
    # столкновение с пакманом
    self.main_pacman_sprite.rect.x = pos[0]
    self.main_pacman_sprite.rect.y = pos[1]
    if pygame.sprite.spritecollideany(self.main_pacman_sprite, self.ghosts):
        return True
    else:
        return False
```

Данное условие производит проверку столкнулся ли спрайт пакмана со спрайтом какого-либо приведения. После добавления условия выше мы должны дополнить код цикла событий следующими строками:

```python
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN:
            ...
        if event.type == TICK:
            score = self.dot.score_update()

            self.ghosts.ghost_calc()
            self.dot.update(self.pacman.PacmanCurrentPos)
            self.dot.update_dots()
            self.pacman.pacman_movement(PacmanCurrentKey, self.pacman.PacmanCurrentPos[1], self.pacman.PacmanCurrentPos[0])
            if self.ghosts.collide_pacman(self.pacman.PacmanCurrentPos):
                self.lose()
            if score == 2050:
                self.win()
        if event.type == PACMAN_MOTION:
            self.pacman.motion_counting()
    pygame.display.flip()
pygame.quit()
```

С этого момента мы добавляем ходы для привидений в каждый тик, а также вводим условия выигрыша по очкам и проигрыша при столкновении с пакманом. Сами функции lose и win показаны ниже. 

```python
def win(self):
    pygame.time.set_timer(TICK, 0)
    pygame.time.set_timer(PACMAN_MOTION, 0)

    font = pygame.font.Font(None, 50)
    text = font.render("You win!", True, (232, 72, 167))
    text_x = 112
    text_y = 150
    place = text.get_rect(
        center=(237, 275))
    text_w = text.get_width()
    text_h = text.get_height()
    pygame.draw.rect(self.screen, (0, 0, 0), (place[0], place[1],
                                        text_w, text_h), 0)
    self.screen.blit(text, place)

def lose(self):
    pygame.time.set_timer(TICK, 0)
    pygame.time.set_timer(PACMAN_MOTION, 0)

    font = pygame.font.Font(None, 50)
    text = font.render("Game Over", True, (232, 72, 167))
    text_x = 112
    text_y = 150
    place = text.get_rect(
        center=(237, 275))
    text_w = text.get_width()
    text_h = text.get_height()
    pygame.draw.rect(self.screen, (0, 0, 0), (place[0], place[1],
                                        text_w, text_h), 0)
    self.screen.blit(text, place)
```

Данные функции являются функциями рендера и делаеют ничего кроме вывода пользователю следующих сообщений.

<p align="center">
  <img src="/docs/imgs/you win.png" alt="coding" width="50%"/>
</p>

<p align="center">
  <img src="/docs/imgs/game over.png" alt="coding" width="50%"/>
</p>
