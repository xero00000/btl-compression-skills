# Behavior-Gated Compression Router

Entry-point skill for choosing the right compression workflow.

## Routes

### Packed-prefix behavioral debugging

Use [`behavior-gated-compression`](../behavior-gated-compression/) when cumulative packed-layer error, tool/formatting behavior, module rescue, compressed-graph repair, or custom low-bit representations dominate the problem.

### MoE range and precision study

Use [`moe-range-before-representation`](../moe-range-before-representation/) when expert tensors dominate storage and the main questions are range selection, reconstruction levels, expert sensitivity, importance-aware packing, and residency.

### Combined

Use both for MoE projects that also develop cumulative behavioral failures requiring prefix localization, precision islands, or low-rank repair.

## Shared rules

- freeze the source and runtime;
- define behavior before optimizing;
- use proxy metrics for diagnosis, not promotion;
- distinguish simulation from packing;
- change one causal variable at a time;
- count physical bytes, not convenient theoretical bpw alone;
- validate the exact release artifact;
- expose weak behavior categories instead of hiding them in aggregates;
- treat single-checkpoint findings as local evidence.

See [`SKILL.md`](./SKILL.md) for the project bootstrap template, evidence ladder, route selection logic, and release checklist.
