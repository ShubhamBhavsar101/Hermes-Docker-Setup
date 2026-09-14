# ☤ Hermes Agent in Docker

Run **Nous Research Hermes Agent** in Docker on macOS, Linux, or Windows with persistent memory, built-in Web Dashboard, and Telegram messaging integration.

---

## 🚀 Quickstart (3 Simple Steps)

### 1. Configure Environment
```bash
cp .env.example .env
```
Open [.env](file:///Users/shubh/Github/Hermes-Docker-Setup/.env) and configure your keys:

- **OpenRouter** *(Required for LLM responses)*:
  ```bash
  OPENROUTER_API_KEY=sk-or-v1-xxxxxxxxxxxxxxxxxxxx
  ```
  *(Grab a key at [openrouter.ai/keys](https://openrouter.ai/keys) — free models available).*

- **Telegram** *(Optional — to chat from your phone)*:
  ```bash
  TELEGRAM_BOT_TOKEN=123456789:ABCdefGhIJKlmNoPQRsTUVwxyZ
  TELEGRAM_ALLOWED_USERS=your_telegram_user_id
  ```
  *(See the [Telegram Setup Guide](#-telegram-setup) below for how to get these).*

---

### 2. Start Hermes
```bash
docker compose up -d
```

Configure OpenRouter with a free default model:
```bash
docker exec hermes hermes config set model.provider openrouter
docker exec hermes hermes config set model.default openrouter/free
docker exec hermes hermes config set terminal.backend local
docker exec hermes hermes auth reset openrouter
```

---

### 3. Start Chatting!

You can talk to Hermes in three ways:

1. **Telegram**:
   Open Telegram, find your bot, and send `/start` or `hi`.
2. **Web Dashboard**:
   Open [http://localhost:9119](http://localhost:9119) in your browser.
   - **Username**: `admin`
   - **Password**: `SecretPass999`
3. **Interactive Terminal Chat**:
   ```bash
   docker exec -it hermes hermes
   ```
   *(Or for the full terminal UI, run `docker exec -it hermes hermes --tui`).*

---

## ✈️ Telegram Setup

Hermes includes a built-in messaging gateway to chat seamlessly from Telegram:

### Step 1: Create Your Bot
1. Open Telegram and search for [@BotFather](https://t.me/BotFather).
2. Send `/newbot`, choose a display name and a unique username ending in `bot` (e.g., `my_hermes_bot`).
3. Copy the **HTTP API Bot Token** provided.

### Step 2: Get Your User ID (Security)
1. Chat with [@userinfobot](https://t.me/userinfobot) on Telegram.
2. Copy your numeric user ID (e.g. `987654321`). This prevents unauthorized people from messaging your bot.

### Step 3: Add to `.env` & Recreate Container
Add both to [.env](file:///Users/shubh/Github/Hermes-Docker-Setup/.env):
```bash
TELEGRAM_BOT_TOKEN=123456789:ABCdefGhIJKlmNoPQRsTUVwxyZ
TELEGRAM_ALLOWED_USERS=987654321
```
Recreate the container:
```bash
docker compose up -d
```

*(Alternatively, you can run the interactive setup wizard inside the container using `docker exec -it hermes hermes gateway setup`, followed by `docker compose restart`).*

---

## 🔑 Updating API Keys (.env)

When changing or adding API keys in [.env](file:///Users/shubh/Github/Hermes-Docker-Setup/.env):

1. **Update [.env](file:///Users/shubh/Github/Hermes-Docker-Setup/.env)** with your new key.
2. **Recreate the container** *(Note: `docker compose restart` does **not** reload `.env`; always use `docker compose up -d`)*:
   ```bash
   docker compose up -d
   ```
3. **Reset cached authentication status** (clears any rate limit / auth exhaustion):
   ```bash
   docker exec hermes hermes auth reset openrouter
   ```

---

## ⚡ Command Cheat Sheet

| Action | Command |
|---|---|
| **Chat in Terminal** | `docker exec -it hermes hermes` |
| **Terminal UI (TUI)** | `docker exec -it hermes hermes --tui` |
| **Change Default Model** | `docker exec hermes hermes config set model.default <model-slug>` |
| **Pick Model Interactively** | `docker exec -it hermes hermes model` |
| **Reset Provider / Rate Limits**| `docker exec hermes hermes auth reset openrouter` |
| **Reload .env / New API Key**| `docker compose up -d && docker exec hermes hermes auth reset openrouter` |
| **Configure Messaging (Telegram, etc.)** | `docker exec -it hermes hermes gateway setup` |
| **View Live Logs** | `docker compose logs -f` |
| **Restart Container** | `docker compose restart` |
| **Stop Container** | `docker compose down` |

---

## 💡 Quick Gotchas

1. **Updating `.env` requires container recreation**:
   `docker compose restart` does not re-read `.env`. Always run `docker compose up -d`. If an invalid key was previously attempted, also run `docker exec hermes hermes auth reset openrouter`.
2. **`/model` inside chat requires an active session**:
   Typing `/model` before sending a message throws `error: config.set model requires a live session`. To change defaults globally, use `docker exec hermes hermes config set model.default ...` or the Web UI under **Settings -> Models**.
3. **Terminal backend must be `local`**:
   Hermes runs inside Docker. Keep `terminal.backend` set to `local` (already default) to prevent nested Docker socket errors.
4. **Connecting to Host Services (Ollama / LM Studio)**:
   Use `http://host.docker.internal:<port>/v1` instead of `localhost`.

---

## 📚 More Documentation

- **For Autonomous Coding Assistants & AI Agents**: See [AGENTS.md](file:///Users/shubh/Github/Hermes-Docker-Setup/AGENTS.md).
- **For Advanced Providers, Email Tools (`himalaya`), & Backups**: See [ADVANCED.md](file:///Users/shubh/Github/Hermes-Docker-Setup/ADVANCED.md).

---

&copy; 2026 Hermes Docker Setup. Developed by Shubham Bhavsar.
