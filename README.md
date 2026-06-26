# 🎙️ Meeting Transcriber Bot — WhatsApp Edition

Bot WhatsApp qui transcrit les messages vocaux et retourne un résumé structuré de réunion en français — **100% gratuit**.

## Architecture

```
Message vocal → WhatsApp Business API (Meta)
      ↓
  [Filtre : audio uniquement]
      ↓
  Meta Graph API → résolution URL → téléchargement audio
      ↓
  Groq Whisper (whisper-large-v3) → Transcription
      ↓
  Groq LLaMA (llama-3.3-70b-versatile) → Résumé structuré
      ↓
  Réponse WhatsApp avec le résumé formaté
```

## Nodes du workflow (11 au total)

| # | Node | Rôle |
|---|------|------|
| 1 | WhatsApp Trigger | Webhook — reçoit les messages entrants |
| 2 | Has Audio? | IF : route audio vs non-audio |
| 3 | Reply No Audio | Envoie "Envoie-moi un message vocal 🎙️" |
| 4 | Get Media URL | Résout l'URL de téléchargement via Graph API |
| 5 | Download Audio | Télécharge le binaire audio |
| 6 | Transcribe Audio | POST vers Groq Whisper (multipart/form-data) |
| 7 | Has Transcript? | IF : vérifie que la transcription a réussi |
| 8 | Send Error | Envoie un message d'erreur si échec |
| 9 | Prepare Summary | Code node — construit le corps de la requête Groq |
| 10 | Summarize | POST vers Groq LLaMA avec prompt structuré |
| 11 | Send Summary | Envoie le résumé formaté à l'expéditeur |

## Prérequis

Avant d'importer le workflow, il te faut :

- **WhatsApp Cloud API** — [Meta for Developers](https://developers.facebook.com) → créer une app → activer WhatsApp → récupérer le `Phone Number ID` et un `Access Token`
- **Groq API Key** — [console.groq.com](https://console.groq.com) (gratuit, sans CB)
- **n8n** — auto-hébergé (Docker) ou déployé sur [Railway.app](https://railway.app)

## Installation

### 1. Variables d'environnement

```env
WHATSAPP_ACCESS_TOKEN=...
WHATSAPP_PHONE_NUMBER_ID=...
GROQ_API_KEY=...
```

### 2. Importer le workflow

Dans n8n : **Workflows → Add Workflow → Import from File** → sélectionner `workflow.json`

### 3. Créer la credential WhatsApp dans n8n

**Credentials → Add Credential → WhatsApp Business Cloud (Trigger)**
- **App Secret** : Meta for Developers → App Settings → Basic
- **Callback Verification Token** : chaîne aléatoire de ton choix

### 4. Activer le workflow et enregistrer le webhook

1. Activer le workflow (toggle en haut à droite)
2. Copier l'URL du webhook depuis le node **WhatsApp Trigger**
3. Dans Meta for Developers → WhatsApp → Configuration → coller l'URL et le Verify Token → **Verify and Save**
4. S'abonner au champ **messages**

### 5. Tester

Envoyer un message vocal au numéro WhatsApp configuré. Réponse attendue en ~10 secondes :

```
📋 Résumé du meeting :

📌 Sujet principal
Discussion sur le lancement du produit Q2...

✅ Décisions prises
- Budget validé à 50k€
- Date de lancement fixée au 15 mai

📋 Actions à faire
- Marie : préparer le deck marketing (vendredi)
- Paul : contacter les partenaires (semaine prochaine)

🕐 Prochaines étapes
Réunion de suivi le 22 avril à 14h
```

## Coût

| Service | Plan | Coût |
|---------|------|------|
| WhatsApp Cloud API | Gratuit : 1 000 conversations/mois | 0 € |
| Groq (Whisper) | Gratuit : 14 400 req/jour | 0 € |
| Groq (LLaMA 3.3 70B) | Gratuit : 14 400 req/jour | 0 € |
| Railway.app | Gratuit : 500h/mois | 0 € |
| **Total** | | **0 €** |

## Personnalisation du prompt

Modifier le node **Prepare Summary** pour changer la langue ou le format du résumé.

## Troubleshooting

**La vérification du webhook échoue**
— Vérifier que le workflow est bien **actif** et que le Verify Token correspond exactement.

**Pas de réponse après un message vocal**
— Consulter l'onglet **Executions** dans n8n pour voir les erreurs.
— Vérifier que l'abonnement au champ `messages` est activé dans Meta.

**"❌ Désolé, je n'ai pas pu transcrire l'audio"**
— Vérifier que `GROQ_API_KEY` est bien défini.

**Access token expiré**
— Le token temporaire Meta expire en 24h. Générer un token permanent via un System User dans Meta Business Manager.
