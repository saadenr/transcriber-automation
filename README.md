# 🎙️ Meeting Transcriber Bot — Telegram Edition

Bot Telegram qui transcrit les messages vocaux et analyse leur contenu avec un LLM — **100% gratuit**.

## Architecture

```
Message vocal → Telegram Bot API
      ↓
  [Filtre : audio uniquement]
      ↓
  Telegram API → getFile → téléchargement audio
      ↓
  Groq Whisper (whisper-large-v3) → Transcription
      ↓
  Groq LLaMA (llama-3.3-70b-versatile) → Analyse libre
      ↓
  Réponse Telegram avec le contenu analysé
```

## Nodes du workflow (12 au total)

| # | Node | Rôle |
|---|------|------|
| 1 | Telegram Trigger | Reçoit tous les messages entrants |
| 2 | Has Audio? | Route audio vs non-audio |
| 3 | Reply No Audio | Demande un message vocal si ce n'est pas un audio |
| 4 | Get File Path | Résout le chemin du fichier via Telegram API |
| 5 | Download Audio | Télécharge le binaire audio |
| 6 | Fix Extension | Renomme `.oga` en `.ogg` pour compatibilité Groq |
| 7 | Transcribe Audio | POST vers Groq Whisper (multipart/form-data) |
| 8 | Has Transcript? | Vérifie que la transcription a réussi |
| 9 | Send Error | Envoie un message d'erreur si échec |
| 10 | Prepare Summary | Code node — construit le body Groq LLaMA |
| 11 | Summarize | POST vers Groq LLaMA 3.3 70B |
| 12 | Send Summary | Envoie la réponse à l'utilisateur |

## Prérequis

- **Telegram Bot** — créer un bot via [@BotFather](https://t.me/botfather) → récupérer le token
- **Groq API Key** — [console.groq.com](https://console.groq.com) (gratuit, sans CB)
- **n8n** — auto-hébergé (Docker) ou déployé sur Railway/Render

## Installation

### 1. Variables d'environnement

```env
TELEGRAM_BOT_TOKEN=...
GROQ_API_KEY=...
WEBHOOK_URL=https://ton-url-publique.com
N8N_BLOCK_ENV_ACCESS_IN_NODE=false
GENERIC_TIMEZONE=Europe/Paris
```

### 2. Docker (local)

```bash
docker compose up -d
```

Puis ouvre **http://localhost:5678**

> Pour exposer n8n publiquement en local, utilise [ngrok](https://ngrok.com) : `ngrok http 5678`

### 3. Importer le workflow

Dans n8n : **Workflows → Import from File** → sélectionner `workflow.json`

### 4. Configurer la credential Telegram

**Credentials → Add Credential → Telegram API** → entrer le `TELEGRAM_BOT_TOKEN`

### 5. Activer le workflow

Toggle **Active** en haut à droite → le bot est opérationnel.

## Déploiement cloud (Railway)

1. Créer un projet sur [railway.app](https://railway.app)
2. **New → Docker Image** → `n8nio/n8n`
3. **Settings → Networking → Generate Domain** (port 5678)
4. **Variables** → ajouter les variables d'environnement ci-dessus
5. Importer le workflow dans l'interface n8n Railway

## Coût

| Service | Plan | Coût |
|---------|------|------|
| Telegram Bot API | Gratuit | 0 € |
| Groq (Whisper) | Gratuit : 14 400 req/jour | 0 € |
| Groq (LLaMA 3.3 70B) | Gratuit : 14 400 req/jour | 0 € |
| Railway.app | Trial : 30 jours / 5$ | 0 € |
| **Total** | | **0 €** |

## Troubleshooting

**Le bot ne répond pas**
— Vérifier que le workflow est **actif** dans n8n.
— Vérifier que `WEBHOOK_URL` pointe vers l'URL publique de n8n.

**"❌ Désolé, je n'ai pas pu transcrire l'audio"**
— Vérifier que `GROQ_API_KEY` est correctement défini.
— Vérifier les logs dans l'onglet **Executions** de n8n.

**access to env vars denied**
— Ajouter `N8N_BLOCK_ENV_ACCESS_IN_NODE=false` aux variables d'environnement.
