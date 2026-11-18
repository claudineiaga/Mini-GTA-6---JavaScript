# Decisões do projeto

## Arquitetura
- **Um arquivo só (`index.html`):** é fácil de baixar, abrir e publicar no GitHub Pages, e quem está aprendendo lê tudo num lugar só.
- **Sem bibliotecas nem imagens:** tudo é Canvas 2D. O objetivo é mostrar que dá para fazer muito com o básico.
- **`PASSO` de 1 a 4:** cada camada do jogo (carro → cidade → vida → noite/polícia) liga com uma tecla, o que permite mostrar e gravar a evolução sem recarregar a página.

## Tela
- **360×640 lógico (9:16) desenhado em 720×1280 (`ESCALA = 2`):** o jogo já é vertical, pensado para Reels e Shorts, e fica nítido.
- **HUD na área segura:** estrelas, minimapa e velocímetro ficam fora da faixa onde o Reels põe a legenda e os botões.
- **Câmera:** olha à frente do carro, se afasta (zoom out) com a velocidade e treme nas batidas.

## Física do carro (`moverVeiculo`)
- A velocidade é decomposta em **pra frente** (produto escalar com a direção do carro) e **de lado**.
- O atrito lateral normal é `0.84`, e o pneu segura o carro. Com o freio de mão vira `0.97`: o carro escorrega e derrapa.
- **Giro proporcional à velocidade, com teto:** parado não vira, devagar faz curva fechada (raio ~62px, cabe nas esquinas) e rápido abre a curva.
- Quando a derrapagem passa de 1.5, aparecem marca de pneu (até 900 segmentos guardados) e fumaça.
- **Colisão com prédio eixo por eixo (x e depois y):** o carro "raspa" na parede em vez de grudar.

## Cidade (`quadra`, `solido`)
- A cidade é uma quadra de 220px (rua de 64px e calçada de 12px) repetida pra sempre.
- Cada quadra é gerada por `hash(i, j)`. Por isso volta igual quando você passa de novo, sem guardar mapa. Fica em cache, limitado a 3000 quadras.
- 12% das quadras são parques. As outras têm um prédio grande ou quatro menores, com altura, cor e neon sorteados pelo hash.

## Prédios em 3D (`desenharPredio`)
- O telhado é a base ampliada a partir do centro da câmera: `escala = CAMERA / (CAMERA - altura)`. Prédio alto "sai" mais.
- Só são desenhadas as paredes viradas para a câmera, porque as de trás ficariam escondidas.
- **Algoritmo do pintor:** os prédios são desenhados do mais longe para o mais perto.
- As janelas são feitas por interpolação bilinear entre a base e o telhado. À noite, 45% ficam acesas, sempre as mesmas.

## Noite
- `cor()` escurece e puxa para o azul (com cache).
- **Luz somada:** faróis, postes, giroflex e faíscas usam `globalCompositeOperation = 'lighter'`.
- O chão é iluminado antes dos prédios, então os prédios cobrem a luz, como na vida real.

## Trânsito e pedestres
- **Trânsito:** 18 carros na mão direita (padrão brasileiro). Nascem fora da tela e nunca em cima do jogador, e são trocados quando ficam a mais de 900px.
- Cada carro só freia por quem está **na mesma faixa e no mesmo sentido**. Se olhasse para todo mundo, os cruzamentos travariam para sempre.
- **Pedestres:** 40, andando em volta das quadras pela calçada. Correm quando um carro rápido chega perto. Com o carro devagar, pulam e desviam; acima de ~45 km/h são atropelados e ficam deitados (sem sangue).

## Polícia (`atualizarPolicia`)
- Cada estrela traz uma viatura (máx. 4), que nasce num cruzamento a ~480px.
- **IA:** mira no jogador, olha 45px à frente para desviar de prédio, dá ré se travar e vai devagar quando está perto, para encurralar.
- **Estrelas:** bater num carro em movimento ou atropelar alguém dá +1. Ser empurrado parado não conta, porque a velocidade é medida **antes** da batida.
- Se ficar 7s longe da polícia, perde 1 estrela (elas piscam antes). Se ficar parado com a polícia colada por ~1,7s, aparece **PRESO!**.
- Cada viatura tem uma fase fixa no giroflex, então a cor não troca quando a ordem da lista muda.

## Controles
- No teclado, as teclas ficam num objeto `teclas`. Se a janela perde o foco, todas são soltas, senão o carro acelera sozinho.
- Sem foco, aparece "clique aqui pra jogar".
- **Toque:** só aparece em tela de toque (`pointer: coarse`). Cada dedo é rastreado pelo `pointerId`, então dá para acelerar e virar ao mesmo tempo e arrastar o dedo entre botões.

## Loop
- **Passo fixo:** 60 atualizações por segundo, independente do monitor. O desenho acontece uma vez por quadro.
- O atraso acumulado tem teto de 100 ms, para o jogo não "disparar" depois de a aba ficar escondida.

## Testes feitos
Chrome headless no computador e no celular emulado: os 4 passos, derrapagem, teclas soltas ao perder o foco, carro do trânsito não nascendo em cima, estrela na batida e no atropelo, perseguição, prisão, toque (passo e acelerador) e ~60 fps.
