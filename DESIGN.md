# Design Notes — eliza-gbrain-docker

**Date:** 2026-07-09

## What It Is

One command gives you a running Docker container with ElizaOS and GBrain wired together on DeepSeek, ready to talk.

## What We Already Know

### GBrain
- PGLite (no DB server), ~200 MB RAM
- MCP stdio + HTTP, 30+ tools
- Native DeepSeek via OpenClaw config
- MCP tools: `brain_add`, `brain_search`, `brain_synthesize`, `brain_gap_analysis`, `brain_graph`, `brain_mcp_list`
- `gbrain init --pglite` → local brain, 2 seconds, no Docker
- `gbrain serve` → MCP stdio server

### ElizaOS
- Bun/TypeScript, plugin architecture
- character.json config
- MCP server support
- Native DeepSeek provider
- Plugins at plugins.elizacloud.ai

### Integration
- No existing plugin bridges GBrain → ElizaOS — we're building the first one
- Both are Bun/TypeScript, both speak MCP
- GBrain exposes MCP (stdio + HTTP)
- ElizaOS supports MCP tools via character.json mcpServers config

## Repo Structure

```
eliza-gbrain-docker/
├── README.md
├── setup.sh              # interactive: asks path, agent name, DeepSeek key
├── docker-compose.yml    # ElizaOS + GBrain + SSH service
├── Dockerfile            # Bun + ElizaOS + GBrain + SSH
├── scripts/
│   ├── enter.sh          # docker exec -it into the container
│   └── stop.sh           # bring it down
├── config/
│   ├── character.json    # ElizaOS character (templated, filled by setup.sh)
│   └── gbrain-config.json # GBrain config (templated, filled by setup.sh)
└── .env.template         # DEEPSEEK_API_KEY, AGENT_NAME, etc.
```

## How It Works

1. `git clone <repo> && cd eliza-gbrain-docker && bash setup.sh`
2. `setup.sh` asks:
   - Path to install everything
   - Agent name
   - DeepSeek API key
3. `setup.sh` writes `.env`, fills `character.json` with agent name + DeepSeek provider, fills `gbrain-config.json`, then runs `docker compose up -d`
4. Container has ElizaOS running with GBrain wired in as an MCP plugin
5. `bash scripts/enter.sh` → SSH into the container
6. `bash scripts/stop.sh` → bring it down

## Inside the Container
- ElizaOS running on port 3000
- GBrain running on port 8787 (MCP stdio)
- GBrain wired into ElizaOS via `character.json` mcpServers config
- DeepSeek set as the LLM provider
- SSH access for debugging

## Open Questions (Not Locked Yet)
- Single container vs two containers (ElizaOS + GBrain separate or combined)
- How GBrain wires into ElizaOS inside the container (MCP stdio pipe or HTTP)
- Whether setup.sh clones ElizaOS + GBrain or bakes them into the image
- What the Dockerfile base image is (Bun official? Debian slim?)
- Whether the container exposes a web UI, CLI, or both for talking to the agent
