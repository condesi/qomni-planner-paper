# Qomni Planner v0.59: Adaptive Cognitive Routing with Online Learning Loops

**Percy Rojas Masgo** · CEO, Condesi Perú · **Qomni AI** · Preprint April 2026

---

## Abstract

We present **Qomni Planner v0.59**, an adaptive multi-factor routing system for edge AI inference that continuously learns from routing outcomes to improve decision thresholds over time.

The planner combines three independent scoring factors into a continuous confidence score, then dispatches queries across five routes — bypassing the LLM entirely when confidence is sufficient.

**Live benchmarks (2026-04-13):**
- 100% LLM bypass rate on domain-specific queries
- Average latency: **205 ms** vs 8,000–45,000 ms for full LLM inference
- Speedup: **39–220×** over neural generation

---

## Repository Structure

```
qomni-planner-paper/
├── arxiv/
│   └── main.tex              # Full LaTeX paper (arXiv-ready)
├── results/
│   ├── benchmark_live.json   # Live benchmark data (Server5, 2026-04-13)
│   ├── crystal_benchmark.json # Crystal retrieval latency per domain
│   └── planner_evolution.json # Phase 0.55→0.59 architecture comparison
├── datasets/
│   ├── test_queries.jsonl    # 12 annotated test queries with expected routes
│   └── routing_decisions.jsonl # Observed routing decisions (live traffic sample)
└── README.md
```

---

## Planner Evolution

| Phase | Name | LLM Bypass | Learning |
|-------|------|-----------|---------|
| 0.55 | Binary Crystal/Free | No | No |
| 0.56 | Multi-Factor Planner | No | No |
| 0.57 | Direct Execution | Yes (crystal score ≥ 3) | No |
| 0.58 | Cognitive Decision Engine | Yes (confidence ≥ 0.50) | No |
| **0.59** | **Learning Loop** | **Yes (adaptive)** | **Yes** |

---

## Confidence Scoring Formula

```
pre_conf = oracle_c + domain_c + numeric_c
conf     = pre_conf + crystal_bonus(0..0.10)
```

| Factor | Score 0 | Score 1 | Score 2 | Score 3 | Score 4+ |
|--------|---------|---------|---------|---------|----------|
| oracle_c | 0.00 | 0.10 | 0.22 | 0.35 | 0.42 |
| domain_c | 0.00 | 0.13 | 0.28 | 0.36 | 0.42 |

---

## Route Decision

```
conf ≥ 0.70  → PIPELINE-DIRECT  (oracle + crystal parallel, no LLM)
conf ≥ 0.50  → ORACLE/CRYSTAL-DIRECT  (best retrieval, no LLM)
conf ≥ 0.28  → CRYSTAL-CTX  (inject into LLM context)
conf  < 0.28 → FREE  (pure LLM)
```

## Adaptive Learning (Phase 0.59)

```
decides → executes → evaluates → saves → improves
```

- **PlannerStats**: 14 `AtomicU64` counters, zero lock overhead
- **Persistence**: `/opt/nexus/planner_stats.json` — atomic rename every 10 queries
- **Adaptation**: thresholds shift ±0.10 based on observed success rates (≥5 samples)
- **Endpoint**: `GET /qomni/planner/stats`

---

## System Stack

- **Runtime**: Rust + Axum + Tokio
- **Crystal Engine**: 8 domains, binary .crystal format (CRYS magic header)
- **CRYS-L JIT**: Engineering calculation DSL compiled to WASM (v2.1)
- **LLM**: qwen2.5-1.5b-instruct-q4_k_m (local, CPU)
- **Hardware**: AMD EPYC 12-core, 48 GB RAM, 500 GB NVMe

> **Note**: Qomni Engine source code is proprietary. This repository contains
> only the paper, benchmark data, datasets, and results for reproducibility.

---

## Citation

```bibtex
@article{rojasmasgo2026qomni,
  title   = {Qomni Planner v0.59: Adaptive Cognitive Routing with Online Learning Loops for Edge AI Inference},
  author  = {Rojas Masgo, Percy and {Qomni AI}},
  year    = {2026},
  month   = {April},
  note    = {Preprint},
  url     = {https://github.com/condesi/qomni-planner-paper}
}
```
