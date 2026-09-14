# Complete Guide: Setting Up Hermes Agent in Docker with PowerShell

This guide walks you through setting up, configuring, and managing **Nous Research Hermes Agent** in Docker from a completely clean slate on Windows using **PowerShell**.

---

## Table of Contents
1. [Prerequisites](#1-prerequisites)
2. [Clean Slate Preparation](#2-clean-slate-preparation)
3. [Step 1: One-Time Initialization & Setup](#3-step-1-one-time-initialization--setup)
4. [Step 2: Running Hermes Agent](#4-step-2-running-hermes-agent)
   - [Mode A: Interactive CLI Chat](#mode-a-interactive-cli-chat)
   - [Mode B: Background Gateway + Web Dashboard (Recommended)](#mode-b-background-gateway--web-dashboard-recommended)
   - [Mode C: Docker Compose (Production / Cleanest)](#mode-c-docker-compose-production--cleanest)
5. [Step 3: Configuring Models & LLM Providers](#5-step-3-configuring-models--llm-providers)
   - [Interactive CLI Picker](#method-1-interactive-cli-picker)
   - [Web Dashboard UI](#method-2-web-dashboard-ui)
   - [Direct File Configuration (.env & config.yaml)](#method-3-direct-file-configuration)
   - [Local Models via Ollama](#local-models-via-ollama-on-windows-host)
   - [Local Models via LM Studio](#local-models-via-lm-studio-on-windows-host)
6. [Windows & Docker Desktop Notes](#6-windows--docker-desktop-notes)
7. [Day-to-Day Operations & Cheat Sheet](#7-day-to-day-operations--cheat-sheet)

---

## 1. Prerequisites

1. **Docker Desktop** installed and actively running on Windows.
   - Verify in PowerShell:
     ```powershell
     docker --version
     docker info
     ```
2. **Windows PowerShell** or **PowerShell 7+**.

---

## 2. Clean Slate Preparation

If you have any older or failed containers, stop and remove them:

```powershell
# Stop and remove any existing hermes container
docker rm -f hermes
```

Create the host directory where all persistent agent data (API keys, memories, configs, and skills) will live:

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.hermes"
```

> [!NOTE]
> The host directory (`$env:USERPROFILE\.hermes` $\rightarrow$ `C:\Users\<username>\.hermes`) maps directly to `/opt/data` inside the Docker container. This ensures your agent's memory, settings, and skills survive container restarts or image upgrades.

---

## 3. Step 1: One-Time Initialization & Setup

Run the interactive setup wizard to configure your initial model provider and API keys:

```powershell
docker run -it --rm `
  -v ${env:USERPROFILE}\.hermes:/opt/data `
  nousresearch/hermes-agent setup
```

Follow the interactive prompts to select your provider (OpenRouter, Anthropic, OpenAI, LM Studio, Ollama, etc.) and enter your API keys. When finished, your credentials are saved in `$env:USERPROFILE\.hermes\.env` and `config.yaml`.

---

## 4. Step 2: Running Hermes Agent

Choose one of three deployment modes in PowerShell:

---

### Mode A: Interactive CLI Chat
Run this when you want to chat directly with Hermes inside your PowerShell window:

```powershell
docker run -it --rm `
  -v ${env:USERPROFILE}\.hermes:/opt/data `
  nousresearch/hermes-agent
```

---

### Mode B: Background Gateway + Web Dashboard (Recommended)
Runs Hermes persistently as a background service. It hosts the built-in Web Dashboard and manages messaging connections (Discord, Telegram, Slack, etc.):

```powershell
docker run -d `
  --name hermes `
  --restart unless-stopped `
  -v ${env:USERPROFILE}\.hermes:/opt/data `
  -p 8642:8642 `
  -p 9119:9119 `
  -e HERMES_DASHBOARD=1 `
  -e HERMES_DASHBOARD_BASIC_AUTH_USERNAME=admin `
  -e HERMES_DASHBOARD_BASIC_AUTH_PASSWORD=SecretPass999 `
  nousresearch/hermes-agent gateway run
```

#### Port Reference & Dashboard Login:
- **Web Dashboard**: Open `http://localhost:9119` in your browser.
  - **Username**: `admin`
  - **Password**: `SecretPass999`
- **API Server**: Port `8642` (OpenAI-compatible endpoints).

---

### Mode C: Docker Compose (Production / Cleanest)

Create a file named `docker-compose.yml` in your project folder:

```yaml
services:
  hermes:
    image: nousresearch/hermes-agent:latest
    container_name: hermes
    restart: unless-stopped
    ports:
      - "8642:8642"   # Gateway / API server
      - "9119:9119"   # Built-in Web Dashboard
    volumes:
      - ${USERPROFILE}/.hermes:/opt/data
    environment:
      - HERMES_HOME=/opt/data
      - HERMES_DASHBOARD=1
      # Web Dashboard basic authentication credentials
      - HERMES_DASHBOARD_BASIC_AUTH_USERNAME=admin
      - HERMES_DASHBOARD_BASIC_AUTH_PASSWORD=SecretPass999
      # Optional: Uncomment to expose the OpenAI-compatible API server externally
      # - API_SERVER_ENABLED=true
      # - API_SERVER_HOST=0.0.0.0
      # - API_SERVER_KEY=replace_with_secure_random_token_min_8_chars
    command: gateway run
```

#### Starting & Managing with Compose in PowerShell:
```powershell
# Start container in background
docker compose up -d

# Check live logs
docker compose logs -f

# Stop container
docker compose down
```

---

## 5. Step 3: Configuring Models & LLM Providers

You can switch models or add new providers at any time using any of these methods:

### Method 1: Interactive CLI Picker
While the container is running, execute in PowerShell:
```powershell
docker exec -it hermes hermes model
```
This opens a menu in your terminal where you can select or switch your provider and model.

### Method 2: Web Dashboard UI
1. Navigate to `http://localhost:9119` in your browser and log in.
2. Click on **Models** in the sidebar.
3. Select your **Main Model** (for primary tasks and reasoning) and **Auxiliary Models** (for summarization, vision, or context compression).

### Method 3: Direct File Configuration
Because your `.hermes` folder is on your Windows machine at `C:\Users\<username>\.hermes`, you can open and edit the configuration directly.

1. **Add API keys in `.env`** (`$env:USERPROFILE\.hermes\.env`):
   ```bash
   OPENROUTER_API_KEY=sk-or-v1-xxxxxxxx
   ANTHROPIC_API_KEY=sk-ant-xxxxxxxx
   OPENAI_API_KEY=sk-proj-xxxxxxxx
   DEEPSEEK_API_KEY=sk-xxxxxxxx
   GROQ_API_KEY=gsk_xxxxxxxx
   ```

2. **Configure the Model in `config.yaml`** (`$env:USERPROFILE\.hermes\config.yaml`):
   ```yaml
   model:
     provider: openrouter   # or anthropic, openai, deepseek, groq
     default: anthropic/claude-3.5-sonnet
   ```

3. **Restart the container in PowerShell to apply changes**:
   ```powershell
   docker restart hermes
   ```

### Local Models via Ollama (on Windows Host)
If you run Ollama locally on Windows, the Docker container cannot access `localhost:11434`. Use `host.docker.internal` instead:

In `config.yaml`:
```yaml
model:
  provider: ollama
  default: hermes3:8b
  base_url: http://host.docker.internal:11434/v1
```

### Local Models via LM Studio (on Windows Host)
If you run LM Studio locally on Windows:
1. In LM Studio, start the local server (default port `1234`).
2. Make sure your model is loaded in memory.
3. Because Hermes is in Docker, it must use `http://host.docker.internal:1234/v1` (NOT `127.0.0.1` or `localhost`).

In `config.yaml`:
```yaml
model:
  provider: lmstudio
  default: qwen2.5-coder-7b-instruct   # or your loaded model identifier
  base_url: http://host.docker.internal:1234/v1
  # context_length: 100000            # optional: Hermes auto-detects this dynamically from LM Studio
```
Or set it directly via CLI in PowerShell:
```powershell
docker exec -it hermes hermes config set model.base_url http://host.docker.internal:1234/v1
docker restart hermes
```

---

## 6. Windows & Docker Desktop Notes

### 💡 Path Syntax in PowerShell
Always use the PowerShell environment variable `${env:USERPROFILE}` to refer to your user profile directory:
```powershell
-v ${env:USERPROFILE}\.hermes:/opt/data
```
This guarantees that Docker receives clean Windows paths without shell path corruption.

### 💡 Connecting to Local Host Services (`host.docker.internal`)
Inside a Docker container, `localhost` or `127.0.0.1` refers to the container itself, **not** your Windows machine.
- If you are running local servers on Windows (such as **LM Studio** on port `1234` or **Ollama** on port `11434`), you **must** use `http://host.docker.internal:<port>/v1`.
- Using `127.0.0.1` or `localhost` from within the container will result in connection refused / network failure.

### 💡 Terminal Backend: `local` vs `docker`
Because Hermes is already running isolated inside a Docker container, you should ensure its terminal execution backend is set to `local` (not `docker`).
- If set to `docker`, Hermes attempts to run nested Docker containers via `/var/run/docker.sock`, which fails with a Docker daemon connection error.
- Setting it to `local` runs file operations and terminal commands safely directly inside the Hermes container:
  ```powershell
  docker exec -it hermes hermes config set terminal.backend local
  docker restart hermes
  ```

### 💡 Web Dashboard Auth Requirement
Hermes Agent enforces mandatory authentication whenever the dashboard binds to `0.0.0.0` (which is required for Docker port forwarding). Always provide both `HERMES_DASHBOARD_BASIC_AUTH_USERNAME` and `HERMES_DASHBOARD_BASIC_AUTH_PASSWORD` when starting the container.

### 💡 Email / Gmail Tool (`himalaya`) Setup & Path Mapping
Hermes uses the `himalaya` CLI to read, search, and send emails via IMAP/SMTP.

#### How Path Mapping Works
Because the Docker container mounts `${env:USERPROFILE}\.hermes:/opt/data`, Hermes sets `HOME=/opt/data/home` for all skill subprocesses.
* **Host (Windows) Path**: `C:\Users\<username>\.hermes\home\.config\himalaya\config.toml`
* **Container Path**: `/opt/data/home/.config/himalaya/config.toml`

Any tool looking in `~/.config/himalaya/config.toml` automatically reads directly from your Windows host folder.

#### 1. Install & Symlink Himalaya in the Container
Run this one-liner in PowerShell (installs Himalaya v1.2.0 and links paths for both `root` and `hermes` users):
```powershell
docker exec -u 0 hermes sh -c "curl -sSL https://github.com/pimalaya/himalaya/releases/download/v1.2.0/himalaya.x86_64-linux.tgz | tar -xz -C /usr/local/bin && mkdir -p /root/.config /home/hermes/.config && ln -sfn /opt/data/home/.config/himalaya /root/.config/himalaya && ln -sfn /opt/data/home/.config/himalaya /home/hermes/.config/himalaya"
```

#### 2. Create the Gmail Configuration File
Save the file at `$env:USERPROFILE\.hermes\home\.config\himalaya\config.toml`:
```toml
[accounts.gmail]
email = "your_email@gmail.com"
display-name = "Your Name"
default = true

backend.type = "imap"
backend.host = "imap.gmail.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "your_email@gmail.com"
backend.auth.type = "password"
backend.auth.raw = "your-16-char-google-app-password"

message.send.backend.type = "smtp"
message.send.backend.host = "smtp.gmail.com"
message.send.backend.port = 587
message.send.backend.encryption.type = "start-tls"
message.send.backend.login = "your_email@gmail.com"
message.send.backend.auth.type = "password"
message.send.backend.auth.raw = "your-16-char-google-app-password"

folder.aliases.inbox = "INBOX"
folder.aliases.sent = "[Gmail]/Sent Mail"
folder.aliases.drafts = "[Gmail]/Drafts"
folder.aliases.trash = "[Gmail]/Trash"
```

---

### 💡 Copying / Migrating `.hermes` to Another User or Machine
Because all persistent state, sessions, memories, skills, and configuration files live exclusively inside the `.hermes` directory, you can easily copy it to set up another user:

1. **Copy the `.hermes` folder**:
   ```powershell
   # Copy from your profile to another user's profile
   Copy-Item -Recurse "$env:USERPROFILE\.hermes" "C:\Users\<other-username>\.hermes"
   ```
2. **Update User-Specific Settings**:
   - In `C:\Users\<other-username>\.hermes\home\.config\himalaya\config.toml`: Update the `email` and `auth.raw` App Password to the other user's credentials.
   - In `C:\Users\<other-username>\.hermes\.env` or `config.yaml`: Update any user-specific keys (Telegram Bot token, chat IDs, etc.) if they should use separate accounts.
3. **Launch the Container for the New User**:
   Run the `docker run` command using their profile path.
4. **Install Himalaya in their Container**:
   Run the one-liner from step 1 above once to install the `himalaya` binary inside the new user's container.

---

## 7. Day-to-Day Operations & Cheat Sheet

Run these commands directly in **PowerShell**:

| Task | PowerShell Command |
|------|--------------------|
| **View Live Container Logs** | `docker logs -f hermes` |
| **Launch Hermes CLI inside Container** | `docker exec -it hermes hermes` |
| **Open a Shell inside Container** | `docker exec -it hermes bash` |
| **Check Gateway Status** | `docker exec hermes hermes gateway status` |
| **Restart Gateway** | `docker restart hermes` |
| **Upgrade to Latest Version** | `docker pull nousresearch/hermes-agent:latest; docker rm -f hermes` then re-run the `docker run` command |
| **Backup Configuration & State** | `Copy-Item -Recurse "$env:USERPROFILE\.hermes" "$env:USERPROFILE\.hermes-backup"` |
