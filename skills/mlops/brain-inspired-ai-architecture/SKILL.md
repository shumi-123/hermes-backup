---
name: brain-inspired-ai-architecture
description: Low-cost breakthrough plan for building cognitive memory and reasoning architecture on top of existing LLMs using skills (cognitive-brain, basal-ganglia, ACC, insula). Phases 0-3 implementation roadmap.
triggers:
  - brain inspired AI architecture
  - cognitive memory system
  - 人脑启发架构
  - AI记忆系统
  - four-layer memory
  - 自主进化
  - agent architecture
---

# Brain-Inspired AI Architecture - Low-Cost Breakthrough Plan

## Core Insight

**Don't build new models — build an architecture layer on existing LLMs.**

```
Current paradigm:          Breakthrough approach:
LLM → direct output        LLM + cognitive architecture layer → output
                              ↑
                          This layer doesn't require retraining
                          Built with skills + memory middleware
```

**Why no one has solved this despite knowing the problems:**
1. Scaling works — capital stays with scale, not risky architecture bets
2. Engineering complexity is exponential — 5+ modules, each needs to work standalone AND together
3. Training paradigm incompatibility — next-token prediction + RLHF doesn't support online learning
4. Missing evaluation metrics — can't prove "more reasoning" vs "better memorization"
5. Capital logic:投入1亿搞架构革命 → 可能失败 → 0产出

**The breakthrough: Build cognitive architecture as an external layer, not inside the model.**

---

## Architecture Blueprint

```
┌─────────────────────────────────────────────────────────────┐
│                    Human Brain-Inspired AI                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  User Input → Sensory Layer (text/voice/image/file parse)  │
│       ↓                                                     │
│  ┌──────────────────────────────────────────────────────┐  │
│  │            Working Memory (context window)            │  │
│  │  ┌────────────┐ ┌────────────┐ ┌────────────┐        │  │
│  │  │Curr context│ │Short cache │ │Attention  │        │  │
│  │  └────────────┘ └────────────┘ └────────────┘        │  │
│  │                                                       │  │
│  │  ┌─────────────────────────────────────────────┐     │  │
│  │  │  Habit System (Basal Ganglia)              │     │  │
│  │  │  Detect: familiar pattern? → fast match   │     │  │
│  │  └─────────────────────────────────────────────┘     │  │
│  │                                                       │  │
│  │  ┌─────────────────────────────────────────────┐     │  │
│  │  │  Conflict Detection (ACC)                   │     │  │
│  │  │  Detect: contradictions? error risk?       │     │  │
│  │  └─────────────────────────────────────────────┘     │  │
│  └──────────────────────────────────────────────────────┘  │
│       ↓                                                     │
│  ┌──────────────────────────────────────────────────────┐  │
│  │          Episodic Memory (vector DB retrieval)        │  │
│  │  Historical conversations, event logs, experience     │  │
│  └──────────────────────────────────────────────────────┘  │
│       ↓                                                     │
│  ┌──────────────────────────────────────────────────────┐  │
│  │          Semantic Memory (knowledge graph)            │  │
│  │  Entity-relation graph, concept hierarchy, skills     │  │
│  └──────────────────────────────────────────────────────┘  │
│       ↓                                                     │
│  ┌──────────────────────────────────────────────────────┐  │
│  │          Internal State (Insula)                     │  │
│  │  Energy (high/med/low) | Mood | Load (idle/busy)    │  │
│  └──────────────────────────────────────────────────────┘  │
│       ↓                                                     │
│  Reasoning Engine (TTT / symbolic-neural hybrid)          │
│       ↓                                                     │
│  Planning & Decision (Prefrontal Cortex)                  │
│       ↓                                                     │
│  Output                                                      │
└─────────────────────────────────────────────────────────────┘
```

---

## Four-Layer Memory System

| Layer | Duration | Capacity | AI Equivalent |
|-------|----------|----------|---------------|
| Sensory | ms | massive | Input preprocessing buffer |
| Working | sec~min | 7±2 items | Current context window |
| Episodic | permanent | unlimited | Vector DB / RAG |
| Semantic | permanent | unlimited | Knowledge graph |

**Brain region → AI module mapping:**
- 基底核(Basal Ganglia) → Habit system / fast path
- 前扣带皮层(ACC) → Conflict/error detection
- 岛叶(Insula) → Internal state (energy/mood/load)
- 海马体(Hippocampus) → Memory index
- 前额叶(Prefrontal) → Planning module

---

## Implementation Phases

### Phase 0: Framework (1 week, ~0 yuan)
**Goal: Make AI have "memory" using existing skills**

Skills already installed:
- basal-ganglia-memory ✓
- anterior-cingulate-memory ✓
- insula-memory ✓

Missing: cognitive-brain (needs PostgreSQL + Redis + pgvector)

**Low-cost alternative for concept validation:**
- Use file storage instead of vector DB
- Use SQLite instead of PostgreSQL
- Verify concept before investing in infrastructure

### Phase 1: Memory System (2-3 weeks)

```
Flow:
User Input → Habit Detection (fast path) → [match?] → yes → direct output
                              ↓ no
                    Working Memory (context)
                              ↓
              Episodic Memory Retrieval (cognitive-brain)
                              ↓
                    Conflict Detection (ACC)
                              ↓
                      LLM Reasoning
                              ↓
                  Internal State Update (insula)
                              ↓
                    Output + Store
```

**Cost estimate:**
| Item | Config | Monthly Cost |
|------|--------|--------------|
| Server | 4 cores 8GB | ~200 yuan |
| PostgreSQL | self-hosted | 0 |
| Redis | self-hosted | 0 |
| pgvector | open source | 0 |
| LLM calls | existing | 0 |
| **Total** | | **~200 yuan** |

### Phase 2: Fast/Slow Thinking Separation (2 weeks)

```
Traditional: every query → LLM → output

Fast/Slow separation:
  High-frequency simple queries → habit system → direct response (no LLM)
  Low-frequency complex queries → LLM reasoning
  
Result: 70%+ requests hit fast path, 70% latency reduction, 50% cost reduction
```

### Phase 3: Self-Evolution (ongoing, long-term)

```
Per interaction:
  Success pattern → reinforce into habit system
  Error pattern → log + fix
  
形成闭环:
  行动 → 结果 → 反馈 → 改进 → 习惯形成
  
This is Route C (自主进化型Agent) from the research
```

---

## Skills Installation Order

1. **cognitive-brain** (core memory — four-layer system)
   - Requires: PostgreSQL + Redis + pgvector
   - Install: `clawhub install cognitive-brain`

2. **self-evolution** (evolution core — self-improvement loop)
   - Rating: 2.0.0 (highest)
   - Install: `clawhub install self-evolution`

3. **basal-ganglia + anterior-cingulate + insula** (complete brain)
   - Already installed (verify with `clawhub list`)

4. **marl-middleware** (reasoning enhancement)
   - Reduces hallucination by 70%
   - 9 specialized reasoning engines

5. **evolver** (optional, evolution assist)

---

## Key Papers

| Paper | arXiv | Relevance |
|-------|-------|-----------|
| Mamba | 2312.00752 | SSM state space, O(N) complexity |
| TTT | 2407.04620 | Test-time training, hidden states learn |
| RWKV | 2412.14847 | Transformer alternative, linear attention |
| Working Memory Networks | 1805.09354 | ACL 2018, MemNN + relational reasoning |

---

## Success Metrics

**Short-term (1 month):**
- [ ] Basic conversation flow stable
- [ ] Habit system learns simple patterns
- [ ] Error detection measurably effective

**Medium-term (3 months):**
- [ ] Four-layer memory running
- [ ] Historical experience reusable
- [ ] Reasoning quality improved

**Long-term (6 months):**
- [ ] System learns new skills autonomously
- [ ] Performance exceeds pure Transformer baseline
- [ ] Handles complex multi-step tasks

---

## Why This Works

| LLM Weakness | Brain-Inspired Solution |
|--------------|-------------------------|
| No cross-session memory | Episodic memory layer |
| No habit system | Basal ganglia → fast path skip |
| No conflict detection | ACC → pre-output validation |
| No internal state | Insula → energy/load awareness |
| High reasoning cost | Fast/slow path separation |

**The essence:**
> Not "build AGI to surpass GPT-5"
> But "build a system that makes existing models 120% effective"

Architecture layer innovation = low cost + low risk + incremental improvement
