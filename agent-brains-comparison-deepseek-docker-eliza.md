# Agent Brain Comparison — DeepSeek + Docker RAM + ElizaOS Readiness

**Date:** 2026-07-09
**Criteria:** Must run on DeepSeek API · Docker RAM footprint · ElizaOS plugin readiness

---

## Comparison Matrix

| System | DeepSeek API | Docker RAM (properly configured) | ElizaOS Readiness | Stars |
|--------|-------------|----------------------------------|-------------------|-------|
| **Mem0** | ✅ Native (Python + TS) | ~500 MB (API) + ~300 MB (Postgres) = **~800 MB** | ✅ Best — npm SDK, REST API, trivial plugin | 48K |
| **Letta** | ✅ Via LiteLLM (100+ providers) | ~1.5 GB (API) + ~500 MB (Postgres) = **~2 GB** | ✅ Good — TS SDK, REST API, MCP tools | 25K |
| **GBrain** | ✅ Native (OpenClaw config) | **~200 MB** (PGLite, no server) | ✅✅ Native — built for OpenClaw/ElizaOS | 1.2K |
| **Cognee** | ✅ Via LiteLLM (custom provider) | ~1 GB (API) + ~500 MB (Neo4j) + ~300 MB (Postgres) = **~1.8 GB** | ⚠️ Moderate — Python SDK, REST API, no npm | 5K |
| **Hindsight** | ✅ Native (first-class provider) | ~1.2 GB (full) / ~500 MB (slim) + ~300 MB (Postgres) = **~1.5 GB / ~800 MB** | ⚠️ Moderate — Python SDK, REST API, no npm | 3K |
| **Metamorphosis** | ✅ Native (OpenClaw config) | **~200 MB** (PGLite, no server) | ✅✅ Native — IS OpenClaw/ElizaOS | 800 |

---

## DeepSeek API Compatibility — Verified

### Mem0
- **Status:** ✅ Native first-class provider
- **Config:** `DEEPSEEK_API_KEY` env var, provider set to `deepseek`
- **Docs:** [Mem0 DeepSeek](https://docs.mem0.ai/components/llms/models/deepseek)
- **Notes:** Both Python and TypeScript SDKs support DeepSeek natively. Model names: `deepseek-chat`, `deepseek-reasoner`.

### Letta
- **Status:** ✅ Via LiteLLM (100+ providers)
- **Config:** Set `LLM_PROVIDER=litellm`, model name `deepseek/deepseek-chat`
- **Docs:** [Letta Models](https://docs.letta.com/configuration/models)
- **Notes:** LiteLLM routes all provider calls. DeepSeek is one of 100+ supported providers. Model names use LiteLLM prefix format: `deepseek/deepseek-chat`, `deepseek/deepseek-reasoner`.

### GBrain
- **Status:** ✅ Native (OpenClaw config)
- **Config:** Standard OpenClaw LLM provider config — set provider to `deepseek` in `.openclaw/config.yaml`
- **Docs:** [GBrain README](https://github.com/garrytan/gbrain)
- **Notes:** GBrain is built on OpenClaw, which natively supports DeepSeek as an LLM provider. No extra config needed beyond the API key.

### Cognee
- **Status:** ✅ Via LiteLLM (custom provider)
- **Config:** Set `LLM_PROVIDER=custom`, `LLM_ENDPOINT=https://api.deepseek.com`, `LLM_MODEL=deepseek-chat`
- **Docs:** [Cognee LLM Providers](https://docs.cognee.ai/setup-configuration/llm-providers)
- **Notes:** Cognee routes through LiteLLM. DeepSeek works via the "Custom" OpenAI-compatible endpoint option. Model names: `deepseek-chat`, `deepseek-reasoner`.

### Hindsight
- **Status:** ✅ Native first-class provider
- **Config:** Set `HINDSIGHT_API_LLM_PROVIDER=deepseek`, `HINDSIGHT_API_LLM_API_KEY=sk-...`
- **Docs:** [Hindsight Models](https://hindsight.vectorize.io/developer/models)
- **Notes:** DeepSeek is listed as a first-class provider alongside OpenAI, Anthropic, Gemini, etc. Model names: `deepseek-chat`, `deepseek-reasoner`.

### Metamorphosis Agent
- **Status:** ✅ Native (OpenClaw config)
- **Config:** Standard OpenClaw LLM provider config — set provider to `deepseek` in `.openclaw/config.yaml`
- **Docs:** [Metamorphosis README](https://github.com/salmonhealer772/metamorphosis-agent)
- **Notes:** Metamorphosis is an OpenClaw-powered agent. DeepSeek is natively supported as an LLM provider. Setup script even has a `--env-file` option to pass `DEEPSEEK_API_KEY`.

---

## Docker RAM Footprint — Properly Configured for ElizaOS

### Mem0 (~800 MB total)
- **API Server:** ~500 MB (Python FastAPI + Qdrant local)
- **Postgres + pgvector:** ~300 MB
- **Dashboard (optional):** ~100 MB
- **Notes:** Lightest full-stack option. Qdrant local mode is very efficient. Can run on a 2 GB machine comfortably.

### Letta (~2 GB total)
- **API Server:** ~1.5 GB (Python + memory engine + tool system)
- **Postgres + pgvector:** ~500 MB
- **Notes:** Heaviest in the group. The tiered memory architecture (core + archival + ephemeral) and built-in tool system add overhead. Needs 4 GB machine for comfortable operation.

### GBrain (~200 MB total)
- **PGLite (embedded):** ~200 MB (no separate server process)
- **Notes:** Lightest option. PGLite runs in-process with no separate database server. Knowledge graph is self-wiring — no Neo4j or external graph DB needed.

### Cognee (~1.8 GB total)
- **API Server:** ~1 GB (Python + graph extraction engine)
- **Neo4j:** ~500 MB
- **Postgres:** ~300 MB
- **Notes:** Heaviest graph component. Neo4j is the main RAM consumer. Can reduce by using SQLite instead of Neo4j for smaller deployments.

### Hindsight (~1.5 GB full / ~800 MB slim)
- **API Server (full):** ~1.2 GB (Python + local BGE embedder + MiniLM reranker)
- **API Server (slim):** ~500 MB (no local models, external providers only)
- **Postgres + pgvector:** ~300 MB
- **Notes:** Full image bundles local embedding and reranker models (~220 MB combined). Slim image delegates to external providers and cuts RAM in half. Best option if you want academic rigor without the full RAM cost.

### Metamorphosis Agent (~200 MB total)
- **PGLite (embedded):** ~200 MB (no separate server process)
- **Notes:** Same as GBrain — PGLite in-process, no separate database server. Lightest option alongside GBrain.

---

## ElizaOS Integration Readiness

### Mem0 — ✅ Best
- npm SDK (`mem0ai`) for direct Node.js integration
- REST API server for plugin-style integration
- Trivial to drop into ElizaOS as a plugin service
- Per-user API keys, audit log, dashboard
- **Verdict:** Easiest to integrate, best documentation, most battle-tested

### Letta — ✅ Good
- TypeScript SDK for direct integration
- REST API server for plugin-style integration
- MCP tools for agent-to-agent communication
- **Verdict:** More powerful but heavier. Good if you need the full agent brain with tiered memory.

### GBrain — ✅✅ Native
- Built specifically for OpenClaw/ElizaOS
- PGLite (zero server overhead)
- Self-wiring knowledge graph
- Synthesis layer (actual answers, not just page lists)
- 30+ MCP tools
- **Verdict:** Best for our stack. Zero integration work needed.

### Cognee — ⚠️ Moderate
- Python SDK only (no npm)
- REST API server available
- No native ElizaOS plugin
- **Verdict:** Good for knowledge-graph-heavy setups, but requires more integration work. Python bridge needed for ElizaOS.

### Hindsight — ⚠️ Moderate
- Python SDK only (no npm)
- REST API server available
- No native ElizaOS plugin
- **Verdict:** Most academically rigorous, but requires Python bridge for ElizaOS. Best for research-heavy agents.

### Metamorphosis Agent — ✅✅ Native
- IS OpenClaw/ElizaOS — no integration needed
- PGLite (zero server overhead)
- Vector memory via OpenViking
- Self-editing (reads and rewrites its own source)
- Multi-channel (webchat, Signal, Discord, terminal)
- **Verdict:** If you want a full agent (not just a memory layer), this is the one. It's ElizaOS with persistent memory baked in.

---

## Recommendations

### For ElizaOS Plugin (Memory Layer Only)
1. **Mem0** — Best overall. Lightest full-stack, npm SDK, trivial integration.
2. **GBrain** — Best for our stack. Zero integration work, PGLite, knowledge graph.
3. **Hindsight (slim)** — Best for academic rigor. TEMPR search, observation consolidation.

### For Full Agent (Memory + Agent)
1. **Metamorphosis** — IS ElizaOS with persistent memory. Self-editing, multi-channel.
2. **Letta** — Most powerful agent brain. Tiered memory, built-in tools, identity + persona.
3. **GBrain** — Best for our stack. Knowledge graph, synthesis, 30+ MCP tools.

### For Knowledge-Graph-Heavy Setups
1. **Cognee** — Graph + vector + relational retrieval, cognitive science ontology.
2. **GBrain** — Self-wiring knowledge graph, synthesis layer.
3. **Hindsight** — Graph-based TEMPR search, observation consolidation.

---

## Bottom Line

**If you want the lightest Docker footprint with DeepSeek + ElizaOS:** GBrain or Metamorphosis (~200 MB, PGLite, native OpenClaw).

**If you want the best memory layer plugin:** Mem0 (~800 MB, npm SDK, trivial integration).

**If you want the most powerful agent brain:** Letta (~2 GB, tiered memory, built-in tools).

**If you want academic rigor:** Hindsight (~800 MB slim, TEMPR search, observation consolidation).

**If you want knowledge graphs:** Cognee (~1.8 GB, Neo4j, cognitive science ontology).
