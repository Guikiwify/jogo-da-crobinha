# jogo-da-crobinha
import pygame
import random
import sys
from cx_Freeze import setup, Executable


# Inicialização do Pygame
pygame.init()

# Definição das cores
WHITE = (255, 255, 255)
RED = (255, 0, 0)
ORANGE = (255, 165, 0)  # Cor laranja
BLUE = (0, 0, 255)  # Cor azul

# Configurações da janela do jogo
pygame.display.set_caption("Snake Game")
SCREEN = pygame.display.set_mode((0, 0), pygame.FULLSCREEN)  # Modo de tela cheia
WIDTH, HEIGHT = SCREEN.get_width(), SCREEN.get_height()
SNAKE_SIZE = 20

# Variáveis de controle do jogo
SNAKE_SPEED = 20
MAX_SCORE = 300
SCORE_INCREMENT = 2
FRUIT_SPAWN_INTERVAL = 2  # Intervalo para surgimento de nova fruta (em pontos)

# Carrega e redimensiona a imagem de fundo
background = pygame.Surface((WIDTH, HEIGHT))
background.fill((0, 128, 0))

# Cores
CROBA = (0, 255, 0)  # Cor da cobra
CAPPLE = (255, 0, 0)  # Cor da maçã
CORANGE = (255, 165, 0)  # Cor da fruta laranja
CBLUE = (0, 0, 255)  # Cor da fruta azul


# Função para desenhar a cobra na tela
def draw_snake(snake):
    for i, pos in enumerate(snake):
        pygame.draw.rect(SCREEN, CROBA, (pos[0], pos[1], SNAKE_SIZE, SNAKE_SIZE))
        # Desenha olhos na cabeça da cobra
        if i == 0:
            eye1_pos = (pos[0] + 8, pos[1] + 5)  # Ajuste na posição do olho 1
            eye2_pos = (pos[0] + 18, pos[1] + 5)  # Ajuste na posição do olho 2
            pygame.draw.circle(SCREEN, (0, 0, 0), eye1_pos, 3)  # Olho 1
            pygame.draw.circle(SCREEN, (0, 0, 0), eye2_pos, 3)  # Olho 2


# Função para desenhar as frutas na tela
def draw_fruit(fruit_pos, color):
    pygame.draw.rect(SCREEN, color, (fruit_pos[0], fruit_pos[1], SNAKE_SIZE, SNAKE_SIZE))


# Função para renderizar texto na tela
def draw_text(text, font, color, surface, x, y):
    text_obj = font.render(text, True, color)
    text_rect = text_obj.get_rect()
    text_rect.topleft = (x, y)
    surface.blit(text_obj, text_rect)


# Função principal do jogo
def main():
    # Variáveis de controle do jogo
    snake = [[WIDTH / 2, HEIGHT / 2]]
    snake_direction = 'RIGHT'
    score = 0
    font = pygame.font.SysFont(None, 30)
    fruit_spawn_counter = 0  # Contador para controlar o surgimento de frutas

    clock = pygame.time.Clock()

    # Geração inicial de uma fruta aleatória
    fruit_color = random.choice([RED, ORANGE, BLUE])
    fruit_pos = [random.randrange(1, (WIDTH - SNAKE_SIZE) // SNAKE_SIZE) * SNAKE_SIZE,
                 random.randrange(1, (HEIGHT - SNAKE_SIZE) // SNAKE_SIZE) * SNAKE_SIZE]

    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            # Controle da direção da cobra
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_LEFT and snake_direction != 'RIGHT':
                    snake_direction = 'LEFT'
                elif event.key == pygame.K_RIGHT and snake_direction != 'LEFT':
                    snake_direction = 'RIGHT'
                elif event.key == pygame.K_UP and snake_direction != 'DOWN':
                    snake_direction = 'UP'
                elif event.key == pygame.K_DOWN and snake_direction != 'UP':
                    snake_direction = 'DOWN'
                elif event.key == pygame.K_ESCAPE:  # Verifica se a tecla Esc foi pressionada
                    pygame.quit()
                    sys.exit()

        # Movimento da cobra
        new_head = [snake[0][0], snake[0][1]]  # Nova posição da cabeça
        if snake_direction == 'LEFT':
            new_head[0] -= SNAKE_SPEED
            if new_head[0] < 0:  # Se ultrapassar a borda esquerda
                new_head[0] = WIDTH - SNAKE_SIZE  # Mover para o lado direito
        elif snake_direction == 'RIGHT':
            new_head[0] += SNAKE_SPEED
            if new_head[0] >= WIDTH:  # Se ultrapassar a borda direita
                new_head[0] = 0  # Mover para o lado esquerdo
        elif snake_direction == 'UP':
            new_head[1] -= SNAKE_SPEED
            if new_head[1] < 0:  # Se ultrapassar a borda superior
                new_head[1] = HEIGHT - SNAKE_SIZE  # Mover para baixo
        elif snake_direction == 'DOWN':
            new_head[1] += SNAKE_SPEED
            if new_head[1] >= HEIGHT:  # Se ultrapassar a borda inferior
                new_head[1] = 0  # Mover para cima

        # Verifica se a cabeça da cobra ultrapassa as bordas da tela e ajusta sua posição
        if new_head[0] < 0 or new_head[0] >= WIDTH or new_head[1] < 0 or new_head[1] >= HEIGHT:
            print("Você perdeu!")
            pygame.quit()
            sys.exit()

        # Adiciona nova cabeça
        snake.insert(0, new_head)

        # Verifica se a cobra comeu uma fruta
        if snake[0][0] == fruit_pos[0] and snake[0][1] == fruit_pos[1]:
            score += SCORE_INCREMENT
            fruit_color = random.choice([RED, ORANGE, BLUE])  # Gerar uma nova fruta aleatória
            fruit_pos = [random.randrange(1, (WIDTH - SNAKE_SIZE) // SNAKE_SIZE) * SNAKE_SIZE,
                         random.randrange(1, (HEIGHT - SNAKE_SIZE) // SNAKE_SIZE) * SNAKE_SIZE]

            # Incrementa o contador de frutas consumidas
            fruit_spawn_counter += 1

            # Se o contador atingir o intervalo definido, gera uma nova fruta
            if fruit_spawn_counter % FRUIT_SPAWN_INTERVAL == 0:
                fruit_color = random.choice([RED, ORANGE, BLUE])  # Gerar uma nova fruta aleatória
                fruit_pos = [random.randrange(1, (WIDTH - SNAKE_SIZE) // SNAKE_SIZE) * SNAKE_SIZE,
                             random.randrange(1, (HEIGHT - SNAKE_SIZE) // SNAKE_SIZE) * SNAKE_SIZE]

        else:
            snake.pop()  # Remove o último segmento da cobra

        # Desenhar na tela
        SCREEN.blit(background, (0, 0))
        draw_snake(snake)
        draw_fruit(fruit_pos, fruit_color)
        draw_text(f"Score: {score}", font, WHITE, SCREEN, WIDTH - 100, 10)

        pygame.display.update()

        # Controle de FPS
        clock.tick(15)

        # Verifica se o jogador venceu
        if score >= MAX_SCORE:
            print("Parabéns! Você venceu!")
            pygame.quit()
            sys.exit()


if _name_ == "_main_":
    main()
