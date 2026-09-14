# Instructions for AI Agents

Guidelines and automated setup procedures for AI coding assistants working in this repository.

---

## 🛑 Step 1: Pre-Launch Environment Setup (Mandatory First Step)

**BEFORE running `docker compose up -d` or executing any container setup commands, the agent MUST:**

1. **Verify `.env` Existence**:
   - Check if [.env](file:///Users/shubh/Github/Hermes-Docker-Setup/.env) exists.
   - If [.env](file:///Users/shubh/Github/Hermes-Docker-Setup/.env) does not exist, copy [.env.example](file:///Users/shubh/Github/Hermes-Docker-Setup/.env.example) to [.env](file:///Users/shubh/Github/Hermes-Docker-Setup/.env) (`cp .env.example .env`).

2. **Prompt the User for Environment Variables First**:
   - **OpenRouter API Key (`OPENROUTER_API_KEY`)**: Ask the user to provide their OpenRouter API key (or confirm they have placed it in `.env`). **Do NOT** launch the container with placeholder (`sk-or-v1-xxxx...`) or empty keys, as this causes authentication failures and exhaustion locks.
   - **Telegram Bot Configuration (Optional)**: Ask the user if they wish to configure Telegram now by adding `TELEGRAM_BOT_TOKEN` and `TELEGRAM_ALLOWED_USERS` to `.env`.
   - **Wait for confirmation**: Only proceed to Step 2 after the user confirms that the required environment variables are set.

---

## 🚀 Step 2: Container Launch & Automated Configuration

Once `.env` has been configured with valid keys:

1. **Launch the Container**:
   ```bash
   docker compose up -d
   ```

2. **Configure OpenRouter Free Tier Model**:
   Automatically run the following commands to configure Hermes:
   ```bash
   docker exec hermes hermes config set model.provider openrouter
   docker exec hermes hermes config set model.default openrouter/free
   docker exec hermes hermes config set terminal.backend local
   docker exec hermes hermes auth reset openrouter
   ```

---

## 🔑 Environment & Credential Guidelines

1. **Reloading `.env` Variables**:
   - Whenever [.env](file:///Users/shubh/Github/Hermes-Docker-Setup/.env) is modified, always reload the container using:
     ```bash
     docker compose up -d
     ```
   - **Never** use `docker compose restart`, which does not re-read `.env`.
   - If an authentication failure occurred prior to key update, clear the exhaustion cache with:
     ```bash
     docker exec hermes hermes auth reset openrouter
     ```

2. **Telegram & Messaging Setup**:
   - Prefer declarative configuration in [.env](file:///Users/shubh/Github/Hermes-Docker-Setup/.env) using `TELEGRAM_BOT_TOKEN` and `TELEGRAM_ALLOWED_USERS`.
   - If the user prefers the interactive wizard (`docker exec -it hermes hermes gateway setup`), remind them that when it prompts `Start manually: hermes gateway`, they must restart the container using:
     ```bash
     docker compose restart
     ```

---

## 💡 Important Rules & Gotchas

- **Terminal Backend**: Keep `terminal.backend` set to `local`. Hermes is already running inside Docker; setting it to Docker causes nested socket errors.
- **Model Selection**: Changing models globally must be done via `docker exec hermes hermes config set model.default <slug>` or via the Web UI at [http://localhost:9119](http://localhost:9119). Typing `/model` inside chat requires an active conversation session.
- **Host Machine Services**: To connect to Ollama or LM Studio running on the host OS, always use `http://host.docker.internal:<port>/v1` rather than `localhost`.

---

&copy; 2026 Hermes Docker Setup. Developed by Shubham Bhavsar.
