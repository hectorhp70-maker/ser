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

O pipeline original **gera** um stickman via Imagen. Com o patch, se você definir
`CHARACTER_REFERENCE_IMAGE`, ele passa a **usar a sua arte do Anderson** como
referência fixa, mantendo o mesmo personagem em todas as cenas (via
subject-customization do Imagen 3 capability model).

Para eu ajustar os prompts/descrição com precisão, **me envie a imagem do Anderson
Patrimônio** (ou o link do Canva / a descrição visual: cor de pele, cabelo, roupa,
estilo — cartoon flat? traço? paleta?). Com a imagem em mãos eu:
- calibro o `CHARACTER_SUBJECT_DESCRIPTION`,
- ajusto o `_NEGATIVE` em `phase2_images.py` (hoje ele proíbe cor/roupa/detalhe,
  o que **conflita** com um personagem colorido — precisa afrouxar para o Anderson),
- e reviso o system prompt do Gemini em `phase1_script.py`, que hoje instrui
  "minimalist black line art stickman" (isso também precisa mudar para o Anderson).

> ⚠️ Ponto de atenção honesto: o projeto inteiro é modelado para **stickman
> preto-e-branco minimalista** (prompts, negative prompts, instrução do Gemini).
> Trocar para um personagem de marca colorido não é só apontar a imagem — os
> prompts de estilo brigam com isso. O patch abre o mecanismo; a calibragem fina
> dos prompts eu faço quando você me mandar a referência.
