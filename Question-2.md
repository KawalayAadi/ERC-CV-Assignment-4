import cv2
import mediapipe as mp
import pygame
import random
import sys


pygame.init()


WIDTH, HEIGHT = 800, 600
SCREEN = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Hand-Tracking Enemy Dodging Game")


WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
BLUE = (0, 0, 255)


PLAYER_WIDTH = 80
PLAYER_HEIGHT = 20
PLAYER_COLOR = BLUE
ENEMY_WIDTH = 50
ENEMY_HEIGHT = 50
ENEMY_COLOR = RED
ENEMY_FALL_SPEED = 7


mp_hands = mp.solutions.hands
hands = mp_hands.Hands(max_num_hands=1)
mp_draw = mp.solutions.drawing_utils


clock = pygame.time.Clock()

def get_hand_position():
    success, img = cap.read()
    img = cv2.flip(img, 1)
    img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    results = hands.process(img_rgb)

    hand_x = WIDTH // 2

    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            mp_draw.draw_landmarks(img, hand_landmarks, mp_hands.HAND_CONNECTIONS)
            hand_x = int(hand_landmarks.landmark[mp_hands.HandLandmark.WRIST].x * WIDTH)

    cv2.imshow("Hand Tracking", img)
    return hand_x


class Player:
    def __init__(self):
        self.x = WIDTH // 2 - PLAYER_WIDTH // 2
        self.y = HEIGHT - 50
        self.width = PLAYER_WIDTH
        self.height = PLAYER_HEIGHT
        self.color = PLAYER_COLOR

    def draw(self):
        pygame.draw.rect(SCREEN, self.color, (self.x, self.y, self.width, self.height))

    def move(self, x):
        self.x = x - self.width // 2
        if self.x < 0:
            self.x = 0
        if self.x + self.width > WIDTH:
            self.x = WIDTH - self.width



class Enemy:
    def __init__(self):
        self.x = random.randint(0, WIDTH - ENEMY_WIDTH)
        self.y = -ENEMY_HEIGHT
        self.width = ENEMY_WIDTH
        self.height = ENEMY_HEIGHT
        self.color = ENEMY_COLOR
        self.speed = ENEMY_FALL_SPEED

    def draw(self):
        pygame.draw.rect(SCREEN, self.color, (self.x, self.y, self.width, self.height))

    def move(self):
        self.y += self.speed



def check_collision(player, enemy):
    return (enemy.x < player.x + player.width and enemy.x + enemy.width > player.x and
            enemy.y < player.y + player.height and enemy.y + enemy.height > player.y)


def main():
    player = Player()
    enemies = []
    score = 0
    run = True

    global cap
    cap = cv2.VideoCapture(0)

    while run:

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False
                break


        player_x = get_hand_position()
        player.move(player_x)


        if random.randint(1, 20) == 1:
            enemies.append(Enemy())


        SCREEN.fill(WHITE)
        player.draw()

        for enemy in enemies[:]:
            enemy.move()
            enemy.draw()


            if check_collision(player, enemy):
                run = False  # End game on collision
                break


            if enemy.y > HEIGHT:
                enemies.remove(enemy)
                score += 1


        pygame.display.update()
        clock.tick(30)

    
    cap.release()
    cv2.destroyAllWindows()
    pygame.quit()
    sys.exit()


if __name__ == "__main__":
    main()
