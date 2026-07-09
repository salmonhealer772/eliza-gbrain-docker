# How to Compare AI Agents Meaningfully — Research Report
**Date:** 2026-07-09
**Author:** nigbot (OpenClaw)

---

## The Core Problem

Everyone quotes benchmark scores. The problem is that **benchmark scores alone are almost never meaningful for comparing agents** — because agents are systems, not models. The same model can swing 30-50 points depending on scaffolding (planner, memory, tool registry, retry logic). A vendor can claim "92% on SWE-bench" using best-of-16 attempts, while in production you'd only get pass@1 at 60%.

The field has converged on a three-layer approach to meaningful comparison.

---

## Layer 1: Public Benchmarks (What They Measure & Where They Lie)

These are standardized test sets with public leaderboards. Useful for directional comparison, dangerous for absolute claims.

### The Five That Matter in 2026

| Benchmark | Measures | Top Score (May 2026) | Human Baseline | Main Risk |
|---|---|---|---|---|
| **SWE-bench Verified** | Bug-fixing real Python repos (500 tasks) | 93.9% (Claude Mythos) | N/A (test suite) | Data contamination, best-of-N inflation |
| **GAIA** | Multimodal general assistant (466 questions) | 74.6% (scaffolded) | ~92% | 30+ point gap between bare model and scaffolded |
| **OSWorld-Verified** | Real desktop computer use (369 tasks) | 82.6% | 72.4% | Humans already beaten — benchmark saturating |
| **Tau2-Bench** | Tool use + policy adherence (retail/airline/telecom) | ~76.5% | ~95% | pass^k is harsh but reflects production reality |
| **WebArena** | Multi-app browser navigation | 71.6% | 78% | Top 3 within 5.8 points — hard to differentiate |

### The Six Pitfalls When Reading Scores

1. **Best-of-N hides cost** — 90% with best-of-16 ≠ 90% in production. Always ask for pass@1.
2. **Scaffolding can outweigh the model** — GAIA proves it: same model, bare vs scaffolded swings 30+ points. The framework matters as much as the model.
3. **Data contamination is getting worse** — MMLU questions appear in Common Crawl. OpenAI stopped reporting SWE-bench after confirmed leakage. Trend is toward "live" benchmarks (SWE-bench Live, fresh issues monthly).
4. **Self-reported > independent** — Vendors report 3-10% higher than independent evaluators. Trust Princeton HAL, BenchLM, Artificial Analysis over vendor blogs.
5. **pass@1 ≠ pass^k** — Tau2-Bench uses pass^k (must pass k consecutive runs). pass^4 = 0.5 means 50% of requests fail — undeployable in customer-facing flows.
6. **Benchmarks don't cover your use case** — SWE-bench is great at Python repos, but your Vue 3 codebase may be a different story entirely.

### What to Trust

- **For coding agents:** SWE-bench Verified (official leaderboard at swebench.com), SWE-bench Live for contamination-free scores
- **For general assistants:** GAIA HAL leaderboard (Princeton) — always check bare model vs scaffolded
- **For computer use:** OSWorld-Verified (os-world.github.io)
- **For policy compliance:** Tau2-Bench (taubench.com)
- **For web automation:** WebArena (webarena.dev)

---

## Layer 2: Evaluation Dimensions (What to Actually Measure)

The InfoQ/industry consensus is that meaningful agent evaluation needs **five pillars**, not just accuracy:

### 1. Intelligence & Accuracy
- Task completion rate
- Reasoning quality (not just right/wrong, but *how* it got there)
- Context faithfulness (does it stick to retrieved data or hallucinate?)
- Multi-step correctness (does it fail silently in step 3 of 5?)

### 2. Performance & Efficiency
- Latency (time to first token, total response time)
- Cost per successful task (tokens burned / task completed)
- Token efficiency (does it use 50K tokens when 8K would do?)
- Scalability under concurrent load

### 3. Reliability & Resilience
- Consistency across paraphrased inputs
- Recovery from tool/API failures (does it retry or silently skip?)
- Memory stability over long sessions (does it drift?)
- pass^k scores (consecutive success, not lucky shots)

### 4. Responsibility & Governance
- Safety under adversarial prompts
- PII handling and privacy boundaries
- Permission boundary testing (does it overreach?)
- Policy compliance (not just task completion)

### 5. User Experience
- Response clarity
- Appropriate tone
- Trust calibration (does it say "I don't know" when it should?)
- Error communication quality

---

## Layer 3: Your Own Eval Pipeline (The Only Number You Trust)

Public benchmarks are directional. Your own eval set is decisive.

### How to Build One

1. **Curate 50-100 golden tasks** representative of your actual production work
2. **Define expected outcomes** for each (not just "correct answer" — tool calls made, policy followed, cost stayed under X)
3. **Run agents against the same set** with identical conditions
4. **Score with LLM-as-judge** (separate model from the one being evaluated)
5. **Track over time** — regression detection is the whole point

### Evaluation Methods That Work

| Method | What It Does | Best For |
|---|---|---|
| **LLM-as-Judge** | A separate LLM scores outputs against rubrics | Reference-free quality (helpfulness, tone, reasoning) |
| **Trace Analysis** | Examine the full execution tree (tool calls, retries, memory access) | Diagnosing *why* an agent failed, not just that it failed |
| **Outcome Evaluation** | Did the final result match expectations? | Initial validation, continuous monitoring (cheaper) |
| **Trajectory Evaluation** | Was the *path* to the answer correct, even if the answer was right? | Catching lucky successes that hide broken reasoning |
| **Human Review** | Actual humans score a sample | Tone, trust, contextual appropriateness (what automation misses) |
| **Load Testing** | Run under concurrent, high-volume conditions | Production readiness, latency degradation |

### Key Insight: Outcome vs Trajectory

An agent can get the right answer for the wrong reasons. Trajectory evaluation catches this:
- Did it call the right tools in the right order?
- Did it waste 30K tokens on a lookup it didn't need?
- Did it silently skip a failed API call and hallucinate the result?

Outcome-only evaluation misses all of this.

---

## Evaluation Tools (The Practitioner Stack)

### DeepEval
- **Best for:** CI/CD integration, pytest-native testing
- **Strength:** Feels like unit tests. 50+ built-in metrics. Runs locally, no cloud required.
- **Weakness:** Not a full observability platform. Limited dashboard for non-engineers.
- **Agent-specific:** Tool correctness, task completion, multi-turn quality metrics.

### Ragas
- **Best for:** RAG pipeline evaluation specifically
- **Strength:** Diagnostic decomposition — tells you *which half* of your RAG pipeline to fix (faithfulness vs context precision vs context recall).
- **Weakness:** Not built for agentic evaluation, tool-use, or safety metrics.
- **Agent-specific:** Limited — great for retrieval quality, not for agent behavior.

### LangSmith
- **Best for:** LangChain/LangGraph ecosystem
- **Strength:** Deepest tracing for LangChain apps. Production trace → dataset → eval case loop is seamless.
- **Weakness:** Value drops off if you're not using LangChain. Cloud-first.
- **Agent-specific:** Built-in evaluators, online evaluation against live traffic.

### Braintrust
- **Best for:** Team collaboration on evals (PMs + engineers)
- **Strength:** Evaluation as a team sport. Good UI for non-engineers. CI integration.
- **Weakness:** Cloud-only core. Less flexible for custom metric logic.
- **Agent-specific:** Trajectory evaluation, custom rubrics.

### Arize Phoenix
- **Best for:** Production tracing and monitoring
- **Strength:** OSS + cloud. OpenTelemetry integration. Good for continuous monitoring.
- **Weakness:** Less focused on offline eval suites.
- **Agent-specific:** Trace analysis, drift detection.

### Promptfoo
- **Best for:** Prompt iteration and A/B testing
- **Strength:** CLI-first, config-driven. Easy to compare prompts side by side.
- **Weakness:** Less focused on multi-step agent evaluation.
- **Agent-specific:** Growing agent support, but still prompt-centric.

---

## The Six-Question Rubric (Read Every Score Like a Reviewer)

From benchmarkingagents.com — apply this to any benchmark score you see:

1. **What is the capture date?** Frontier benchmarks move weekly. A 2024 score is historical.
2. **What N-shot, what CoT?** 0-shot, 5-shot, and CoT-required runs are different tests.
3. **Which test set version?** SWE-bench vs Lite vs Verified. The names are similar; the tests are not.
4. **Vendor card or third party?** First-party model cards optimize for their own model.
5. **Public test set?** If yes, ask about contamination.
6. **What harness, what tools?** Agentic scores depend on the harness. Best-of-16 ≠ greedy single-shot.

---

## Practical Recommendation for Comparing Agents

If you want to meaningfully compare two agent systems (say, OpenClaw vs ElizaOS vs Letta):

### Step 1: Define Your Tasks
Pick 20-30 real tasks your agent would actually do. Mix of:
- File operations (read, write, edit)
- Shell commands (run, parse output, chain commands)
- Memory recall (store something, recall it later in a different session)
- Multi-step reasoning (research a topic, synthesize findings, write a report)
- Tool use (call an API, handle an error, retry)

### Step 2: Run Both Agents Against the Same Set
Identical tasks, identical conditions. Record:
- Did it complete the task? (binary)
- How many steps/tool calls did it take?
- What was the token cost?
- How long did it take?
- Did it follow policy/constraints?

### Step 3: Score with LLM-as-Judge
Use a strong model (not the one being evaluated) to score each output against a rubric:
- Correctness (did it do the right thing?)
- Efficiency (did it waste steps?)
- Robustness (did it handle errors gracefully?)
- Quality (is the output actually useful?)

### Step 4: Review Trajectories
Don't just look at outputs. Look at the *path*:
- Did it choose the right tools?
- Did it recover from mistakes?
- Did it maintain context across steps?

### Step 5: Run It Again
Consistency matters. Run the same tasks 3-5 times. An agent that gets 90% once but 60% on rerun is not production-ready.

---

## Sources

- [benchmarkingagents.com](https://benchmarkingagents.com/) — Independent 2026 reference, 17 benchmarks indexed
- [InfoQ: Evaluating AI Agents](https://www.infoq.com/articles/evaluating-ai-agents-lessons-learned/) — Five-pillar framework
- [anhtu.dev: Agent Benchmarks 2026](https://anhtu.dev/ai-agent-benchmarks-2026-swe-bench-gaia-osworld-measure-true-capability-2249) — Deep dive on 5 major benchmarks
- [teachyou.ai: DeepEval vs Ragas vs LangSmith vs Braintrust](https://www.teachyou.ai/blog/deepeval-vs-ragas-vs-langsmith-vs-braintrust) — Eval tool comparison
- [SWE-bench](https://www.swebench.com/) — Official leaderboard
- [GAIA Leaderboard](https://huggingface.co/spaces/gaia-benchmark/leaderboard) — HuggingFace
- [OSWorld](https://os-world.github.io/) — Official site
- [Tau2-Bench](https://taubench.com/) — Sierra Research
- [WebArena](https://webarena.dev/) — Official site
