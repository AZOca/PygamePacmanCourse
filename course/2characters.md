# Глава 2: Создание Персонажей и Объектов

В предыдущей главе мы изучили основы Pygame: инициализацию, создание окна, обработку событий и рисование простых фигур. Теперь пришло время сделать наши игры более интересными! В этой главе мы узнаем, как создавать персонажей и объекты, которые смогут двигаться, взаимодействовать друг с другом и оживлять нашу игровую вселенную.

### 2.1. Поверхности (Surfaces):

Напомним, что поверхность (Surface) в Pygame – это прямоугольный участок, на котором мы можем рисовать. Обычно мы используем объект screen как главную поверхность для отображения игры. 

### 2.2. Создание персонажа:

Для создания персонажа мы можем использовать несколько подходов:

• Прямоугольник: Простейший вариант – использовать pygame.Rect():

```python
import pygame

# ... (инициализация Pygame, создание окна)

player_rect = pygame.Rect(100, 300, 50, 50) # Создаем прямоугольник для игрока

running = True

while running:
  # ... (обработка событий)

  screen.fill((255, 255, 255)) # Заливка экрана белым
  pygame.draw.rect(screen, (0, 255, 0), player_rect) # Отрисовка прямоугольника игрока
  
  # ... (обновление экрана)

pygame.quit()
```

• Изображение: Мы можем загрузить изображение и использовать его как визуальное представление персонажа:

```python
import pygame

# ... (инициализация Pygame, создание окна)

player_image = pygame.image.load("player.png").convert_alpha() # Загружаем изображение персонажа
player_rect = player_image.get_rect(center=(100, 300)) # Создаем Rect с центром в (100, 300)

running = True

while running:
  # ... (обработка событий)

  screen.fill((255, 255, 255)) # Заливка экрана белым
  screen.blit(player_image, player_rect) # Отображение изображения персонажа

  # ... (обновление экрана)

pygame.quit()
```

### 2.3. Перемещение персонажа:

Чтобы сделать персонажа подвижным, мы будем обновлять его координаты в цикле игры:

```python
import pygame

# ... (инициализация, создание окна, загрузка изображения персонажа)

player_speed = 5 # Скорость движения

running = True

while running:
  # ... (обработка событий)

  keys = pygame.key.get_pressed() # Получаем состояние всех клавиш
  if keys[pygame.K_LEFT]:
    player_rect.x -= player_speed 
  if keys[pygame.K_RIGHT]:
    player_rect.x += player_speed

  # ... (очистка экрана, отрисовка, обновление экрана)

pygame.quit()
```

### 2.4. Создание объектов:

Процесс создания других объектов (например, препятствий, врагов, бонусов) очень похож на создание персонажа. Вы можете использовать прямоугольники, изображения или даже сочетание обоих методов.

### 2.5. Столкновения:

Чтобы реализовать взаимодействие между объектами, нам необходимо уметь определять, столкнулись ли они. Pygame предоставляет метод colliderect(), который проверяет, пересекаются ли два прямоугольника:

```python
if player_rect.colliderect(enemy_rect):
  # Персонаж столкнулся с врагом
```

### 2.6. Пример: Простая игра с движущимся персонажем:

```python
import pygame

pygame.init()
screen = pygame.display.set_mode((800, 600))
pygame.display.set_caption("Движущийся Персонаж")

player_image = pygame.image.load("player.png").convert_alpha()
player_rect = player_image.get_rect(center=(100, 300))

player_speed = 5

running = True
while running:
  for event in pygame.event.get():
    if event.type == pygame.QUIT:
      running = False

  keys = pygame.key.get_pressed()
  if keys[pygame.K_LEFT]:
    player_rect.x -= player_speed
  if keys[pygame.K_RIGHT]:
    player_rect.x += player_speed

  screen.fill((255, 255, 255))
  screen.blit(player_image, player_rect)
  pygame.display.flip()

pygame.quit()
```

### Заключение:
В этой главе мы узнали, как создавать персонажей и объекты в Pygame, а также как сделать их подвижными и реализовать взаимодействие между ними. В следующих главах мы продолжим развивать наши навыки, создавая более сложные игры.
