# Design Notes — eliza-gbrain-docker

**Date:** 2026-07-09

## What It Is

One command. Running agent with persistent memory on DeepSeek — contained in Docker. Multiple agents by name, each isolated in its own container.

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
├── setup.sh              # one-time: asks for DeepSeek API key, writes .env
├── docker-compose.yml    # template for agent containers
├── Dockerfile            # Bun + ElizaOS + GBrain + SSH
├── scripts/
│   ├── up.sh             # bash scripts/up.sh --alice  (create or restart)
│   ├── down.sh           # bash scripts/down.sh --alice  (stop, memory persists)
│   └── enter.sh          # bash scripts/enter.sh --alice  (talk to it)
├── config/
│   ├── character.json.template   # ElizaOS character (filled by up.sh)
│   └── gbrain-config.json.template # GBrain config (filled by up.sh)
└── .env.template         # DEEPSEEK_API_KEY (filled by setup.sh)
```

## How It Works

1. `git clone <repo> && cd eliza-gbrain-docker && bash setup.sh`
   - Asks for DeepSeek API key once, writes `.env`

2. `bash scripts/up.sh --alice`
   - If "alice" doesn't exist → creates container, brain, config
   - If "alice" exists but stopped → restarts it
   - Each name gets its own container, its own PGLite brain, its own memory

3. `bash scripts/enter.sh --alice` → SSH into alice's container

4. `bash scripts/down.sh --alice` → stop alice (memory persists in volume)

## Naming Convention

- Container name: `eliza-<name>` (e.g., `eliza-alice`, `eliza-bob`)
- Volume name: `eliza-<name>-data` (persists across up/down)
- Port mapping: base port + offset per name (alice=3000, bob=3001, etc.)

## Inside Each Container
- ElizaOS running on its assigned port
- GBrain running on MCP stdio
- GBrain wired into ElizaOS via `character.json` mcpServers config
- DeepSeek set as the LLM provider
- SSH access for debugging
- **Cannot reach the host** — even with sudo inside, nothing leaves the container unless a port is explicitly exposed

## Open Questions (Not Locked Yet)
- How GBrain wires into ElizaOS inside the container (MCP stdio pipe or HTTP)
- Whether up.sh clones ElizaOS + GBrain or bakes them into the image
- What the Dockerfile base image is (Bun official? Debian slim?)
- Whether the container exposes a web UI, CLI, or both for talking to the agent
- Port assignment strategy for multiple agents (fixed offset? dynamic?)
