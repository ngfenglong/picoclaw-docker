# PicoClaw on Docker — Portable AI Personal Assistant

## Objective

This project provides a simple, portable Docker Compose setup to run [PicoClaw](https://github.com/sipeed/picoclaw) — a lightweight AI agent — on any VPS or local machine with minimal configuration.

PicoClaw sits between your messaging channel (e.g. Telegram) and an LLM backend, handling conversation memory, tool execution, and agent logic. This repo makes it easy to self-host that entire stack in two containers.

Two backend options are supported:

- **Ollama** (default) — runs an open-source model locally, no cloud API needed, works on low-spec hardware
- **OpenAI** — use GPT models via API key for stronger responses, no local model required

```
[Telegram] → [PicoClaw] → [Ollama (local model)]
                       OR → [OpenAI API]
```

---

## Requirements

- Docker and Docker Compose
- A Telegram bot token (from [@BotFather](https://t.me/BotFather))
- Your Telegram numeric user ID (from [@userinfobot](https://t.me/userinfobot))
- For OpenAI: an OpenAI API key

---

## Setup

### 1. Clone and configure

```bash
git clone <this-repo>
cd <this-repo>

cp config/config.example.json config/config.json
```

Edit `config/config.json` and fill in your Telegram credentials:

```json
"telegram": {
  "enabled": true,
  "token": "YOUR_TELEGRAM_BOT_TOKEN",
  "allow_from": ["YOUR_TELEGRAM_USER_ID"]
}
```

### 2. Choose your LLM backend

#### Option A — Ollama (local model, no API key needed)

Update the `model_list` in `config/config.json`:

```json
"model_list": [
  {
    "model_name": "qwen2.5",
    "model": "qwen2.5:0.5b",
    "api_key": "ollama",
    "api_base": "http://ollama:11434/v1"
  }
]
```

Set the default model in `agents.defaults`:

```json
"agents": {
  "defaults": {
    "model_name": "qwen2.5"
  }
}
```

Also update `docker-compose.yml` to pull your chosen model:

```yaml
entrypoint: ["ollama", "pull", "qwen2.5:0.5b"]
```

See the model table below to pick the right model for your hardware.

#### Option B — OpenAI (cloud API, stronger models)

Update `config/config.json`:

```json
"model_list": [
  {
    "model_name": "gpt4o-mini",
    "model": "gpt-4o-mini",
    "api_key": "sk-YOUR_OPENAI_API_KEY",
    "api_base": "https://api.openai.com/v1"
  }
]
```

Set the default model in `agents.defaults`:

```json
"agents": {
  "defaults": {
    "model_name": "gpt4o-mini"
  }
}
```

Then remove the Ollama services from `docker-compose.yml` — only the `picoclaw-gateway` service is needed. Remove `ollama`, `ollama-init`, and the `ollama-models` volume, and remove the `depends_on` block from `picoclaw-gateway`.

> OpenAI models fully support tool calling. `gpt-4o-mini` is a cost-effective starting point. Use `gpt-4o` for best quality.

### 3. Start

```bash
docker compose up -d
```

With Ollama: the model is downloaded automatically on first run. PicoClaw starts after the model is ready.

With OpenAI: PicoClaw starts immediately and calls the API on demand.

### 4. Test

Message your Telegram bot — it will respond using your configured model.

To test Ollama directly while running (Ollama mode only):

```bash
curl http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"qwen2.5:0.5b","messages":[{"role":"user","content":"Hello!"}]}'
```

---

## Choosing a Model (Ollama)

> **Important:** The model must support **tool calling** to work with PicoClaw's agent loop. Models without tool support will return an error.

### Tool-capable models by VPS spec

| VPS RAM | Model | Size on disk | Tool Support | Notes |
|---|---|---|---|---|
| 512 MB | `qwen2.5:0.5b` | ~400 MB | Yes | Minimum viable; add 512MB swap recommended |
| 1 GB | `llama3.2:1b` | ~700 MB | Yes | Good alternative for 1GB |
| 1 GB | `qwen2.5:1.5b` | ~1 GB | Yes | Good balance of size and quality |
| 2 GB | `qwen2.5:3b` | ~2 GB | Yes | Noticeably better reasoning |
| 2 GB | `llama3.2:3b` | ~2 GB | Yes | Strong general-purpose small model |
| 4 GB | `qwen2.5:7b` | ~4.7 GB | Yes | Solid quality for a local model |
| 4 GB | `mistral:7b` | ~4.1 GB | Yes | Good instruction following |
| 8 GB+ | `llama3.1:8b` | ~4.7 GB | Yes | High quality, best for complex tasks |

> All sizes are for default Q4 quantization. Q8 uses ~2x the space and RAM for marginally better quality.

### Models that do NOT work with PicoClaw

| Model | Reason |
|---|---|
| `smollm2:360m-instruct-q8_0` | No tool calling support |
| Most embedding-only models | No chat/tool support |

### Choosing between Ollama and OpenAI

| | Ollama (local) | OpenAI (cloud) |
|---|---|---|
| Cost | Free (electricity only) | Pay per token |
| Privacy | Fully local, no data leaves your server | Data sent to OpenAI |
| Quality (small VPS) | Limited by hardware | GPT-4o quality regardless of VPS spec |
| Setup | Requires choosing a model | Just an API key |
| Works offline | Yes | No |

---

## VPS Preparation (512 MB RAM, Ollama only)

Add swap before starting Docker to prevent OOM kills:

```bash
fallocate -l 512M /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

---

## Stopping

```bash
docker compose down
```

To remove downloaded Ollama model data (frees disk space):

```bash
docker volume rm open-models_ollama-models
```

---

## Project structure

```
.
├── docker-compose.yml          # Ollama + PicoClaw services
├── config/
│   ├── config.example.json     # Template — commit this
│   └── config.json             # Your secrets — gitignored, never commit
└── README.md
```
