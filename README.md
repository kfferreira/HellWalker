# HellWalker
Um jogo de tiro programado em Python e Pygames no Visual Studio Code


import pygame
import sys
import random
import math

# =========================================================
# INICIALIZAÇÃO DO PYGAME
# =========================================================
pygame.init()

# =========================================================
# CONFIGURAÇÕES DA JANELA
# =========================================================
LARGURA, ALTURA = 1600, 900  # Dimensões da tela
FPS = 60  # Quadros por segundo
TELA = pygame.display.set_mode((LARGURA, ALTURA))
pygame.display.set_caption("Hellwalker")
clock = pygame.time.Clock()  # Controla a taxa de atualização

# =========================================================
# PALETA DE CORES
# =========================================================
BRANCO = (255, 255, 255)
PRETO = (0, 0, 0)
VERMELHO = (255, 0, 0)
VERMELHO_ESCURO = (139, 0, 0)
ROSA = (255, 105, 180)
AZUL = (0, 0, 255)
VERDE = (0, 200, 0)
LARANJA = (255, 165, 0)
AMARELO = (255, 255, 0)

# =========================================================
# FONTES DE TEXTO
# =========================================================
fonte = pygame.font.SysFont(None, 36)  # Fonte padrão para UI
fonte_gameover = pygame.font.SysFont(None, 72)  # Fonte da tela de game over
fonte_titulo = pygame.font.SysFont(None, 150)  # Fonte do título principal
fonte_menu = pygame.font.SysFont(None, 72)  # Fonte dos botões do menu
fonte_godmode = pygame.font.SysFont(None, 90)  # Fonte para mensagem de God Mode

# =========================================================
# FUNÇÕES AUXILIARES
# =========================================================

def vetor_direcao(origem, destino):
    """
    Calcula um vetor unitário (normalizado) da origem até o destino.
    Usado para mover inimigos e projéteis em direção a alvos.
    
    Args:
        origem: Tupla (x, y) da posição inicial
        destino: Tupla (x, y) da posição final
    
    Returns:
        Tupla (dx, dy) normalizada, ou (0, 0) se origem == destino
    """
    dx, dy = destino[0] - origem[0], destino[1] - origem[1]
    dist = math.hypot(dx, dy)  # Distância euclidiana
    return (dx / dist, dy / dist) if dist else (0, 0)


def desenhar_botao(texto, retangulo, mouse_pos, pisca, cor_normal, cor_hover):
    """
    Desenha um botão interativo com efeito hover e piscante.
    
    Args:
        texto: Texto do botão
        retangulo: pygame.Rect com posição e tamanho
        mouse_pos: Posição atual do mouse
        pisca: Estado do efeito piscante (True/False)
        cor_normal: Cor quando o mouse não está sobre o botão
        cor_hover: Cor quando o mouse está sobre o botão
    """
    hover = retangulo.collidepoint(mouse_pos)  # Detecta se o mouse está sobre o botão
    cor_fundo = cor_hover if hover else cor_normal
    
    # Desenha o retângulo do botão com bordas arredondadas
    pygame.draw.rect(TELA, cor_fundo, retangulo, border_radius=12)
    
    # Alterna cor do texto quando hover + pisca (efeito piscante)
    cor_texto = PRETO if hover and pisca else BRANCO
    texto_render = fonte_menu.render(texto, True, cor_texto)
    
    # Centraliza o texto no botão
    TELA.blit(texto_render, (retangulo.centerx - texto_render.get_width() // 2,
                             retangulo.centery - texto_render.get_height() // 2))


def jogador_recebe_dano(jogador, tempo_atual):
    """
    Aplica dano ao jogador e ativa invulnerabilidade temporária.
    Muda a cor do jogador para azul como feedback visual.
    
    Args:
        jogador: Instância do Jogador
        tempo_atual: Tempo atual do jogo em milissegundos
    """
    jogador.vida -= 1
    jogador.invulneravel = True
    jogador.invulneravel_timer = tempo_atual
    jogador.image.fill(AZUL)  # Feedback visual de dano

# =========================================================
# TELA INICIAL (COM CÓDIGO SECRETO)
# =========================================================

def tela_inicial():
    """
    Exibe o menu principal com título tremendo e botões interativos.
    Possui código secreto "penes" que ativa o God Mode.
    
    Returns:
        bool: True se God Mode foi ativado, False caso contrário
    """
    # === Variáveis de controle de efeitos visuais ===
    pisca = True
    tempo_pisca = pygame.time.get_ticks()
    intervalo_pisca = 500  # Tempo entre piscadas (ms)
    
    amplitude_tremor = 6  # Intensidade do tremor
    tempo_tremor = 0
    intervalo_tremor = 40  # Atualiza tremor a cada 40ms

    # === Posicionamento dos botões ===
    botao_jogar = pygame.Rect(LARGURA // 2 - 150, 500, 300, 80)
    botao_sair = pygame.Rect(LARGURA // 2 - 150, 620, 300, 80)

    offset_x = offset_y = 0  # Offset para o efeito de tremor
    
    # === Sistema de código secreto ===
    codigo_digitado = ""  # Armazena as teclas digitadas
    codigo_secreto = "penes"  # Código que ativa God Mode
    god_mode_ativo = False
    tempo_mensagem_godmode = 0  # Momento em que God Mode foi ativado
    duracao_mensagem = 3000  # Mensagem fica 3 segundos na tela

    while True:
        clock.tick(FPS)
        tempo_atual = pygame.time.get_ticks()
        TELA.fill(PRETO)

        # === EFEITO DE TREMOR ===
        # Atualiza posição do tremor em intervalos regulares
        if tempo_atual - tempo_tremor > intervalo_tremor:
            offset_x = random.randint(-amplitude_tremor, amplitude_tremor)
            offset_y = random.randint(-amplitude_tremor, amplitude_tremor)
            tempo_tremor = tempo_atual

        # === TÍTULO COM TREMOR ===
        titulo = fonte_titulo.render("HELLWALKER", True, VERMELHO)
        TELA.blit(titulo, (LARGURA // 2 - titulo.get_width() // 2 + offset_x, 150 + offset_y))

        # === EFEITO DE PISCAR ===
        # Alterna estado a cada intervalo
        if tempo_atual - tempo_pisca > intervalo_pisca:
            pisca = not pisca
            tempo_pisca = tempo_atual

        mouse_pos = pygame.mouse.get_pos()

        # === DESENHA BOTÕES ===
        desenhar_botao("JOGAR", botao_jogar, mouse_pos, pisca, VERMELHO, (150, 0, 0))
        desenhar_botao("SAIR", botao_sair, mouse_pos, pisca, VERMELHO, (150, 0, 0))

        # === MENSAGEM GOD MODE (se ativado recentemente) ===
        if god_mode_ativo and tempo_atual - tempo_mensagem_godmode < duracao_mensagem:
            # Efeito de pulsação usando função seno
            escala = 1.0 + 0.2 * math.sin(tempo_atual * 0.01)
            texto_godmode = fonte_godmode.render("GOD MODE", True, AZUL)
            
            # Aplica escala ao texto
            largura_original = texto_godmode.get_width()
            altura_original = texto_godmode.get_height()
            nova_largura = int(largura_original * escala)
            nova_altura = int(altura_original * escala)
            texto_escalado = pygame.transform.scale(texto_godmode, (nova_largura, nova_altura))
            
            # Centraliza o texto na tela
            pos_x = LARGURA // 2 - texto_escalado.get_width() // 2
            pos_y = 350
            TELA.blit(texto_escalado, (pos_x, pos_y))

        pygame.display.flip()

        # === PROCESSAMENTO DE EVENTOS ===
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            
            elif evento.type == pygame.KEYDOWN:
                # Captura teclas alfabéticas para o código secreto
                if evento.unicode.isalpha():
                    codigo_digitado += evento.unicode.lower()
                    
                    # Mantém apenas os últimos N caracteres (tamanho do código)
                    if len(codigo_digitado) > len(codigo_secreto):
                        codigo_digitado = codigo_digitado[-len(codigo_secreto):]
                    
                    # Verifica se o código correto foi digitado
                    if codigo_digitado == codigo_secreto:
                        god_mode_ativo = True
                        tempo_mensagem_godmode = tempo_atual
                        codigo_digitado = ""  # Reseta para evitar ativações múltiplas
            
            elif evento.type == pygame.MOUSEBUTTONDOWN and evento.button == 1:
                if botao_jogar.collidepoint(mouse_pos):
                    return god_mode_ativo  # Inicia o jogo
                elif botao_sair.collidepoint(mouse_pos):
                    pygame.quit()
                    sys.exit()

# =========================================================
# CLASSE: JOGADOR
# =========================================================

class Jogador(pygame.sprite.Sprite):
    """
    Personagem controlado pelo jogador.
    - Movimenta-se com WASD
    - Pode ter God Mode (invencibilidade total)
    - Acumula power-ups ao derrotar bosses
    """
    def __init__(self, x, y, god_mode=False):
        super().__init__()
        self.image = pygame.Surface((32, 32))
        self.image.fill(VERDE)
        self.rect = self.image.get_rect(center=(x, y))
        
        # === Atributos de movimento ===
        self.vel = 5  # Velocidade de movimento
        
        # === Atributos de combate ===
        self.vida = 3  # Vida inicial
        
        # === Sistema de invulnerabilidade temporária ===
        self.invulneravel = False  # Estado de invulnerabilidade
        self.invulneravel_timer = 0  # Momento em que ficou invulnerável
        self.invulneravel_duracao = 1500  # Duração da invulnerabilidade (1.5s)
        
        # === Power-ups desbloqueáveis ===
        self.tem_penetracao = False  # Tiros atravessam múltiplos inimigos
        self.tem_cadencia = False  # Aumenta taxa de disparo
        self.tem_magnetismo = False  # Tiros perseguem inimigos (homing)
        
        # === God Mode (código secreto) ===
        self.god_mode = god_mode  # Invencibilidade total

    def update(self, teclas, tempo_atual):
        """
        Atualiza estado do jogador a cada frame.
        
        Args:
            teclas: Estado de todas as teclas (pygame.key.get_pressed())
            tempo_atual: Tempo atual do jogo em ms
        """
        # === Remove invulnerabilidade após o tempo ===
        if self.invulneravel and tempo_atual - self.invulneravel_timer >= self.invulneravel_duracao:
            self.invulneravel = False
            self.image.fill(VERDE)  # Volta à cor normal

        # === Movimento com WASD ===
        # Calcula movimento horizontal (D - A)
        dx = (teclas[pygame.K_d] - teclas[pygame.K_a]) * self.vel
        # Calcula movimento vertical (S - W)
        dy = (teclas[pygame.K_s] - teclas[pygame.K_w]) * self.vel
        
        # Move o jogador
        self.rect.move_ip(dx, dy)
        
        # Mantém o jogador dentro dos limites da tela
        self.rect.clamp_ip(TELA.get_rect())

# =========================================================
# CLASSE: TIRO (PROJÉTIL DO JOGADOR)
# =========================================================

class Tiro(pygame.sprite.Sprite):
    """
    Projétil disparado pelo jogador.
    Pode ter homing (perseguição automática) se o power-up estiver ativo.
    """
    def __init__(self, x, y, dir_x, dir_y, homing_enabled=False, target_group=None):
        super().__init__()
        self.image = pygame.Surface((6, 6))
        self.image.fill((255, 255, 0))  # Amarelo
        self.rect = self.image.get_rect(center=(x, y))
        
        # === Atributos de movimento ===
        self.vel = 10
        self.dir_x = dir_x  # Direção X normalizada
        self.dir_y = dir_y  # Direção Y normalizada
        
        # === Sistema de penetração ===
        self.penetrado = False  # Flag para saber se já atravessou um inimigo

        # === Sistema de homing (perseguição) ===
        self.homing_enabled = homing_enabled  # Se homing está ativo
        self.target_group = target_group  # Grupo de inimigos para perseguir
        self.homing_range = 350  # Alcance máximo da perseguição
        self.homing_strength = 0.15  # Força da curva (0-1)

    def update(self):
        """
        Atualiza posição do tiro e aplica homing se ativo.
        """
        # === SISTEMA DE HOMING (PERSEGUIÇÃO) ===
        if self.homing_enabled and self.target_group and len(self.target_group.sprites()) > 0:
            alvo_mais_proximo = None
            menor_dist = float('inf')
            cx, cy = self.rect.center
            
            # Encontra o inimigo mais próximo dentro do alcance
            for inim in self.target_group:
                if not hasattr(inim, 'rect'):
                    continue
                dist = math.hypot(inim.rect.centerx - cx, inim.rect.centery - cy)
                if dist < menor_dist and dist <= self.homing_range:
                    menor_dist = dist
                    alvo_mais_proximo = inim

            # Se encontrou um alvo, ajusta a direção gradualmente
            if alvo_mais_proximo is not None:
                tx = alvo_mais_proximo.rect.centerx - cx
                ty = alvo_mais_proximo.rect.centery - cy
                distt = math.hypot(tx, ty)
                
                if distt != 0:
                    # Normaliza o vetor para o alvo
                    tx, ty = tx / distt, ty / distt
                    
                    # Interpola entre direção atual e direção do alvo
                    self.dir_x = (1 - self.homing_strength) * self.dir_x + self.homing_strength * tx
                    self.dir_y = (1 - self.homing_strength) * self.dir_y + self.homing_strength * ty
                    
                    # Normaliza novamente o vetor resultante
                    dnorm = math.hypot(self.dir_x, self.dir_y)
                    if dnorm != 0:
                        self.dir_x /= dnorm
                        self.dir_y /= dnorm

        # === MOVIMENTO ===
        self.rect.x += self.dir_x * self.vel
        self.rect.y += self.dir_y * self.vel
        
        # Remove o tiro se sair da tela
        if not TELA.get_rect().contains(self.rect):
            self.kill()

# =========================================================
# CLASSE: INIMIGO BÁSICO
# =========================================================

class Inimigo(pygame.sprite.Sprite):
    """
    Inimigo vermelho básico.
    Persegue o jogador constantemente.
    Morre com 1 tiro.
    """
    def __init__(self, x, y, alvo):
        super().__init__()
        tamanho = 28
        self.image = pygame.Surface((tamanho, tamanho))
        self.image.fill((200, 0, 0))  # Vermelho
        self.rect = self.image.get_rect(center=(x, y))
        
        # === Atributos ===
        self.vel = 2
        self.alvo = alvo  # Referência ao jogador
        self.vida = 1  # Morre com 1 tiro

    def update(self):
        """Move-se em direção ao jogador"""
        dir_x, dir_y = vetor_direcao(self.rect.center, self.alvo.rect.center)
        self.rect.x += dir_x * self.vel
        self.rect.y += dir_y * self.vel

# =========================================================
# CLASSE: INIMIGO TANQUE (VERMELHO ESCURO)
# =========================================================

class InimigoVermelhoEscuro(pygame.sprite.Sprite):
    """
    Inimigo tanque: grande, lento, mas com muita vida.
    Desbloqueado após derrotar o primeiro boss.
    """
    def __init__(self, x, y, alvo):
        super().__init__()
        tamanho_quadrado = 28
        tamanho = tamanho_quadrado * 2  # 2x maior que o inimigo normal
        self.image = pygame.Surface((tamanho, tamanho))
        self.image.fill(VERMELHO_ESCURO)
        self.rect = self.image.get_rect(center=(x, y))
        
        # === Atributos ===
        self.vel = 1.0  # Mais lento
        self.alvo = alvo
        self.vida = 10  # 10x mais vida

    def update(self):
        """Move-se em direção ao jogador"""
        dir_x, dir_y = vetor_direcao(self.rect.center, self.alvo.rect.center)
        self.rect.x += dir_x * self.vel
        self.rect.y += dir_y * self.vel

# =========================================================
# CLASSE: PROJÉTIL INIMIGO
# =========================================================

class ProjetilInimigo(pygame.sprite.Sprite):
    """
    Projétil disparado pelos inimigos atiradores.
    """
    def __init__(self, x, y, dir_x, dir_y):
        super().__init__()
        self.image = pygame.Surface((10, 10))
        self.image.fill(ROSA)
        self.rect = self.image.get_rect(center=(x, y))
        
        # === Atributos ===
        self.vel = 5
        self.dir_x = dir_x
        self.dir_y = dir_y

    def update(self):
        """Move o projétil e remove se sair da tela"""
        self.rect.x += self.dir_x * self.vel
        self.rect.y += self.dir_y * self.vel
        
        # Remove se sair da tela
        if not TELA.get_rect().colliderect(self.rect):
            self.kill()

# =========================================================
# CLASSE: INIMIGO ATIRADOR
# =========================================================

class InimigoAtirador(pygame.sprite.Sprite):
    """
    Inimigo laranja que dispara projéteis no jogador.
    Desbloqueado após derrotar o primeiro boss.
    """
    def __init__(self, x, y, alvo):
        super().__init__()
        self.image = pygame.Surface((28, 28))
        self.image.fill(LARANJA)
        self.rect = self.image.get_rect(center=(x, y))
        
        # === Atributos ===
        self.vel = 1.5
        self.alvo = alvo
        
        # === Sistema de disparo ===
        self.ultimo_tiro = 0
        self.cooldown_tiro = 1500  # Atira a cada 1.5 segundos

    def update(self):
        """Move-se em direção ao jogador"""
        dir_x, dir_y = vetor_direcao(self.rect.center, self.alvo.rect.center)
        self.rect.x += dir_x * self.vel
        self.rect.y += dir_y * self.vel

    def pode_atirar(self, tempo_atual):
        """Verifica se o cooldown do tiro já passou"""
        return tempo_atual - self.ultimo_tiro >= self.cooldown_tiro

    def atirar(self, grupo_projeteis, tempo_atual, todos_group=None):
        """
        Dispara um projétil em direção ao jogador.
        
        Args:
            grupo_projeteis: Grupo de sprites dos projéteis inimigos
            tempo_atual: Tempo atual do jogo
            todos_group: Grupo com todos os sprites (opcional)
        """
        dir_x, dir_y = vetor_direcao(self.rect.center, self.alvo.rect.center)
        if dir_x == 0 and dir_y == 0:
            return
        
        # Cria o projétil
        proj = ProjetilInimigo(self.rect.centerx, self.rect.centery, dir_x, dir_y)
        grupo_projeteis.add(proj)
        
        if todos_group is not None:
            todos_group.add(proj)
        
        self.ultimo_tiro = tempo_atual

# =========================================================
# CLASSE: PROJÉTIL DO BOSS
# =========================================================

class ProjetilBoss(pygame.sprite.Sprite):
    """
    Projétil maior e mais rápido disparado pelos bosses.
    """
    def __init__(self, x, y, dir_x, dir_y):
        super().__init__()
        self.image = pygame.Surface((32, 16))
        self.image.fill(ROSA)
        self.rect = self.image.get_rect(center=(x, y))
        
        # === Atributos ===
        self.vel = 6  # Mais rápido que projéteis normais
        self.dir_x = dir_x
        self.dir_y = dir_y

    def update(self):
        """Move o projétil e remove se sair da tela"""
        self.rect.x += self.dir_x * self.vel
        self.rect.y += self.dir_y * self.vel
        
        if not TELA.get_rect().colliderect(self.rect):
            self.kill()

# =========================================================
# CLASSE: BOSS
# =========================================================

class Boss(pygame.sprite.Sprite):
    """
    Boss roxo gigante que aparece a cada 100 pontos.
    - 5x maior que inimigo normal
    - 100 de vida
    - Dispara 3 projéteis em leque
    """
    def __init__(self, x, y, alvo):
        super().__init__()
        tamanho = 28 * 5  # 5x maior
        self.image = pygame.Surface((tamanho, tamanho))
        self.image.fill((100, 0, 200))  # Roxo
        self.rect = self.image.get_rect(center=(x, y))
        
        # === Atributos ===
        self.vel = 1
        self.alvo = alvo
        self.vida = 100  # Muita vida
        
        # === Sistema de disparo ===
        self.ultimo_disparo = 0
        self.cooldown_disparo = 3000  # Atira a cada 3 segundos

    def update(self):
        """Move-se em direção ao jogador"""
        dir_x, dir_y = vetor_direcao(self.rect.center, self.alvo.rect.center)
        self.rect.x += dir_x * self.vel
        self.rect.y += dir_y * self.vel

    def pode_atirar(self, tempo_atual):
        """Verifica se o cooldown do disparo já passou"""
        return tempo_atual - self.ultimo_disparo >= self.cooldown_disparo

    def atirar(self, grupo_projeteis, tempo_atual):
        """
        Dispara 3 projéteis em leque (spread) em direção ao jogador.
        
        Args:
            grupo_projeteis: Grupo de sprites dos projéteis do boss
            tempo_atual: Tempo atual do jogo
        """
        dir_x, dir_y = vetor_direcao(self.rect.center, self.alvo.rect.center)
        if dir_x == 0 and dir_y == 0:
            return
        
        spread = 0.2  # Ângulo de separação entre os projéteis
        angle_central = math.atan2(dir_y, dir_x)
        
        # Cria 3 ângulos: central, esquerda, direita
        angulos = [angle_central, angle_central - spread, angle_central + spread]
        
        for angulo in angulos:
            dx_i = math.cos(angulo)
            dy_i = math.sin(angulo)
            proj = ProjetilBoss(self.rect.centerx, self.rect.centery, dx_i, dy_i)
            grupo_projeteis.add(proj)
        
        self.ultimo_disparo = tempo_atual

# =========================================================
# CLASSE: PARTÍCULA DE DISPARO
# =========================================================

class ParticulaDisparo(pygame.sprite.Sprite):
    """
    Efeito visual que aparece ao disparar armas.
    Cria um flash amarelo/laranja que desaparece rapidamente.
    """
    def __init__(self, x, y):
        super().__init__()
        # Escolhe cor aleatória (tons de amarelo/laranja)
        cor = random.choice([(255, 200, 50), (255, 150, 0), (255, 220, 100)])
        tamanho = random.randint(3, 6)
        
        # Cria círculo com transparência
        self.image = pygame.Surface((tamanho, tamanho), pygame.SRCALPHA)
        pygame.draw.circle(self.image, cor, (tamanho // 2, tamanho // 2), tamanho // 2)
        self.rect = self.image.get_rect(center=(x, y))
        
        # === Atributos de movimento ===
        self.vel_x = random.uniform(-1.5, 1.5)
        self.vel_y = random.uniform(-1.5, 1.5)
        
        # === Atributos de vida ===
        self.tempo_vida = random.randint(10, 20)  # Frames até desaparecer
        self.alpha = 255  # Opacidade inicial

    def update(self):
        """Move a partícula e diminui sua opacidade até desaparecer"""
        self.rect.x += self.vel_x
        self.rect.y += self.vel_y
        
        # Diminui tempo de vida e opacidade
        self.tempo_vida -= 1
        self.alpha = max(0, self.alpha - 25)
        self.image.set_alpha(self.alpha)
        
        # Remove quando acabar a vida ou ficar invisível
        if self.tempo_vida <= 0 or self.alpha <= 0:
            self.kill()

# =========================================================
# CLASSE: ORB (ORBE ORBITAL)
# =========================================================

class Orb(pygame.sprite.Sprite):
    """
    Orbe branca que orbita o jogador causando dano aos inimigos.
    Power-up obtido ao derrotar bosses (máximo 3 orbes).
    """
    def __init__(self, jogador, posicao_index, total_orbes):
        super().__init__()
        self.jogador = jogador
        tamanho = 48
        
        # Cria círculo branco
        self.image_original = pygame.Surface((tamanho, tamanho), pygame.SRCALPHA)
        pygame.draw.circle(self.image_original, BRANCO, (tamanho // 2, tamanho // 2), tamanho // 2)
        self.image = self.image_original.copy()
        self.rect = self.image.get_rect(center=jogador.rect.center)
        
        # === Atributos ===
        self.dano = 5  # Dano por frame de contato
        self.raio = 80  # Distância do jogador
        self.posicao_index = posicao_index  # Índice desta orbe
        self.total_orbes = total_orbes  # Total de orbes ativas

        # === Define posição inicial baseada na quantidade de orbes ===
        if total_orbes == 1:
            # 1 orbe: acima do jogador
            self.angulo_base = math.radians(-90)
        elif total_orbes == 2:
            # 2 orbes: esquerda e direita
            self.angulo_base = [math.radians(180), math.radians(0)][posicao_index]
        else:
            # 3 orbes: formam um triângulo
            self.angulo_base = [math.radians(-90), math.radians(30), math.radians(150)][posicao_index]

        # === Controle de rotação ===
        self.angulo_giro = 0  # Ângulo de rotação orbital atual
        self.velocidade_giro = 0.02  # Velocidade de rotação
        self.rotacao_visual = 0  # Rotação visual da orbe

    def update(self):
        """
        Atualiza posição da orbe girando ao redor do jogador.
        """
        # Incrementa o ângulo de rotação
        self.angulo_giro += self.velocidade_giro
        angulo_total = self.angulo_base + self.angulo_giro
        
        # Posição do jogador
        cx, cy = self.jogador.rect.center
        
        # Calcula nova posição usando trigonometria circular
        x = cx + math.cos(angulo_total) * self.raio
        y = cy + math.sin(angulo_total) * self.raio
        
        # Rotação visual da orbe (gira sobre si mesma)
        self.rotacao_visual += 2
        self.image = pygame.transform.rotate(self.image_original, self.rotacao_visual)
        self.rect = self.image.get_rect(center=(x, y))

# =========================================================
# FUNÇÃO: TELA DE GAME OVER
# =========================================================

def tela_gameover(pontuacao):
    """
    Exibe a tela de game over com opções de reiniciar ou sair.
    
    Args:
        pontuacao: Pontuação final do jogador
    """
    TELA.fill(PRETO)
    
    # === Renderiza textos ===
    texto1 = fonte_gameover.render("GAME OVER", True, VERMELHO)
    texto2 = fonte.render(f"Pontos: {pontuacao}", True, BRANCO)
    texto4 = fonte.render("Pressione R para jogar novamente ou ESC para sair", True, BRANCO)

    meio_x = LARGURA // 2
    meio_y = ALTURA // 2

    # Centraliza e desenha os textos
    TELA.blit(texto1, (meio_x - texto1.get_width() // 2, meio_y - 100))
    TELA.blit(texto2, (meio_x - texto2.get_width() // 2, meio_y))
    TELA.blit(texto4, (meio_x - texto4.get_width() // 2, meio_y + 100))

    pygame.display.flip()

    # Loop de espera por input
    while True:
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_r:
                    return  # Reinicia o jogo
                elif evento.key == pygame.K_ESCAPE:
                    pygame.quit()
                    sys.exit()

# =========================================================
# FUNÇÃO: CRIAR INIMIGO OU BOSS
# =========================================================

def criar_inimigo_ou_boss(pontos, boss_intervalo_atual, jogador, desbloqueou_atirador=False, desbloqueou_vermelho_escuro=False):
    """
    Cria inimigos normais ou bosses baseado na pontuação.
    Boss aparece a cada 100 pontos.
    
    Args:
        pontos: Pontuação atual
        boss_intervalo_atual: Último intervalo em que um boss foi criado
        jogador: Referência ao jogador
        desbloqueou_atirador: Se inimigos atiradores foram desbloqueados
        desbloqueou_vermelho_escuro: Se inimigos tanque foram desbloqueados
    
    Returns:
        Tupla (inimigo/boss, novo_boss_intervalo_atual)
    """
    intervalo = pontos // 100  # A cada 100 pontos = 1 intervalo
    
    # === SPAWNA BOSS ===
    if intervalo > boss_intervalo_atual:
        # Posições possíveis (fora da tela)
        posicoes = [
            (random.randint(0, LARGURA), -140),  # Topo
            (random.randint(0, LARGURA), ALTURA + 140),  # Baixo
            (-140, random.randint(0, ALTURA)),  # Esquerda
            (LARGURA + 140, random.randint(0, ALTURA)),  # Direita
        ]
        x, y = random.choice(posicoes)
        boss = Boss(x, y, jogador)
        boss_intervalo_atual = intervalo
        return boss, boss_intervalo_atual
    
    # === SPAWNA INIMIGO NORMAL ===
    else:
        # Posições possíveis (fora da tela, mais próximas)
        posicoes = [
            (random.randint(0, LARGURA), -30),
            (random.randint(0, LARGURA), ALTURA + 30),
            (-30, random.randint(0, ALTURA)),
            (LARGURA + 30, random.randint(0, ALTURA)),
        ]
        x, y = random.choice(posicoes)

        # Determina tipo de inimigo baseado em probabilidade
        if desbloqueou_vermelho_escuro and random.random() < 0.10:
            # 10% de chance: Inimigo Tanque
            inimigo = InimigoVermelhoEscuro(x, y, jogador)
        elif desbloqueou_atirador and random.random() < 0.2:
            # 20% de chance: Inimigo Atirador
            inimigo = InimigoAtirador(x, y, jogador)
        else:
            # Inimigo básico
            inimigo = Inimigo(x, y, jogador)
        
        return inimigo, boss_intervalo_atual

# =========================================================
# FUNÇÃO: ATIRAR ESCOPETA
# =========================================================

def atirar_escopeta(jogador, tiros, todos, particulas, dx, dy, tempo_atual, ultimo_tiro_escopeta, inimigos_group):
    """
    Dispara múltiplos projéteis em leque (padrão de escopeta).
    
    Args:
        jogador: Instância do jogador
        tiros: Grupo de sprites dos tiros
        todos: Grupo com todos os sprites
        particulas: Grupo de partículas visuais
        dx, dy: Direção normalizada do disparo
        tempo_atual: Tempo atual do jogo
        ultimo_tiro_escopeta: Momento do último disparo de escopeta
        inimigos_group: Grupo de inimigos (para homing)
    
    Returns:
        Tempo do último tiro (atualizado se disparou)
    """
    cooldown_escopeta = 1000  # Cooldown de 1 segundo
    
    if tempo_atual - ultimo_tiro_escopeta >= cooldown_escopeta:
        spread = 0.2  # Ângulo de abertura do leque
        
        # Número de projéteis baseado no power-up de cadência
        num_tiros = 10 if jogador.tem_cadencia else 5
        
        # Ângulo central do disparo
        angle = math.atan2(dy, dx)
        
        # Cria múltiplos projéteis em leque
        for i in range(num_tiros):
            # Calcula offset angular para cada projétil
            offset = spread * (i - num_tiros // 2)
            dir_x_i = math.cos(angle + offset)
            dir_y_i = math.sin(angle + offset)
            
            # Cria o projétil
            tiro = Tiro(jogador.rect.centerx, jogador.rect.centery, dir_x_i, dir_y_i,
                        homing_enabled=jogador.tem_magnetismo, target_group=inimigos_group)
            tiros.add(tiro)
            todos.add(tiro)

            # Cria partículas visuais
            for _ in range(5):
                p = ParticulaDisparo(jogador.rect.centerx, jogador.rect.centery)
                p.vel_x = random.uniform(0.5, 2.0) * dx + random.uniform(-0.5, 0.5)
                p.vel_y = random.uniform(0.5, 2.0) * dy + random.uniform(-0.5, 0.5)
                particulas.add(p)
                todos.add(p)
        
        return tempo_atual  # Atualiza o tempo do último tiro
    
    return ultimo_tiro_escopeta  # Mantém o tempo anterior

# =========================================================
# FUNÇÃO PRINCIPAL: MAIN (LOOP DO JOGO)
# =========================================================

def main(god_mode=False):
    """
    Loop principal do jogo.
    
    Args:
        god_mode: Se o jogador tem invencibilidade ativa
    """
    # === GRUPOS DE SPRITES ===
    todos = pygame.sprite.Group()  # Todos os sprites visíveis
    tiros = pygame.sprite.Group()  # Projéteis do jogador
    inimigos = pygame.sprite.Group()  # Inimigos normais
    bosses = pygame.sprite.Group()  # Bosses
    projeteis_boss = pygame.sprite.Group()  # Projéteis dos bosses
    projeteis_inimigos = pygame.sprite.Group()  # Projéteis dos inimigos
    particulas = pygame.sprite.Group()  # Partículas visuais
    orbes = pygame.sprite.Group()  # Orbes orbitais

    # === INICIALIZA JOGADOR ===
    jogador = Jogador(LARGURA // 2, ALTURA // 2, god_mode)
    todos.add(jogador)

    # === VARIÁVEIS DE CONTROLE DO JOGO ===
    pontos = 0
    desbloqueou_atirador = False  # Inimigos atiradores
    desbloqueou_vermelho_escuro = False  # Inimigos tanque

    # === EVENTOS CUSTOMIZADOS ===
    # Evento para spawnar inimigos periodicamente
    SPAWN_EVENTO = pygame.USEREVENT + 1
    pygame.time.set_timer(SPAWN_EVENTO, 250)  # A cada 250ms

    # Evento para disparo contínuo da metralhadora
    DISPARO_CONTINUO_EVENTO = pygame.USEREVENT + 2
    pygame.time.set_timer(DISPARO_CONTINUO_EVENTO, 0)  # Desativado inicialmente

    # === VARIÁVEIS DE ARMAS ===
    arma_atual = "Metralhadora"
    disparando_continuo = False
    intervalo_metralhadora = 100  # Intervalo entre disparos (ms)
    ultimo_tiro_escopeta = 0
    boss_intervalo_atual = 0

    # =========================================================
    # LOOP PRINCIPAL
    # =========================================================
    while True:
        clock.tick(FPS)  # Controla FPS
        TELA.fill(PRETO)
        tempo_atual = pygame.time.get_ticks()

        # === PROCESSAMENTO DE EVENTOS ===
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            # === EVENTO: SPAWN DE INIMIGOS ===
            if evento.type == SPAWN_EVENTO:
                inimigo_ou_boss, boss_intervalo_atual = criar_inimigo_ou_boss(
                    pontos, boss_intervalo_atual, jogador, desbloqueou_atirador, desbloqueou_vermelho_escuro
                )
                todos.add(inimigo_ou_boss)
                
                # Adiciona ao grupo correto
                if isinstance(inimigo_ou_boss, Boss):
                    bosses.add(inimigo_ou_boss)
                else:
                    inimigos.add(inimigo_ou_boss)

            # === EVENTO: CLIQUE DO MOUSE ===
            if evento.type == pygame.MOUSEBUTTONDOWN:
                # Botão esquerdo: dispara
                if evento.button == 1:
                    mx, my = pygame.mouse.get_pos()
                    dx, dy = vetor_direcao(jogador.rect.center, (mx, my))
                    if dx == 0 and dy == 0:
                        continue

                    if arma_atual == "Metralhadora":
                        # Ativa disparo contínuo
                        disparando_continuo = True
                        intervalo_metralhadora = 60 if jogador.tem_cadencia else 100
                        pygame.time.set_timer(DISPARO_CONTINUO_EVENTO, intervalo_metralhadora)
                    else:
                        # Dispara escopeta
                        ultimo_tiro_escopeta = atirar_escopeta(
                            jogador, tiros, todos, particulas, dx, dy, tempo_atual, ultimo_tiro_escopeta, inimigos
                        )

                # Botão direito: troca de arma
                elif evento.button == 3:
                    if arma_atual == "Metralhadora":
                        arma_atual = "Escopeta"
                        disparando_continuo = False
                        pygame.time.set_timer(DISPARO_CONTINUO_EVENTO, 0)
                    else:
                        arma_atual = "Metralhadora"

            # === EVENTO: SOLTAR BOTÃO DO MOUSE ===
            if evento.type == pygame.MOUSEBUTTONUP:
                # Para o disparo contínuo da metralhadora
                if evento.button == 1 and arma_atual == "Metralhadora":
                    disparando_continuo = False
                    pygame.time.set_timer(DISPARO_CONTINUO_EVENTO, 0)

            # === EVENTO: DISPARO CONTÍNUO DA METRALHADORA ===
            if evento.type == DISPARO_CONTINUO_EVENTO and disparando_continuo and arma_atual == "Metralhadora":
                mx, my = pygame.mouse.get_pos()
                dx, dy = vetor_direcao(jogador.rect.center, (mx, my))
                
                if dx != 0 or dy != 0:
                    # Deslocamento da origem do tiro (para parecer que sai da frente)
                    deslocamento = 20
                    origem_x = jogador.rect.centerx + dx * deslocamento
                    origem_y = jogador.rect.centery + dy * deslocamento

                    # Cria o tiro
                    tiro = Tiro(origem_x, origem_y, dx, dy,
                                homing_enabled=False, target_group=None)
                    tiros.add(tiro)
                    todos.add(tiro)

                    # Cria partículas visuais
                    for _ in range(5):
                        p = ParticulaDisparo(origem_x, origem_y)
                        p.vel_x = random.uniform(0.5, 2.0) * dx + random.uniform(-0.5, 0.5)
                        p.vel_y = random.uniform(0.5, 2.0) * dy + random.uniform(-0.5, 0.5)
                        particulas.add(p)
                        todos.add(p)

        # === ATUALIZA TODOS OS SPRITES ===
        teclas = pygame.key.get_pressed()
        jogador.update(teclas, tempo_atual)
        tiros.update()
        inimigos.update()
        bosses.update()
        projeteis_boss.update()
        projeteis_inimigos.update()
        particulas.update()
        orbes.update()

        # === BOSSES ATIRAM ===
        for boss in bosses:
            if boss.pode_atirar(tempo_atual):
                boss.atirar(projeteis_boss, tempo_atual)
                todos.add(projeteis_boss)

        # === INIMIGOS ATIRADORES ATIRAM ===
        for inimigo in list(inimigos):
            if isinstance(inimigo, InimigoAtirador) and inimigo.pode_atirar(tempo_atual):
                inimigo.atirar(projeteis_inimigos, tempo_atual, todos_group=todos)

        # === COLISÃO: TIROS vs INIMIGOS ===
        for tiro in list(tiros):
            atingidos = pygame.sprite.spritecollide(tiro, inimigos, False)
            if atingidos:
                for inim in list(atingidos):
                    # Aplica dano ao inimigo
                    if hasattr(inim, 'vida'):
                        inim.vida -= 1
                        if inim.vida <= 0:
                            inim.kill()
                            pontos += 1
                    else:
                        inim.kill()
                        pontos += 1
                
                # Remove o tiro se não tiver penetração ou já penetrou
                if not jogador.tem_penetracao or tiro.penetrado:
                    tiro.kill()
                else:
                    tiro.penetrado = True

        # === COLISÃO: ORBES vs INIMIGOS ===
        for orb in list(orbes):
            atingidos = pygame.sprite.spritecollide(orb, inimigos, False)
            for inim in atingidos:
                if hasattr(inim, 'vida'):
                    inim.vida -= orb.dano
                    if inim.vida <= 0:
                        inim.kill()
                        pontos += 1
                else:
                    inim.kill()
                    pontos += 1

            # Orbes destroem projéteis inimigos
            pygame.sprite.spritecollide(orb, projeteis_inimigos, True)
            pygame.sprite.spritecollide(orb, projeteis_boss, True)

        # === COLISÃO: TIROS vs BOSSES ===
        colisoes_boss = pygame.sprite.groupcollide(tiros, bosses, True, False)
        if colisoes_boss:
            for boss_list in colisoes_boss.values():
                for boss in boss_list:
                    boss.vida -= 1
                    
                    # Boss morreu
                    if boss.vida <= 0:
                        boss.kill()
                        jogador.vida += 2  # Recupera 2 vidas
                        
                        # === SISTEMA DE POWER-UPS ===
                        powerups_disponiveis = []
                        if not jogador.tem_penetracao:
                            powerups_disponiveis.append("Penetração")
                        if not jogador.tem_cadencia:
                            powerups_disponiveis.append("Cadência")
                        if len(orbes) < 3:
                            powerups_disponiveis.append("Orb")
                        if not jogador.tem_magnetismo:
                            powerups_disponiveis.append("Magnetismo")

                        # Escolhe um power-up aleatório
                        if powerups_disponiveis:
                            melhoria = random.choice(powerups_disponiveis)

                            if melhoria == "Penetração":
                                jogador.tem_penetracao = True

                            elif melhoria == "Cadência":
                                jogador.tem_cadencia = True
                                intervalo_metralhadora = 60
                                # Atualiza timer se estiver disparando
                                if arma_atual == "Metralhadora" and disparando_continuo:
                                    pygame.time.set_timer(DISPARO_CONTINUO_EVENTO, intervalo_metralhadora)

                            elif melhoria == "Orb":
                                if len(orbes) < 3:
                                    # Recria todas as orbes com nova distribuição
                                    num_orbes_existentes = len(orbes)
                                    total_orbes = num_orbes_existentes + 1
                                    
                                    for orb in list(orbes):
                                        orb.kill()
                                    orbes.empty()

                                    for i in range(total_orbes):
                                        orb = Orb(jogador, i, total_orbes)
                                        orbes.add(orb)
                                        todos.add(orb)

                            elif melhoria == "Magnetismo":
                                jogador.tem_magnetismo = True

                        # Desbloqueia novos tipos de inimigos
                        if not desbloqueou_atirador:
                            desbloqueou_atirador = True
                        if not desbloqueou_vermelho_escuro:
                            desbloqueou_vermelho_escuro = True

        # === SISTEMA DE DANO AO JOGADOR ===
        def jogador_recebe_dano():
            """Função interna para aplicar dano ao jogador"""
            # God Mode: não recebe dano
            if jogador.god_mode:
                return
            
            jogador.vida -= 1
            jogador.invulneravel = True
            jogador.invulneravel_timer = tempo_atual
            jogador.image.fill(AZUL)

        # Colisão com projéteis
        if (pygame.sprite.spritecollideany(jogador, projeteis_boss) or
            pygame.sprite.spritecollideany(jogador, projeteis_inimigos)) and not jogador.invulneravel:
            jogador_recebe_dano()

        # Colisão com inimigos/bosses
        if (pygame.sprite.spritecollideany(jogador, inimigos) or 
            pygame.sprite.spritecollideany(jogador, bosses)) and not jogador.invulneravel:
            jogador_recebe_dano()

        # === VERIFICA GAME OVER ===
        if jogador.vida <= 0:
            tela_gameover(pontos)
            return  # Retorna ao menu

        # === DESENHA TUDO NA TELA ===
        todos.draw(TELA)
        particulas.draw(TELA)
        projeteis_boss.draw(TELA)
        projeteis_inimigos.draw(TELA)

        # === INTERFACE (HUD) ===
        # Vida (infinito se God Mode)
        if jogador.god_mode:
            texto_vida = fonte.render("Vida: ∞", True, AZUL)
        else:
            texto_vida = fonte.render(f"Vida: {jogador.vida}", True, BRANCO)
        
        texto_pontos = fonte.render(f"Pontos: {pontos}", True, BRANCO)
        texto_arma = fonte.render(arma_atual, True, BRANCO)

        TELA.blit(texto_vida, (10, ALTURA - 40))
        TELA.blit(texto_pontos, (10, 10))
        TELA.blit(texto_arma, (LARGURA - texto_arma.get_width() - 10, 10))

        # === LISTA DE MELHORIAS ATIVAS ===
        melhorias = []
        if jogador.tem_penetracao:
            melhorias.append("Penetração")
        if jogador.tem_cadencia:
            melhorias.append("Cadência")
        if len(orbes) > 0:
            melhorias.append("Orb")
        if jogador.tem_magnetismo:
            melhorias.append("Magnetismo")
        if jogador.god_mode:
            melhorias.append("GOD MODE")
        
        if melhorias:
            texto_melhorias = fonte.render(" | ".join(melhorias), True, BRANCO)
            TELA.blit(texto_melhorias, (LARGURA - texto_melhorias.get_width() - 10, 40))

        pygame.display.flip()

# =========================================================
# EXECUÇÃO DO JOGO
# =========================================================

if __name__ == "__main__":
    while True:
        # Exibe menu e verifica se God Mode foi ativado
        god_mode_ativo = tela_inicial()
        
        # Inicia o jogo
        main(god_mode_ativo)
        
        # Quando o jogo termina, volta ao menu



