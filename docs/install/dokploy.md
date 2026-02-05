---
summary: "Deploy OpenClaw on Dokploy (self-hosted PaaS)"
read_when:
  - You want to deploy OpenClaw using Dokploy
  - You want a GUI-based deployment solution
  - You're using Dokploy as your self-hosted PaaS
---

# Dokploy Deployment

Deploy OpenClaw on [Dokploy](https://dokploy.com), a self-hosted Platform as a Service (PaaS) alternative to Vercel, Netlify, and Heroku.

## Prerequisites

- Dokploy installed on your server ([Installation Guide](https://docs.dokploy.com/docs/get-started/installation))
- At least 2GB RAM (4GB recommended)
- A domain pointed to your Dokploy server (optional but recommended)

## Quick Start

### 1. Create a Docker Compose Application

1. Log in to your Dokploy dashboard
2. Click **Create Application** → **Docker Compose**
3. Give it a name (e.g., `openclaw`)

### 2. Configure the Compose File

In the **General** tab, paste the following or use the Git source option:

**Option A: Paste directly**

Copy the contents of `docker-compose.dokploy.yml` from the OpenClaw repository.

**Option B: Git source**

- Repository: `https://github.com/openclaw/openclaw`
- Branch: `main`
- Compose Path: `docker-compose.dokploy.yml`

### 3. Configure Environment Variables

Go to the **Environment** tab and add your variables:

```env
# REQUIRED
OPENCLAW_GATEWAY_TOKEN=<generate-with-openssl-rand-hex-32>

# AI Provider (at least one required)
ANTHROPIC_API_KEY=sk-ant-...

# Optional: Messaging channels
TELEGRAM_BOT_TOKEN=123456:ABC...
DISCORD_BOT_TOKEN=...
```

Generate a secure gateway token:
```bash
openssl rand -hex 32
```

### 4. Configure Domain

Go to the **Domains** tab:

1. Click **Add Domain**
2. Select service: `openclaw-gateway`
3. Enter your domain: `openclaw.yourdomain.com`
4. Port: `18789`
5. Enable HTTPS (recommended)
6. Save

Dokploy will automatically configure Traefik routing and SSL.

### 5. Deploy

Click **Deploy** in the **General** tab.

Monitor the deployment in the **Deployments** tab.

## Environment Variables

### Required

| Variable | Description |
|----------|-------------|
| `OPENCLAW_GATEWAY_TOKEN` | Gateway authentication token |

### AI Providers (configure at least one)

| Variable | Provider |
|----------|----------|
| `ANTHROPIC_API_KEY` | Anthropic Claude |
| `OPENAI_API_KEY` | OpenAI |
| `GEMINI_API_KEY` | Google Gemini |
| `GROQ_API_KEY` | Groq |
| `OPENROUTER_API_KEY` | OpenRouter |
| `ZAI_API_KEY` | Z.AI (GLM models) |
| `SYNTHETIC_API_KEY` | Synthetic |
| `XAI_API_KEY` | xAI (Grok) |
| `MISTRAL_API_KEY` | Mistral |
| `VENICE_API_KEY` | Venice AI |
| `MOONSHOT_API_KEY` | Moonshot |
| `MINIMAX_API_KEY` | MiniMax |
| `OPENCODE_API_KEY` | OpenCode |

### Messaging Channels

| Variable | Channel |
|----------|---------|
| `TELEGRAM_BOT_TOKEN` | Telegram |
| `DISCORD_BOT_TOKEN` | Discord |
| `SLACK_BOT_TOKEN` | Slack |
| `SLACK_APP_TOKEN` | Slack |

### Tools

| Variable | Description |
|----------|-------------|
| `BRAVE_API_KEY` | Brave Search API |
| `ELEVENLABS_API_KEY` | ElevenLabs TTS |

See `.env.vps.example` for the complete list of environment variables.

## Channel Setup

After deployment, use Dokploy's terminal feature or SSH to run channel commands:

### Using Dokploy Terminal

1. Go to **Advanced** → **Terminal**
2. Select container: `openclaw-gateway`
3. Run commands:

```bash
# WhatsApp (scan QR in terminal)
node dist/index.js channels login

# Telegram
node dist/index.js channels add --channel telegram --token "<token>"

# Discord
node dist/index.js channels add --channel discord --token "<token>"
```

### Using Docker Exec

SSH to your server and run:

```bash
docker exec -it openclaw-gateway node dist/index.js channels login
```

## Data Persistence

The compose file uses Docker named volumes for data persistence:

- `openclaw-config` - Configuration and credentials
- `openclaw-workspace` - Agent workspaces

### Volume Backups

Named volumes are compatible with Dokploy's **Volume Backups** feature:

1. Go to **Volume Backups** tab
2. Configure S3 destination
3. Schedule automatic backups

### Using Bind Mounts Instead

If you prefer bind mounts (direct file access), edit the compose file:

```yaml
volumes:
  # Comment out named volumes
  # - openclaw-config:/home/node/.openclaw
  # - openclaw-workspace:/home/node/.openclaw/workspace

  # Use bind mounts with ../files
  - ../files/config:/home/node/.openclaw
  - ../files/workspace:/home/node/.openclaw/workspace
```

Note: Bind mounts don't work with Dokploy Volume Backups.

## Custom Packages

If you need additional system packages (ffmpeg, etc.):

### Option 1: Custom Image

1. Create a `Dockerfile.custom` in your project:

```dockerfile
FROM ghcr.io/openclaw/openclaw:main
USER root
RUN apt-get update && apt-get install -y ffmpeg && rm -rf /var/lib/apt/lists/*
USER node
```

2. Update the compose file:

```yaml
services:
  openclaw-gateway:
    build:
      context: .
      dockerfile: Dockerfile.custom
    # Remove or comment out the image line
    # image: ghcr.io/openclaw/openclaw:main
```

3. Deploy - Dokploy will build the custom image

### Option 2: Use Pre-built Custom Image

Build and push to a registry, then update the compose:

```yaml
services:
  openclaw-gateway:
    image: your-registry.com/openclaw-custom:latest
```

## Health Monitoring

The compose file includes a health check. Monitor in the **Monitoring** tab:

- Container status
- CPU/Memory usage
- Health check results

## Logs

View logs in the **Logs** tab:

- Select service: `openclaw-gateway`
- Enable auto-refresh for live logs

## Manual Traefik Configuration

If you prefer manual Traefik labels instead of Dokploy's Domains tab:

```yaml
services:
  openclaw-gateway:
    # ... other config ...
    labels:
      - traefik.enable=true
      - traefik.http.routers.openclaw.rule=Host(`openclaw.yourdomain.com`)
      - traefik.http.routers.openclaw.entrypoints=websecure
      - traefik.http.routers.openclaw.tls=true
      - traefik.http.routers.openclaw.tls.certresolver=letsencrypt
      - traefik.http.services.openclaw.loadbalancer.server.port=18789
```

## WebSocket Support

For WebSocket connections (bridge port 18790), add a second domain:

1. **Domains** tab → **Add Domain**
2. Service: `openclaw-gateway`
3. Domain: `ws.openclaw.yourdomain.com`
4. Port: `18790`

Or with manual labels:

```yaml
labels:
  # ... gateway labels ...
  - traefik.http.routers.openclaw-ws.rule=Host(`ws.openclaw.yourdomain.com`)
  - traefik.http.routers.openclaw-ws.entrypoints=websecure
  - traefik.http.routers.openclaw-ws.tls=true
  - traefik.http.services.openclaw-ws.loadbalancer.server.port=18790
```

## Troubleshooting

### Container won't start

Check logs in **Logs** tab for errors. Common issues:
- Missing `OPENCLAW_GATEWAY_TOKEN`
- Invalid API keys

### Domain not working

1. Verify DNS A record points to your server
2. Check **Domains** tab for correct port (18789)
3. Wait for SSL certificate provisioning

### Health check failing

The health check hits `/health` endpoint. If failing:
- Check container logs for startup errors
- Verify port 18789 is correct

### Permission errors

If you see permission errors with volumes:
1. SSH to server
2. Fix ownership:
```bash
docker exec -u root openclaw-gateway chown -R node:node /home/node/.openclaw
```

## Updating

To update to the latest version:

1. Go to **General** tab
2. Click **Pull** to get the latest image
3. Click **Deploy**

Or enable **Auto Deploy** with webhooks for automatic updates on push.

## Resources

- [Dokploy Documentation](https://docs.dokploy.com)
- [OpenClaw Documentation](https://github.com/openclaw/openclaw)
- [Dokploy Discord](https://discord.gg/dokploy)
