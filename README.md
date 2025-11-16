# HellWalker
Um jogo de tiro programado em Python e Pygames no Visual Studio Code

|
|
V


import pygame
import sys
import random
import math


pygame.init()


# =========================================================
# CONFIGURAÇÕES GERAIS
# =========================================================
LARGURA, ALTURA = 1920, 1080
FPS = 60
TELA = pygame.display.set_mode((LARGURA, ALTURA))
pygame.display.set_caption("Hellwalker")
clock = pygame.time.Clock()


# Cores
BRANCO = (255, 255, 255)
PRETO = (0, 0, 0)
VERMELHO = (255, 0, 0)
VERMELHO_ESCURO = (139, 0, 0)
ROSA = (255, 105, 180)
AZUL = (0, 0, 255)
VERDE = (0, 200, 0)
LARANJA = (255, 165, 0)
AMARELO = (255, 255, 0)


# Fontes
fonte = pygame.font.SysFont(None, 36)
fonte_gameover = pygame.font.SysFont(None, 72)
fonte_titulo = pygame.font.SysFont(None, 150)
fonte_menu = pygame.font.SysFont(None, 72)


# =========================================================
# FUNÇÕES AUXILIARES
# =========================================================
def vetor_direcao(origem, destino):
    dx, dy = destino[0] - origem[0], destino[1] - origem[1]
    dist = math.hypot(dx, dy)
    return (dx / dist, dy / dist) if dist else (0, 0)


def desenhar_botao(texto, retangulo, mouse_pos, pisca, cor_normal, cor_hover):
    hover = retangulo.collidepoint(mouse_pos)
    cor_fundo = cor_hover if hover else cor_normal
    pygame.draw.rect(TELA, cor_fundo, retangulo, border_radius=12)
    cor_texto = PRETO if hover and pisca else BRANCO
    texto_render = fonte_menu.render(texto, True, cor_texto)
    TELA.blit(texto_render, (retangulo.centerx - texto_render.get_width() // 2,
                             retangulo.centery - texto_render.get_height() // 2))


def jogador_recebe_dano(jogador, tempo_atual):
    jogador.vida -= 1
    jogador.invulneravel = True
    jogador.invulneravel_timer = tempo_atual
    jogador.image.fill(AZUL)


# =========================================================
# 🟥 TELA INICIAL
# =========================================================
def tela_inicial():
    pisca = True
    tempo_pisca = pygame.time.get_ticks()
    intervalo_pisca = 500
    amplitude_tremor = 6
    tempo_tremor = 0
    intervalo_tremor = 40


    botao_jogar = pygame.Rect(LARGURA // 2 - 150, 500, 300, 80)
    botao_sair = pygame.Rect(LARGURA // 2 - 150, 620, 300, 80)


    offset_x = offset_y = 0


    while True:
        clock.tick(FPS)
        tempo_atual = pygame.time.get_ticks()
        TELA.fill(PRETO)


        # Tremor
        if tempo_atual - tempo_tremor > intervalo_tremor:
            offset_x = random.randint(-amplitude_tremor, amplitude_tremor)
            offset_y = random.randint(-amplitude_tremor, amplitude_tremor)
            tempo_tremor = tempo_atual


        # Título tremendo
        titulo = fonte_titulo.render("HELLWALKER", True, VERMELHO)
        TELA.blit(titulo, (LARGURA // 2 - titulo.get_width() // 2 + offset_x, 150 + offset_y))


        # Piscar
        if tempo_atual - tempo_pisca > intervalo_pisca:
            pisca = not pisca
            tempo_pisca = tempo_atual


        mouse_pos = pygame.mouse.get_pos()


        # Botões
        desenhar_botao("JOGAR", botao_jogar, mouse_pos, pisca, VERMELHO, (150, 0, 0))
        desenhar_botao("SAIR", botao_sair, mouse_pos, pisca, VERMELHO, (150, 0, 0))


        pygame.display.flip()


        # Eventos
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif evento.type == pygame.MOUSEBUTTONDOWN and evento.button == 1:
                if botao_jogar.collidepoint(mouse_pos):
                    return
                elif botao_sair.collidepoint(mouse_pos):
                    pygame.quit()
                    sys.exit()


# =========================================================
# CLASSES PRINCIPAIS
# =========================================================
class Jogador(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = pygame.Surface((32, 32))
        self.image.fill(VERDE)
        self.rect = self.image.get_rect(center=(x, y))
        self.vel = 5
        self.vida = 3
        self.invulneravel = False
        self.invulneravel_timer = 0
        self.invulneravel_duracao = 1500
        self.tem_penetracao = False
        self.tem_cadencia = False
        self.tem_magnetismo = False  # <<< NOVO: Magnetismo


    def update(self, teclas, tempo_atual):
        if self.invulneravel and tempo_atual - self.invulneravel_timer >= self.invulneravel_duracao:
            self.invulneravel = False
            self.image.fill(VERDE)


        dx = (teclas[pygame.K_d] - teclas[pygame.K_a]) * self.vel
        dy = (teclas[pygame.K_s] - teclas[pygame.K_w]) * self.vel
        self.rect.move_ip(dx, dy)
        self.rect.clamp_ip(TELA.get_rect())


class Tiro(pygame.sprite.Sprite):
    def __init__(self, x, y, dir_x, dir_y, homing_enabled=False, target_group=None):
        super().__init__()
        self.image = pygame.Surface((6, 6))
        self.image.fill((255, 255, 0))
        self.rect = self.image.get_rect(center=(x, y))
        self.vel = 10
        self.dir_x = dir_x
        self.dir_y = dir_y
        self.penetrado = False


        # Homing
        self.homing_enabled = homing_enabled
        self.target_group = target_group
        self.homing_range = 350  # alcance para buscar inimigos
        self.homing_strength = 0.15  # quanto mais alto, mais rápido ajusta a direção


    def update(self):
        # comportamento homing: busca inimigo mais próximo no grupo target_group
        if self.homing_enabled and self.target_group and len(self.target_group.sprites()) > 0:
            # encontra inimigo mais próximo dentro do alcance
            alvo_mais_proximo = None
            menor_dist = float('inf')
            cx, cy = self.rect.center
            for inim in self.target_group:
                # só considerar inimigos vivos (tem rect)
                if not hasattr(inim, 'rect'):
                    continue
                dist = math.hypot(inim.rect.centerx - cx, inim.rect.centery - cy)
                if dist < menor_dist and dist <= self.homing_range:
                    menor_dist = dist
                    alvo_mais_proximo = inim


            if alvo_mais_proximo is not None:
                # direção alvo
                tx = alvo_mais_proximo.rect.centerx - cx
                ty = alvo_mais_proximo.rect.centery - cy
                distt = math.hypot(tx, ty)
                if distt != 0:
                    tx, ty = tx / distt, ty / distt
                    # lerp (suaviza a mudança de direção)
                    self.dir_x = (1 - self.homing_strength) * self.dir_x + self.homing_strength * tx
                    self.dir_y = (1 - self.homing_strength) * self.dir_y + self.homing_strength * ty
                    # normalizar direção
                    dnorm = math.hypot(self.dir_x, self.dir_y)
                    if dnorm != 0:
                        self.dir_x /= dnorm
                        self.dir_y /= dnorm


        self.rect.x += self.dir_x * self.vel
        self.rect.y += self.dir_y * self.vel
        if not TELA.get_rect().contains(self.rect):
            self.kill()


class Inimigo(pygame.sprite.Sprite):
    def __init__(self, x, y, alvo):
        super().__init__()
        tamanho = 28
        self.image = pygame.Surface((tamanho, tamanho))
        self.image.fill((200, 0, 0))
        self.rect = self.image.get_rect(center=(x, y))
        self.vel = 2
        self.alvo = alvo
        self.vida = 1  # inimigos padrões: 1 HP


    def update(self):
        dir_x, dir_y = vetor_direcao(self.rect.center, self.alvo.rect.center)
        self.rect.x += dir_x * self.vel
        self.rect.y += dir_y * self.vel


# Inimigo Vermelho Escuro
class InimigoVermelhoEscuro(pygame.sprite.Sprite):
    def __init__(self, x, y, alvo):
        super().__init__()
        tamanho_quadrado = 28
        tamanho = tamanho_quadrado * 2
        self.image = pygame.Surface((tamanho, tamanho))
        self.image.fill(VERMELHO_ESCURO)
        self.rect = self.image.get_rect(center=(x, y))
        self.vel = 1.0
        self.alvo = alvo
        self.vida = 10


    def update(self):
        dir_x, dir_y = vetor_direcao(self.rect.center, self.alvo.rect.center)
        self.rect.x += dir_x * self.vel
        self.rect.y += dir_y * self.vel


# Projetil inimigo rosa
class ProjetilInimigo(pygame.sprite.Sprite):
    def __init__(self, x, y, dir_x, dir_y):
        super().__init__()
        self.image = pygame.Surface((10, 10))
        self.image.fill(ROSA)
        self.rect = self.image.get_rect(center=(x, y))
        self.vel = 5
        self.dir_x = dir_x
        self.dir_y = dir_y


    def update(self):
        self.rect.x += self.dir_x * self.vel
        self.rect.y += self.dir_y * self.vel
        if not TELA.get_rect().colliderect(self.rect):
            self.kill()


# Inimigo atirador laranja
class InimigoAtirador(pygame.sprite.Sprite):
    def __init__(self, x, y, alvo):
        super().__init__()
        self.image = pygame.Surface((28, 28))
        self.image.fill(LARANJA)
        self.rect = self.image.get_rect(center=(x, y))
        self.vel = 1.5
        self.alvo = alvo
        self.ultimo_tiro = 0
        self.cooldown_tiro = 1500  # ms


    def update(self):
        dir_x, dir_y = vetor_direcao(self.rect.center, self.alvo.rect.center)
        self.rect.x += dir_x * self.vel
        self.rect.y += dir_y * self.vel


    def pode_atirar(self, tempo_atual):
        return tempo_atual - self.ultimo_tiro >= self.cooldown_tiro


    def atirar(self, grupo_projeteis, tempo_atual, todos_group=None):
        dir_x, dir_y = vetor_direcao(self.rect.center, self.alvo.rect.center)
        if dir_x == 0 and dir_y == 0:
            return
        proj = ProjetilInimigo(self.rect.centerx, self.rect.centery, dir_x, dir_y)
        grupo_projeteis.add(proj)
        if todos_group is not None:
            todos_group.add(proj)
        self.ultimo_tiro = tempo_atual


class ProjetilBoss(pygame.sprite.Sprite):
    def __init__(self, x, y, dir_x, dir_y):
        super().__init__()
        self.image = pygame.Surface((32, 16))
        self.image.fill(ROSA)
        self.rect = self.image.get_rect(center=(x, y))
        self.vel = 6
        self.dir_x = dir_x
        self.dir_y = dir_y


    def update(self):
        self.rect.x += self.dir_x * self.vel
        self.rect.y += self.dir_y * self.vel
        if not TELA.get_rect().colliderect(self.rect):
            self.kill()


class Boss(pygame.sprite.Sprite):
    def __init__(self, x, y, alvo):
        super().__init__()
        tamanho = 28 * 5
        self.image = pygame.Surface((tamanho, tamanho))
        self.image.fill((100, 0, 200))
        self.rect = self.image.get_rect(center=(x, y))
        self.vel = 1
        self.alvo = alvo
        self.vida = 100
        self.ultimo_disparo = 0
        self.cooldown_disparo = 3000


    def update(self):
        dir_x, dir_y = vetor_direcao(self.rect.center, self.alvo.rect.center)
        self.rect.x += dir_x * self.vel
        self.rect.y += dir_y * self.vel


    def pode_atirar(self, tempo_atual):
        return tempo_atual - self.ultimo_disparo >= self.cooldown_disparo


    def atirar(self, grupo_projeteis, tempo_atual):
        dir_x, dir_y = vetor_direcao(self.rect.center, self.alvo.rect.center)
        if dir_x == 0 and dir_y == 0:
            return
        spread = 0.2
        angle_central = math.atan2(dir_y, dir_x)
        angulos = [angle_central, angle_central - spread, angle_central + spread]
        for angulo in angulos:
            dx_i = math.cos(angulo)
            dy_i = math.sin(angulo)
            proj = ProjetilBoss(self.rect.centerx, self.rect.centery, dx_i, dy_i)
            grupo_projeteis.add(proj)
        self.ultimo_disparo = tempo_atual


# Partículas de disparo
class ParticulaDisparo(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        cor = random.choice([(255, 200, 50), (255, 150, 0), (255, 220, 100)])
        tamanho = random.randint(3, 6)
        self.image = pygame.Surface((tamanho, tamanho), pygame.SRCALPHA)
        pygame.draw.circle(self.image, cor, (tamanho // 2, tamanho // 2), tamanho // 2)
        self.rect = self.image.get_rect(center=(x, y))
        self.vel_x = random.uniform(-1.5, 1.5)
        self.vel_y = random.uniform(-1.5, 1.5)
        self.tempo_vida = random.randint(10, 20)
        self.alpha = 255


    def update(self):
        self.rect.x += self.vel_x
        self.rect.y += self.vel_y
        self.tempo_vida -= 1
        self.alpha = max(0, self.alpha - 25)
        self.image.set_alpha(self.alpha)
        if self.tempo_vida <= 0 or self.alpha <= 0:
            self.kill()


# =========================================================
# ⚪ Classe de Orb (triângulo giratório ao redor do jogador)
# =========================================================
class Orb(pygame.sprite.Sprite):
    def __init__(self, jogador, posicao_index, total_orbes):
        super().__init__()
        self.jogador = jogador
        tamanho = 48
        self.image_original = pygame.Surface((tamanho, tamanho), pygame.SRCALPHA)
        pygame.draw.circle(self.image_original, BRANCO, (tamanho // 2, tamanho // 2), tamanho // 2)
        self.image = self.image_original.copy()
        self.rect = self.image.get_rect(center=jogador.rect.center)
        self.dano = 5
        self.raio = 80
        self.posicao_index = posicao_index
        self.total_orbes = total_orbes


        if total_orbes == 1:
            # Um orb acima do jogador
            self.angulo_base = math.radians(-90)
        elif total_orbes == 2:
            # Dois orbs paralelos: um à esquerda (180°) e outro à direita (0°)
            self.angulo_base = [math.radians(180), math.radians(0)][posicao_index]
        else:
            # Três orbs formando triângulo ao redor do jogador
            self.angulo_base = [math.radians(-90), math.radians(30), math.radians(150)][posicao_index]


        self.angulo_giro = 0
        self.velocidade_giro = 0.02
        self.rotacao_visual = 0


    def update(self):
        self.angulo_giro += self.velocidade_giro
        angulo_total = self.angulo_base + self.angulo_giro
        cx, cy = self.jogador.rect.center
        x = cx + math.cos(angulo_total) * self.raio
        y = cy + math.sin(angulo_total) * self.raio
        self.rotacao_visual += 2
        self.image = pygame.transform.rotate(self.image_original, self.rotacao_visual)
        self.rect = self.image.get_rect(center=(x, y))


def tela_gameover(pontuacao):
    TELA.fill(PRETO)
    texto1 = fonte_gameover.render("GAME OVER", True, VERMELHO)
    texto2 = fonte.render(f"Pontos: {pontuacao}", True, BRANCO)
    texto4 = fonte.render("Pressione R para jogar novamente ou ESC para sair", True, BRANCO)


    meio_x = LARGURA // 2
    meio_y = ALTURA // 2


    TELA.blit(texto1, (meio_x - texto1.get_width() // 2, meio_y - 100))
    TELA.blit(texto2, (meio_x - texto2.get_width() // 2, meio_y))
    TELA.blit(texto4, (meio_x - texto4.get_width() // 2, meio_y + 100))


    pygame.display.flip()


    while True:
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_r:
                    return
                elif evento.key == pygame.K_ESCAPE:
                    pygame.quit()
                    sys.exit()


# Modificado para aceitar flags opcionais
def criar_inimigo_ou_boss(pontos, boss_intervalo_atual, jogador, desbloqueou_atirador=False, desbloqueou_vermelho_escuro=False):
    intervalo = pontos // 100
    if intervalo > boss_intervalo_atual:
        posicoes = [
            (random.randint(0, LARGURA), -140),
            (random.randint(0, LARGURA), ALTURA + 140),
            (-140, random.randint(0, ALTURA)),
            (LARGURA + 140, random.randint(0, ALTURA)),
        ]
        x, y = random.choice(posicoes)
        boss = Boss(x, y, jogador)
        boss_intervalo_atual = intervalo
        return boss, boss_intervalo_atual
    else:
        posicoes = [
            (random.randint(0, LARGURA), -30),
            (random.randint(0, LARGURA), ALTURA + 30),
            (-30, random.randint(0, ALTURA)),
            (LARGURA + 30, random.randint(0, ALTURA)),
        ]
        x, y = random.choice(posicoes)


        if desbloqueou_vermelho_escuro and random.random() < 0.10:
            inimigo = InimigoVermelhoEscuro(x, y, jogador)
        elif desbloqueou_atirador and random.random() < 0.2:
            inimigo = InimigoAtirador(x, y, jogador)
        else:
            inimigo = Inimigo(x, y, jogador)
        return inimigo, boss_intervalo_atual


# ---------- FUNÇÃO DE DISPARO DA ESCOPETA (agora recebe grupo de inimigos) ----------
def atirar_escopeta(jogador, tiros, todos, particulas, dx, dy, tempo_atual, ultimo_tiro_escopeta, inimigos_group):
    cooldown_escopeta = 1000
    if tempo_atual - ultimo_tiro_escopeta >= cooldown_escopeta:
        spread = 0.2
        num_tiros = 10 if jogador.tem_cadencia else 5
        angle = math.atan2(dy, dx)
        for i in range(num_tiros):
            offset = spread * (i - num_tiros // 2)
            dir_x_i = math.cos(angle + offset)
            dir_y_i = math.sin(angle + offset)
            # passa homing_enabled baseado na flag do jogador e o grupo de inimigos
            tiro = Tiro(jogador.rect.centerx, jogador.rect.centery, dir_x_i, dir_y_i,
                        homing_enabled=jogador.tem_magnetismo, target_group=inimigos_group)
            tiros.add(tiro)
            todos.add(tiro)


            for _ in range(5):
                p = ParticulaDisparo(jogador.rect.centerx, jogador.rect.centery)
                p.vel_x = random.uniform(0.5, 2.0) * dx + random.uniform(-0.5, 0.5)
                p.vel_y = random.uniform(0.5, 2.0) * dy + random.uniform(-0.5, 0.5)
                particulas.add(p)
                todos.add(p)
        return tempo_atual
    return ultimo_tiro_escopeta


def main():
    todos = pygame.sprite.Group()
    tiros = pygame.sprite.Group()
    inimigos = pygame.sprite.Group()
    bosses = pygame.sprite.Group()
    projeteis_boss = pygame.sprite.Group()
    projeteis_inimigos = pygame.sprite.Group()
    particulas = pygame.sprite.Group()
    orbes = pygame.sprite.Group()


    jogador = Jogador(LARGURA // 2, ALTURA // 2)
    todos.add(jogador)


    pontos = 0
    desbloqueou_atirador = False
    desbloqueou_vermelho_escuro = False


    SPAWN_EVENTO = pygame.USEREVENT + 1
    pygame.time.set_timer(SPAWN_EVENTO, 250)


    DISPARO_CONTINUO_EVENTO = pygame.USEREVENT + 2
    pygame.time.set_timer(DISPARO_CONTINUO_EVENTO, 0)


    arma_atual = "Metralhadora"
    disparando_continuo = False
    intervalo_metralhadora = 100
    ultimo_tiro_escopeta = 0
    boss_intervalo_atual = 0


    while True:
        clock.tick(FPS)
        TELA.fill(PRETO)
        tempo_atual = pygame.time.get_ticks()


        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                pygame.quit()
                sys.exit()


            if evento.type == SPAWN_EVENTO:
                inimigo_ou_boss, boss_intervalo_atual = criar_inimigo_ou_boss(
                    pontos, boss_intervalo_atual, jogador, desbloqueou_atirador, desbloqueou_vermelho_escuro
                )
                todos.add(inimigo_ou_boss)
                if isinstance(inimigo_ou_boss, Boss):
                    bosses.add(inimigo_ou_boss)
                else:
                    inimigos.add(inimigo_ou_boss)


            if evento.type == pygame.MOUSEBUTTONDOWN:
                if evento.button == 1:
                    mx, my = pygame.mouse.get_pos()
                    dx, dy = vetor_direcao(jogador.rect.center, (mx, my))
                    if dx == 0 and dy == 0:
                        continue


                    if arma_atual == "Metralhadora":
                        disparando_continuo = True
                        intervalo_metralhadora = 60 if jogador.tem_cadencia else 100
                        pygame.time.set_timer(DISPARO_CONTINUO_EVENTO, intervalo_metralhadora)
                    else:
                        # passe o grupo de inimigos para habilitar homing nas balas da escopeta
                        ultimo_tiro_escopeta = atirar_escopeta(
                            jogador, tiros, todos, particulas, dx, dy, tempo_atual, ultimo_tiro_escopeta, inimigos
                        )


                elif evento.button == 3:
                    if arma_atual == "Metralhadora":
                        arma_atual = "Escopeta"
                        disparando_continuo = False
                        pygame.time.set_timer(DISPARO_CONTINUO_EVENTO, 0)
                    else:
                        arma_atual = "Metralhadora"


            if evento.type == pygame.MOUSEBUTTONUP:
                if evento.button == 1 and arma_atual == "Metralhadora":
                    disparando_continuo = False
                    pygame.time.set_timer(DISPARO_CONTINUO_EVENTO, 0)


            if evento.type == DISPARO_CONTINUO_EVENTO and disparando_continuo and arma_atual == "Metralhadora":
                mx, my = pygame.mouse.get_pos()
                dx, dy = vetor_direcao(jogador.rect.center, (mx, my))
                if dx != 0 or dy != 0:
                    deslocamento = 20
                    origem_x = jogador.rect.centerx + dx * deslocamento
                    origem_y = jogador.rect.centery + dy * deslocamento


                    # ao criar tiro para metralhadora, também passamos homing baseado no jogador
                    tiro = Tiro(origem_x, origem_y, dx, dy,
                                homing_enabled=False, target_group=None)
                    tiros.add(tiro)
                    todos.add(tiro)


                    for _ in range(5):
                        p = ParticulaDisparo(origem_x, origem_y)
                        p.vel_x = random.uniform(0.5, 2.0) * dx + random.uniform(-0.5, 0.5)
                        p.vel_y = random.uniform(0.5, 2.0) * dy + random.uniform(-0.5, 0.5)
                        particulas.add(p)
                        todos.add(p)


        teclas = pygame.key.get_pressed()
        jogador.update(teclas, tempo_atual)
        tiros.update()
        inimigos.update()
        bosses.update()
        projeteis_boss.update()
        projeteis_inimigos.update()
        particulas.update()
        orbes.update()


        for boss in bosses:
            if boss.pode_atirar(tempo_atual):
                boss.atirar(projeteis_boss, tempo_atual)
                todos.add(projeteis_boss)


        # inimigos atiradores
        for inimigo in list(inimigos):
            if isinstance(inimigo, InimigoAtirador) and inimigo.pode_atirar(tempo_atual):
                inimigo.atirar(projeteis_inimigos, tempo_atual, todos_group=todos)


        # colisões tiros x inimigos
        for tiro in list(tiros):
            atingidos = pygame.sprite.spritecollide(tiro, inimigos, False)
            if atingidos:
                for inim in list(atingidos):
                    if hasattr(inim, 'vida'):
                        inim.vida -= 1
                        if inim.vida <= 0:
                            inim.kill()
                            pontos += 1
                    else:
                        inim.kill()
                        pontos += 1
                if not jogador.tem_penetracao or tiro.penetrado:
                    tiro.kill()
                else:
                    tiro.penetrado = True


        # Orb colide com inimigos e projéteis
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


            pygame.sprite.spritecollide(orb, projeteis_inimigos, True)
            pygame.sprite.spritecollide(orb, projeteis_boss, True)


        # tiros x boss
        colisoes_boss = pygame.sprite.groupcollide(tiros, bosses, True, False)
        if colisoes_boss:
            for boss_list in colisoes_boss.values():
                for boss in boss_list:
                    boss.vida -= 1
                    if boss.vida <= 0:
                        boss.kill()
                        jogador.vida += 2
                        powerups_disponiveis = []
                        if not jogador.tem_penetracao:
                            powerups_disponiveis.append("Penetração")
                        if not jogador.tem_cadencia:
                            powerups_disponiveis.append("Cadência")
                        if len(orbes) < 3:
                            powerups_disponiveis.append("Orb")
                        # NOVO: adicionar Magnetismo se ainda não tiver
                        if not jogador.tem_magnetismo:
                            powerups_disponiveis.append("Magnetismo")


                        if powerups_disponiveis:
                            melhoria = random.choice(powerups_disponiveis)


                            if melhoria == "Penetração":
                                jogador.tem_penetracao = True


                            elif melhoria == "Cadência":
                                jogador.tem_cadencia = True
                                intervalo_metralhadora = 60
                                if arma_atual == "Metralhadora" and disparando_continuo:
                                    pygame.time.set_timer(DISPARO_CONTINUO_EVENTO, intervalo_metralhadora)


                            elif melhoria == "Orb":
                                if len(orbes) < 3:
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
                                # opcional: poderia tocar som / efeito visual aqui


                        # desbloqueios
                        if not desbloqueou_atirador:
                            desbloqueou_atirador = True
                        if not desbloqueou_vermelho_escuro:
                            desbloqueou_vermelho_escuro = True


        def jogador_recebe_dano():
            jogador.vida -= 1
            jogador.invulneravel = True
            jogador.invulneravel_timer = tempo_atual
            jogador.image.fill(AZUL)


        # projéteis do boss e do inimigo laranja acertam o jogador
        if (pygame.sprite.spritecollideany(jogador, projeteis_boss) or
            pygame.sprite.spritecollideany(jogador, projeteis_inimigos)) and not jogador.invulneravel:
            jogador_recebe_dano()


        # colisão direta inimigo x jogador
        if (pygame.sprite.spritecollideany(jogador, inimigos) or pygame.sprite.spritecollideany(jogador, bosses)) and not jogador.invulneravel:
            jogador_recebe_dano()


        if jogador.vida <= 0:
            tela_gameover(pontos)
            return


        todos.draw(TELA)
        particulas.draw(TELA)
        projeteis_boss.draw(TELA)
        projeteis_inimigos.draw(TELA)


        texto_vida = fonte.render(f"Vida: {jogador.vida}", True, BRANCO)
        texto_pontos = fonte.render(f"Pontos: {pontos}", True, BRANCO)
        texto_arma = fonte.render(arma_atual, True, BRANCO)


        TELA.blit(texto_vida, (10, ALTURA - 40))
        TELA.blit(texto_pontos, (10, 10))
        TELA.blit(texto_arma, (LARGURA - texto_arma.get_width() - 10, 10))


        melhorias = []
        if jogador.tem_penetracao:
            melhorias.append("Penetração")
        if jogador.tem_cadencia:
            melhorias.append("Cadência")
        if len(orbes) > 0:
            melhorias.append("Orb")
        if jogador.tem_magnetismo:  # exibe Magnetismo na HUD
            melhorias.append("Magnetismo")
        if melhorias:
            texto_melhorias = fonte.render(" | ".join(melhorias), True, BRANCO)
            TELA.blit(texto_melhorias, (LARGURA - texto_melhorias.get_width() - 10, 40))


        pygame.display.flip()


if __name__ == "__main__":
    while True:
        tela_inicial()
        main()





