# 🎬 Pacote de Vídeo — Anderson Patrimônio

**Tema:** "Rotativo do cartão de crédito — como R$500 viram uma dívida gigante"
**Canal:** Surto Financeiro · **Formato:** Short educativo viral · **Cenas:** 5 · **Duração:** ~40s

> Este pacote traz tudo que o pipeline precisa **e** os prompts prontos para uso
> manual (Imagen/Veo/outra ferramenta): personagem, roteiro pt-BR, e por cena o
> prompt de imagem, o prompt de animação (Veo) e a narração.

---

## 0. Modelos e parâmetros recomendados (`.env`)

```dotenv
GEMINI_MODEL=gemini-1.5-pro-002
IMAGEN_GENERATE_MODEL=imagen-3.0-generate-002
IMAGEN_CAPABILITY_MODEL=imagen-3.0-capability-001

# Veo 3 (precisa de acesso allowlist no seu projeto)
VEO_MODEL=veo-3.0-generate-001        # ou veo-3.0-fast-generate-001 (mais barato/rápido)
VIDEO_SECONDS=8                       # Veo 3 renderiza ~8s por clipe
VEO_GENERATE_AUDIO=0                  # 0 = clipe mudo (usamos a narração pt-BR); 1 = áudio nativo do Veo

SCENE_COUNT=5
TTS_VOICE=pt-BR-AntonioNeural
NARRATION_LANGUAGE=Brazilian Portuguese (português do Brasil)

CHARACTER_REFERENCE_IMAGE=/caminho/anderson_patrimonio.png
```

> ⚠️ **Veo 3 + narração:** o Veo 3 gera áudio nativo (fala/SFX). Como o pipeline já
> adiciona a narração pt-BR do edge-tts na montagem, mantenha `VEO_GENERATE_AUDIO=0`
> pra não sobrepor dois áudios. Se quiser SFX de ambiente do Veo, ligue com `=1` e
> aí controle o volume na montagem.
>
> 📐 **Vertical (Shorts/Reels):** o projeto gera em **16:9** por padrão. Para 9:16,
> troque `aspect_ratio="16:9"` por `"9:16"` em `phase2_images.py` (Imagen) e em
> `phase3_video.py` (Veo).

---

## 1. Personagem (Character Reference Prompt) — EN

Use como referência global. Ideal: apontar `CHARACTER_REFERENCE_IMAGE` para o PNG do
Canva (mais fiel). Se for gerar do zero, use este prompt:

```
Anderson Patrimônio: a hand-drawn black ink line-art stickman man with short spiky
tousled hair, a simple expressive face (round eyes, worried eyebrows, small nose,
expressive mouth), wearing a short-sleeved collared shirt and a necktie, thin stick
arms and legs, monochrome black lines on a solid cream / warm off-white background,
flat, no shading, no color. Full body, centered, neutral standing pose.
```

**Negative prompt (global):**
```
colorful character, saturated colors, photorealistic, 3d render, shadows, gradients,
text, watermark, clutter, extra characters, background scenery clutter
```

---

## 2. Roteiro completo (pt-BR)

> Você paga só o mínimo do cartão e acha que tá tudo certo? Esse é o começo do surto.
> O que você não paga vira "rotativo" — o crédito mais caro que existe, com juros de
> até 15% ao mês. Aí a conta vira bola de neve: uma dívida de 500 reais não para de
> crescer. Em um ano, sem quitar, ela quase triplica. Não é azar, é matemática.
> A saída? Troque essa dívida por um empréstimo mais barato e nunca mais pague só o
> mínimo. Se esse vídeo te salvou de um surto, segue o Anderson Patrimônio.

---

## 3. Cenas (imagem + animação + narração)

Para cada cena: **Image Prompt** (EN, para Imagen — versão standalone com personagem
e estilo), **Scene action** (EN, o que vai no `storyboard.json`), **Veo prompt**
(EN, movimento) e **Narração** (pt-BR).

### 🎬 Cena 1 — The Hook (choque com a fatura)
- **Narração:** *Você paga só o mínimo do cartão e acha que tá tudo certo? Esse é o começo do surto.*
- **Image Prompt (EN):**
  ```
  Hand-drawn black line-art stickman "Anderson" (spiky hair, expressive face, collared
  shirt and necktie) standing frozen, holding a comically oversized credit-card bill,
  eyes huge and jaw dropped, sweat drops flying off his head, a couple of coins spilling
  off the paper edge. Monochrome black ink, solid cream background, flat, no shading. 16:9.
  ```
- **Scene action (storyboard):** `stands frozen holding a comically oversized credit-card bill, eyes huge and jaw dropped, sweat drops flying off his head, a couple of coins spilling off the edge of the paper`
- **Veo prompt (EN):**
  ```
  Animate this hand-drawn stickman scene with subtle motion: Anderson's eyes widen
  further, jaw drops lower, the oversized bill trembles and pulses, sweat drops shake
  loose, a coin slips off. Very slow camera push-in. Keep the black-line-art on cream
  style. ~8s, silent.
  ```

### 🎬 Cena 2 — The Trap (o rotativo)
- **Narração:** *O que você não paga vira "rotativo" — o crédito mais caro que existe, com juros de até 15% ao mês.*
- **Image Prompt (EN):**
  ```
  Hand-drawn black line-art stickman "Anderson" (spiky hair, shirt and tie) pointing
  nervously at a tiny pile of coins in his open hand, while a large dark swirling storm
  cloud looms and grows behind his back, his necktie flipping upward, glancing over his
  shoulder. Monochrome black ink, solid cream background, flat, no shading. 16:9.
  ```
- **Scene action (storyboard):** `points nervously at a tiny pile of coins in his open hand while a large dark swirling storm cloud grows behind his back, his necktie flipping upward, glancing over his shoulder`
- **Veo prompt (EN):**
  ```
  Animate with subtle motion: the dark storm cloud swells and slowly swirls behind him,
  his tie flutters upward, he leans back and glances over his shoulder nervously. Slight
  handheld sway. Keep the black-line-art on cream style. ~8s, silent.
  ```

### 🎬 Cena 3 — The Snowball (bola de neve)
- **Narração:** *Aí a conta vira bola de neve: uma dívida de 500 reais não para de crescer.*
- **Image Prompt (EN):**
  ```
  Hand-drawn black line-art stickman "Anderson" (spiky hair, shirt and tie) running
  forward in panic, arms flailing, chased by a giant rolling snowball made of crumpled
  bills and coins, bigger than he is. Motion lines. Monochrome black ink, solid cream
  background, flat, no shading. 16:9.
  ```
- **Scene action (storyboard):** `runs forward in panic with arms flailing, chased by a giant rolling snowball made of crumpled bills and coins that is bigger than he is`
- **Veo prompt (EN):**
  ```
  Animate with dynamic motion: Anderson runs in place toward the camera, legs blurring,
  while the giant money snowball rolls and grows behind him with bills fluttering off it.
  Fast dolly-back. Keep the black-line-art on cream style. ~8s, silent.
  ```

### 🎬 Cena 4 — The Math Explosion (a matemática)
- **Narração:** *Em um ano, sem quitar, ela quase triplica. Não é azar, é matemática.*
- **Image Prompt (EN):**
  ```
  Hand-drawn black line-art stickman "Anderson" (spiky hair, shirt and tie) with jaw
  dropped and head tilted up, staring at a steep line-graph arrow rocketing sharply
  upward and shooting off the top edge of the frame. Monochrome black ink, solid cream
  background, flat, no shading. 16:9.
  ```
- **Scene action (storyboard):** `jaw dropped and head tilted up, staring at a steep line-graph arrow rocketing sharply upward and shooting off the top edge of the frame`
- **Veo prompt (EN):**
  ```
  Animate with subtle motion: the steep arrow rockets further upward off the frame,
  Anderson's head tilts up following it, his jaw drops lower and his tie lifts. Subtle
  zoom-out. Keep the black-line-art on cream style. ~8s, silent.
  ```

### 🎬 Cena 5 — The Way Out (a saída)
- **Narração:** *A saída? Troque essa dívida por um empréstimo mais barato e nunca mais pague só o mínimo. Se esse vídeo te salvou de um surto, segue o Anderson Patrimônio.*
- **Image Prompt (EN):**
  ```
  Hand-drawn black line-art stickman "Anderson" (spiky hair, shirt and tie) smiling
  calmly and relieved, cutting an oversized credit card in half with big scissors, a
  simple check-mark shape floating above his head, shoulders relaxed. Monochrome black
  ink, solid cream background, flat, no shading. 16:9.
  ```
- **Scene action (storyboard):** `smiles calmly and relieved, cutting an oversized credit card in half with big scissors, a simple check-mark shape floating above his head, shoulders relaxed`
- **Veo prompt (EN):**
  ```
  Animate with subtle motion: Anderson closes the scissors and the oversized card splits
  in two, the check-mark pops in above him, he exhales and his shoulders drop, a small
  confident nod. Gentle push-in. Keep the black-line-art on cream style. ~8s, silent.
  ```

---

## 4. Como rodar (drop-in, pula o Gemini)

O pipeline usa `projects/<slug>/storyboard.json` como cache da Fase 1. Basta colocar o
arquivo pronto lá:

```bash
cd Stickman-Studio
mkdir -p projects/rotativo_do_cartao_de_credito
cp storyboard-rotativo.json projects/rotativo_do_cartao_de_credito/storyboard.json

# Animação com Veo 3 (precisa de GCS_STAGING_BUCKET no .env):
python orchestrator.py "Rotativo do cartao de credito" --video-mode animation

# OU versão barata sem Veo (Ken Burns nas imagens do Imagen):
python orchestrator.py "Rotativo do cartao de credito" --video-mode slideshow
```

O `slug` da pasta **precisa** ser `rotativo_do_cartao_de_credito` (é o `slugify` do
tópico `"Rotativo do cartao de credito"`, sem acentos). Já validei que o JSON carrega
no modelo do pipeline. ✅

Ordem das fases: **imagens (Imagen)** → **animação (Veo 3)** ou **slideshow** →
**narração (edge-tts pt-BR)** → **montagem (ffmpeg)**.

---

## 5. Dicas finais
- **Custo:** Veo 3 é caro por clipe. Teste primeiro com `--video-mode slideshow` (só
  Imagen + Ken Burns) pra validar imagens e narração; depois gaste o Veo.
- **Consistência do personagem:** mantenha `CHARACTER_REFERENCE_IMAGE` apontando pro
  PNG do Anderson — é o que garante o mesmo personagem em todas as cenas.
- **Sem texto nas imagens:** os prompts evitam palavras/labels de propósito (Imagen
  erra texto). Números como "15%" e "R$500" ficam na **narração**, não na imagem.
- **Trocar de tema:** é só editar as narrações/prompts deste arquivo (ou gerar um novo
  storyboard com o Gemini rodando sem o cache).
