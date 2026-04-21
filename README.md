# Self-Hosted AI Stack

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Docker Compose](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![Ollama](https://img.shields.io/badge/Ollama-local%20LLM-000?logo=ollama&logoColor=white)](https://ollama.com)
[![LiteLLM](https://img.shields.io/badge/LiteLLM-proxy-blueviolet)](https://github.com/BerriAI/litellm)

A production-ready Docker Compose stack for running a complete self-hosted AI infrastructure. Designed for privacy-conscious users and GDPR-sensitive environments.

**Ollama + LiteLLM + Open WebUI + PostgreSQL + Prometheus** — everything you need to run local LLMs behind an OpenAI-compatible API, with a ChatGPT-like UI and observability built in.

---

## Stack components

| Service | Purpose |
|---|---|
| **Ollama** | Local LLM inference with NVIDIA GPU acceleration |
| **LiteLLM** | OpenAI-compatible proxy — unified API for local + cloud models |
| **Open WebUI** | ChatGPT-like multi-user interface |
| **PostgreSQL** | LiteLLM backend (usage tracking, keys, model registry) |
| **Prometheus** | Metrics collection for request-level observability |

## Why this stack?

- **Zero API costs** — inference runs locally on your GPU
- **OpenAI-compatible** — drop-in replacement for any app that speaks the OpenAI API
- **Multi-user** — Open WebUI provides accounts, chat history, and access control
- **GDPR-friendly** — by default no data leaves your infrastructure
- **Observable** — Prometheus scrapes LiteLLM out of the box
- **Extensible** — add OpenAI / Anthropic / Gemini / OpenRouter keys only if you want to

---

## Requirements

- Docker Engine + Docker Compose v2
- NVIDIA GPU with CUDA support (tested on RTX 3080 10GB)
- [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) installed and configured
- At least 16GB system RAM recommended

---

## Quick start

### 1. Clone the repository

```bash
git clone https://github.com/bertorico/self-hosted-ai-stack.git
cd self-hosted-ai-stack
```

### 2. Create the `.env` file

```bash
cp .env.example .env
```

Generate strong random values for the secrets:

```bash
echo "LITELLM_MASTER_KEY=sk-$(openssl rand -hex 16)"
echo "LITELLM_SALT_KEY=sk-$(openssl rand -hex 16)"
echo "WEBUI_SECRET_KEY=$(openssl rand -hex 32)"
echo "POSTGRES_PASSWORD=$(openssl rand -hex 16)"
```

Paste the generated values into `.env`. **Remember to keep `DATABASE_URL`'s password in sync with `POSTGRES_PASSWORD`.**

### 3. Create the external Ollama network

```bash
docker network create ollamanet
```

This network is declared as `external` in the compose file so other stacks can join it.

### 4. Start the stack

```bash
docker compose up -d
```

### 5. Pull your first model

```bash
docker exec ollama ollama pull qwen2.5:14b
```

### 6. Access the interfaces

| Interface | URL |
|---|---|
| Open WebUI | http://localhost:3055 |
| LiteLLM API | http://localhost:4000 |
| LiteLLM Admin UI | http://localhost:4000/ui |
| Prometheus | http://localhost:9090 |

On first access to Open WebUI, create an admin account. Signups are disabled by default after that (`ENABLE_SIGNUP=false`).

---

## LiteLLM configuration

Models are declared in [`config.yaml`](./config.yaml). The default set includes a handful of Ollama models; add more or enable cloud providers by uncommenting the relevant blocks.

```yaml
model_list:
  - model_name: qwen2.5-14b
    litellm_params:
      model: ollama_chat/qwen2.5:14b
      api_base: http://ollama:11434
```

Any application that supports the OpenAI API can now use your local models by pointing to `http://localhost:4000/v1` with your `LITELLM_MASTER_KEY`.

---

## Using with n8n

Ollama credential (direct, local-only):

- Base URL: `http://YOUR_SERVER_IP:11434`

OpenAI-compatible credential (LiteLLM, all providers):

- Base URL: `http://YOUR_SERVER_IP:4000/v1`
- API Key: your `LITELLM_MASTER_KEY`

---

## Building LiteLLM from source (optional)

The provided [`Dockerfile`](./Dockerfile) builds LiteLLM from source. By default the compose file uses the prebuilt image `ghcr.io/berriai/litellm:main-stable`. To build locally, uncomment the `build:` block and comment out the `image:` line in `docker-compose.yml`.

---

## Security notes

- PostgreSQL is bound to `127.0.0.1` only (not exposed externally)
- Prometheus is bound to `127.0.0.1` only
- All containers run with `no-new-privileges:true`
- `ENABLE_SIGNUP=false` prevents unauthorized Open WebUI registrations
- Ollama port `11434` is exposed on all interfaces by default — restrict via firewall or change the binding in `docker-compose.yml` if needed
- Never commit `.env` — the included `.gitignore` already excludes it

---

## GPU resource usage (RTX 3080 10GB)

| Model | VRAM usage | Notes |
|---|---|---|
| qwen2.5:14b (Q4) | ~8.5GB | Fits comfortably |
| deepseek-r1:14b (Q4) | ~8.5GB | Fits comfortably |
| qwen3.5:9b (Q4) | ~6GB | Leaves room for other processes |

---

## Troubleshooting

**`could not select device driver "nvidia"`**
Install the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) and run `sudo nvidia-ctk runtime configure --runtime=docker && sudo systemctl restart docker`.

**`network ollamanet declared as external, but could not be found`**
Run `docker network create ollamanet` before `docker compose up`.

**Port already in use (3055 / 4000 / 9090 / 11434)**
Change the host-side port in `docker-compose.yml`, e.g. `"3066:8080"` for Open WebUI.

**`LITELLM_MASTER_KEY is not set` / permission denied on Open WebUI**
Check that `.env` exists and contains all required variables. Re-run `docker compose up -d` to pick up changes.

**LiteLLM reports `database connection failed`**
Make sure `POSTGRES_PASSWORD` in `.env` matches the password inside `DATABASE_URL`.

---

## Contributing

Issues and pull requests are welcome. For significant changes please open an issue first to discuss what you'd like to change.

---

## License

[MIT](./LICENSE)

---

## Author

[@bertorico](https://github.com/bertorico) — self-hosted AI infrastructure, n8n automation, Italian fiscal domain.
