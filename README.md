# Qomni Engine v7.4 — Cognitive Inference Cascade for Engineering Decision Support

**12-module Rust AI system · 207 features · 9-layer inference cascade · Lock-free architecture**

[![Live System](https://img.shields.io/badge/system-live-00e5ff)](https://qomni.clanmarketer.com/crysl/)
[![Engine v7.4](https://img.shields.io/badge/engine-v7.4-e040fb)](https://qomni.clanmarketer.com/)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue)](LICENSE)
[![arXiv](https://img.shields.io/badge/arXiv-preprint-red)](arxiv/main.tex)

> **Percy Rojas Masgo** · CEO Condesi Perú · Qomni AI Lab
> percy.rojas@condesi.pe · https://qomni.clanmarketer.com/

---

## What is Qomni Engine?

Qomni is a **hybrid neuro-symbolic AI system** that routes engineering queries through a 9-layer cognitive cascade. Instead of sending every query to an LLM (expensive, slow, non-deterministic), Qomni resolves most queries at shallow layers via pattern recognition, reflex memory, or crystallized inference — reaching the LLM layer only for genuinely novel queries.

**Key insight:** 97% of engineering queries in production are variations of previously seen queries. Caching their resolution at multiple abstraction levels reduces mean latency from 846ms to 47ms — **18× speedup** on real query distributions.

---

## Architecture Evolution (v0.59 → v7.4)

| Version | Key advancement |
|---------|----------------|
| v0.59 (original preprint) | Adaptive routing with online learning loops |
| v4.1–v4.7 | 7 cognitive features (SemanticProfiler, ThemeDetector, PatternMemory, IntuitionEngine, SleepCycle, MeshProtocol, ReflexEngine) |
| v4.8 | SemanticClusterer (N-gram clustering) |
| v5.3 | NeuralGating (selective layer activation) |
| v5.7 | HDC Memory (10,000-dim binary hypervectors) |
| v5.8 | Solid State Inference (crystallized patterns, 0ms retrieval) |
| v6.4 | Lock-free engine (59 write locks → DashMap + atomics) |
| v7.1–v7.3 | C4 deduplication, H5 delimiters, TUI dashboard, mesh peer auth |
| **v7.4** | **CRYS-L as primary compute backend · Universal Intent Router** |

---

## The 9-Layer Cognitive Cascade

```
Query
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│ Layer 9: Universal Intent Router (0ms)                  │
│   keyword classification → 8 intent classes             │
│   BLOCKS fallback to wrong engine                       │
└─────────────────┬───────────────────────────────────────┘
                  │ if intent = "calculation"
  ┌───────────────▼────────────────────────────────────┐
  │ Layer 5: Reflex Engine (0ms) — 34% hit rate        │
  │   Exact match in reflex store → DirectResponse     │
  └──────────────────┬─────────────────────────────────┘
                     │ miss
  ┌──────────────────▼─────────────────────────────────┐
  │ Layer 6: Solid State (3ms) — 28% hit rate          │
  │   Crystallized inference patterns (frequent)       │
  └──────────────────┬─────────────────────────────────┘
                     │ miss
  ┌──────────────────▼─────────────────────────────────┐
  │ Layer 4: HDC Intuition (5ms) — 19% hit rate        │
  │   10,000-dim binary hypervector similarity         │
  └──────────────────┬─────────────────────────────────┘
                     │ miss
  ┌──────────────────▼─────────────────────────────────┐
  │ Layer 3: Pattern Memory (12ms) — 11% hit rate      │
  │   N-gram cluster match (SemanticClusterer v4.8)    │
  └──────────────────┬─────────────────────────────────┘
                     │ miss
  ┌──────────────────▼─────────────────────────────────┐
  │ Layer 2: Concept Graph (35ms) — 5% hit rate        │
  │   Concept traversal, domain inference              │
  └──────────────────┬─────────────────────────────────┘
                     │ miss (3% of queries)
  ┌──────────────────▼─────────────────────────────────┐
  │ Layer 1: CRYS-L Oracle (9µs compute, 200ms round)  │
  │   JIT-compiled domain oracle via plan engine       │
  └────────────────────────────────────────────────────┘
```

**Mean latency across production query distribution: 47ms**
(vs 846ms if all queries routed to compute layer)

---

## Universal Intent Router

The Universal Intent Router (UIR) is the critical v7.4 addition. It intercepts queries **before** the plan engine and routes to 8 distinct response types:

| Intent detected | Decision label | Response type | Example query |
|-----------------|---------------|--------------|--------------|
| `throughput_compare` | `COMPARISON` | Throughput table | "compara speedup vs paper" |
| `slo_metrics` | `LATENCY_STATS` | p50/p95/p99 card | "muéstrame p50 p95 p99" |
| `adversarial` | `ADVERSARIAL_RESULT` | NaN-Shield card | "qué pasa con NaN e infinito" |
| `repeatability` | `REPEATABILITY` | Determinism card | "100 veces mismo resultado" |
| `meta` | `EVIDENCE` | Benchmark card | "cuánto vs GPT-4" |
| `pareto` | `PARETO_ANALYSIS` | Pareto card | "frente de pareto" |
| `calculation` | `VALIDATED/CALCULATED` | Pipeline card | "bomba 500 gpm 100 psi" |
| `validation` | `VALIDATED` | Compliance card | "revisa mi cálculo estructural" |

**Design invariant:** A query about adversarial inputs **cannot** route to `plan_pump_sizing`. This was the main failure mode of the original architecture — meta-queries fell through to the default engineering plan.

---

## Feature Count by Version

| Version range | Features added | Running total |
|--------------|----------------|--------------|
| v1.0–v3.9 | Core engine, LLM routing | ~80 |
| v4.1–v4.7 | 7 cognitive layers | 87 |
| v4.8 | SemanticClusterer | 88 |
| v5.1–v5.2 | Pipeline + rate limiter | 90 |
| v5.3 | NeuralGating | 91 |
| v5.4–v5.6 | Training, mesh, auth | 94 |
| v5.7 | HDC Memory | 95 |
| v5.8 | Solid State Inference | 96 |
| v6.1–v6.4 | Lock-free (59 → 0 write locks) | 120 |
| v7.1–v7.3 | C4, H5, TUI, mesh auth | 140 |
| v7.4 | CRYS-L integration, UIR | **207** |

---

## 12-Module Rust Architecture

```
src/
├── main.rs        ← Entry point, CLI, --tui flag
├── cognitive.rs   ← Layer 3–4 (Pattern, Intuition)
├── handlers.rs    ← HTTP request routing
├── advanced.rs    ← Layer 5–6 (Reflex, Solid State)
├── engine_core.rs ← Layer cascade orchestration
├── auth.rs        ← API key management
├── agents.rs      ← Multi-agent coordination
├── training.rs    ← Online learning loops
├── types.rs       ← Shared type definitions
├── tools.rs       ← Tool registry
├── mesh.rs        ← Peer mesh (H4 auth)
└── voice.rs       ← Voice interface
```

main.rs was reduced from **15,122 lines → 2,843 lines** (81% reduction) through modularization.

---

## Lock-Free Architecture (v6.4)

The v5.x engine had 59 write locks (Mutex/RwLock) creating serialization bottlenecks under concurrent load. v6.4 replaced these with:

- `DashMap<K, V>` — sharded concurrent hash maps (no global lock)
- `Arc<AtomicUsize>` — lock-free counters
- `crossbeam::queue` — wait-free SPSC queues for reflex dispatch
- Remaining: 0 Mutex, 0 RwLock in hot paths

**Result:** Linear throughput scaling to 12 cores (previously plateaued at 4 cores due to lock contention).

---

## Security Architecture (v7.3 — 10/10)

| Feature | Implementation |
|---------|---------------|
| API authentication | HMAC-SHA256 bearer tokens |
| Mesh peer auth (H4) | `QOMNI_MESH_SECRET` → `X-Qomni-Mesh-Auth` header |
| Content dedup (C4) | SHA-256 content addressing |
| Rate limiting | Middleware on ALL endpoints |
| Input validation | NaN-Shield (branchless guards) |
| TLS | nginx proxy with Let's Encrypt |
| UFW firewall | Only 2291/80/443 open |
| fail2ban | 4 jails (SSH, nginx, API) |
| Endlessh honeypot | Port 2222 |
| rkhunter | Rootkit detection |

---

## TUI Dashboard (v7.2)

```bash
./qomni-server --tui   # Launch with terminal dashboard
```

Real-time display: active layer, query throughput, cache hit rates per layer, mesh peer status, reflex store size, HDC vector count.

---

## Performance Numbers (Production — Server5)

**Server:** AMD EPYC 7282, 12 cores, 48GB RAM, Ubuntu 24.04.4

| Query class | Mean latency | Layer resolved at |
|------------|-------------|------------------|
| Reflex hit | 0ms | L5 |
| Solid State hit | 3ms | L6 |
| HDC Intuition hit | 5ms | L4 |
| Pattern match | 12ms | L3 |
| Concept graph | 35ms | L2 |
| CRYS-L oracle | 9µs compute + ~200ms roundtrip | L1 |
| **Production mean** | **47ms** | (weighted average) |

**Cascade speedup vs all-LLM routing: 18×**

---

## Live Endpoints

```bash
# Engine status
curl https://qomni.clanmarketer.com/qomni/v4/status

# Cognitive cascade state
curl https://qomni.clanmarketer.com/qomni/v4/gating

# Reflex store toggle
curl -X POST https://qomni.clanmarketer.com/qomni/v4/reflex/toggle

# Sleep/consolidation cycle
curl -X POST https://qomni.clanmarketer.com/qomni/v4/sleep/trigger

# Full chat with intent routing
curl -X POST https://qomni.clanmarketer.com/crysl/orch/orchestrate \
  -H "Content-Type: application/json" \
  -d '{"q": "bomba contra incendio 500 gpm 100 psi eficiencia 75%"}'
```

---

## Benchmark Results (vs Original v0.59)

| Metric | v0.59 (original paper) | v7.4 (current) |
|--------|----------------------|---------------|
| Mean query latency | ~400ms | **47ms** (18× faster) |
| Max throughput | ~50 req/s | **117M ops/s** (CRYS-L mode) |
| Domain coverage | General NLU | 13 engineering domains, 56 plans |
| Stochastic outputs | Yes (LLM) | No for 97% of queries |
| Intent routing accuracy | ~70% | **~99%** (keyword router) |
| Concurrent users (no degradation) | 8 | 100+ (lock-free) |
| Module count (Rust) | 1 (monolith) | 12 modules |
| Lines of code (main.rs) | ~15,122 | **2,843** (81% reduction) |

---

## Results Directory (Updated)

```
results/
├── benchmark_live.json        ← Live benchmark numbers (v7.4)
├── crystal_benchmark.json     ← CRYS-L throughput proofs
├── planner_evolution.json     ← Version-by-version metrics
└── cascade_hit_rates.json     ← Per-layer hit rates (NEW)
```

---

## Cite This Work

```bibtex
@misc{rojas2026qomni,
  title  = {Qomni Engine v7.4: Hybrid Neuro-Symbolic Cognitive Inference Cascade
            for Deterministic Engineering Decision Support},
  author = {Rojas Masgo, Percy},
  year   = {2026},
  month  = {April},
  note   = {Preprint. Qomni AI Lab, Condesi Perú},
  url    = {https://github.com/condesi/qomni-planner-paper}
}
```

---

**Live system:** https://qomni.clanmarketer.com/crysl/
**CRYS-L source:** https://github.com/condesi/crysl
**Contact:** percy.rojas@condesi.pe
**License:** Apache-2.0
