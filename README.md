
#coding=utf-8
import pygame
from pygame.locals import *
import time
import random

SCREEN_WIDTH = 480
SCREEN_HEIGHT = 852


class Plane(pygame.sprite.Sprite):
    """定义一个飞机的基类"""

    def __init__(self, screen, imageName, init_pos):

        super(Plane, self).__init__()

        # 每架飞机默认的血量
        self.blood = 100

        #设置要显示内容的窗口
        self.screen = screen
        self.image = pygame.image.load(imageName)
        self.rect = self.image.get_rect()
        self.rect.topleft = init_pos  # 初始化矩形的左上角坐标

        #用来存储英雄飞机发射的所有子弹
        self.bulletList = []

        # 被击中时，用来存储播放图片的需要
        self.bomd_index = 0
        # 被击中时，多少次的循环 更换一张图片
        self.bomd_loop_current_num = 0
        self.bomd_loop_all_num = 5

        self.speed = 10

    def display(self, plane_temp):
        #用来存放需要删除的对象引用
        needDelItemList = []

        #保存需要删除的对象
        for i in self.bulletList:
            # 判断子弹是否越界
            if i.judge():
                needDelItemList.append(i)
                continue
            
            # 判断子弹是否击中相应的飞机
            if not plane_temp.is_hit:
                if pygame.sprite.collide_circle(i, plane_temp):
                    # print("击中")
                    needDelItemList.append(i)
                    # 播放被击中的声音
                    plane_temp.bomb_sound.play()
                    plane_temp.is_hit = True  # 被击中了
                    continue
        
        #删除self.bulletList中需要删除的对象
        for i in needDelItemList:
            self.bulletList.remove(i)

        for bullet in self.bulletList:
            bullet.display()#显示一个子弹的位置
            bullet.move()#让这个子弹进行移动，下次再显示的时候就会看到子弹在修改后的位置

        # 判断飞机是否被击中，如果被击中那么就显示相应的动画效果，否则还是显示默认的图片
        if not self.is_hit:
            # 如果没有击中，那么就显示默认的图片
            self.screen.blit(self.image, self.rect)

        else:
            self.bomd_loop_current_num += 1
            if self.bomd_loop_current_num == self.bomd_loop_all_num:
                self.bomd_loop_current_num = 0
                
                self.bomd_index += 1
                if self.bomd_index >= len(self.bomb_images):
                    self.is_hit = False
                    self.bomd_loop_current_num = 0
                    self.bomd_index = 0
                    self.blood -= 30

            # 注意下面的这句话，不要放到了if里，否则会闪烁
            self.screen.blit(self.bomb_images[self.bomd_index], self.rect)        


class HeroPlane(Plane):
    """英雄飞机"""

    def __init__(self, screen):
        super(HeroPlane, self).__init__(screen, "./feiji/hero1.png", [230, 700])  # [230, 700]是这个飞机默认的左上角坐标

        # 飞机被击中时播放的声音
        self.bomb_sound = pygame.mixer.Sound('./sound/game_over.wav')
        self.bomb_sound.set_volume(0.3)

        # 用来存储声音对象
        self.bullet_sound = pygame.mixer.Sound('./sound/bullet.wav')
        self.bullet_sound.set_volume(0.3)

        # 被击中时，播放的动画
        self.bomb_images = []
        self.bomb_images.append(pygame.image.load("./feiji/hero1.png"))
        self.bomb_images.append(pygame.image.load("./feiji/hero_blowup_n1.png"))
        self.bomb_images.append(pygame.image.load("./feiji/hero_blowup_n2.png"))
        self.bomb_images.append(pygame.image.load("./feiji/hero_blowup_n3.png"))
        self.bomb_images.append(pygame.image.load("./feiji/hero_blowup_n4.png"))

        # 默认是没有被击中的
        self.is_hit = False

    def moveUp(self):
        if self.rect.top <= 0:
            self.rect.top = 0
        else:
            self.rect.top -= self.speed

    def moveDown(self):
        if self.rect.top >= SCREEN_HEIGHT - self.rect.height:
            self.rect.top = SCREEN_HEIGHT - self.rect.height
        else:
            self.rect.top += self.speed

    def moveLeft(self):
        if self.rect.left <= 0:
            self.rect.left = 0
        else:
            self.rect.left -= self.speed

    def moveRight(self):
        if self.rect.left >= SCREEN_WIDTH - self.rect.width:
            self.rect.left = SCREEN_WIDTH - self.rect.width
        else:
            self.rect.left += self.speed

    def sheBullet(self):
        # 发射子弹
        newBullet = Bullet(self.rect.left, self.rect.top, self.screen, "hero")
        self.bulletList.append(newBullet)

        # 用来存储声音对象
        self.bullet_sound.play()


class EnemyPlane(Plane):
    """敌机类"""

    def __init__(self, screen):
        super(EnemyPlane, self).__init__(screen, "./feiji/enemy0.png", [0, 0])

        self.direction = "right"

        # 这架敌机被击中时播放的声音
        self.bomb_sound = pygame.mixer.Sound('./sound/enemy1_down.wav')
        self.bomb_sound.set_volume(0.3)

        # 被击中时，播放的动画
        self.bomb_images = []
        self.bomb_images.append(pygame.image.load("./feiji/enemy0.png"))
        self.bomb_images.append(pygame.image.load("./feiji/enemy0_down1.png"))
        self.bomb_images.append(pygame.image.load("./feiji/enemy0_down2.png"))
        self.bomb_images.append(pygame.image.load("./feiji/enemy0_down3.png"))
        self.bomb_images.append(pygame.image.load("./feiji/enemy0_down4.png"))

        # 默认是没有被击中的
        self.is_hit = False

    def move(self):
        #如果碰到了右边的边界，那么就往左走，如果碰到了左边的边界，那么就往右走
        if self.direction == "right":
            self.rect.left += 4
        elif self.direction == "left":
            self.rect.left -= 4

        if self.rect.left > 480-50:
            self.direction = "left"
        elif self.rect.left < 0:
            self.direction = "right"

    def sheBullet(self):
        num = random.randint(1,50)
        if num == 38:
            newBullet = Bullet(self.rect.left, self.rect.top, self.screen, "enemy")
            self.bulletList.append(newBullet)


class Bullet(pygame.sprite.Sprite):
    """子弹类，将敌机发射的子弹、英雄发射的子弹合并到这个类中了"""

    def __init__(self,x,y,screen,planeName):

        super(Bullet, self).__init__()

        self.name = planeName
        self.screen = screen

        if self.name == "hero":
            x += 40
            y -= 20
            imageName = "./feiji/bullet.png"

        elif self.name == "enemy":
            x += 30
            y += 30
            imageName = "./feiji/bullet1.png"
        self.image = pygame.image.load(imageName)
        self.rect = self.image.get_rect()
        self.rect.topleft = [x, y]
    
    def move(self):
        if self.name == "hero":
            self.rect.top -= 6
        elif self.name == "enemy":
            self.rect.top += 6

    def display(self):
        self.screen.blit(self.image, self.rect)

    def judge(self):
        if self.rect.top>SCREEN_HEIGHT or self.rect.top<0:
            return True
        else:
            return False


def key_control(heroPlane):
    """用来检测玩家按下的键盘"""

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            exit()
            
    # 监听键盘事件
    key_pressed = pygame.key.get_pressed()  # 注意这种方式是能够检测到连续按下的，比之前的版本要新
    # 若玩家被击中，则无效
    if key_pressed[K_w] or key_pressed[K_UP]:
        heroPlane.moveUp()
    if key_pressed[K_s] or key_pressed[K_DOWN]:
        heroPlane.moveDown()
    if key_pressed[K_a] or key_pressed[K_LEFT]:
        heroPlane.moveLeft()
    if key_pressed[K_d] or key_pressed[K_RIGHT]:
        heroPlane.moveRight()
    if key_pressed[K_SPACE]:
        heroPlane.sheBullet()


def init_pygame():
    """进行初始化，否则的话像声音等都不能使用"""

    pygame.init()
    pygame.mixer.music.load('./sound/game_music.wav')
    pygame.mixer.music.set_volume(0.05)
    pygame.mixer.music.play(-1, 0.0)


def score(screen, hero_plane, enemy_plane):
    """绘制每个飞机的血量信息"""

    # 绘制玩家飞机血量
    score_font = pygame.font.Font(None, 36)
    score_text = score_font.render(str(hero_plane.blood), True, (128, 128, 128))
    text_rect = score_text.get_rect()
    text_rect.topleft = [10, 10]
    screen.blit(score_text, text_rect)

    # 绘制敌机飞机血量
    score_font = pygame.font.Font(None, 36)
    score_text = score_font.render(str(enemy_plane.blood), True, (128, 128, 128))
    text_rect = score_text.get_rect()
    text_rect.topleft = [10, 50]
    screen.blit(score_text, text_rect)


def judge_win_or_lose(hero_plane, enemy_plane):
    """用来完成输赢的判断"""

    if hero_plane.blood <= 0 and enemy_plane.blood > 0:
        return "lose"
    elif hero_plane.blood >=0 and enemy_plane.blood <= 0:
        return "win"


def main():
    """整体的控制"""

    # 0. 初始化pygame系统
    init_pygame()

    #1. 创建一个窗口，用来显示内容
    screen = pygame.display.set_mode((480,852),0,32)

    #2. 创建一个和窗口大小的图片，用来充当背景
    background = pygame.image.load("./feiji/background.png")

    #3.1 创建一个飞机对象
    heroPlane = HeroPlane(screen)
    #3.2 创建一个敌人飞机
    enemyPlane = EnemyPlane(screen)

    # 获取时间对象，用来控制刷新频率
    clock = pygame.time.Clock()

    result = None

    # 主循环，用来循环所有的事情：键盘检测、飞机绘制、子弹绘制、输赢判断等
    while True:
        #4. 把背景图片放到窗口中显示
        screen.blit(background,(0,0))

        heroPlane.display(enemyPlane)

        enemyPlane.move()
        enemyPlane.display(heroPlane)
        enemyPlane.sheBullet()

        key_control(heroPlane)

        score(screen, heroPlane, enemyPlane)

        result = judge_win_or_lose(heroPlane, enemyPlane)
        if result is not None:
            break

        pygame.display.update()

        # 控制游戏最大帧率为60
        clock.tick(90)


    # 整理游戏结束后显示的输赢结果
    font = pygame.font.Font(None, 48)
    # 显示最终的输赢结果
    if result == "win":
        text = font.render("Very NB", True, (255, 0, 0))
    else:
        text = font.render("You are loser", True, (255, 0, 0))        
    
    text_rect = text.get_rect()
    text_rect.centerx = screen.get_rect().centerx
    text_rect.centery = screen.get_rect().centery + 24
    game_over = pygame.image.load('./feiji/gameover2.png')
    screen.blit(game_over, (0, 0))
    screen.blit(text, text_rect)

    # 等待用户关闭游戏窗口
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
        pygame.display.update()
        clock.tick(100)

if __name__ == "__main__":
    main()
