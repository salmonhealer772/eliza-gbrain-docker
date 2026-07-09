# OSS Agent Brains — Research Report
**Date:** 2026-07-09
**Author:** nigbot (OpenClaw)

---

## What We're Looking For

Plug-and-play agent brains that:
- Auto-store and auto-recall memories across sessions
- Support bash/shell tool use (or at least tool-use in general)
- Are open source and self-hostable
- Are easy to hook up to ElizaOS (plugin architecture or MCP)

---

## Top Contenders

### 1. Mem0 ⭐ Best Overall Pick

| | |
|---|---|
| **Repo** | [mem0ai/mem0](https://github.com/mem0ai/mem0) |
| **Stars** | ~48K |
| **License** | Apache 2.0 |
| **Language** | Python (primary), Node.js SDK |
| **Self-hosted** | Yes — Docker Compose, one command |
| **Cloud option** | Yes (mem0.ai platform) |

**What it does:**
Universal memory layer. Auto-extracts facts from conversations, stores them, and auto-recalls relevant memories on each new interaction. Multi-level memory (user, session, agent). Hybrid search (semantic + BM25 keyword + entity linking). Temporal reasoning (time-aware retrieval).

**Why it's strong:**
- Dead simple API: `memory.add()` and `memory.search()` — that's the core loop
- Self-hosted in one Docker Compose command
- Python + Node.js SDKs (ElizaOS is TypeScript/Node, so the npm package works)
- Agent self-signup: an agent can mint its own API key in 4 commands, no human needed
- Benchmarks: 91.6 on LoCoMo, 94.8 on LongMemEval — best-in-class recall
- Skills for Claude Code, Codex, Cursor, OpenClaw already exist

**ElizaOS compatibility:**
ElizaOS has a Python SDK (`elizaos` on PyPI) and a TypeScript core. Mem0 has both Python and npm packages. You can wire Mem0 into ElizaOS as a plugin service — the ElizaOS plugin model supports registering services (long-lived singletons), and Mem0's client is exactly that. There's also an `elizaos-plugin-memory` package on PyPI suggesting the community is already bridging them.

**Verdict:** 🏆 Top pick. Easiest to drop in, best recall, self-hostable, dual-language SDK.

---

### 2. Letta (formerly MemGPT) ⭐ Best for Stateful Agents

| | |
|---|---|
| **Repo** | [letta-ai/letta](https://github.com/letta-ai/letta) / [letta-ai/letta-code](https://github.com/letta-ai/letta-code) |
| **Stars** | ~21K |
| **License** | Apache 2.0 |
| **Language** | Python (server), TypeScript (Agent SDK) |
| **Self-hosted** | Yes — Docker, or local via CLI |
| **Cloud option** | Yes (Constellation) |

**What it does:**
Full agent harness with tiered memory architecture (inspired by OS memory management). Agents have identity, persona, and persistent memory that survives across sessions. MemFS (git-tracked memory filesystem) with agent dreaming. Built-in tool use including bash/shell execution. Skills and subagents system.

**Why it's strong:**
- Not just a memory layer — it's a complete agent runtime with memory built in
- Tiered memory: core memory (small, fast), recall memory (vector search), archival memory (long-term storage)
- Agents can rewrite their own memory, skills, and prompts
- Bash/shell tool use is first-class
- Model-agnostic (Anthropic, OpenAI, local models)
- TypeScript Agent SDK for embedding into apps

**ElizaOS compatibility:**
Both are TypeScript-first frameworks. Letta's Agent SDK (`@letta-ai/letta-agent-sdk`) is npm-installable and could run as an ElizaOS plugin service. The overlap is significant — both are agent runtimes. You could potentially use Letta as the memory backend for an ElizaOS agent, or run them side by side. Less plug-and-play than Mem0, but more powerful if you want a full agent brain with identity.

**Verdict:** 🥈 Best if you want a full agent brain with identity + memory + tool use, not just a memory layer.

---

### 3. GBrain ⭐ Best for OpenClaw Users

| | |
|---|---|
| **Repo** | [garrytan/gbrain](https://github.com/garrytan/gbrain) |
| **Stars** | ~5.4K (growing fast) |
| **License** | MIT |
| **Language** | TypeScript (Bun) |
| **Self-hosted** | Yes — PGLite (zero server) or Postgres/Supabase |
| **Cloud option** | No (designed self-hosted) |

**What it does:**
Garry Tan's (YC CEO) opinionated agent brain. Markdown + pgvector hybrid. Self-wiring knowledge graph with typed entity edges (works_at, invested_in, founded, etc.). Synthesis layer that gives actual answers with citations, not just page lists. Gap analysis (tells you what it doesn't know). 34+ skills. Overnight dream cycle for enrichment.

**Why it's strong:**
- Built specifically for OpenClaw/Hermes — the README literally says "Garry's Opinionated OpenClaw/Hermes Agent Brain"
- 30+ MCP tools out of the box
- PGLite means 2-second local brain, zero Docker, zero server
- Knowledge graph is automatic — no manual schema design
- Synthesis + gap analysis is unique in the space
- Designed to be installed BY an agent (paste a URL, agent does the work)

**ElizaOS compatibility:**
GBrain is TypeScript/Bun, same runtime as ElizaOS. MCP support means it can connect to anything that speaks MCP. ElizaOS has MCP integration. The bridge is straightforward — GBrain as an MCP server, ElizaOS agent calling it. Not a native ElizaOS plugin yet, but the architecture fits perfectly.

**Verdict:** 🥉 Best if you're already in the OpenClaw ecosystem or want a knowledge-graph-first brain. Less mature than Mem0/Letta but growing fast and designed for our stack.

---

### 4. Cognee ⭐ Best for Knowledge Graph Heavy Users

| | |
|---|---|
| **Repo** | [topoteretes/cognee](https://github.com/topoteretes/cognee) |
| **Stars** | ~12K |
| **License** | Open core |
| **Language** | Python |
| **Self-hosted** | Yes — Docker Compose |
| **Cloud option** | Yes (Cognee Cloud) |

**What it does:**
Knowledge engine combining vector search, graph databases, and cognitive-science-grounded ontology generation. Ingests any format (text, PDF, code, structured data). Builds a continuously evolving knowledge graph. Four core memory operations: retain, recall, reflect, reason.

**Why it's strong:**
- Strongest on institutional/organizational knowledge
- Graph + vector + relational retrieval in one system
- Cognitive science ontology (not just flat embeddings)
- MCP integration available
- Good for multi-agent teams with shared knowledge

**ElizaOS compatibility:**
Python SDK, MCP support. Can be wired in as an external service. Less direct than Mem0's npm package, but the MCP bridge works.

**Verdict:** Strong for teams/orgs needing shared institutional memory. Heavier setup than Mem0.

---

### 5. Hindsight ⭐ Best for Structured Memory

| | |
|---|---|
| **Repo** | [vectorize-io/hindsight](https://github.com/vectorize-io/hindsight) |
| **Stars** | ~4K (fastest-growing in 2026) |
| **License** | MIT |
| **Language** | Python |
| **Self-hosted** | Yes |
| **Cloud option** | Yes (Vectorize.io) |

**What it does:**
Biomimetic memory with four logical networks: World Facts, Experiences, Observations, Opinions. Separates objective facts from subjective beliefs. Three core operations: Retain, Recall (TEMPR hybrid search), Reflect. ACL 2026 paper.

**Why it's strong:**
- Unique four-network architecture (facts vs beliefs separation)
- TEMPR hybrid search (temporal + semantic)
- Reflect operation lets agents learn from experience
- Fastest-growing OSS memory project in 2026
- MIT licensed

**ElizaOS compatibility:**
Python SDK, MCP support. Same story as Cognee — works via MCP bridge.

**Verdict:** Most academically rigorous. Great if you care about separating facts from opinions in agent memory.

---

## Comparison Matrix

| Feature | Mem0 | Letta | GBrain | Cognee | Hindsight |
|---|---|---|---|---|---|
| **Auto-store** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Auto-recall** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Bash/tool use** | Via agent | ✅ Built-in | Via MCP | Via MCP | Via MCP |
| **Self-hosted** | ✅ Docker | ✅ Docker | ✅ PGLite | ✅ Docker | ✅ |
| **Knowledge graph** | Entity linking | ❌ | ✅ Self-wiring | ✅ Full KG | 4-network |
| **Synthesis** | ❌ | Partial | ✅ | ✅ | ✅ Reflect |
| **npm/TS SDK** | ✅ | ✅ | ✅ Native | ❌ | ❌ |
| **Python SDK** | ✅ | ✅ | ❌ | ✅ | ✅ |
| **MCP support** | ✅ | ✅ | ✅ 30+ tools | ✅ | ✅ |
| **ElizaOS fit** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| **Setup complexity** | Low | Medium | Low | Medium | Medium |
| **License** | Apache 2.0 | Apache 2.0 | MIT | Open core | MIT |

---

## Recommendation for ElizaOS Integration

**If you want the easiest drop-in:** **Mem0** — npm package, simple API, self-hosted in one command, already has ElizaOS community bridges.

**If you want a full agent brain with identity:** **Letta** — complete runtime, tiered memory, bash built-in, TypeScript SDK.

**If you're in the OpenClaw ecosystem:** **GBrain** — built for it, PGLite zero-server, knowledge graph, synthesis layer.

**For our specific case (nigbot on OpenClaw):**
We already use OpenViking (vector memory via all-minilm). The question is whether to replace it or complement it. GBrain would be the most natural fit since it's designed for OpenClaw and adds the knowledge graph + synthesis layer we don't have yet. Mem0 would be the easiest swap if we wanted to replace OpenViking entirely.

---

## Sources

- [Mem0 GitHub](https://github.com/mem0ai/mem0)
- [Letta GitHub](https://github.com/letta-ai/letta)
- [GBrain GitHub](https://github.com/garrytan/gbrain)
- [Cognee GitHub](https://github.com/topoteretes/cognee)
- [Hindsight GitHub](https://github.com/vectorize-io/hindsight)
- [ElizaOS GitHub](https://github.com/elizaOS/eliza)
- [Best AI Agent Memory Systems 2026](https://vectorize.io/articles/best-ai-agent-memory-systems)
- [Mem0 vs Zep vs Letta Comparison](https://rohitraj.tech/en/notes/open-source-ai-agent-memory-mem0-vs-zep-letta-2026)
