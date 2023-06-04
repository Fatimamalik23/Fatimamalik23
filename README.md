# Intialize the pygame
import math
import random
import pygame
import os
import time


# Fonts
pygame.font.init()
main_font = pygame.font.SysFont("comicsans", 50)
lost_font = pygame.font.SysFont("comicsans", 60)

# Colors
WHITE = (255, 255, 255)
RED = (255, 0, 0)
GREEN = (0, 255, 0)


# create the screen
width=1200
height=700
ssize=(width,height)
screen = pygame.display.set_mode((ssize))

# Caption and Icon
pygame.display.set_caption("Space Invader")
icon = pygame.image.load('./aircraft.jpg')
pygame.display.set_icon(icon)


# Background
background = pygame.image.load('./background.jpg')


# Load images
RED_SPACE_SHIP = pygame.image.load(os.path.join("assets", "pixel_ship_red_small.png"))
GREEN_SPACE_SHIP = pygame.image.load(os.path.join("assets", "pixel_ship_green_small.png"))
BLUE_SPACE_SHIP = pygame.image.load(os.path.join("assets", "pixel_ship_blue_small.png"))

# Player player
YELLOW_SPACE_SHIP = pygame.image.load(os.path.join("assets", "pixel_ship_yellow.png"))

# Lasers
RED_LASER = pygame.image.load(os.path.join("assets", "pixel_laser_red.png"))
GREEN_LASER = pygame.image.load(os.path.join("assets", "pixel_laser_green.png"))
BLUE_LASER = pygame.image.load(os.path.join("assets", "pixel_laser_blue.png"))
YELLOW_LASER = pygame.image.load(os.path.join("assets", "pixel_laser_yellow.png"))

#sound
# Initialize pygame mixer
pygame.mixer.init()


bullet_sound = pygame.mixer.Sound("bullet_.mp3")
laser_sound=pygame.mixer.Sound("laser_.mp3")

"""
class Score:
    def __init__(self, initial_value=0):
        self.value = initial_value

    def get_value(self):
        return self.value

    def increment_value(self, amount=1):
        self.value += amount

    def reset(self):
        self.value = 0
"""
#@abstraction
class GameObject:
    def __init__(self, x, y, img):
        self.x = x
        self.y = y
        self.img = img
        self.mask = pygame.mask.from_surface(self.img)

    def draw(self, window):
        window.blit(self.img, (self.x, self.y))

    def get_health(self):
        return self.health

class Laser(GameObject):
    def __init__(self, x, y, img):
        self.x = x
        self.y = y
        self.img = img
        self.mask = pygame.mask.from_surface(self.img)

    def draw(self, window):
        window.blit(self.img, (self.x, self.y)) 

    def move(self, vel):
        self.y += vel

    def off_screen(self, height):
        return not(self.y <= height and self.y >= 0)

    def collision(self, obj):
        return collide(self, obj)


class Ship(GameObject):
    COOLDOWN = 30

    def __init__(self, x, y, health=100):
        self.x = x
        self.y = y
        self.health = health
        self.ship_img = None
        self.laser_img = None
        self.lasers = []
        self.cool_down_counter = 0

    def draw(self, window):
        window.blit(self.ship_img, (self.x, self.y))
        for laser in self.lasers:
            laser.draw(window)

    def move_lasers(self, vel, obj):
        self.cooldown()
        for laser in self.lasers:
            laser.move(vel)
            if laser.off_screen(height):
                self.lasers.remove(laser)
            elif laser.collision(obj):
                obj.health -= 10
                self.lasers.remove(laser)

    def cooldown(self):
        if self.cool_down_counter >= self.COOLDOWN:
            self.cool_down_counter = 0
        elif self.cool_down_counter > 0:
            self.cool_down_counter += 1

    def shoot(self):
        if self.cool_down_counter == 0:
            laser = Laser(self.x, self.y, self.laser_img)
            self.lasers.append(laser)
            self.cool_down_counter = 1

    def get_width(self):
        return self.ship_img.get_width()

    def get_height(self):
        return self.ship_img.get_height()




class Player(Ship):
    def __init__(self, x, y, health=100):
        super().__init__(x, y, health)
        self.ship_img = YELLOW_SPACE_SHIP
        self.laser_img = YELLOW_LASER
        self.mask = pygame.mask.from_surface(self.ship_img)
        self.max_health = health

    def move_lasers(self, vel, objs):
        self.cooldown()
        score=0
        for laser in self.lasers:
            laser.move(vel)
            if laser.off_screen(height):
                self.lasers.remove(laser)
            else:
                for obj in objs:
                    if laser.collision(obj):
                        objs.remove(obj)
                        if laser in self.lasers:
                            self.lasers.remove(laser)
                            score+=10
                            
                            laser_sound.play()
                            laser_sound.set_volume(0.5)

    def draw(self, window):
        super().draw(window)
        self.healthbar(window)

    def healthbar(self, window):
        pygame.draw.rect(window, (255,0,0), (self.x, self.y + self.ship_img.get_height() + 10, self.ship_img.get_width(), 10))
        pygame.draw.rect(window, (0,255,0), (self.x, self.y + self.ship_img.get_height() + 10, self.ship_img.get_width() * (self.health/self.max_health), 10))


"""
    def draw(self):
        if self.shield:
            screen.blit(shield_image, (self.x, self.y))



    def activate_shield(self):
        self.shield = True
        self.shield_timer = pygame.time.get_ticks()

    def activate_weapon(self):
        self.weapon = True
        self.weapon_timer = pygame.time.get_ticks()

    def update_power_ups(self):
        if self.shield and pygame.time.get_ticks() - self.shield_timer >= SHIELD_DURATION:
            self.shield = False

        if self.weapon and pygame.time.get_ticks() - self.weapon_timer >= WEAPON_DURATION:
            self.weapon = False


# PowerUp class
class PowerUp:
    def __init__(self, x, y, power_type):
        self.x = x
        self.y = y
        self.width = 50
        self.height = 50
        self.power_type = power_type

    def draw(self):
        if self.power_type == 'shield':
            screen.blit(shield_image, (self.x, self.y))
        elif self.power_type == 'weapon':
            screen.blit(weapon_image, (self.x, self.y))
            

for event in pygame.event.get():
    if event.type == pygame.QUIT:
        game_over = True

    if event.type == pygame.KEYDOWN:
        if event.key == pygame.K_LEFT:
            player.x -= player.vel
        elif event.key == pygame.K_RIGHT:
            player.x += player.vel

        power_type = random.choice(['shield', 'weapon'])
        power_up = PowerUp(random.randint(0, width - 50), 0, power_type)
        power_ups.append(power_up)

        screen.fill((0, 0, 0))

        # Update and draw power-ups
        for power_up in power_ups:
            power_up.y += ASTEROID_SPEED
            power_up.draw()
            if collide(player, power_up):
                if power_up.power_type == 'shield':
                    player.activate_shield()
                elif power_up.power_type == 'weapon':
                    player.activate_weapon()
                power_ups.remove(power_up)

"""


class Enemy(Ship):
    COLOR_MAP = {
                "red": (RED_SPACE_SHIP, RED_LASER),
                "green": (GREEN_SPACE_SHIP, GREEN_LASER),
                "blue": (BLUE_SPACE_SHIP, BLUE_LASER)
                }

    def __init__(self, x, y, color, health=100):
        super().__init__(x, y, health)
        self.ship_img, self.laser_img = self.COLOR_MAP[color]
        self.mask = pygame.mask.from_surface(self.ship_img)

    def move(self, vel):
        self.y += vel

    def shoot(self):
        if self.cool_down_counter == 0:
            laser = Laser(self.x, self.y, self.laser_img)
            self.lasers.append(laser)
            self.cool_down_counter = 1

    def update_score(self, score):
        score.increase(10)        
"""
class GameStats:
    def __init__(self, player, enemies):
        self.player = player
        self.enemies = enemies

    def get_total_score(self):
        total_score = self.player.health
        for enemy in self.enemies:
            total_score += enemy.health
        return total_score
"""
def collide(obj1, obj2):
    offset_x = obj2.x - obj1.x
    offset_y = obj2.y - obj1.y
    return obj1.mask.overlap(obj2.mask, (offset_x, offset_y)) != None

def main():
    run = True
 
    FPS=60

    level = 0
    lives = 5
    score= 0
    

    enemies = []
    wave_length = 5
    enemy_vel = 1

    player_vel = 5
    laser_vel = 3
    player = Player(300, 630)
    
    timer=60000
    clock = pygame.time.Clock()

    lost = False
    lost_count = 0

    def redraw_window():
        screen.blit(background, (0, 0))
    
        
        lives_label = main_font.render(f"Lives: {lives}", 1, (255,255,255))
        level_label = main_font.render(f"Level: {level}", 1, (255,255,255))
        score_label = main_font.render(f"Score: {score}", 1,(255,255,255))
        time_label = main_font.render(f"Time: {timer} ms", 1, (255,255,255))

        screen.blit(lives_label, (10, 10))
        screen.blit(level_label, (width - level_label.get_width() - 10, 10))
        screen.blit(score_label, (10, 60))
        screen.blit(time_label, (10, 100))

        for enemy in enemies:
            enemy.draw(screen)
        player.draw(screen)

        if lost:
            lost_label = lost_font.render("You Lost!!", 1, WHITE)
            screen.blit(lost_label, (width / 2 - lost_label.get_width() / 2, 350))
            score_label = main_font.render("new high score}", 1, WHITE)
            screen.blit(score_label, (width / 2 - score_label.get_width() / 4, 550))
            

        pygame.display.update()

    while run:
     
        clock.tick(FPS)

        if timer>0:
            timer-=1
        redraw_window()

        if lives <= 0 or player.health <= 0 or timer<=0:
            lost = True
            lost_count += 1

        if lost:
            if lost_count > FPS * 3:
                run = False
            else:
                continue

        if len(enemies) == 0:
            level += 1
            wave_length += 5
            for _ in range(wave_length):
                enemy = Enemy(random.randrange(50, width - 100), random.randrange(-1500, -100), random.choice(["red", "green", "blue"]))
                enemies.append(enemy)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False

        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] and player.x - player_vel > 0:  # Left
            player.x -= player_vel
        if keys[pygame.K_RIGHT] and player.x + player_vel + player.get_width() < width:  # Right
            player.x += player_vel
        if keys[pygame.K_UP] and player.y - player_vel > 0:  # Up
            player.y -= player_vel
        if keys[pygame.K_DOWN] and player.y + player_vel + player.get_height() + 15 < height:  # Down
            player.y += player_vel
        if keys[pygame.K_SPACE]:
            player.shoot()


        for enemy in enemies[:]:
             
            enemy.move(enemy_vel)
            enemy.move_lasers(laser_vel, player)

                          

            if random.randrange(0, 2 * 60) == 1:
                enemy.shoot()
                

            if collide(enemy, player):
                player.health -= 10

                bullet_sound.play()
                bullet_sound.set_volume(0.5)
                score-=1 
                enemies.remove(enemy)

                
            elif enemy.y + enemy.get_height() > height:
                lives -= 1
                enemies.remove(enemy)
                score-=2


            for laser in player.lasers:
                if collide(enemy, laser):
                    enemy.health -= 10

                    player.lasers.remove(laser)
                    if enemy.health <= 0:
        
                        enemies.remove(enemy)
                        score+=10


            #player.move_lasers(laser_vel, enemies)
            player.move_lasers(-laser_vel, enemies)


def game_start():

    run = True
    


    while run:
        screen.blit(background, (0, 0))
        start_label = main_font.render("Press the mouse to start", 1, WHITE)
        screen.blit(start_label, (width / 2 - start_label.get_width() / 2, 350))
        pygame.display.update()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False
            if event.type == pygame.MOUSEBUTTONDOWN:
                main()

    pygame.quit()


if __name__ == "__main__":
    game_start()

