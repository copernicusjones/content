+++
tags = ['openclaw', 'daytona', 'ai-development', 'getting-started', 'tutorial']
---

# OpenClaw with Daytona: Up and Running in 2 Minutes

## Introduction

OpenClaw and Daytona are a natural pairing. OpenClaw gives you a self-hosted AI agent that can automate your workflows. Daytona gives you instant, reproducible development environments. Together, you can go from zero to a fully configured agent environment in under two minutes — no manual dependency installation, no "works on my machine" drama.

This guide walks through the fastest path to a running OpenClaw + Daytona setup.

## Prerequisites

- A machine running macOS, Linux, or Windows (via WSL2)
- Docker installed and running
- A Git provider account (GitHub, GitLab, or Bitbucket)
- ~2 GB of free disk space

## Step 1: Install Daytona

**macOS/Linux:**
```bash
curl -fsSL https://get.daytona.io | bash
```

**Verify:**
```bash
daytona version
```

**Start the server:**
```bash
daytona serve
```

The Daytona server starts on `localhost:10001` by default.

## Step 2: Create Your OpenClaw Workspace

Create a `.devcontainer/devcontainer.json` in your project:

```json
{
  "name": "OpenClaw Workspace",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {},
    "ghcr.io/devcontainers/features/node:1": {}
  },
  "forwardPorts": [3000, 10001],
  "postCreateCommand": "npm install -g @anthropic/anthropic-ai",
  "customizations": {
    "vscode": {
      "extensions": ["anthropic.claude-code"]
    }
  }
}
```

This gives you:
- Ubuntu base with Docker-in-Docker support
- Node.js for running OpenClaw
- Claude Code VS Code extension pre-installed
- Port forwarding for the OpenClaw gateway (3000) and Daytona server (10001)

## Step 3: Spin Up the Environment

```bash
daytona create .
```

Daytona reads the devcontainer config, pulls the image, installs features, and runs the post-create command. Typical spin-up time: **30–90 seconds**.

## Step 4: Connect Your IDE

**VS Code:**
1. Open VS Code
2. Install the Daytona extension (search "Daytona" in Extensions)
3. Click the Daytona icon in the sidebar
4. Select your environment and click "Connect"

**Terminal:**
```bash
daytona ssh <env-name>
```

## Step 5: Initialize OpenClaw

Inside the Daytona environment:

```bash
# Clone the OpenClaw repo
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Run the onboard wizard
openclaw onboard
```

The onboard wizard walks through:
- Gateway configuration
- Provider/model routing (choose your LLM provider)
- Channel setup (Telegram, Discord, iMessage, etc.)
- Skill installation
- Workspace structure

## Step 6: Verify the Gateway

```bash
openclaw gateway status
```

You should see:
```
Gateway: running
Port: 3000
Channels: configured
Skills: loaded
```

## What You Have Now

- **OpenClaw gateway** running inside a Daytona-managed environment
- **Docker-in-Docker** support for running containers from within your agent
- **Persistent workspace** — your OpenClaw config is version-controlled in the devcontainer
- **Reproducible** — any team member gets the same setup with `daytona create`

## Common Configurations

### Self-Hosted LLM

Point OpenClaw to a local Ollama instance:

```bash
openclaw config set provider.ollama.baseUrl http://localhost:11434
openclaw config set provider.ollama.model llama3:70b
```

### Multi-Channel Setup

Run OpenClaw with multiple channels simultaneously:

```yaml
# ~/.openclaw/config.yaml
channels:
  telegram:
    bot_token: ${TELEGRAM_BOT_TOKEN}
  discord:
    bot_token: ${DISCORD_BOT_TOKEN}
  imessage:
    enabled: true
```

### Skills

Install community skills:

```bash
clawhub install web-search
clawhub install desktop-automation
clawhub install file-manager
```

## Troubleshooting

**Environment won't start:**
```bash
daytona logs <env-name>
# Check devcontainer build output
```

**OpenClaw gateway won't bind:**
```bash
# Check port 3000 isn't already in use
lsof -i :3000
# Change port in config
openclaw config set gateway.port 3001
```

**Claude Code can't see OpenClaw:**
- Verify the VS Code extension is installed in the Dayton environment (not your local machine)
- Reload the window after connecting

## Performance Tips

1. **Use prebuilds**: Configure Daytona to prebuild your environment so new branches spin up instantly
2. **Resource allocation**: Give the Daytona environment at least 2 CPU cores and 4 GB RAM for smooth agent operation
3. **Persistent storage**: Mount a volume for OpenClaw's memory files so they survive environment restarts

## Comparison: Manual vs. Daytona Setup

| Step | Manual | Daytona |
|------|--------|---------|
| Install dependencies | 30–60 min | 0 min (pre-configured) |
| Configure IDE | 15–20 min | 0 min (pre-configured) |
| Onboard new developer | 1–2 hours | `daytona create` + 5 min |
| Environment reproduction | Days of debugging | Identical every time |
| CI/CD integration | Complex | Built-in |

## Next Steps

- [Daytona documentation](https://daytona.io/docs) — advanced provider configuration
- [OpenClaw documentation](https://docs.openclaw.ai) — skills, channels, and agent hosting
- [ClawHub](https://clawhub.ai) — browse community skills

## Conclusion

Daytona + OpenClaw removes the biggest friction point in AI agent setup: environment configuration. Spend your time building skills and automations, not fighting dependency conflicts.

Two minutes. That's all it takes.
