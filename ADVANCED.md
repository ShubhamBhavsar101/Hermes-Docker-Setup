# Hermes Agent: Advanced Configuration

This document covers optional and advanced configurations for power users.

---

## 1. Other LLM Providers (OpenAI, Anthropic, DeepSeek, Local)

You can switch away from OpenRouter at any time:

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

### Local Models via Ollama / LM Studio
Inside Docker, connect to host machine services using `http://host.docker.internal:<port>/v1`:
```bash
# Ollama
docker exec hermes hermes config set model.provider ollama
docker exec hermes hermes config set model.base_url http://host.docker.internal:11434/v1
docker exec hermes hermes config set model.default hermes3:8b

# LM Studio
docker exec hermes hermes config set model.provider lmstudio
docker exec hermes hermes config set model.base_url http://host.docker.internal:1234/v1
docker exec hermes hermes config set model.default qwen2.5-coder-7b-instruct
```

---

## 2. Email / Gmail Tool (`himalaya`) Setup

Hermes uses the `himalaya` CLI to read, search, and send emails via IMAP/SMTP.

### Step 1: Install & Symlink Himalaya in the Container
```bash
docker exec -u 0 hermes sh -c "curl -sSL https://github.com/pimalaya/himalaya/releases/download/v1.2.0/himalaya.x86_64-linux.tgz | tar -xz -C /usr/local/bin && mkdir -p /root/.config /home/hermes/.config && ln -sfn /opt/data/home/.config/himalaya /root/.config/himalaya && ln -sfn /opt/data/home/.config/himalaya /home/hermes/.config/himalaya"
```

### Step 2: Create the Gmail Configuration File
Create `~/.hermes/home/.config/himalaya/config.toml` (Windows: `$env:USERPROFILE\.hermes\home\.config\himalaya\config.toml`):
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

## 3. Backing Up & Migrating State
All memories, chat history, and configuration files live in `~/.hermes` (mounted to `/opt/data` in the container).

```bash
# macOS / Linux
cp -r ~/.hermes ~/.hermes-backup

# Windows PowerShell
Copy-Item -Recurse "$env:USERPROFILE\.hermes" "$env:USERPROFILE\.hermes-backup"
```
