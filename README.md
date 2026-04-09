# SAMWAD-showcase

> Real-time multilingual chat app where every message is auto-translated into each recipient's preferred language — powered by Groq's LLaMA AI.

---

## What is Samwad?

**Samwad** (संवाद — Hindi for *conversation*) is a real-time messaging app with built-in AI translation. You write in your language, your friend reads in theirs — automatically, on every message, with no manual translate button.

It supports three modes of communication:
- **Direct Messages (DM)** — one-on-one private chat
- **Room Chat** — join or create a room with a code, chat with multiple people
- **Global Net** — broadcast to everyone currently online, each person receiving it in their own language

---

## Features

- AI-powered per-message translation via Groq (LLaMA)
- Google OAuth login — no passwords
- Unique 5-digit SamwadID for adding friends
- Persistent message history stored per conversation
- Per-user preferred language setting (persists across sessions)
- Virtual multilingual keyboard with 10+ script layouts (Hindi, Punjabi, Gujarati, Arabic, etc.)
- Global Net live user count
- Read receipts on DMs
- Holographic dark UI with Three.js animated landing page
- Fully responsive — works on mobile and desktop

---

## Tech Stack

### Backend
| | |
|---|---|
| **Node.js** | JavaScript runtime |
| **Express.js** | HTTP server and routing |
| **Socket.IO** | Real-time WebSocket communication |
| **Groq SDK** | LLaMA AI for message translation |
| **Passport.js** | Authentication middleware |
| **passport-google-oauth20** | Google OAuth 2.0 login strategy |
| **express-session** | Session management |
| **Nodemailer** | Contact form email dispatch |
| **dotenv** | Environment variable management |

### Frontend
| | |
|---|---|
| **Vanilla HTML/CSS/JS** | No framework — zero bundle size |
| **Socket.IO client** | Real-time events in the browser (loaded via CDN) |
| **Three.js r128** | 3D animated landing page background |
| **Google Fonts** | Rajdhani, Share Tech Mono, Noto Sans (10+ script variants) |

### Storage
| | |
|---|---|
| **users.json** | User directory — Google ID, SamwadID, friends, preferred language |
| **messages.json** | All conversation history — keyed by sorted SamwadID pairs |

### Hosting
| | |
|---|---|
| **ngrok** | Exposes local server to the public internet |

---

## Project Structure

```
samwad/
├── server.js          # Main server — Express, Socket.IO, Groq, Passport
├── index.html         # Full chat app UI (DM + Room + Global Net)
├── landing.html       # Public landing page with Three.js background
├── users.json         # User database (flat file)
├── messages.json      # Message history (flat file)
├── .env               # Secret keys (not committed)
└── public/            # Static assets served at /app
```

---

## Setup & Running

### 1. Prerequisites

- Node.js v18+
- ngrok account (free tier works)
- Google Cloud project with OAuth 2.0 credentials
- Groq API key (free at console.groq.com)

### 2. Install dependencies

```bash
npm install express socket.io groq-sdk express-session passport passport-google-oauth20 nodemailer dotenv
```

### 3. Create `.env` file

```env
GROQ_API_KEY=your_groq_api_key
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
SESSION_SECRET=any_random_secret_string
EMAIL_USER=your_gmail_address
EMAIL_PASS=your_gmail_app_password
```

### 4. Update Google OAuth callback URL

In `server.js`, update the `callbackURL` to match your ngrok domain:

```js
callbackURL: 'https://YOUR-NGROK-SUBDOMAIN.ngrok-free.app/auth/google/callback'
```

Also add this same URL in your Google Cloud Console under **Authorized redirect URIs**.

### 5. Start the server

```bash
node server.js
```

Server runs on `127.0.0.1:3002`.

### 6. Expose with ngrok

```bash
ngrok http 3002
```

Copy the `https://...ngrok-free.app` URL — that's your public link. Share it with anyone to use the app.

---

## How Translation Works

Every time a message is sent, the server:

1. Receives the original message and the sender's language
2. Looks up each recipient's `preferredLang` from `users.json`
3. Calls Groq's LLaMA model once per unique recipient language:
   ```
   "Translate from [senderLang] to [recipientLang]. Output ONLY the translated text."
   ```
4. Delivers the translated version to each recipient via Socket.IO
5. Saves both `text` (original) and `translatedText` (translated) to `messages.json`

In a group chat with 4 users speaking 4 different languages, one message triggers 4 separate Groq API calls — each recipient gets the message in their own language.

---

## How SamwadID Works

On first login, a unique 5-digit ID is generated using a **collision-retry** algorithm:

```js
function generateSamwadId(db) {
    let newId;
    const existingIds = Object.values(db).map(u => u.samwadId);
    do {
        newId = Math.floor(10000 + Math.random() * 90000).toString();
    } while (existingIds.includes(newId));
    return newId;
}
```

- Range: `10000` to `99999` → 90,000 possible IDs
- If the random number is already taken, it retries until a free one is found
- Users share this ID to add friends (like a phone number)

---

## Message Storage Format

Conversations are stored in `messages.json` keyed by both SamwadIDs sorted and joined:

```json
"57993_65664": [
  {
    "from": "57993",
    "fromName": "Abhay Gupta",
    "text": "Hello my friend is Shivam",
    "translatedText": "ਸਤ ਸ੍ਰੀ ਅਕਾਲ ਮੇਰਾ ਮਿੱਤਰ ਸ਼ਿਵਮ ਹੈ",
    "timestamp": 1775655990635,
    "read": true
  }
]
```

The key is always `[smallerId]_[largerId]` — so `57993_65664` is the same conversation whether Abhay or Kunwar looks it up.

---

## Supported Languages

The Noto Sans font variants loaded in the frontend support rendering of:

Hindi · Punjabi (Gurmukhi) · Gujarati · Bengali · Tamil · Telugu · Kannada · Malayalam · Odia · Arabic · Japanese · German · English · and more

Translation is handled by Groq/LLaMA which supports virtually all major languages.

---

## Accessing the App

The server runs locally on a **MacBook** at `127.0.0.1:3002`. ngrok creates a secure tunnel from that local port to a public URL, so anyone on the internet can reach it without any cloud server or deployment but provided the server of localhost is on.

**Public URL:**
```
https://missy-homemade-rainily.ngrok-free.dev/
```

Here's exactly what happens when a user visits that link:

```
User opens https://missy-homemade-rainily.ngrok-free.dev/
        ↓
Request hits ngrok's servers (on the internet)
        ↓
ngrok tunnels it to MacBook → localhost:3002
        ↓
Express serves landing.html  ← the public landing page
        ↓
User clicks "Launch" / "Sign in with Google"
        ↓
Redirected to Google OAuth → user approves → callback to /auth/google/callback
        ↓
Passport creates session → redirected to /app
        ↓
Express serves index.html  ← the full chat app
        ↓
Socket.IO client connects back to the same ngrok URL over WebSocket
        ↓
User is live — messages flow in real time through the MacBook
```

> **Note:** The ngrok free tier assigns a fixed subdomain (`missy-homemade-rainily`) only if you've reserved it in your ngrok dashboard. If the tunnel is restarted without a reserved domain, the URL changes and the Google OAuth callback URL in both `server.js` and Google Cloud Console must be updated to match.

---

## Known Limitations

- **No database** — `users.json` and `messages.json` are read/written on every request. Not suitable for high concurrency or large scale.
- **ngrok free tier** — the public URL changes every time ngrok restarts. You'll need to update the Google OAuth callback URL each time.
- **No media messages** — text only.
- **Single server** — no horizontal scaling.

---

## Author

Built by **Kunwar Gaba** · IIIT Allahabad
