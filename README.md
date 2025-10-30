# n8n-Ollama-WebUI Stack

This repository provides a complete self-hosted workflow automation and AI environment featuring:

* **n8n** – visual workflow automation
* **Ollama** – local LLM model server
* **Open-WebUI** – browser UI for chatting with Ollama models
* **PostgreSQL** – persistent database backend for n8n
* **Nginx** – TLS-terminating reverse proxy for all services

All services run on **Ubuntu** using Docker Compose, with optional GPU support for Ollama.

---

## Directory layout

```
.
├── docker-compose.yml        # All services: n8n, postgres, ollama, open-webui, nginx
├── nginx/
│   └── default.conf          # Nginx reverse proxy config
├── certs/
│   ├── fullchain.pem         # SSL certificate
│   └── privkey.pem           # SSL private key
├── .env                      # Environment variables
├── db_data/                  # Postgres volume
├── n8n_data/                 # n8n persistent data
├── webui_data/               # Open-WebUI data
└── ollama/                   # Ollama model and config cache
```

---

## Prerequisites

* Ubuntu 22.04+ host
* Docker Engine + Docker Compose plugin
* IPv4 connectivity on your public interface (IPv6 may be used only for management)
* Valid TLS certificate and key under `certs/` (or self-signed for testing)

---

## Quick start

### 1. Clone this repository

```bash
git clone https://github.com/<yourname>/n8n-ollama-stack.git
cd n8n-mcp-agents
```

### 2. Prepare environment file

Copy `.env.template` → `.env` and edit:

```bash
POSTGRES_USER=n8n
POSTGRES_PASSWORD=n8n
POSTGRES_DB=n8n

# Random secret for n8n encryption
N8N_ENCRYPTION_KEY=$(openssl rand -hex 32)

# Timezone and hostname
TZ=America/New_York
N8N_HOST=localhost
WEBHOOK_URL=https://<your-public-hostname>:8443/
MODEL_NAME=llama3
```

### 3. Launch the stack

```bash
docker compose up -d
```

The first start will pull images and initialize Postgres and Ollama.

---

## Service endpoints

| Service             | Port  | Description / URL                                                                      |
| ------------------- | ----- |----------------------------------------------------------------------------------------|
| Nginx reverse proxy | 443   | Entry point for all HTTPS traffic                                                      |
| n8n                 | 5678  | Exposed via Nginx at `/` → https://<host>/  or directly http://<host>:5678/            |
| Open-WebUI          | 8080  | Exposed via Nginx at `/webui/` → https://<host>/webui/ or directly http://<host>:8080/ |
| Ollama API          | 11434 | Exposed via Nginx at `/ollama/` → https://<host>/ollama/                               |
| PostgreSQL          | 5432  | Local only, used by n8n                                                                |

> Note: Nginx proxies internally to `127.0.0.1` (IPv4). Ensure your host networking prefers IPv4 (see below).

---

## GPU support (optional)

If your host has an NVIDIA GPU and drivers installed:

```bash
docker compose up -d ollama
```

The `ollama` service already includes:

```yaml
deploy:
  resources:
    reservations:
      devices:
        - capabilities: [gpu]
```

so Docker will pass GPU devices automatically if available.

---

## Logs & troubleshooting

```bash
docker compose logs -f nginx
docker compose logs -f n8n
docker compose logs -f open-webui
docker compose logs -f ollama
```

To test upstream reachability:

```bash
curl -vk https://<your-host>/ollama/api/tags
curl -vk https://<your-host>/
curl -vk https://<your-host>/webui/
```

If `502 Bad Gateway` appears, verify that:

* Nginx `proxy_pass` uses `127.0.0.1` (not `localhost`)
* Services are listening on `0.0.0.0` or `127.0.0.1` ports
* IPv4 preference (`/etc/gai.conf`) is applied

---

## Shut down and cleanup

```bash
docker compose down
```

To remove volumes (wipe Postgres/n8n data):

```bash
docker compose down -v
```

---

## License

MIT License © 2025 Komal Thareja
