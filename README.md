# SPACE-INVADER-GAME
import pygame
import random  # for enemies to come at random...so import this package
import math  # for applying the distance formula
from pygame.constants import NUMEVENTS
from pygame import mixer  # handles all kind of music in our game

# Initialize the pygame
pygame.init()

# To access the methods inside a pygame module
# To create the screen
# width and the height of the screen
screen = pygame.display.set_mode((800, 600))

# Background
background = pygame.image.load('background.jpg')

# Background music
mixer.music.load('background.wav')
mixer.music.play(-1)  # adding -1 means play on loop

# Title & Icon
pygame.display.set_caption("SPACE INVADERS")
icon = pygame.image.load('space-invaders.png')  # creating a icon variable
pygame.display.set_icon(icon)

# PLAYER
playerImg = pygame.image.load('space-invaders.png')
playerX = 370  # closer to middle..including the image we give 370, and not 400
playerY = 480  # closer to 600..beneath
playerX_change = 0  # if left or right keystroke is pressed

# Bullet
bulletImg = pygame.image.load('bullet.png')
bulletX = 0
bulletY = 480
bulletX_change = 0  # since bullet is not going to move in x-direction....so 0
bulletY_change = 1  # bullet speed
bullet_state = "ready"  # ready --> you cant see the bullet on the screen

# ENEMY
enemyImg = []     # for creating multiple enemies
enemyX = []
enemyY = []
enemyX_change = []
enemyY_change = []
num_of_enemies = 6

for i in range(num_of_enemies):
    enemyImg.append(pygame.image.load('enemy.png'))
    # method inside random package to choose random integers between to values
    enemyX.append(random.randint(0, 735))
    enemyY.append(random.randint(50, 150))
    enemyX_change.append(0.2)
    enemyY_change.append(40)  # so that we can move it down by 40 pixels

score = 0  # the initial score of the game...which will be manipulated in collision in while loop
# SCORE -->> ADDING TEXT AND DISPLAYING SCORE
score_value = 0
# ttf is the extension of the font and 32 is the size
font = pygame.font.Font('freesansbold.ttf', 32)
# since we want he text to appear on the top left corner
textX = 10  # 10 pixels
textY = 10  # 10 pixels

# GAME_OVER_TEXT
over_font = pygame.font.Font('freesansbold.ttf', 64)


def show_score(x, y):  # function created to show score
    # instead of blit , we first render and blit the text on screen
    score = font.render("score: " + str(score_value), True, (255, 255, 255))
    screen.blit(score, (x, y))


def game_over_text(x, y):
    over_text = over_font.render("GAME OVER", True, (255, 255, 255))
    # 200 and 250 pixels are the middle of the screen
    screen.blit(over_text, (200, 250))


def player(x, y):  # this function should be called inside the while loop...to keep the player running
    # it means to basically to draw an image on screen
    screen.blit(playerImg, (x, y))


def enemy(x, y, i):  # this function should be called inside the while loop...to keep the player running
    # it means to basically to draw an image on screen
    screen.blit(enemyImg[i], (x, y))


def fire_bullet(x, y):
    global bullet_state  # global so we access the variable even inside the function
    bullet_state = 'fire'
    # x+16 and y+10 because to appear the bullet from the center of spaceship and little bit on top of the spaceship
    screen.blit(bulletImg, (x+16, y+10))

# defines whether the collision between the bullet and enemy has occurred.


def isCollision(enemyX, enemyY, bulletX, bulletY):
    # will store the distance between the bullet and our enemy
    distance = math.sqrt((math.pow(enemyX-bulletX, 2)) +
                         (math.pow(enemyY-bulletY, 2)))
    # this is the distance formula
    if distance < 27:  # Through trial and error I found out the best distance between enemy and bullet to call it a collision
        return True
    else:
        return False


# Game loop
# To make sure the screen stays ..we need an infinite loop
running = True
while running:  # event is anything that happens inside your window
    # to change the background color of screen # the tuple is rgb color
    screen.fill((0, 0, 0))

    # background image
    # co-ordinates..since we need it to appear from top-left corner
    screen.blit(background, (0, 0))

    # we need to loop through all the events in the game window..it makes sure that all of the events that are happening get into this pygame.event.get()
    # and then we can loop all of these events using for loop
    # playerX += 0.01 #Movement mechanics in game
    # playerY -= 0.01
    for event in pygame.event.get():  # every event gets logged inside pygame.event.get()
        if event.type == pygame.QUIT:  # we check if the quit function has been accessed ...ie close button has been pressed
            # Then running = False
            running = False
        # if keystroke is pressed check whether right or left
        if event.type == pygame.KEYDOWN:  # this checks if any keystroke is pressed
            if event.key == pygame.K_LEFT:  # if keystroke is pressed ...then checks if left arrow is pressed
                playerX_change = -0.5  # This helps in moving the player towards left if K_LEFT pressed
            if event.key == pygame.K_RIGHT:
                playerX_change = 0.5  # same as above
            if event.key == pygame.K_SPACE:
                if bullet_state is 'ready':
                    bullet_sound = mixer.Sound(
                        'shooting.wav')  # for shooting sound
                    bullet_sound.play()
                    # now the bullet wont change along the movement of spaceship  (and gets the current co-ordinate of the spaceship)
                    bulletX = playerX
                    # playerX ..since it changes with the direction of spaceship
                    fire_bullet(bulletX, bulletY)

        if event.type == pygame.KEYUP:  # to check when the keystroke is released
            if event.key == pygame.K_LEFT or event.key == pygame.K_RIGHT:
                playerX_change = 0

    # Checking for boundaries for spaceship so it doesn't go out of bounds
    # Player movement
    # it doesn't matter if it is added...since given -0.1 and 0.1 above
    playerX += playerX_change
    # Adding boundaries to the game
    if playerX <= 0:      # here only for x axis since only left and right is given
        playerX = 0
    elif playerX >= 736:  # since png image 64*64 pixels ...so 800-64 = 736
        playerX = 736

    # Multiple Enemy movement
    for i in range(num_of_enemies):
        # GAME OVER
        if enemyY[i] > 440:  # since spaceship at 480th pixel ....so 440 would be idle
            for j in range(num_of_enemies):  # to remove all the enemies out of the screen
                enemyY[j] = 2000  # arbitrary no out of the screen plane
            game_over_text(200, 250)
            break

        # we need to specify which enemy we are here for...since multiple enemies created
        enemyX[i] += enemyX_change[i]
        if enemyX[i] <= 0:
            # ie when it hits the left boundary it moves towards the right direction
            enemyX_change[i] = 0.5
            # if it hits the boundary it should move down by 40 pixels
            enemyY[i] += enemyY_change[i]
        elif enemyX[i] >= 736:
            # when it hits the right direction ...it moves towards the left
            enemyX_change[i] = -0.5
            enemyY[i] += enemyY_change[i]

        # collision detection
        collision = isCollision(enemyX[i], enemyY[i], bulletX, bulletY)
        if collision:
            explosion_sound = mixer.Sound('explosion.wav')
            explosion_sound.play()
            bulletY = 480  # reset the bullet to starting point
            bullet_state = 'ready'
            score_value += 1
        #    print(score)
            enemyX[i] = random.randint(0, 735)
            enemyY[i] = random.randint(50, 150)

        # called the enemy function [i] and which enemy is to be blit..
        enemy(enemyX[i], enemyY[i], i)

    # bullet movement
    if bulletY <= 0:
        bulletY = 480
        bullet_state = 'ready'

    if bullet_state is "fire":
        # for bullet to appear continuously on the screen..
        fire_bullet(bulletX, bulletY)
        bulletY -= bulletY_change

    player(playerX, playerY)  # called the player function
    # call it in while loop for displaying text on screen continuously...
    show_score(textX, textY)
    # if we add anything inside display window ...we need to update...
    pygame.display.update()
