# Handoff: Site de Aniversário — "Feliz 21 anos, Ana Luiza"

Uma experiência digital de aniversário: microsite mobile-first, romântico e "tecnológico"
(coração holográfico de partículas, fundo de constelação, efeito holograma ao inclinar o
celular, contagem regressiva ao vivo até o reencontro, cartas que viram em 3D e a história
real do casal contada em capítulos por data).

---

## Overview

Presente de aniversário para **Ana Luiza (21 anos)**, feito pelo namorado que está no **Japão**
enquanto ela está em **Brasília**. O site conta a história de vocês em capítulos datados, com
a foto dela dentro de um coração de luz que pulsa, e termina numa contagem regressiva até o
reencontro (**02/09/2026, 23:55**). Tudo é rolável de cima pra baixo, uma seção após a outra.

Tom da escrita: romântico, íntimo, poético mas natural (voz de um namorado apaixonado que
conhece ela de verdade). **Não** deixar infantil.

---

## About the Design Files

Os arquivos deste bundle são **referências de design criadas em HTML** — um protótipo que mostra
o visual e o comportamento pretendidos, **não** código de produção pra copiar direto.

O arquivo principal (`Feliz 21 Ana Luiza.dc.html`) é um **Design Component**: ele roda com um
runtime próprio (`support.js`), usa um template com _holes_ `{{ ... }}` e uma classe de lógica
`Component extends DCLogic` embutida no fim do arquivo dentro de um `<script type="text/x-dc">`.
Isso é ótimo pra visualizar, mas **não é a arquitetura alvo**.

**A tarefa é recriar este design no ambiente do seu codebase.** Como o pedido original era
**React / Next.js + React Three Fiber + Framer Motion**, a recomendação é implementar assim:

- **Next.js (App Router)** + React
- **three.js** via **@react-three/fiber** + **@react-three/drei** (coração de partículas, arco 3D, AR opcional)
- **Framer Motion** para os reveals no scroll, microinterações de botões e o flip dos cards
- Estilização com o que o projeto já usa (CSS Modules, Tailwind, styled-components…). Os
  valores exatos estão em **Design Tokens** abaixo.

Se ainda não existe um app, comece um Next.js limpo e implemente lá.

> Não é pra "shippar" o HTML como está. É pra reconstruir as telas com os componentes e padrões
> do app real, preservando pixel, tipografia, cores, timings e a sensação.

---

## Fidelity

**Alta fidelidade (hifi).** Cores, tipografia, espaçamentos, timings de animação e interações
são finais. Recrie a UI fielmente. Onde este doc e o HTML divergirem, **o HTML vence** (o dono
fez edições diretas no protótipo).

---

## Estrutura geral / layout global

- **Mobile-first.** Largura de leitura das seções: `max-width` entre **560–640px**, centralizado,
  com padding lateral de **22px**. No desktop o conteúdo continua nessa coluna estreita e centrada
  (é uma experiência de celular, mesmo no desktop).
- **Fundo fixo** (`position:fixed; inset:0`) em duas camadas atrás de tudo (`z-index:0`):
  1. `<canvas>` de constelação/partículas (three.js) — cobre a viewport inteira.
  2. Um overlay de **grão/textura** (SVG `feTurbulence`) com `mix-blend-mode:multiply`, `opacity ~0.5`.
- **Conteúdo** rola por cima em `z-index:1`.
- **Gradiente base do documento** (atrás do canvas):
  ```
  background:
    radial-gradient(1000px 760px at 16% 92%, rgba(150,104,60,0.15), transparent 60%),
    radial-gradient(1200px 820px at 50% -12%, rgba(74,108,240,0.20), transparent 60%),
    radial-gradient(900px 720px at 82% 24%, rgba(240,170,205,0.11), transparent 55%),
    linear-gradient(180deg,#070912,#05060e 42%,#04050b);
  ```
- **Reveal on scroll**: cada bloco marcado com `data-reveal` entra com fade + subida (translateY
  ~26px → 0), duração ~**780ms**, easing `cubic-bezier(.2,.7,.2,1)`, com `data-delay` opcional
  (0/60/80/120ms…). Em Framer Motion: `whileInView` + `viewport={{ once:true, margin:'-8%' }}`.
  (`data-reveal="scale"` = entra de `scale(.94)`; `"left"`/`"right"` = entra de ±26px no eixo X.)
- **Parallax/holograma**: elementos com `data-depth="N"` deslocam `translate3d(pX*N, pY*N*0.5, 0)`
  onde `pX/pY` ∈ [-1,1] vêm do **giroscópio** no celular (`deviceorientation`) ou do **mouse** no
  desktop, suavizados (lerp ~0.06/frame). Profundidades usadas no hero: **10** (coração) e **22** (texto).

---

## Screens / Views (todas empilhadas verticalmente, na ordem)

### 1. Hero — `#sec-hero`
- **Propósito**: abertura. A foto da Ana dentro de um coração de luz pulsando.
- **Layout**: `min-height:100svh`, flex coluna, centralizado, `text-align:center`, `padding:48px 22px 36px`.
- **Coração (o elemento-assinatura)**: container `min(86vw,400px)` quadrado, `data-depth="10"`,
  `data-reveal="scale"`, com 3 camadas sobrepostas (`position:relative; display:flex; center`):
  1. `<canvas>` do **coração de partículas** (three.js), `position:absolute; inset:0`, com máscara
     radial `mask-image:radial-gradient(circle at 50% 50%,#000 68%,transparent 92%)` (esfumaça a borda).
  2. **Glow atrás da foto**: `div` `70%×70%`, `border-radius:50%`,
     `background:radial-gradient(circle, rgba(255,120,170,0.30), rgba(217,168,119,0.12) 52%, transparent 72%)`,
     `filter:blur(10px)`.
  3. **Foto da Ana** recortada em coração: `<img src="assets/ana-heart.png">`, `width/height:63%`,
     `object-fit:contain`, `filter:drop-shadow(0 8px 22px rgba(0,0,0,0.4))`. **Pulsa** junto do
     batimento: `transform: scale(1 + 0.032*beat)` a cada frame.
- **Kicker** (mono, uppercase, `letter-spacing:.32em`, cor `--blue2`): "uma mensagem do outro lado do mundo"
- **Título** (`--serif`, `font-weight:500`, `clamp(40px,11vw,72px)`, `line-height:1.0`):
  "Feliz 21 anos," + quebra + **"Ana Luiza"** em itálico com gradiente de texto
  `linear-gradient(100deg,#ffffff,var(--rose) 52%,var(--blue2))` (via `background-clip:text`).
- **Subtítulo** (`--muted`, `clamp(15px,4.4vw,18px)`, `max-width:32ch`):
  "21 anos hoje. E eu, com sorte de te amar em todos os que ainda vêm."
- **Frase em itálico** (`--serif`, `--rose`): "bom dia, flor do dia."
- **Botão "começar"**: pílula translúcida (glass), `padding:15px 32px`, `border-radius:999px`,
  `border:1px solid rgba(255,255,255,0.18)`, `background:linear-gradient(180deg,rgba(255,255,255,.13),rgba(255,255,255,.04))`,
  `backdrop-filter:blur(10px)`, sombra rosada. Hover: `translateY(-2px)` + sombra mais forte.
  Ação: **pede permissão de giroscópio** (iOS `DeviceOrientationEvent.requestPermission()`) e faz
  scroll suave até `#sec-inicio`.
- **Dica**: "incline o celular para explorar" (mono, com um pontinho pulsando).

### 2. Onde começou — `#sec-inicio`  (capítulo **21.09.2025**)
- Kicker "21.09.2025 · onde tudo começou" → H2 "Você chegou numa festa".
- 2 parágrafos narrativos (o dono editou o texto; **use o que está no HTML**), um trecho da música
  em blockquote (borda-esquerda 2px `--rose`), e um **slot de foto** (`#js-foto-inicio`, altura 210px,
  card com borda `--line`, `border-radius:18px`).
- Fecho: "No dia seguinte, você saiu no meio da aula do cursinho só pra me ver de novo… Ainda bem que você veio."

### 3. A distância — `#sec-distancia`
- Kicker "17.000 km entre nós" → H2 "E então veio a distância".
- **Arco Brasília↔Tóquio (SVG)**: `path d="M78,152 Q300,8 522,104"`, tracejado (`stroke-dasharray:3 6`),
  `stroke:url(#arcGrad)` (gradiente `#ff9ec2`→`#8fb4ff`), com dois nós (rosa em Brasília, azul em Tóquio)
  e um **ponto de luz** que percorre o arco em loop (usa `path.getPointAtLength`, progresso ~`(t*0.16)%1`,
  opacidade pulsando com `sin(prog*π)`).
- Blockquote do verso 2 da música + fecho em itálico.

### 4. O pedido — `#sec-pedido`  (capítulo **24.12.2025**)
- Kicker "24.12.2025 · o sim" → H2 "Aí eu voltei — só pra te pedir uma coisa".
- 2 parágrafos + **slot de foto** `#js-foto-pedido` (já preenchido pelo dono com
  `media/pedido-por-do-sol.jpeg`, `height:300px`, `background:center/cover`).

### 5. Ano Novo, na roça — `#sec-roca`  (capítulo **31.12.2025**)
- Kicker "31.12.2025" → H2 "O Ano Novo, na roça". Parágrafo + **2 slots** lado a lado
  (`grid-template-columns:1fr 1fr; gap:12px`): `#js-foto-roca1`, `#js-foto-roca2`.

### 6. A música — `#sec-musica`
- Kicker "a nossa música" → H2 "Eu escrevi uma canção / tentando te explicar".
- **Card de vídeo** (clicável): `#js-video-poster` como capa + overlay escuro + botão play
  redondo (glass). Ao clicar → **modal de vídeo** (ver Interactions). Se `videoUrl` estiver vazio,
  o modal mostra um placeholder "seu vídeo cantando entra aqui".
- **Card da letra**: bloco `--cream`/glass com 3 estrofes; o refrão em `--serif` itálico maior,
  com a linha final destacada em `--rose`: _"é só você dizer: bom dia, flor do dia."_

### 7. A trilha — `#sec-trilha`
- H2 "As músicas que são nossas". Lista de 3 itens (cards):
  1. **"Flor do Dia" — a que eu fiz pra você** → **clicar toca o áudio** (`audioUrl`).
  2. **Jota Pê** — "o MPB que você ama" (decorativo).
  3. **Djavan** — "pros nossos dias" (decorativo).

### 8. 21 motivos pra te amar — `#sec-motivos`
- H2 "21 motivos pra te amar" + linha em `--hand`/itálico "um pra cada ano seu — toque em cada carta".
- **Grid** `repeat(auto-fill,minmax(150px,1fr))`, `gap:12px`, **21 cards que viram em 3D** (flip).
  - Frente: número **01–21** (`--serif`, ~30px, `--rose`/terracota) + "toque".
  - Verso (`rotateY(180deg)`): o motivo, `font-size:12.5px`, fundo rosado translúcido.
  - Flip: `transform-style:preserve-3d`; ao tocar, anima `rotateY` 0↔180 em ~**520ms** (cúbico),
    e troca a opacidade das faces no ponto de 90° (ver nota técnica em Interactions).
  - **Os 21 textos estão no array `MOTIVOS`** no topo do `<script>` — copie-os na íntegra.

### 9. O reencontro (contagem) — `#sec-reencontro`  (**02.09.2026, 23:55**)
- Kicker "02.09.2026 · 23:55" → H2 "E tem um dia marcado no céu".
- **Card de contagem** (glass): "faltam, até te reencontrar" + **dias : horas : min : seg** ao vivo
  (atualiza a cada 1s; segundos em `--rose`). Rótulo com a data formatada em pt-BR.
- **Mensagem lacrada** (card tracejado): enquanto `agora < data` mostra "tem uma mensagem lacrada
  aqui / ela abre quando você chegar · 02.09, 23:55". Quando a data chega (`arrived`), revela o
  texto: _"Você chegou. Acabou a distância. Chega mais perto — agora é de verdade. Bem-vinda de
  volta pra mim, minha flor do dia."_
- Dois parágrafos de fecho (o que ela precisa ouvir).

### Footer
- Timeline em `--hand`: "21.09 nos conhecemos · 24.12 o sim · 02.09 o reencontro" + "feito do
  Japão, com amor" + "pros seus 21, minha flor do dia".

### Modais (overlay `position:fixed; inset:0; z-index:120`, fundo escurecido + `backdrop-filter:blur`)
- **Vídeo** (`modalVideo`): `<video controls playsinline>` com `src = videoUrl`; ou placeholder.
- **Carta** (`modalLetter`): card de "papel" com a carta completa (texto abaixo em **Conteúdo/Copy**).
  ⚠️ **Estado atual: a carta NÃO tem gatilho na UI** — a seção "Ler minha mensagem" foi removida a
  pedido do dono. O modal e o handler `openLetter` continuam no código. Decida com o dono se quer
  religar um botão (ex.: no fim do `#sec-reencontro` ou no footer) ou remover de vez.

---

## Interactions & Behavior

- **Reveals no scroll**: fade+slide, ~780ms, `cubic-bezier(.2,.7,.2,1)`, dispara quando o topo do
  elemento passa de ~92% da viewport, **uma vez**. (Em React: `IntersectionObserver` ou
  `motion` `whileInView`.)
- **Parallax/holograma** (giroscópio no mobile, mouse no desktop): ver "Estrutura geral". Requer
  permissão de movimento no iOS — pedida no clique do botão "começar".
- **Botões**: microinteração hover `translateY(-2px)` + sombra; active `scale(0.98)`.
- **Flip dos cards**: 3D flip 520ms. Nota técnica: no protótipo, `backface-visibility` não era
  confiável no ambiente de preview, então a troca de face é feita por **opacidade** no meio do giro
  (ângulo > 90° mostra o verso). Em React/Framer Motion com `rotateY`, `backfaceVisibility:'hidden'`
  costuma funcionar — mantenha o crossfade como fallback.
- **Contagem regressiva**: `setInterval` 1s até `reunionDate`; quando chega a zero, seta `arrived`
  e revela a mensagem lacrada.
- **Música**: botão flutuante fixo (canto inferior direito) com 3 barrinhas estilo equalizer
  animando quando toca. **Nunca** dá autoplay alto — só toca no toque do usuário. Tooltip
  "toque para ouvir a nossa música". Tocar "Flor do Dia" na seção Trilha também aciona.
- **Modais**: abrir trava o scroll (`documentElement.style.overflow='hidden'`); fecha no X, no
  clique fora, ou `Esc`. O vídeo pausa ao fechar.
- **prefers-reduced-motion**: desliga pulsos/parallax/reveals (conteúdo aparece estático).

---

## State Management

Props (configuráveis; hoje editáveis no protótipo, viram props/CMS/env no app):
- `nome` (string, default "Ana Luiza")
- `reunionDate` (ISO local, default `"2026-09-02T23:55"`)
- `videoUrl` (string; vazio = placeholder no modal)
- `audioUrl` (string; a música "Flor do Dia")
- `tilt` (boolean, default true) — liga/desliga o holograma por giroscópio
- `showHearts` (boolean, default true) — coraçõezinhos flutuando no fundo

Estado de runtime:
- `modal`: `null | 'video' | 'letter'`
- `playing`: boolean (áudio)
- `hint`: boolean (tooltip do player, some após ~9s)
- `arrived`: boolean (data do reencontro chegou)
- Refs para canvases (coração, fundo), `<audio>`, `<video>`, dígitos da contagem, path/ponto do arco.
- Loop de animação único via `requestAnimationFrame` (atualiza fundo, coração, arco, parallax,
  e agenda checagem de reveals). Fallback por `setTimeout` garante que o conteúdo apareça mesmo
  se o rAF pausar.

Data fetching: nenhum. Tudo local/estático. O `<video>`/`<audio>` podem ser arquivos hospedados
ou embeds; hoje esperam URL direta.

---

## Design Tokens

**Fontes** (Google Fonts): 
- Serif de display/emoção: **Cormorant Garamond** (400/500/600, itálico) — títulos, frases românticas.
- Sans de UI: **Space Grotesk** (400/500/600/700) — corpo, botões, subtítulos.
- Mono de rótulos: **Space Mono** (400/700) — kickers, datas, labels uppercase.
- (No protótipo há também referências a `--hand` como caligrafia p/ alguns rótulos do footer/motivos;
  se quiser esse toque manuscrito use **Caveat**. Caso contrário, mantenha Cormorant itálico.)

**Cores** (o protótipo usa OKLCH; equivalentes hex aproximados ao lado):
```
--blue    oklch(0.68 0.15 250)   ≈ #4a6cf0   (azul profundo)
--blue2   oklch(0.80 0.10 240)   ≈ #8fb4ff   (azul claro / kickers)
--rose    oklch(0.82 0.09 353)   ≈ #f5b8cf   (rosa perolado / destaques)
--rose2   oklch(0.72 0.13 358)   ≈ #e8849a   (rosa mais forte)
--gold    oklch(0.85 0.08 85)    ≈ #ecd6a6   (dourado suave)
--brown   oklch(0.78 0.075 55)   ≈ #d9a877   (marrom claro/caramelo — cor favorita dela)
--brown2  oklch(0.52 0.07 48)    ≈ #8a6a45   (marrom médio)
--pearl   oklch(0.96 0.012 90)   ≈ #f6f3ec   (branco perolado — texto principal)
--muted   rgba(226,231,255,0.60)             (texto secundário)
--glass   rgba(255,255,255,0.045)            (fundo de cards translúcidos)
--line    rgba(255,255,255,0.09)             (bordas hairline)
Base bg   #04050b / #05060e / #070912        (quase-preto azulado)
```
Partículas do coração: rosa `#ff8fbc`, perolado `#ffe9f0`, azul `#8fb4ff`, caramelo `#d9a877`.
Glow do coração (sprite aditivo): `#ff5f9a`.

**Tipografia (escala real usada)**
- H1 hero: `clamp(40px,11vw,72px)`, weight 500, line-height 1.0
- H2 seções: `clamp(27px,7.6vw,44px)` (algumas `7.4vw,42px`), weight 500, line-height ~1.1
- Corpo: `clamp(14px,4.2vw,16.5px)`, line-height 1.8
- Kicker/label: 10.5px, `letter-spacing:0.3em`, uppercase, mono
- Contagem (dígitos): `clamp(30px,9vw,44px)`, mono 700

**Espaçamento**: seções com `padding` vertical ~20–84px; padding lateral 22px; gaps de 10–18px.
**Radius**: cards 14–24px; pílulas/botões 999px; slots de foto 18–20px.
**Sombras**: sombras longas e suaves, ex. `0 30px 60px -44px rgba(0,0,0,0.9)`; `inset 0 1px 0 rgba(255,255,255,0.06)` pra brilho de topo.
**Batimento (heartbeat)**: período ~1.15s, dois picos (duplo-batimento) em ~12% e ~34% do ciclo.

---

## 3D / three.js (o que recriar em @react-three/fiber)

1. **Fundo (canvas fixo)**: ~440–920 estrelas (Points, cores perola/azul/rosa, blending aditivo),
   ~32–70 "motes" de luz subindo devagar, e ~4–7 sprites de coração translúcidos flutuando
   (`showHearts`). Câmera reage levemente ao parallax (pX/pY) e ao scroll.
2. **Coração de partículas (hero)**: ~1700 (mobile) a 3800 pontos amostrados na curva paramétrica
   de coração (`x=16sin³t`, `y=13cos t−5cos2t−2cos3t−cos4t`), com profundidade em Z (mais fino nas
   bordas), textura de ponto suave, blending aditivo. Pulsa no batimento (escala + opacidade do
   glow). Rotação sutil por tilt (sem giro completo, pra não desalinhar com a foto na frente).
3. **Arco Brasília↔Tóquio**: é **SVG** (não three), com ponto de luz correndo pelo path.
4. **AR (opcional, futuro)**: ver abaixo.

Textura da foto no coração: a imagem `assets/ana-heart.png` já é a foto da Ana **recortada em
coração com bordas esfumaçadas** (PNG com alpha). Para trocar a foto: gerar novo PNG com a mesma
máscara de coração (ou aplicar `mask-image` de um SVG de coração sobre um `<img>`/textura).

---

## AR de verdade (preparado pro futuro — pendente de asset)

O dono quer, no futuro, um botão discreto **"ver em AR"** com bonecos 3D dele e da Ana.
- **iOS (Safari)**: AR Quick Look — precisa de um arquivo **`.usdz`**. Basta um
  `<a rel="ar" href="modelo.usdz"><img></a>`.
- **Android**: **Scene Viewer** com **`.glb`** via intent, ou `<model-viewer ar>`.
- Recomendado: usar **`<model-viewer>`** (`@google/model-viewer`) que cobre `.glb` + `.usdz` e já
  dá o botão de AR. **Bloqueado até o dono fornecer os modelos 3D** (photo→3D confiável não é
  gerável automaticamente; provável caminho é encomendar figurinhas 3D a partir de fotos).
- Deixe o botão AR **condicional**: só aparece se houver `modelGlbUrl`/`modelUsdzUrl`.

---

## Assets

- `assets/ana-heart.png` — foto da Ana recortada em coração (bordas suaves, alpha). **Usada no hero.**
- `assets/ana-circle.png` — mesma foto em recorte circular suave (alternativa, não usada agora).
- `media/pedido-por-do-sol.jpeg` — foto real do pôr do sol do pedido (usada em `#js-foto-pedido`).
- **Slots a preencher** (o dono troca as fotos depois): `#js-foto-inicio`, `#js-foto-roca1`,
  `#js-foto-roca2`, `#js-video-poster`. No app, troque por `<img>`/`next/image` com upload/props.
- **Pendentes do dono**: vídeo dele cantando (`videoUrl`), áudio da música "Flor do Dia"
  (`audioUrl`), fotos das seções, e (futuro) modelos 3D pra AR.
- Fonte original da foto do hero: `uploads/pasted-1782966133918-0.png` (não incluída; o recorte
  final já está em `assets/ana-heart.png`).

**Letra da música "Flor do Dia"** (de autoria do dono — usar como está, na seção `#sec-musica`):
está reproduzida no HTML da seção. O refrão-chave: _"Você chegou tão leve na minha vida, / fez
minha manhã virar pura poesia. / E se quiser me acordar sorrindo… / é só você dizer: 'bom dia,
flor do dia'."_

**Carta (modal, texto integral)** — está no HTML dentro de `<!-- LETTER MODAL -->`. Começa em
"Meu amor, minha flor do dia," e passa por 21.09 (a festa/o sofá/a voz), 24.12 (o gramado/o
violão/o pôr do sol), o orgulho da medicina ("ninguém do seu nível"), os 21 anos, e fecha com
"Feliz 21 anos, minha flor do dia." Copie na íntegra do arquivo.

---

## Files (neste bundle)

- `Feliz 21 Ana Luiza.dc.html` — **o design de referência completo** (template + lógica no
  `<script type="text/x-dc">` no fim do arquivo). É a fonte da verdade pra copy, medidas e lógica.
- `support.js` — runtime do Design Component (**só pra abrir o protótipo**; não portar).
- `image-slot.js` — web component dos slots de foto arrastáveis (**só pro protótipo**; no app use `<img>`).
- `assets/ana-heart.png`, `assets/ana-circle.png` — recortes da foto da Ana.
- `media/pedido-por-do-sol.jpeg` — foto do pedido.

### Como abrir o protótipo localmente
Sirva a pasta por HTTP (não `file://`, por causa dos módulos/CORS):
```
cd design_handoff_aniversario_ana_luiza
python3 -m http.server 8000
# abra http://localhost:8000/Feliz%2021%20Ana%20Luiza.dc.html
```

---

## Checklist de implementação (sugerida)

- [ ] Scaffold Next.js + R3F + Framer Motion.
- [ ] Tokens (cores/fontes/escala) no tema do projeto.
- [ ] Fundo fixo (constelação + motes + coraçõezinhos) em `<Canvas>` R3F.
- [ ] Hero: coração de partículas + foto em coração pulsando + parallax por giroscópio/mouse.
- [ ] Seções-capítulo (21.09 / distância / 24.12 / roça) com reveals no scroll e slots de foto.
- [ ] Seção música (modal de vídeo) + seção trilha (tocar áudio).
- [ ] 21 cards com flip 3D (array `MOTIVOS` na íntegra).
- [ ] Contagem regressiva + mensagem lacrada que abre na data.
- [ ] Player de música flutuante (sem autoplay alto).
- [ ] Decidir com o dono: religar a **carta** (modal existe, falta gatilho).
- [ ] Placeholders prontos pra: vídeo, áudio, fotos e (futuro) AR `.glb/.usdz`.
- [ ] Respeitar `prefers-reduced-motion` e performance no celular.
