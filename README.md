import pygame
import random

# تهيئة Pygame
pygame.init()

# إعداد الشاشة
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("لعبة الذئب والأرانب والخراف")

# ألوان
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)

# تحميل الصور
wolf_image = pygame.Surface((50, 50))
wolf_image.fill(RED)
rabbit_image = pygame.Surface((30, 30))
rabbit_image.fill(GREEN)
sheep_image = pygame.Surface((40, 40))
sheep_image.fill(WHITE)

# تحميل الصوتيات
pygame.mixer.init()
rabbit_sound = pygame.mixer.Sound("rabbit_catch.wav")
sheep_sound = pygame.mixer.Sound("sheep_catch.wav")

# الفئة الأساسية للعناصر
class GameObject:
    def __init__(self, x, y, width, height, color):
        self.rect = pygame.Rect(x, y, width, height)
        self.color = color

    def draw(self, screen):
        pygame.draw.rect(screen, self.color, self.rect)

# فئة الذئب
class Wolf(GameObject):
    def __init__(self, x, y):
        super().__init__(x, y, 50, 50, RED)
        self.speed = 5

    def move(self, keys):
        if keys[pygame.K_UP] and self.rect.top > 0:
            self.rect.y -= self.speed
        if keys[pygame.K_DOWN] and self.rect.bottom < HEIGHT:
            self.rect.y += self.speed
        if keys[pygame.K_LEFT] and self.rect.left > 0:
            self.rect.x -= self.speed
        if keys[pygame.K_RIGHT] and self.rect.right < WIDTH:
            self.rect.x += self.speed

# فئة الأهداف المتحركة (أرانب وخراف)
class MovingObject(GameObject):
    def __init__(self, x, y, width, height, color, speed):
        super().__init__(x, y, width, height, color)
        self.speed = speed

    def move_random(self):
        self.rect.x += random.choice([-self.speed, self.speed])
        self.rect.y += random.choice([-self.speed, self.speed])
        # حدود الشاشة
        self.rect.x = max(0, min(self.rect.x, WIDTH - self.rect.width))
        self.rect.y = max(0, min(self.rect.y, HEIGHT - self.rect.height))

# دالة لتحديث النقاط
def update_score(screen, score, font):
    score_text = font.render(f"Score: {score}", True, WHITE)
    screen.blit(score_text, (10, 10))

# دالة لتحديث المؤقت
def update_timer(screen, start_time, font):
    elapsed_time = pygame.time.get_ticks() - start_time
    time_left = max(0, 60 - (elapsed_time // 1000))
    timer_text = font.render(f"Time Left: {time_left}", True, WHITE)
    screen.blit(timer_text, (10, 40))
    return time_left

# إنشاء كائنات اللعبة
wolf = Wolf(WIDTH // 2, HEIGHT // 2)
rabbits = [MovingObject(random.randint(0, WIDTH - 30), random.randint(0, HEIGHT - 30), 30, 30, GREEN, 2) for _ in range(5)]
sheep = [MovingObject(random.randint(0, WIDTH - 40), random.randint(0, HEIGHT - 40), 40, 40, WHITE, 2) for _ in range(3)]

# النقاط
score = 0
font = pygame.font.SysFont(None, 36)

# حلقة اللعبة
running = True
clock = pygame.time.Clock()
start_time = pygame.time.get_ticks()

while running:
    screen.fill(BLACK)

    # معالجة الأحداث
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    # التحكم في الذئب
    keys = pygame.key.get_pressed()
    wolf.move(keys)

    # التحقق من الاصطياد
    for rabbit in rabbits[:]:
        if wolf.rect.colliderect(rabbit.rect):
            rabbits.remove(rabbit)
            score += 5
            rabbit_sound.play()

    for s in sheep[:]:
        if wolf.rect.colliderect(s.rect):
            sheep.remove(s)
            score += 10
            sheep_sound.play()

    # تحريك الأرانب والخراف
    for rabbit in rabbits:
        rabbit.move_random()

    for s in sheep:
        s.move_random()

    # رسم العناصر
    wolf.draw(screen)
    for rabbit in rabbits:
        rabbit.draw(screen)
    for s in sheep:
        s.draw(screen)

    # تحديث النقاط والمؤقت
    update_score(screen, score, font)
    time_left = update_timer(screen, start_time, font)

    # إذا انتهى الوقت
    if time_left == 0:
        running = False
        game_over_text = font.render("Game Over!", True, WHITE)
        screen.blit(game_over_text, (WIDTH // 2 - 100, HEIGHT // 2))

    # تحديث الشاشة
    pygame.display.flip()
    clock.tick(30)

pygame.quit()
