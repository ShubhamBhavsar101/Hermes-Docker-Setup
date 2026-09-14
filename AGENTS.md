# Instructions for AI Agents

Guidelines and automated setup procedures for AI coding assistants working in this repository.

---

## 🤖 Post-Launch Container Configuration

After launching the container with `docker compose up -d`, automatically configure **OpenRouter** with a **free model** as the preferred default option:

```bash
docker exec hermes hermes config set model.provider openrouter
docker exec hermes hermes config set model.default openrouter/free
docker exec hermes hermes config set terminal.backend local
docker exec hermes hermes auth reset openrouter
```

---

## 🔑 Environment & Credential Guidelines

1. **OpenRouter API Key**:
   - Prompt the user to add their OpenRouter API key (`OPENROUTER_API_KEY`) to [.env](file:///Users/shubh/Github/Hermes-Docker-Setup/.env).
   - Whenever [.env](file:///Users/shubh/Github/Hermes-Docker-Setup/.env) is modified, always reload the container using `docker compose up -d` (never `docker compose restart`, which does not re-read `.env`).
   - If authentication failure occurred prior to key update, clear the exhaustion cache with:
     ```bash
     docker exec hermes hermes auth reset openrouter
     ```

2. **Telegram & Messaging Setup**:
   - Prompt the user if they wish to configure messaging platforms (Telegram, Discord, WhatsApp, etc.).
   - Interactive wizard can be triggered with:
     ```bash
     docker exec -it hermes hermes gateway setup
     ```
   - If the wizard is used and prompts `Start manually: hermes gateway`, restart the container to resume the supervised gateway process:
     ```bash
     docker compose restart
     ```
   - Alternatively, encourage declarative configuration via [.env](file:///Users/shubh/Github/Hermes-Docker-Setup/.env) using `TELEGRAM_BOT_TOKEN` and `TELEGRAM_ALLOWED_USERS`.

---

## 💡 Important Rules & Gotchas

- **Terminal Backend**: Keep `terminal.backend` set to `local`. Hermes is already running inside Docker; setting it to Docker causes nested socket errors.
- **Model Selection**: Changing models globally must be done via `docker exec hermes hermes config set model.default <slug>` or via the Web UI at [http://localhost:9119](http://localhost:9119). Typing `/model` inside chat requires an active conversation session.
- **Host Machine Services**: To connect to Ollama or LM Studio running on the host OS, always use `http://host.docker.internal:<port>/v1` rather than `localhost`.

---

&copy; 2026 Hermes Docker Setup. Developed by Shubham Bhavsar.
