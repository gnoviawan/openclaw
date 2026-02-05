---
summary: "Deploy OpenClaw on any VPS without cloning the repository"
read_when:
  - You want to run OpenClaw on a VPS or remote server
  - You want the simplest possible deployment
  - You don't need to modify the source code
---

# VPS Deployment (Standalone)

Deploy OpenClaw on any VPS with Docker, without cloning the repository. Uses pre-built images from GitHub Container Registry.

## Requirements

- VPS with Docker Engine + Docker Compose v2
- At least 1GB RAM (2GB recommended)
- Network access to ports 18789, 18790

## Quick Start

### 1. Download deployment files

```bash
# Create deployment directory
mkdir -p ~/openclaw && cd ~/openclaw

# Download docker-compose and env example
curl -O https://raw.githubusercontent.com/openclaw/openclaw/main/docker-compose.vps.yml
curl -O https://raw.githubusercontent.com/openclaw/openclaw/main/.env.vps.example
```

### 2. Configure environment

```bash
# Copy example config
cp .env.vps.example .env

# Generate secure gateway token
TOKEN=$(openssl rand -hex 32)
sed -i "s/your-secure-token-here/$TOKEN/" .env

# Review and edit settings
nano .env
```

### 3. Run onboarding

```bash
# Pull the image
docker compose -f docker-compose.vps.yml pull

# Run interactive onboarding
docker compose -f docker-compose.vps.yml run --rm openclaw-cli onboard --no-install-daemon
```

During onboarding:
- Gateway bind: `lan`
- Gateway auth: `token`
- Gateway token: (paste from your .env file)
- Tailscale exposure: `Off`
- Install Gateway daemon: `No`

### 4. Start the gateway

```bash
docker compose -f docker-compose.vps.yml up -d openclaw-gateway
```

### 5. Verify it's running

```bash
# Check container status
docker compose -f docker-compose.vps.yml ps

# Check logs
docker compose -f docker-compose.vps.yml logs -f openclaw-gateway

# Health check
docker compose -f docker-compose.vps.yml exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

## Channel Setup

After the gateway is running, configure your messaging channels:

### WhatsApp (QR code)

```bash
docker compose -f docker-compose.vps.yml run --rm openclaw-cli channels login
```

### Telegram (bot token)

```bash
docker compose -f docker-compose.vps.yml run --rm openclaw-cli channels add --channel telegram --token "<your-bot-token>"
```

### Discord (bot token)

```bash
docker compose -f docker-compose.vps.yml run --rm openclaw-cli channels add --channel discord --token "<your-bot-token>"
```

### Slack

```bash
docker compose -f docker-compose.vps.yml run --rm openclaw-cli channels add --channel slack --bot-token "xoxb-..." --app-token "xapp-..."
```

## Environment Variables Reference

### Required

| Variable | Description |
|----------|-------------|
| `OPENCLAW_GATEWAY_TOKEN` | Authentication token for gateway access. Generate with `openssl rand -hex 32` |

### Docker & Gateway

| Variable | Default | Description |
|----------|---------|-------------|
| `OPENCLAW_IMAGE` | `ghcr.io/openclaw/openclaw:main` | Docker image to use |
| `OPENCLAW_GATEWAY_BIND` | `lan` | Network bind: `localhost`, `lan`, or `all` |
| `OPENCLAW_GATEWAY_PORT` | `18789` | Gateway HTTP port |
| `OPENCLAW_BRIDGE_PORT` | `18790` | Bridge WebSocket port |
| `OPENCLAW_GATEWAY_PASSWORD` | - | Password for Tailscale funnel |
| `OPENCLAW_CONFIG_DIR` | `./data/config` | Config directory mount path |
| `OPENCLAW_WORKSPACE_DIR` | `./data/workspace` | Workspace directory mount path |
| `OPENCLAW_PROFILE` | `default` | Profile name for multi-instance |
| `TZ` | `UTC` | Timezone (IANA format) |

### AI Provider API Keys

Configure at least one provider:

| Variable | Provider | Get Key At |
|----------|----------|------------|
| `ANTHROPIC_API_KEY` | Anthropic (Claude) | https://console.anthropic.com/ |
| `OPENAI_API_KEY` | OpenAI | https://platform.openai.com/api-keys |
| `GEMINI_API_KEY` | Google Gemini | https://makersuite.google.com/app/apikey |
| `GROQ_API_KEY` | Groq | https://console.groq.com/ |
| `OPENROUTER_API_KEY` | OpenRouter | https://openrouter.ai/keys |
| `XAI_API_KEY` | xAI (Grok) | - |
| `ZAI_API_KEY` | Z.AI (GLM models) | - |
| `SYNTHETIC_API_KEY` | Synthetic | - |
| `MISTRAL_API_KEY` | Mistral | https://console.mistral.ai/ |
| `CEREBRAS_API_KEY` | Cerebras | - |
| `DEEPGRAM_API_KEY` | Deepgram | https://console.deepgram.com/ |
| `VENICE_API_KEY` | Venice AI | - |
| `MOONSHOT_API_KEY` | Moonshot | - |
| `KIMICODE_API_KEY` | Kimi Code | - |
| `MINIMAX_API_KEY` | MiniMax | - |
| `XIAOMI_API_KEY` | Xiaomi | - |
| `OPENCODE_API_KEY` | OpenCode | - |
| `PERPLEXITY_API_KEY` | Perplexity | https://www.perplexity.ai/settings/api |
| `AI_GATEWAY_API_KEY` | Vercel AI Gateway | - |
| `CHUTES_API_KEY` | Chutes | - |
| `QWEN_PORTAL_API_KEY` | Qwen Portal | - |
| `COPILOT_GITHUB_TOKEN` | GitHub Copilot | - |

### AWS Bedrock

For Amazon Bedrock models, configure AWS credentials:

| Variable | Description |
|----------|-------------|
| `AWS_ACCESS_KEY_ID` | AWS access key |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key |
| `AWS_SESSION_TOKEN` | Session token (for temporary credentials) |
| `AWS_REGION` | AWS region (e.g., `us-east-1`) |
| `AWS_DEFAULT_REGION` | Default AWS region |
| `AWS_PROFILE` | AWS profile name |
| `AWS_BEARER_TOKEN_BEDROCK` | Bedrock bearer token (alternative auth) |

See [Bedrock documentation](/bedrock) for detailed setup.

### Messaging Channels

| Variable | Channel | Description |
|----------|---------|-------------|
| `TELEGRAM_BOT_TOKEN` | Telegram | Bot token from @BotFather |
| `DISCORD_BOT_TOKEN` | Discord | Bot token from Developer Portal |
| `SLACK_BOT_TOKEN` | Slack | Bot token (xoxb-...) |
| `SLACK_APP_TOKEN` | Slack | App token (xapp-...) |
| `TWILIO_ACCOUNT_SID` | WhatsApp/Twilio | Twilio Account SID |
| `TWILIO_AUTH_TOKEN` | WhatsApp/Twilio | Twilio Auth Token |
| `TWILIO_WHATSAPP_FROM` | WhatsApp/Twilio | WhatsApp sender number |

### Tools

| Variable | Description |
|----------|-------------|
| `BRAVE_API_KEY` | Brave Search API key for web_search |
| `FIRECRAWL_API_KEY` | Firecrawl API key for web scraping |
| `ELEVENLABS_API_KEY` | ElevenLabs API key for TTS |
| `XI_API_KEY` | Alternative ElevenLabs key |
| `OPENAI_TTS_BASE_URL` | Custom OpenAI TTS endpoint |

### Debug & Development

| Variable | Default | Description |
|----------|---------|-------------|
| `OPENCLAW_DEBUG_TELEGRAM_ACCOUNTS` | - | Enable Telegram account debug logs |
| `OPENCLAW_ANTHROPIC_PAYLOAD_LOG` | - | Log Anthropic API payloads |
| `OPENCLAW_CACHE_TRACE` | - | Enable cache trace logging |
| `OPENCLAW_SKIP_CANVAS_HOST` | - | Skip canvas host (headless) |
| `FORCE_COLOR` | - | Force terminal colors |
| `NO_COLOR` | - | Disable terminal colors |

## Custom Packages & Binaries

The pre-built image includes Node.js but may not have all system packages you need.
There are three ways to add custom packages:

### Option 1: Build a Custom Image (Recommended)

Best for packages that need to persist across container restarts.

```bash
# Download the custom Dockerfile
curl -O https://raw.githubusercontent.com/openclaw/openclaw/main/Dockerfile.vps.custom

# Edit to add your packages
nano Dockerfile.vps.custom
# Change: ARG OPENCLAW_APT_PACKAGES="ffmpeg imagemagick git"

# Build the image
docker build -f Dockerfile.vps.custom -t openclaw-custom:latest .

# Update .env to use your custom image
echo 'OPENCLAW_IMAGE=openclaw-custom:latest' >> .env

# Start with your custom image
docker compose -f docker-compose.vps.yml up -d openclaw-gateway
```

### Option 2: Runtime Installation

Packages are installed every time the container starts. Good for testing.

```bash
# Set packages in .env
echo 'OPENCLAW_APT_PACKAGES=ffmpeg imagemagick git' >> .env

# Use the custom gateway service
docker compose -f docker-compose.vps.yml --profile custom up -d openclaw-gateway-custom
```

You can also use a custom init script:

```bash
# Create init script
cat > init.sh << 'EOF'
#!/bin/sh
apt-get update
apt-get install -y ffmpeg
# Add more setup commands here
EOF

# Set in .env
echo 'OPENCLAW_INIT_SCRIPT=./init.sh' >> .env

# Start with custom profile
docker compose -f docker-compose.vps.yml --profile custom up -d openclaw-gateway-custom
```

### Option 3: Mount Host Binaries

If binaries are already installed on your VPS, mount them into the container.

Edit `docker-compose.vps.yml` and uncomment/add volume mounts:

```yaml
volumes:
  - /usr/bin/ffmpeg:/usr/bin/ffmpeg:ro
  - /usr/bin/imagemagick:/usr/bin/imagemagick:ro
  - ./custom-bin:/usr/local/custom-bin:ro
```

For custom directories, set PATH prepend in `.env`:

```bash
OPENCLAW_PATH_PREPEND=/usr/local/custom-bin
```

### Common Packages

| Package | Use Case |
|---------|----------|
| `ffmpeg` | Video/audio processing |
| `imagemagick` | Image manipulation |
| `build-essential` | Compilers and build tools |
| `python3 python3-pip` | Python runtime |
| `git` | Version control |
| `jq` | JSON processor |
| `ripgrep` | Fast text search |
| `pandoc` | Document conversion |

## Management Commands

```bash
# View logs
docker compose -f docker-compose.vps.yml logs -f openclaw-gateway

# Restart gateway
docker compose -f docker-compose.vps.yml restart openclaw-gateway

# Stop gateway
docker compose -f docker-compose.vps.yml down

# Update to latest version
docker compose -f docker-compose.vps.yml pull
docker compose -f docker-compose.vps.yml up -d openclaw-gateway

# Run CLI commands
docker compose -f docker-compose.vps.yml run --rm openclaw-cli <command>

# Examples:
docker compose -f docker-compose.vps.yml run --rm openclaw-cli config show
docker compose -f docker-compose.vps.yml run --rm openclaw-cli models list
docker compose -f docker-compose.vps.yml run --rm openclaw-cli status
```

## Using a Specific Version

To use a specific release instead of the latest main branch:

```bash
# Edit .env and change:
OPENCLAW_IMAGE=ghcr.io/openclaw/openclaw:v2026.1.29
```

Available image tags:
- `main` - Latest from main branch (default)
- `vX.Y.Z` - Specific version (e.g., `v2026.1.29`)
- `vX.Y.Z-amd64` - AMD64 architecture only
- `vX.Y.Z-arm64` - ARM64 architecture only

## Data Persistence

Data is stored in local directories (default: `./data/`):
- `./data/config/` - Configuration files
- `./data/workspace/` - Agent workspaces

To backup:
```bash
tar -czvf openclaw-backup.tar.gz ./data/
```

To restore:
```bash
tar -xzvf openclaw-backup.tar.gz
```

## Firewall Configuration

If using UFW:
```bash
sudo ufw allow 18789/tcp  # Gateway
sudo ufw allow 18790/tcp  # Bridge
```

If using firewalld:
```bash
sudo firewall-cmd --permanent --add-port=18789/tcp
sudo firewall-cmd --permanent --add-port=18790/tcp
sudo firewall-cmd --reload
```

## Reverse Proxy (optional)

### Nginx

```nginx
server {
    listen 443 ssl http2;
    server_name openclaw.yourdomain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 86400;
    }
}
```

### Caddy

```caddyfile
openclaw.yourdomain.com {
    reverse_proxy localhost:18789
}
```

### Traefik (Docker labels)

Add these labels to the `openclaw-gateway` service:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.openclaw.rule=Host(`openclaw.yourdomain.com`)"
  - "traefik.http.routers.openclaw.tls=true"
  - "traefik.http.routers.openclaw.tls.certresolver=letsencrypt"
  - "traefik.http.services.openclaw.loadbalancer.server.port=18789"
```

## Multi-Instance Deployment

To run multiple OpenClaw instances on the same server:

```bash
# Instance 1 (default)
mkdir -p ~/openclaw-main && cd ~/openclaw-main
# Download files and configure with OPENCLAW_PROFILE=default

# Instance 2 (work)
mkdir -p ~/openclaw-work && cd ~/openclaw-work
# Download files and configure:
# - OPENCLAW_PROFILE=work
# - OPENCLAW_GATEWAY_PORT=18791
# - OPENCLAW_BRIDGE_PORT=18792
# - Different OPENCLAW_CONFIG_DIR and OPENCLAW_WORKSPACE_DIR
```

## Troubleshooting

### Permission errors
```bash
# Fix data directory ownership
sudo chown -R 1000:1000 ./data/
```

### Image not found
```bash
# Login to GitHub Container Registry (public, no auth needed for read)
docker pull ghcr.io/openclaw/openclaw:main
```

### Gateway not accessible
- Check firewall rules
- Verify `OPENCLAW_GATEWAY_BIND=lan`
- Check container logs for errors

### Container keeps restarting
```bash
# Check logs for errors
docker compose -f docker-compose.vps.yml logs --tail=100 openclaw-gateway

# Common issues:
# - Missing OPENCLAW_GATEWAY_TOKEN
# - Invalid API keys
# - Port already in use
```

### Memory issues
```bash
# Add memory limits to the service:
# In docker-compose.vps.yml, add under openclaw-gateway:
deploy:
  resources:
    limits:
      memory: 2G
```

### Network connectivity
```bash
# Test from inside the container
docker compose -f docker-compose.vps.yml exec openclaw-gateway curl -v http://localhost:18789/health
```
