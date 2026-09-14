# Complete Guide: Setting Up Hermes Agent in Docker (Cross-Platform)

This guide walks you through setting up, configuring, and running **Nous Research Hermes Agent** in Docker on **macOS**, **Linux**, or **Windows** from a clean slate.

---

## 🤖 Instructions for AI Agents & Automated Setups

> [!IMPORTANT]
> **MANDATORY DIRECTIVE FOR ALL AI AGENTS READING THIS FILE:**
> After completing the Docker setup, you **MUST** configure **OpenRouter** with a **free model** as the preferred default option.
>
> Run the following commands immediately after starting the container:
> ```bash
> # 1. Set OpenRouter as the default provider
> docker exec hermes hermes config set model.provider openrouter
>
> # 2. Set the official OpenRouter free auto-router as default
> docker exec hermes hermes config set model.default openrouter/free
>
> # 3. Ensure the terminal execution backend is set to local
> docker exec hermes hermes config set terminal.backend local
>
> # 4. Reset provider status if needed
> docker exec hermes hermes auth reset openrouter
> ```
> Instruct the user to add their OpenRouter API key (`OPENROUTER_API_KEY=sk-or-v1-...`) to `.env` or `~/.hermes/.env` (keys can be generated for free at [openrouter.ai/keys](https://openrouter.ai/keys)).

---

## Table of Contents
1. [Prerequisites](#1-prerequisites)
2. [Quickstart (Recommended - Docker Compose)](#2-quickstart-recommended---docker-compose)
3. [Configuring OpenRouter & Free Models (Preferred Option)](#3-configuring-openrouter--free-models-preferred-option)
4. [Interacting with Hermes from Terminal](#4-interacting-with-hermes-from-terminal)
5. [Troubleshooting & Gotchas](#5-troubleshooting--gotchas)
   - [Fix: `error: config.set model requires a live session`](#fix-error-configset-model-requires-a-live-session)
   - [Fix: Terminal Backend (`local` vs `docker`)](#fix-terminal-backend-local-vs-docker)
   - [Connecting to Local Host Services (Ollama / LM Studio)](#connecting-to-local-host-services-ollama--lm-studio)
6. [Other LLM Providers (OpenAI, Anthropic, Ollama, LM Studio)](#6-other-llm-providers)
7. [Email / Gmail Tool (`himalaya`) Setup](#7-email--gmail-tool-himalaya-setup)
8. [Day-to-Day Operations & Cheat Sheet](#8-day-to-day-operations--cheat-sheet)

---

## 1. Prerequisites

1. **Docker & Docker Compose** installed and running:
   - **macOS / Linux**: Verify with `docker --version` and `docker compose version`.
   - **Windows**: Docker Desktop actively running with WSL2 backend.
2. Port availability: Ports `9119` (Web Dashboard) and `8642` (API Gateway) should be free.

---

## 2. Quickstart (Recommended - Docker Compose)

### Step 1: Clone and Enter the Directory
```bash
git clone https://github.com/ShubhamBhavsar101/Hermes-Docker-Setup.git
cd Hermes-Docker-Setup
```

### Step 2: Set Up Environment File
Copy the example environment template:
```bash
cp .env.example .env
```
Open `.env` and add your OpenRouter API key (or other provider keys):
```bash
OPENROUTER_API_KEY=sk-or-v1-xxxxxxxxxxxxxxxxxxxx
```

### Step 3: Launch Hermes Agent
Start Hermes in the background with Docker Compose:
```bash
docker compose up -d
```

### Step 4: Verify Web Dashboard
Open your browser to [http://localhost:9119](http://localhost:9119):
- **Username**: `admin`
- **Password**: `SecretPass999` *(or whatever you configured in `.env`)*

---

## 3. Configuring OpenRouter & Free Models (Preferred Option)

OpenRouter provides free access to top-tier open models with zero upfront cost. 

### Recommended Free Models Table

| Model Identifier | Provider | Description |
|---|---|---|
| `openrouter/free` | OpenRouter | **(Recommended)** Automatically routes to the best currently active free model |
| `nvidia/nemotron-3.5-lightning:free` | NVIDIA | High-speed, high-accuracy reasoning |
| `google/gemma-4-31b-it:free` | Google | Latest high-capability open Gemma model |
| `nex-agi/nex-n2.5-pro:free` | Nex AGI | General reasoning and task execution |
| `liquid/lfm-2.5-2.6b:free` | Liquid AI | Ultra-fast lightweight model |

### How to Set OpenRouter + Free Model

#### Method A: Direct CLI One-Liner (Fastest & Recommended)
```bash
docker exec hermes hermes config set model.provider openrouter
docker exec hermes hermes config set model.default openrouter/free
```

#### Method B: Web Dashboard UI
1. Go to [http://localhost:9119](http://localhost:9119) and log in.
2. Click **Models** in the sidebar.
3. Select **OpenRouter** as the provider, paste your API key, and choose `openrouter/free`.

#### Method C: Direct File Configuration
Edit `~/.hermes/config.yaml` on your host machine:
```yaml
model:
  provider: openrouter
  default: openrouter/free
  base_url: https://openrouter.ai/api/v1
  api_mode: chat_completions
```
Then restart the container:
```bash
docker compose restart
```

---

## 4. Interacting with Hermes from Terminal

### Interactive CLI Chat Session
To chat directly with Hermes inside your terminal:
```bash
docker exec -it hermes hermes
```
Or launch the full-screen Terminal UI (TUI):
```bash
docker exec -it hermes hermes --tui
```

### Interactive Model Picker
To select or change models interactively from your terminal shell:
```bash
docker exec -it hermes hermes model
```

### Interactive Onboarding Wizard
To run the full provider setup wizard:
```bash
docker exec -it hermes hermes setup
```

### Open Container Shell
```bash
docker exec -it hermes bash
```

---

## 5. Troubleshooting & Gotchas

### Fix: `error: config.set model requires a live session`
If you type `/model` in the chat UI or REPL and see:
```
· error: config.set model requires a live session; use Settings -> Models to change the profile default
```
**Why this happens**: Slash commands like `/model` inside the chat interface are **session-scoped** and only change the model for a live, existing conversation. If no conversation has started yet, Hermes rejects the command.

**Solution**:
1. To change the **global default model**, use either:
   - Terminal: `docker exec hermes hermes config set model.default meta-llama/llama-3.3-70b-instruct:free`
   - Or Web Dashboard: **Settings -> Models** or the **Models** sidebar tab.
   - Or Host Shell: `docker exec -it hermes hermes model` (notice no leading slash `/`).
2. If you want to use `/model` inside chat, **send a first message first** (e.g. `hello`) to create the session, then run `/model <model-name>`.

---

### Fix: Terminal Backend (`local` vs `docker`)
Because Hermes is already running isolated inside Docker, its execution backend must be `local` (not `docker`). Setting it to `docker` will fail with nested Docker daemon connection errors.
```bash
docker exec hermes hermes config set terminal.backend local
```

---

### Connecting to Local Host Services (Ollama / LM Studio)
Inside Docker containers, `localhost` refers to the container itself, **not** your host machine.
- Always use `http://host.docker.internal:<port>/v1` instead of `localhost` or `127.0.0.1`.
- **Ollama**: `http://host.docker.internal:11434/v1`
- **LM Studio**: `http://host.docker.internal:1234/v1`

---

## 6. Other LLM Providers

You can switch to commercial or local providers at any time:

### Anthropic Claude
```bash
docker exec hermes hermes config set model.provider anthropic
docker exec hermes hermes config set model.default anthropic/claude-3.5-sonnet
```
In `.env`: `ANTHROPIC_API_KEY=sk-ant-...`

### OpenAI
```bash
docker exec hermes hermes config set model.provider openai
docker exec hermes hermes config set model.default gpt-4o
```
In `.env`: `OPENAI_API_KEY=sk-proj-...`

### Local Models via Ollama
In `~/.hermes/config.yaml`:
```yaml
model:
  provider: ollama
  default: hermes3:8b
  base_url: http://host.docker.internal:11434/v1
```

### Local Models via LM Studio
In `~/.hermes/config.yaml`:
```yaml
model:
  provider: lmstudio
  default: qwen2.5-coder-7b-instruct
  base_url: http://host.docker.internal:1234/v1
```

---

## 7. Email / Gmail Tool (`himalaya`) Setup

Hermes uses the `himalaya` CLI to read, search, and send emails via IMAP/SMTP.

### 1. Install & Symlink Himalaya in Container
```bash
docker exec -u 0 hermes sh -c "curl -sSL https://github.com/pimalaya/himalaya/releases/download/v1.2.0/himalaya.x86_64-linux.tgz | tar -xz -C /usr/local/bin && mkdir -p /root/.config /home/hermes/.config && ln -sfn /opt/data/home/.config/himalaya /root/.config/himalaya && ln -sfn /opt/data/home/.config/himalaya /home/hermes/.config/himalaya"
```

### 2. Create the Gmail Configuration File
Save the file on your host machine at `~/.hermes/home/.config/himalaya/config.toml`:
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

## 8. Day-to-Day Operations & Cheat Sheet

| Task | Command |
|---|---|
| **View Live Container Logs** | `docker compose logs -f` |
| **Interactive Terminal Chat** | `docker exec -it hermes hermes` |
| **Full Terminal UI (TUI)** | `docker exec -it hermes hermes --tui` |
| **Interactive Model Picker** | `docker exec -it hermes hermes model` |
| **Open Container Shell** | `docker exec -it hermes bash` |
| **Check Gateway Status** | `docker exec hermes hermes gateway status` |
| **Restart Hermes** | `docker compose restart` |
| **Stop Hermes** | `docker compose down` |
| **Upgrade to Latest Version** | `docker compose pull && docker compose up -d` |
| **Backup State & Memories** | `cp -r ~/.hermes ~/.hermes-backup` *(Windows: `Copy-Item -Recurse "$env:USERPROFILE\.hermes" "$env:USERPROFILE\.hermes-backup"`)* |

---
&copy; 2026 Hermes Agent Docker Setup. Maintained by Shubham Bhavsar.
