from pygame import *
from random import randint
# вынесем размер окна в константы для удобства
# W - width, ширина
# H - height, высота
WIN_W = 700
WIN_H = 500
FPS = 60
WHITE = 255, 255, 255
size = 52
step = 4
direction = 'left'
speed = 2
x1 = 60
x2 = 100
GREEN = 0, 255, 0
RED = 255, 0, 0
UFOS = 10
class GameSprite(sprite.Sprite):
    def __init__(self, img, x, y, w, h):
        super().__init__()
        self.image = transform.scale(
            image.load(img),
            # здесь - размеры картинки
            (w, h)
        )
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y
    
    def draw(self, window):
        window.blit(self.image, (self.rect.x, self.rect.y))

class Enemy(GameSprite):
    def __init__(self, img, x, y, w, h, speed=step):
        super().__init__(img, x, y, w, h)
        self.speed = speed
        self.rect.x = randint(0, WIN_W - self.rect.width)
        self.rect.y = randint(0, 40)
    def update(self, is_ufo = True):
        if self.rect.y >= WIN_H:
            self.rect.x = randint(0, WIN_W - self.rect.width)
            self.rect.y = randint(0, 40)
        self.rect.y += self.speed

class Bullet(GameSprite):
    def __init__(self, img, x, y, w, h, speed=step):
        super().__init__(img, x, y, w, h)
        self.speed = speed
    def update(self):
        if self.rect.y <= 0:
            self.kill()
        self.rect.y -= self.speed
class Player(GameSprite):
    def __init__(self, x, y, img, sizeX, sizeY, step):
        super().__init__(img, x, y, sizeX, sizeY)
        self.step = step
        self.shot = 0
        self.missed = 0
        self.bullets = sprite.Group()

    
    def update(self, up, down, left, right):
        keys = key.get_pressed()
        if keys[up] and self.rect.y >= 0:
            self.rect.y -= self.step

        if keys[down] and self.rect.y <= WIN_H - self.rect.height:
            self.rect.y += self.step

        if keys[left] and self.rect.x >= 0:
            self.rect.x -= self.step
        
        if keys[right] and self.rect.x <= WIN_W - self.rect.width:
            self.rect.x += self.step

    def fire(self):
        bullet = Bullet('src/bullet.png', self.rect.x + self.rect.width / 2, self.rect.y, 10, 20)
        self.bullets.add(bullet)

# создание окна размером 700 на 500
window = display.set_mode((WIN_W, WIN_H))
# создание таймера
clock = time.Clock()

# название окна 
display.set_caption("Чё смотришь? Стреляй давай!")

mixer.init()
mixer.music.load('space.ogg')
mixer.music.play(-1)
mixer.music.set_volume(0.5)

win_sound = mixer.Sound('money.ogg')
loose_sound = mixer.Sound('kick.ogg')

font.init()
title_font = font.SysFont('Comic Sans', 70)
win = title_font.render('Лох :) / dumb', True, GREEN)
lost = title_font.render('Победа/ Win!', True, RED)

label_font = font.SysFont('Papyrus', 40)


# задать картинку фона такого же размера, как размер окна
x1 = -100
y1 = 30
background = transform.scale(
    image.load("galaxy.jpg"),
    # здесь - размеры картинки
    (WIN_W, WIN_H)
)
player = Player(0, 365, 'rocket.png', size, size, step)
ufos = sprite.Group()
for i in range(UFOS):
    enemy = Enemy('ufo.png', 0, 0, 70, 50)
    ufos.add(enemy)

# игровой цикл
game = True
finish = False
while game:
    if not finish:
        # отобразить картинку фона
        window.blit(background,(0, 0))

        count = label_font.render(str(player.shot), True, WHITE)
        missed = label_font.render(str(player.missed), True, WHITE)

        player.draw(window)
        player.update(K_UP, K_DOWN, K_LEFT, K_RIGHT)
        ufos.draw(window)
        ufos.update()
        player.bullets.draw(window)
        player.bullets.update()

        sprites_list = sprite.groupcollide(
            ufos, player.bullets, True, True
        )   

        for collide in sprites_list:
            player.shot += 1
            enemy = Enemy('ufo.png', 0, 0, 70, 50)
            ufos.add(enemy)
        if sprite.spritecollide(player, ufos, False):
            window.blit(win, (100, 200))
            display.update()
            finish = True

    # слушать события и обрабатывать
    for e in event.get():
        # выйти, если нажат "крестик"
        if e.type == QUIT:
            game = False
        if e.type == KEYDOWN:
            if e.key == K_SPACE:
                player.fire()
    # обновить экран, чтобы отобрзить все изменения
    display.update()
    clock.tick(FPS)
#задай фон сцены

#создай 2 спрайта и размести их на сцене

#обработай событие «клик по кнопке "Закрыть окно"»
