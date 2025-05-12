import pygame
import random
import sys

# ИНИЦИАЛИЗАЦИЯ
pygame.init()
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Маг против Ведьмы и Босса")
clock = pygame.time.Clock()
font = pygame.font.SysFont(None, 24)

# ЦВЕТА
WHITE = (255,255,255)
RED = (255,0,0)
BLUE = (50, 50, 255)
GREEN = (0, 255, 0)
BLACK = (0, 0, 0)
PURPLE = (150, 0, 150)
YELLOW = (255, 255, 0)

# ПЕРЕМЕННЫЕ ИГРОКА
player = pygame.Rect(100, 300, 40, 40)
player_speed = 5
player_bullets = []
player_health = 5

# ВЕДЬМА
witch = pygame.Rect(400, 100, 40, 40)
witch_bullets = []
witch_cooldown = 0
witch_health = 5
witch_direction = 1
witch_speed = 2

# БОСС
boss = pygame.Rect(650, 400, 60, 60)
boss_bullets = []
boss_cooldown = 0
boss_direction = -1
boss_speed = 2
boss_health = 15

# GAME LOOP
running = True
while running:
    clock.tick(60)
    screen.fill(BLACK)

    # СОБЫТИЯ
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    # УПРАВЛЕНИЕ
    keys = pygame.key.get_pressed()
    if keys[pygame.K_w]: player.y -= player_speed
    if keys[pygame.K_s]: player.y += player_speed
    if keys[pygame.K_a]: player.x -= player_speed
    if keys[pygame.K_d]: player.x += player_speed
    if keys[pygame.K_SPACE]:
        if len(player_bullets) < 5:
            player_bullets.append(pygame.Rect(player.centerx, player.centery, 10, 5))

    # ДВИЖЕНИЕ БОССА
    boss.y += boss_direction * boss_speed
    if boss.top <= 0 or boss.bottom >= HEIGHT:
        boss_direction *= -1

    # ДВИЖЕНИЕ ВЕДЬМЫ
    witch.y += witch_direction * witch_speed
    if witch.top <= 0 or witch.bottom >= HEIGHT:
        witch_direction *= -1

    # ДВИЖЕНИЕ ПУЛЬ ИГРОКА
    for bullet in player_bullets[:]:
        bullet.x += 10
        if bullet.x > WIDTH:
            player_bullets.remove(bullet)
        elif bullet.colliderect(witch):
            witch_health -= 1
            player_bullets.remove(bullet)
        elif bullet.colliderect(boss):
            boss_health -= 1
            player_bullets.remove(bullet)

    # ПУЛИ ВЕДЬМЫ
    witch_cooldown += 1
    if witch_cooldown > 50:
        witch_bullets.append(pygame.Rect(witch.x, witch.y + 80, 10, 5))
        witch_cooldown = 0
    for bullet in witch_bullets[:]:
        bullet.x -= 6
        if bullet.colliderect(player):
            player_health -= 1
            witch_bullets.remove(bullet)
        elif bullet.x < 0:
            witch_bullets.remove(bullet)

    # ПУЛИ БОССА (НАПРАВЛЕНЫ ТОЛЬКО НА ИГРОКА)
    boss_cooldown += 1  
    if boss_cooldown > 60:
        boss_bullets.append(pygame.Rect(boss.x, boss.y + 90, 20, 10))
        boss_cooldown = 0
    for bullet in boss_bullets[:]:
        bullet.x -= 1
        if bullet.colliderect(player):
            player_health -= 2 # Урон больше!
            boss_bullets.remove(bullet)
        elif bullet.x < 0:
            boss_bullets.remove(bullet)

    # ОТРИСОВКА
    pygame.draw.rect(screen, BLUE, player)
    pygame.draw.rect(screen, GREEN, witch)
    pygame.draw.rect(screen, PURPLE, boss)

    for bullet in player_bullets:
        pygame.draw.rect(screen, WHITE, bullet)
    for bullet in witch_bullets:
        pygame.draw.rect(screen, RED, bullet)
    for bullet in boss_bullets:
        pygame.draw.rect(screen, YELLOW, bullet)

    # ИМЕНА
    screen.blit(font.render("Маг", True, WHITE), (player.x, player.y - 20))
    screen.blit(font.render("Ведьма", True, WHITE), (witch.x, witch.y - 20))
    screen.blit(font.render("БОСС", True, WHITE), (boss.x, boss.y - 20))

    # ЖИЗНИ
    screen.blit(font.render(f"Здоровье мага: {player_health}", True, WHITE), (10, 80))
    screen.blit(font.render(f"Здоровье ведьмы: {witch_health}", True, WHITE), (10, 90))
    screen.blit(font.render(f"Здоровье босса: {boss_health}", True, WHITE), (10, 100))

    pygame.display.flip()

    # ПРОВЕРКА КОНЦА ИГРЫ
    if player_health <= 0:
        print("Ты проиграл!")
        running = False
    elif witch_health <= 0 and boss_health <= 0:
        print("Ты победил всех!")
        running = False

pygame.quit()
sys.exit()
