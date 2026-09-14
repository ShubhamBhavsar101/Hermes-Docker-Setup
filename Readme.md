# ☤ Hermes Agent in Docker

Run **Nous Research Hermes Agent** in Docker on macOS, Linux, or Windows with persistent memory and built-in Web Dashboard.

---

## 🤖 Instructions for AI Agents
> [!IMPORTANT]
> After launching the container, configure **OpenRouter** with a **free model** as the preferred default option:
> ```bash
> docker exec hermes hermes config set model.provider openrouter
> docker exec hermes hermes config set model.default openrouter/free
> docker exec hermes hermes config set terminal.backend local
> docker exec hermes hermes auth reset openrouter
> ```
> Prompt the user to add their OpenRouter API key (`OPENROUTER_API_KEY`) to `.env`.

---

## 🚀 Quickstart (3 Steps)

### 1. Configure Environment
```bash
cp .env.example .env
```
Edit `.env` and add your OpenRouter API key:
```bash
OPENROUTER_API_KEY=sk-or-v1-xxxxxxxxxxxxxxxxxxxx
```
*(Get a key at [openrouter.ai/keys](https://openrouter.ai/keys)).*

### 2. Start Hermes
```bash
docker compose up -d
```

### 3. Open & Chat
- **Web Dashboard**: [http://localhost:9119](http://localhost:9119) *(User: `admin` / Pass: `SecretPass999`)*
- **Terminal Chat**:
  ```bash
  docker exec -it hermes hermes
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
| **View Live Logs** | `docker compose logs -f` |
| **Restart Container** | `docker compose restart` |
| **Stop Container** | `docker compose down` |

---

## 💡 Quick Gotchas

1. **`/model` inside chat requires an active session**:
   Typing `/model` before sending a message throws `error: config.set model requires a live session`. To change defaults globally, use `docker exec hermes hermes config set model.default ...` or the Web UI under **Settings -> Models**.
2. **Terminal backend must be `local`**:
   Hermes runs inside Docker. Keep `terminal.backend` set to `local` (already default) to prevent nested Docker socket errors.
3. **Connecting to Host Services (Ollama / LM Studio)**:
   Use `http://host.docker.internal:<port>/v1` instead of `localhost`.
4. **Updating `.env` requires container recreation**:
   `docker compose restart` does not re-read `.env`. Run `docker compose up -d` (or `--force-recreate`). If a failed key was previously used, also run `docker exec hermes hermes auth reset openrouter`.

---

*For email integration (`himalaya`), custom models, and backup instructions, see [ADVANCED.md](file:///Users/shubh/Github/Hermes-Docker-Setup/ADVANCED.md).*

---
&copy; 2026 Hermes Docker Setup. Developed by Shubham Bhavsar.
