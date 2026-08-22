---
name: behavior-gated-compression-router
description: Chooses and coordinates behavior-gated LLM compression workflows for dense/agentic versus mixture-of-experts models. Use at the start of a compression project to define evidence classes, gates, experiments, artifact accounting, and whether to invoke the packed-prefix or MoE range-selection workflow.
---

# Behavior-Gated Compression Router

Use this skill at the beginning of an aggressive LLM compression project or when an existing project has become a collection of unrelated quantization experiments.

It coordinates two companion skills:

- `behavior-gated-compression` - packed-prefix calibration, behavioral bisection, precision islands, compressed-graph repair, vocabulary/head rescue, exact byte ledger, sealed artifact gate;
- `moe-range-before-representation` - fixed-budget range ablations, reconstruction-level cliffs, expert sensitivity, importance-aware stock-format packing, residency analysis.

## First decision: identify the architecture and failure mode

### Route A - packed-prefix behavioral debugging

Prefer `behavior-gated-compression` when:

- the model is dense or expert routing is not the dominant compression issue;
- you are building a custom low-bit representation;
- behavioral failures emerge only after a sequence of packed layers;
- you need module-level precision islands;
- you need low-rank repair on the final compressed graph;
- tool syntax, stopping, or vocabulary serialization is fragile.

### Route B - MoE range/level study

Prefer `moe-range-before-representation` when:

- expert tensors dominate parameter storage;
- only a subset of experts is active per token but all weights must remain resident;
- the target is around two bits per expert weight;
- binary/ternary/four-level choices are under consideration;
- you need to determine whether range selection matters more than codec novelty;
- an upstream IQ/importance-matrix path may avoid a custom runtime.

### Route C - combined workflow

Use both when an MoE also needs behavioral-prefix localization, module rescue, or correction training.

Do not assume that the dense/agentic model's precision map transfers to MoE. Do not assume that a successful MoE range choice eliminates downstream behavioral cliffs.

## Universal rules shared by both workflows

1. **Freeze the source.** Pin model, tokenizer, adapter, evaluator, runtime, and target budget.
2. **Separate data.** Keep calibration/development data distinct from a sealed release gate.
3. **Define behaviors before optimizing.** State exactly what a successful generation must do.
4. **Use proxy metrics for diagnosis, not promotion.** Perplexity, reconstruction error, KL, and token agreement never replace free-running behavior.
5. **Separate simulation from packing.** Fake quantization is causal evidence about a knob, not artifact evidence.
6. **Change one causal variable at a time.** If two variables change, mark the comparison confounded.
7. **Measure the exact artifact.** The shipping file, runtime path, and generation settings are part of the experiment.
8. **Count physical bytes.** Include metadata, alignment, scales, codebooks, adapters, corrections, and mixed-precision exceptions.
9. **Report teacher-conditional retention narrowly.** Do not turn a contract score into a universal capability percentage.
10. **Expose weak categories.** Never hide a collapsed behavior family inside an aggregate result.
11. **Prefer measured precision allocation.** Promote bytes only where they restore behavior.
12. **Treat one-checkpoint findings as local.** Convert them into hypotheses, not universal laws.

## Project bootstrap template

Create a project record before the first quantization run:

```yaml
project:
  name:
  architecture:
  parameter_count:
  active_parameters_per_token: null
  layers:
  experts_per_layer: null
  routed_experts_per_token: null

source:
  model_revision:
  model_checksum:
  adapter_revision: null
  adapter_checksum: null
  tokenizer_revision:
  runtime_revision:

artifact_scope:
  text_decoder: true
  vision_tower: null
  speculative_or_mtp_head: null
  target_bytes:
  target_bpw: null

behavior_contract:
  categories: []
  development_gate_hash:
  sealed_release_gate_hash:
  scoring_version:

calibration:
  corpus_hash:
  overlap_checks: [id, source_id, content_hash, family]

experiment_policy:
  simulation_and_release_separate: true
  one_variable_at_a_time: true
  physical_byte_accounting: true
  no_dense_fallback: true
```

## Experiment ladder

Run experiments in this order unless there is a specific reason not to.

### Stage 0 - Reference characterization

- run the teacher/reference on the development gate;
- identify teacher-correct and completed items;
- classify failures;
- record baseline memory, speed, and artifact size.

### Stage 1 - Cheap controlled probes

Use simulated quantization to answer narrow causal questions:

- range selection;
- group size;
- level count;
- tensor-class sensitivity;
- candidate representation families.

Do not spend days packing a representation before learning whether the underlying low-bit regime preserves the target behavior.

### Stage 2 - Architecture-specific optimization

For dense/agentic route:

- replay the real packed prefix;
- collect downstream state from that prefix;
- gate short prefixes;
- bisect behavioral flips;
- add only behavior-proven precision islands.

For MoE route:

- hold budget fixed;
- compare range methods;
- test reconstruction-level count explicitly;
- determine whether experts or non-experts dominate functional damage.

### Stage 3 - Minimal repair

If a stable residual failure remains:

- train bounded correction on the exact compressed graph;
- or allocate a measured rescue budget to sensitive tensors/rows;
- price the intervention in bytes.

Reject repair that improves proxy metrics without restoring target behavior.

### Stage 4 - Deployment representation decision

Ask:

- Can an upstream-supported format reproduce enough of the controlled-study behavior?
- Does a custom codec create meaningful behavioral or size benefit after counting runtime complexity?
- Is the custom representation required, or merely interesting?

Prefer the simplest runtime path that passes the behavior and budget gates.

### Stage 5 - Exact artifact proof

- export one immutable artifact;
- verify tensor coverage and byte ranges;
- reject fallback paths;
- checksum the file;
- measure disk bytes;
- test intended hardware/runtime combinations;
- measure workspace and KV-cache headroom.

### Stage 6 - Sealed release evaluation

Open the sealed gate only after the artifact is frozen.

Report:

- teacher-correct denominator;
- retained count;
- per-category results;
- malformed/non-stopping outputs;
- exact artifact bytes;
- runtime revision;
- any known compatibility constraints.

If the model changes, invalidate that release gate for future tuning.

## Decision rules

### If perplexity improves but behavior worsens

Keep the behavioral winner. Use perplexity only to diagnose where the two candidates differ.

### If token agreement is high but the model cannot execute the contract

Reject the candidate.

### If a high-error tensor override does not restore the failing behavior

Do not promote it solely because the error is high.

### If a lower-bpw candidate preserves more behavior than a higher-bpw candidate

Look at range allocation, representable level count, and tensor-class precision before concluding that the result is anomalous.

### If head/embedding protection helped a previous model

Retest it on the current architecture. Do not inherit it automatically.

### If simulated and packed results disagree

Trust the exact packed artifact for release decisions. Use the simulation only to revise the causal hypothesis.

### If a stock-format artifact is nearly as good as a custom codec

Include runtime/deployment complexity in the decision. A small score difference may not justify a private decoder path.

## Required result table

Every serious experiment should end with a row like:

```text
ID | evidence class | representation | expert levels | group size | range method | precision exceptions | correction bytes | artifact bytes | dev retention | release retention | notes
```

`evidence class` must be one of:

- `simulation`;
- `packed-prefix`;
- `packed-artifact`;
- `sealed-release`.

Never merge these classes in one unlabeled percentage.

## Stop conditions

Stop optimizing and prepare a release when:

- the exact artifact satisfies the declared byte ceiling;
- free-running behavior passes the predeclared overall threshold;
- category-level failures are understood and disclosed;
- no dense/reference fallback is active;
- the artifact is reproducible from pinned inputs;
- runtime and hardware requirements are known;
- further complexity provides less value than its maintenance/deployment cost.

Stop a route early when:

- it fails the behavior gate catastrophically despite acceptable proxy metrics;
- reconstruction-level experiments reveal a hard cliff incompatible with the target budget;
- a repair stage repeatedly improves proxies but not emitted behavior;
- the representation requires substantial runtime complexity with no meaningful artifact-level gain.

## Final release note template

```markdown
### Artifact
- Model scope:
- Exact bytes:
- Decimal GB / GiB:
- Whole-artifact bpw:
- SHA-256:
- Runtime revision:
- Required tensor/codec support:

### Behavior gate
- Gate size:
- Teacher-correct completed items:
- Artifact-retained items:
- Conditional retention:
- Category breakdown:
- Malformed/non-stop failures:

### Evidence boundaries
- Simulation results:
- Packed-prefix results:
- Exact artifact results:
- Sealed release results:

### Known limitations
- Architectures tested:
- Behaviors not tested:
- Context limits not validated:
- Hardware/runtime coverage:
- Confounded comparisons:

### Claim
One narrow sentence stating exactly what the artifact demonstrated, without converting contract retention into general capability.
```

## Philosophy

The reusable contribution is not one fixed quantizer.

It is the experimental loop:

**hold the budget and behavior contract steady -> manipulate one compression decision -> run the real behavior -> localize failure -> spend precision only where behavior returns -> count every byte -> validate the exact artifact -> make the narrowest claim the evidence supports.**
