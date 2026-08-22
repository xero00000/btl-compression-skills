# BTL Compression Skills

Reusable agent skills distilled from two Bad Theory Labs compression studies:

- **Behavior Before Perplexity** — failure-driven compression of a 27B agentic model, where free-running deployed behavior rather than proxy metrics decides what ships.
- **Range Before Representation** — behavior-gated ultra-low-bit MoE quantization, emphasizing range selection, reconstruction-level cliffs, expert sensitivity, and exact artifact validation.

These are **operational skills**, not paper summaries. They are intended for coding/research agents that need a repeatable procedure for aggressive LLM compression experiments.

## Skills

| Skill | Use it for |
|---|---|
| [`behavior-gated-compression`](./behavior-gated-compression/) | Packed-prefix calibration, behavioral bisection, measured precision islands, compressed-graph repair, vocabulary/head handling, exact byte accounting, and sealed release gates. |
| [`moe-range-before-representation`](./moe-range-before-representation/) | MoE range-selection ablations, reconstruction-level testing, expert sensitivity, importance-aware stock-format packing, residency analysis, and exact artifact validation. |
| [`behavior-gated-compression-router`](./behavior-gated-compression-router/) | Choosing between the two workflows or coordinating them in a combined compression project. |

## Core idea

> Behavior promotes candidates; proxy metrics diagnose them.

Perplexity, KL divergence, token agreement, and reconstruction error are useful for triage and localization, but the final decision should come from free-running behavior of the exact packed artifact.

## Repository layout

```text
.
├── README.md
├── SOURCES.md
├── behavior-gated-compression/
│   ├── README.md
│   └── SKILL.md
├── moe-range-before-representation/
│   ├── README.md
│   └── SKILL.md
└── behavior-gated-compression-router/
    ├── README.md
    └── SKILL.md
```

## Usage

Copy any skill folder into the skills directory used by your agent/tooling, or point the agent directly at the corresponding `SKILL.md`.

For a new project, start with the router. It will route a dense/agentic compression problem toward the packed-prefix workflow, an MoE precision/range problem toward the MoE workflow, or combine both when needed.

## Scope and claims

The numerical settings and measured thresholds in these skills come from specific experiments. They should be treated as tested presets or hypotheses, **not universal laws**. The durable part is the experimental method: freeze inputs, separate evidence classes, change one causal variable at a time, gate on behavior, count physical bytes, and validate the exact release artifact.

## Sources

See [`SOURCES.md`](./SOURCES.md).
