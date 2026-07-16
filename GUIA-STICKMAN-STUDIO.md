# Guia de Setup — Stickman Studio (pt-BR + Vertex AI)

> Preparado para rodar no **seu Mac**. A sessão do Claude roda num container Linux
> efêmero na nuvem, então o que foi instalado lá **não vai para a sua máquina** —
> siga os passos abaixo localmente. O que já está pronto: o repo foi clonado e
> inspecionado, o `.env` foi criado, e as mudanças de código (pt-BR + referência
> de personagem) estão no patch `stickman-studio-ptbr-e-referencia.patch`.

---

## 1. Clonar e instalar dependências (macOS)

```bash
# 1. Clonar
git clone https://github.com/saiedpod-bot/Stickman-Studio.git
cd Stickman-Studio

# 2. Python 3.11 + ffmpeg (via Homebrew)
brew install python@3.11 ffmpeg
python3.11 --version      # confirmar 3.11.x
ffmpeg -version | head -1 # confirmar que o ffmpeg está no PATH

# 3. Ambiente virtual isolado
python3.11 -m venv .venv
source .venv/bin/activate

# 4. Dependências
pip install --upgrade pip
pip install -r requirements.txt
```

> ⚠️ **Correção importante que apliquei:** o `edge-tts` era `import`ado em
> `tts_engine.py` mas **faltava no `requirements.txt`**. Adicionei. Sem isso o
> pipeline quebra na fase de narração. (Faz parte do patch.)

---

## 2. Aplicar o patch (mudanças pt-BR + referência de personagem)

Copie o arquivo `stickman-studio-ptbr-e-referencia.patch` para dentro da pasta
`Stickman-Studio/` e rode:

```bash
git apply --stat stickman-studio-ptbr-e-referencia.patch   # pré-visualizar
git apply stickman-studio-ptbr-e-referencia.patch          # aplicar
```

### O que o patch muda

| Arquivo | Mudança |
|---|---|
| `tts_engine.py` | Voz padrão passa a ser **`pt-BR-AntonioNeural`**; voz/rate/pitch configuráveis por `.env` (`TTS_VOICE`, `TTS_RATE`, `TTS_PITCH`). |
| `requirements.txt` | Adiciona `edge-tts` (estava faltando). |
| `stickman_studio/phases/phase2_images.py` | Se `CHARACTER_REFERENCE_IMAGE` apontar para uma imagem, o Imagen usa **essa imagem** como referência do personagem (ex.: Anderson Patrimônio) em vez de gerar um stickman. Descrição do sujeito configurável via `CHARACTER_SUBJECT_DESCRIPTION`. |
| `.env.example` | Novas variáveis documentadas. |

---

## 3. Configurar o `.env`

Já criei o `.env` a partir do `.env.example`. Preencha:

```dotenv
# Autenticação (uma das duas opções — ver seção 4)
GOOGLE_APPLICATION_CREDENTIALS=/caminho/absoluto/service-account.json

# Projeto Google Cloud
GCP_PROJECT_ID=seu-project-id
GCP_LOCATION=us-central1

# Bucket GCS para o Veo (sem prefixo gs://) — só se for gerar vídeo com Veo
GCS_STAGING_BUCKET=seu-bucket-staging

# Narração pt-BR
TTS_VOICE=pt-BR-AntonioNeural
TTS_RATE=+0%
TTS_PITCH=+0Hz

# Personagem Anderson Patrimônio (referência visual do Canva)
CHARACTER_REFERENCE_IMAGE=/caminho/absoluto/anderson_patrimonio.png
CHARACTER_SUBJECT_DESCRIPTION=a friendly Brazilian male cartoon character named Anderson, consistent face and outfit
```

Vozes pt-BR disponíveis (liste com `edge-tts --list-voices | grep pt-BR`):
- `pt-BR-AntonioNeural` — masculina, calorosa (boa para o Anderson)
- `pt-BR-FranciscaNeural` — feminina
- `pt-BR-ThalitaNeural` — feminina

---

## 4. Google Cloud + Service Account para Vertex AI (Gemini + Imagen)

### 4.1 Criar a conta e o projeto
1. Acesse **https://console.cloud.google.com** e faça login com sua conta Google.
2. Aceite os termos. Se for a primeira vez, ative o **Free Trial** (crédito inicial;
   exige um cartão para verificação, mas não cobra automaticamente ao fim do trial).
3. No topo, clique no seletor de projeto → **New Project** → dê um nome
   (ex.: `stickman-studio`) → **Create**.
4. Anote o **Project ID** (não o nome) → vai no `.env` como `GCP_PROJECT_ID`.

### 4.2 Ativar billing e a API do Vertex AI
1. Menu → **Billing** → associe o projeto a uma conta de faturamento.
2. Menu → **APIs & Services → Enable APIs and Services** → busque e ative:
   - **Vertex AI API** (`aiplatform.googleapis.com`) — Gemini + Imagen
   - **Cloud Storage API** — só se for usar o Veo (staging de vídeo)
   - Rápido pelo terminal (com o `gcloud` instalado):
     ```bash
     gcloud config set project SEU_PROJECT_ID
     gcloud services enable aiplatform.googleapis.com storage.googleapis.com
     ```

### 4.3 Criar a Service Account
1. Menu → **IAM & Admin → Service Accounts → Create Service Account**.
2. Nome: `stickman-vertex` → **Create and Continue**.
3. Em **Grant this service account access**, adicione a role:
   - **Vertex AI User** (`roles/aiplatform.user`)
   - (Opcional, só p/ Veo) **Storage Object Admin** (`roles/storage.objectAdmin`)
4. **Done**.

Pelo terminal, o equivalente:
```bash
gcloud iam service-accounts create stickman-vertex \
  --display-name="Stickman Studio Vertex"

gcloud projects add-iam-policy-binding SEU_PROJECT_ID \
  --member="serviceAccount:stickman-vertex@SEU_PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"
```

### 4.4 Gerar a chave JSON
1. Na service account criada → aba **Keys → Add Key → Create new key → JSON → Create**.
2. O navegador baixa um arquivo `.json`. **Guarde em local seguro** (nunca
   commite no git — o `.gitignore` já ignora `.env`, mas a chave deve ficar fora do repo).
3. No `.env`, aponte:
   ```dotenv
   GOOGLE_APPLICATION_CREDENTIALS=/Users/voce/segredos/stickman-vertex.json
   ```

Pelo terminal:
```bash
gcloud iam service-accounts keys create ~/segredos/stickman-vertex.json \
  --iam-account=stickman-vertex@SEU_PROJECT_ID.iam.gserviceaccount.com
```

> **Alternativa sem chave JSON** (mais simples e mais segura para uso local):
> deixe `GOOGLE_APPLICATION_CREDENTIALS` em branco e rode
> `gcloud auth application-default login`. O SDK usa suas credenciais de usuário.

### 4.5 Validar
O repo tem um script de verificação:
```bash
python verify_models.py
```
Ele confirma o acesso a Gemini/Imagen/Veo com as credenciais do `.env`.

> ℹ️ **Acesso ao Imagen/Veo:** dependendo da conta, Imagen 3 e Veo podem exigir
> solicitação de acesso ("allowlist") no Vertex AI. Se `verify_models.py` acusar
> permissão negada só para imagem/vídeo (mas Gemini funcionar), é isso — peça
> acesso no console do Vertex AI.

---

## 5. Anderson Patrimônio — referência visual do personagem

Acessei o design no Canva (`DAHPilbjiBM` — "Surto Financeiro Stickman with Open
Arms on Cream"). O Anderson é um **stickman desenhado à mão, traço preto sobre
fundo creme**, com **cabelo espetado, rosto expressivo (expressão preocupada),
camisa de colarinho + gravata** e braços abertos. Ou seja: é line art
monocromático (combina com o projeto), mas **mais detalhado** que o stickman
"cabeça redonda sem detalhes" que o código assumia por padrão.

### Baixar o PNG do Anderson (export do Canva)
Exportei o design em PNG (qualidade pro, fundo transparente). Baixe pelo navegador
e salve localmente (o link expira em ~10h; se expirar, eu gero outro):

```
https://export-download.canva.com/bjiBM/DAHPilbjiBM/-1/0/0001-3860094548993545421.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAQYCGKMUH5AO7UJ26%2F20260716%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20260716T015726Z&X-Amz-Expires=35634&X-Amz-Signature=6bf8f269e715776346fe8148ec3e6f28ea6049ccd03459898c376bbe03300093&X-Amz-SignedHeaders=host%3Bx-amz-expected-bucket-owner&response-expires=Thu%2C%2016%20Jul%202026%2011%3A51%3A20%20GMT
```
Salve como, por exemplo, `~/stickman/anderson_patrimonio.png` e aponte no `.env`:
```dotenv
CHARACTER_REFERENCE_IMAGE=/Users/voce/stickman/anderson_patrimonio.png
```

### Calibragem já aplicada no patch
Como o Anderson é um stickman detalhado (cabelo/rosto/roupa), o patch agora:
- **`CHARACTER_SUBJECT_DESCRIPTION`** → descreve o Anderson de fato (cabelo espetado,
  rosto expressivo, camisa + gravata, traço preto monocromático).
- **`CHARACTER_NEGATIVE_PROMPT`** (override novo) → afrouxa o negative prompt, que
  antes proibia `clothing`/`detailed illustration`/`shading` e brigava com ele.
- **`CHARACTER_STYLE_CONSTRAINTS`** (override novo) → mantém "black ink line art
  monocromático", mas deixa passar cabelo/rosto/roupa.
- O prompt de cena agora reforça: *"manter a mesma identidade, cabelo, rosto e
  roupa da referência"*.

### O que ainda vale revisar (opcional)
- `phase1_script.py` (Gemini): o system prompt ainda diz "minimalist black line
  art stickman, simple round head". Como a **identidade** é garantida pela imagem
  de referência, isso não quebra a consistência — mas se quiser que o roteiro
  descreva o Anderson explicitamente (gestos com a gravata etc.), dá pra ajustar.
- **Fundo:** configurado para **creme** (casando com a arte do Anderson) via
  `CHARACTER_STYLE_CONSTRAINTS`. O `CHARACTER_NEGATIVE_PROMPT` bloqueia só cores
  no personagem (`colorful character, saturated colors`), deixando o fundo creme
  passar. Para voltar a fundo branco, troque `solid cream / warm off-white
  background` por `plain white background`.
