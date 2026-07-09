# eliza-gbrain-docker

**One command. Running agent with persistent memory on DeepSeek — contained.**

## What It Does

- **Auto memory** — everything you say gets stored. No commands, no prompts, no opt-in.
- **Auto recall** — when you talk about something relevant, the context just appears. No search, no triggers, no manual lookup.
- **Self-editing** — reads and rewrites its own config and prompts. Learns from mistakes.
- **Multi-channel** — talk to it from webchat, terminal, or any messaging surface.
- **Knowledge graph** — remembers how things relate, not just what you said. Gives you answers with citations, not page lists.

## What It Can't Do (Yet)

- Run local LLMs — DeepSeek API only
- Multi-agent orchestration — single agent, single brain
- Custom plugins beyond GBrain — locked to the shipped MCP tools
- **Touch your host** — it runs inside a Docker container. Even with sudo inside the container, it cannot reach outside. Nothing leaves the box unless you explicitly mount a volume or open a port.

## Quick Start

First time — `setup.sh` asks for your DeepSeek API key once:

```bash
git clone https://github.com/salmonhealer772/eliza-gbrain-docker && cd eliza-gbrain-docker && bash setup.sh
```

Then run agents by name:

```bash
bash scripts/up.sh --alice        # create or restart "alice"
bash scripts/enter.sh --alice     # talk to "alice"
bash scripts/down.sh --alice      # stop "alice" (memory persists)
```

Run multiple at once:

```bash
bash scripts/up.sh --alice
bash scripts/up.sh --bob
bash scripts/enter.sh --alice     # talks to alice
bash scripts/enter.sh --bob       # talks to bob
```

Each name gets its own container, its own brain, its own memory. Bring it down and it remembers everything. Bring it back up and it's where you left off.
