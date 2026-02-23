# openclaw-demo

A Docker Compose deployment configuration for [OpenClaw](https://openclaw.dev) — an AI gateway that proxies requests to LLM providers (Anthropic, etc.) and exposes a Web UI, deployed behind a Traefik reverse proxy with TLS.

## Overview

This repository contains the infrastructure configuration to self-host an OpenClaw gateway instance. It includes:

- **Docker Compose** service definition for the `openclaw-gateway` container
- **Traefik dynamic configuration** for HTTPS routing
- **Environment variable template** for secrets and settings

## Prerequisites

- Docker and Docker Compose installed on the host
- A running [Traefik](https://traefik.io/) instance with:
  - An external Docker network named `traefik-public`
  - A `websecure` entrypoint configured
  - TLS certificates available at `/certs/cert.pem` and `/certs/key.pem`
- An [Anthropic API key](https://console.anthropic.com/)

## Setup

### 1. Configure environment variables

Copy the example environment file and fill in your values:

```bash
cp .env.example .env
```

Edit `.env`:

```env
# Secure access token for the Web UI
OPENCLAW_GATEWAY_TOKEN=your-secure-token-here

# Anthropic API key
ANTHROPIC_API_KEY=sk-ant-...

# System settings
NODE_ENV=production
TZ=GMT+3
```

### 2. Configure Traefik routing

Copy `traefik-dynamic-openclaw.yml` to your Traefik dynamic configuration directory (e.g. `~/traefik/dynamic/openclaw.yml`) and update the `Host` rule to match your domain:

```yaml
rule: "Host(`your-domain.example.com`)"
```

### 3. Start the service

```bash
docker compose up -d
```

The gateway will be available at `https://your-domain.example.com` once Traefik picks up the dynamic configuration.

## Project Structure

```
.
├── .env.example                   # Environment variable template
├── .gitignore                     # Ignores .env, data/, workspace/
├── docker-compose.yml             # OpenClaw gateway service definition
└── traefik-dynamic-openclaw.yml   # Traefik HTTP router and service config
```

Runtime directories (created automatically, git-ignored):

- `data/` — persistent OpenClaw configuration and state
- `workspace/` — OpenClaw workspace files

## Health Check

The container exposes a health endpoint at `http://localhost:18789/health`. Docker checks it every 30 seconds (3 retries, 15s start period).

## Stopping the Service

```bash
docker compose down
```
