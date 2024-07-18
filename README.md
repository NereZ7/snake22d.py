# snake22d.py
 #Uma tentativa de criação de um jogo em PYTHON.
 
import pygame
import random

# Inicialização do Pygame
pygame.init()

# Definição das constantes do jogo
SCREEN_WIDTH = 900
SCREEN_HEIGHT = 600
GRID_SIZE = 30
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
YELLOW = (255, 255, 0)

# Enum para os estados do jogo
class GameState:
    START = 0
    PLAYING = 1
    GAME_OVER = 2

# Classe da cobrinha
class Snake:
    def __init__(self):
        self.body = [(SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2)]
        self.direction = (1, 0) # Começa indo para a direita
        self.grow_count = 0
        self.color = WHITE

    def move(self):
        # Adiciona uma nova cabeça na direção atual
        new_head = (self.body[0][0] + self.direction[0] * GRID_SIZE,
                    self.body[0][1] + self.direction[1] * GRID_SIZE)
        self.body.insert(0, new_head)
        if self.grow_count == 0:
            self.body.pop()
        else:
            self.grow_count -= 1 

    def grow(self):
        # Adiciona uma nova parte ao corpo (cresce)
        self.grow_count += 1

    def collide_with_self(self):
        # Verifica se a cabeça da cobra colidiu com alguma parte do corpo
        return len(self.body) != len(set(self.body))

    def change_color(self, score):
        # Muda a cor da cobra quando a pontuação atinge 50
        if score >= 500:
            self.color = GREEN

# Função para desenhar a cobra na tela
def draw_snake(snake):
    for segment in snake.body:
        pygame.draw.rect(screen, snake.color, (segment[0], segment[1], GRID_SIZE, GRID_SIZE))

# Função para desenhar a fruta na tela
def draw_fruit(fruit):
    pygame.draw.rect(screen, YELLOW, (fruit[0], fruit[1], GRID_SIZE, GRID_SIZE))

# Função para gerar uma nova posição aleatória para a fruta
def generate_fruit():
    x = random.randint(0, SCREEN_WIDTH - GRID_SIZE) // GRID_SIZE * GRID_SIZE
    y = random.randint(0, SCREEN_HEIGHT - GRID_SIZE) // GRID_SIZE * GRID_SIZE
    return (x, y)

# Função para exibir o texto na tela
def draw_text(text, font, color, surface, x, y):
    text_obj = font.render(text, True, color)
    text_rect = text_obj.get_rect()
    text_rect.topleft = (x, y)
    surface.blit(text_obj, text_rect)

# Configuração da tela
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Snake Game")

# Variáveis de estado do jogo
game_state = GameState.START
score = 0
snake = None
fruit = None # Inicialização da fruta

# Variáveis para piscar a cobra quando perde
blink_timer = 0
blink_interval = 210 # Intervalo de piscagem em milissegundos
blink_color = GREEN # Cor inicial

# Fonte para o texto "Start"
font_start = pygame.font.Font(None, 50)
# Fonte para a mensagem de "Game Over"
font_game_over = pygame.font.Font(None, 16)

# Loop principal do jogo
clock = pygame.time.Clock()
running = True
while running:
    # Limita a taxa de quadros
    clock.tick(10)

    # Processamento de eventos
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.MOUSEBUTTONDOWN:
            # Inicia o jogo quando o jogador clicar o mouse
            if game_state == GameState.START:
                snake = Snake() # Inicialização da cobra ao iniciar o jogo
                fruit = generate_fruit() # Inicialização da fruta ao iniciar o jogo
                game_state = GameState.PLAYING
            # Reinicia o jogo quando o jogador clicar o mouse após a derrota
            elif game_state == GameState.GAME_OVER:
                snake = Snake() # Reinicialização da cobra ao jogar novamente
                fruit = generate_fruit() # Reinicialização da fruta ao jogar novamente
                score = 0
                game_state = GameState.PLAYING

    if game_state == GameState.PLAYING:
        # Processamento do jogo enquanto estiver jogando
        keys = pygame.key.get_pressed()
        if keys[pygame.K_UP] and snake.direction != (0, 1):
            snake.direction = (0, -1)
        elif keys[pygame.K_DOWN] and snake.direction != (0, -1):
            snake.direction = (0, 1)
        elif keys[pygame.K_LEFT] and snake.direction != (1, 0):
            snake.direction = (-1, 0)
        elif keys[pygame.K_RIGHT] and snake.direction != (-1, 0):
            snake.direction = (1, 0)

        # Move a cobra
        snake.move()

        # Verifica se a cobra comeu a fruta
        if snake.body[0] == fruit:
            snake.grow()
            fruit = generate_fruit()
            score += 1
            # Verifica se a cobra deve mudar de cor
            snake.change_color(score)
            # Aumenta o score da fruta a cada 4 pontos
            if score % 4 == 0:
                score += 10

        # Verifica colisões
        if snake.collide_with_self():
            game_state = GameState.GAME_OVER

        # Mantém a cobra dentro dos limites da tela
        if snake.body[0][0] < 0:
            snake.body[0] = (SCREEN_WIDTH - GRID_SIZE, snake.body[0][1])
        elif snake.body[0][0] >= SCREEN_WIDTH:
            snake.body[0] = (0, snake.body[0][1])
        elif snake.body[0][1] < 0:
            snake.body[0] = (snake.body[0][0], SCREEN_HEIGHT - GRID_SIZE)
        elif snake.body[0][1] >= SCREEN_HEIGHT:
            snake.body[0] = (snake.body[0][0], 0)

    # Renderiza a tela
    screen.fill(BLACK)

    if game_state == GameState.START:
        # Exibe a tela de início com a opção de "Start"
        draw_text("Start", font_start, WHITE, screen, SCREEN_WIDTH // 2.8, SCREEN_HEIGHT // 2.8)
    elif game_state == GameState.PLAYING:
        # Exibe o jogo enquanto estiver jogando
        draw_snake(snake)
        draw_fruit(fruit)
        font = pygame.font.Font(None, 36)
        draw_text(f"Score: {score}", font, WHITE, screen, 60, 60)
    elif game_state == GameState.GAME_OVER:
        # Exibe a tela de derrota com a opção de jogar novamente
        draw_snake(snake)
        draw_fruit(fruit)
        font = pygame.font.Font(None, 36)
        draw_text("Game Over", font, YELLOW, screen, SCREEN_WIDTH // 0, SCREEN_HEIGHT // 0)
        draw_text("Clique para jogar novamente", font, WHITE, screen, SCREEN_WIDTH // 0, SCREEN_HEIGHT // 0 + 50)

    # Piscagem da cobra quando perde
    if game_state == GameState.GAME_OVER:
        blink_timer += clock.get_rawtime() # Incrementa o temporizador de piscagem
        if blink_timer >= blink_interval:
            # Troca a cor da piscagem entre branco e vermelho
            if blink_color == WHITE:
                blink_color = RED
            else:
                blink_color = WHITE
            blink_timer = 0 # Reinicia o temporizador de piscagem

        # Desenha a cobra com a cor de piscagem
        for segment in snake.body:
            pygame.draw.rect(screen, blink_color, (segment[0], segment[1], GRID_SIZE, GRID_SIZE))

    pygame.display.flip()

# Encerramento do jogo
pygame.quit()
