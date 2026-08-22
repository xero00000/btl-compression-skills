---
name: behavior-gated-compression
description: Failure-driven workflow for extreme post-training LLM compression where free-running deployed behavior, exact packed-prefix execution, and physical byte accounting decide what ships. Use when compressing a model aggressively, debugging low-bit behavioral regressions, allocating precision islands, fitting correction adapters, or validating an exact packed artifact.
---

# Behavior-Gated LLM Compression

Use this skill when the objective is not merely to minimize reconstruction error or perplexity, but to preserve a model's **deployed behavioral contract** under an aggressive storage budget.

This workflow is derived from the BTL-3 "Behavior Before Perplexity" procedure. Treat its exact numeric settings as a proven preset for that checkpoint, not as universal constants.

## Core principle

**Behavior promotes candidates; proxy metrics only diagnose them.**

Perplexity, KL divergence, teacher-forced token agreement, local tensor error, and reconstruction loss are useful for triage and localization. They are not release gates. A candidate advances only after the exact packed prefix or final artifact reproduces the required free-running behavior.

## Use this skill when

- targeting extreme PTQ or mixed-precision compression;
- a low-bit model looks numerically healthy but fails tool use, formatting, stopping, factual contracts, or other deployment behaviors;
- deciding which tensors or modules deserve more precision;
- calibrating later layers after earlier layers have already been quantized;
- introducing low-rank correction on a compressed graph;
- quantizing serialization-sensitive embeddings or output heads;
- proving that a final GGUF or other packed artifact has no dense fallback;
- comparing candidate recipes under a strict byte ceiling.

## Required inputs

Before changing weights, establish:

1. frozen base-model revision;
2. frozen adapter/checkpoint checksum if applicable;
3. tokenizer and chat/tool template revision;
4. runtime/backend revision;
5. target artifact byte ceiling;
6. calibration set;
7. validation/development behavior gate;
8. sealed release gate that is not used to make engineering choices;
9. deterministic evaluator for the behaviors that matter;
10. artifact manifest and checksum strategy.

If any of these are unknown, record the uncertainty before experimentation.

## Non-negotiable data discipline

Keep calibration, validation/development, and release evaluation separated by more than row order.

Exclude overlap by every identifier available, including:

- item ID;
- source ID;
- content hash;
- tool/function name;
- prompt family or template family.

Balance the behavior families that matter in deployment. Do not allow a category-sorted calibration file to make early prefixes appear stronger than they are.

## Workflow

### 1. Freeze the source and the budget

Record immutable source hashes and the physical byte ceiling before tuning.

Do not silently move the budget after seeing results. If the budget changes, start a new experiment identity.

### 2. Calibrate on the state the packed model will really see

For layer `l`:

1. run calibration samples through layers `0..l-1` using the **already packed prefix**;
2. capture the actual layer input `X_l`;
3. accumulate the input second moment in high precision;
4. fit the current layer against those packed-prefix activations;
5. install the packed layer;
6. replay and capture the next layer from the new real prefix.

Do not calibrate every layer from full-precision hidden states once the deployed graph will contain upstream quantization error.

The BTL-3 executed path used an FP64 full second moment:

`H_l = (1/N) X_l^T X_l`

Centered covariance can be recorded as a diagnostic, but it is not interchangeable with the executed objective unless explicitly tested.

### 3. Fit a low-bit representation with explicit curvature/state information

For the BTL-3 preset:

- block width: 128;
- vector size: 4 weights;
- seeded randomized Hadamard transforms on input and output block axes;
- affine 4x4 lattice code;
- block-LDLQ assignment with propagated quantization error;
- scale candidates: `0.8`, `1.0`, `1.2` times initial scale;
- ten coordinate-refinement passes;
- exact 2-bit packed codes plus counted metadata.

Use these numbers only as a reproducible starting preset. For a new architecture, ablate them rather than assuming transfer.

### 4. Treat interacting modules as functional units

Do not assume every matrix can be optimized independently.

For a gated MLP, evaluate the interacting computation as a unit:

`y = W_down ( SiLU(W_gate x) * W_up x )`

If a behavioral failure localizes to an MLP, test joint treatment of `up_proj`, `gate_proj`, and `down_proj` before concluding that one matrix alone is responsible.

Reject extra recovery stages when they improve a proxy metric but worsen emitted behavior.

### 5. Gate short packed prefixes

Do not wait for the full model to fail.

At regular prefix checkpoints:

1. run the free-running behavior gate;
2. compare against the last healthy prefix;
3. identify the first prefix where a behavior flips;
4. use numerical metrics only to rank likely suspects.

The first behavioral flip is the start of debugging, not an invitation to globally increase precision.

### 6. Behavioral bisection and module override

At the first failing prefix:

1. replay the identical failing cases;
2. restore one module to reference precision;
3. if needed, restore a tightly interacting module group;
4. accept an override only if the same emitted behavior is restored;
5. measure its physical byte cost;
6. choose the smallest proven intervention;
7. install it;
8. replay the prefix;
9. capture fresh downstream activations.

Never promote a tensor solely because it has high reconstruction error.

### 7. Build an empirical precision map

Precision allocation is an observed map of behavioral cliffs, not a blanket rule such as:

- "attention must be INT4";
- "MLPs can always be 2-bit";
- "early layers need more precision";
- "the head must stay BF16."

Record every precision island with:

- layer/module/tensor;
- previous precision;
- promoted precision;
- bytes added;
- exact behavior restored;
- gate item IDs affected;
- before/after proxy metrics for diagnosis only.

### 8. Repair only on the deployed compressed graph

If the frozen compressed model still shows systematic residual failure, train bounded correction **against the exact compressed forward graph**.

For the BTL-3 preset, the final behavior correction was:

- rank-8 LoRA;
- alpha 16;
- no dropout;
- AdamW betas `(0.9, 0.95)`;
- zero weight decay;
- gradient norm 1.0;
- sequence length 1024;
- 100 optimizer steps in saved state;
- assistant-token-only labels;
- restricted to measured late-layer module families.

Do not merge a correction into low-bit codes unless that merged representation is separately validated. Runtime composition is acceptable when it is part of the declared artifact.

### 9. Handle embeddings and output heads as serialization-sensitive components

If exact delimiters, punctuation, function names, argument keys, stop tokens, or structural vocabulary matter, evaluate them directly.

Possible interventions include:

- row-addressable quantization;
- a fixed rescue budget for structural/frequent embedding rows;
- a low-rank residual fitted to output-head error;
- measured higher precision for only the rows/tensors that restore behavior.

Do not leave the entire vocabulary stack at high precision by reflex. Do not quantize it blindly either.

### 10. Solve the physical byte ledger

Count the complete shipping artifact, not only the low-bit decoder core.

Include:

- quantized tensor payloads;
- precision islands;
- embedding/head payloads;
- adapters and low-rank corrections;
- scales;
- codebooks;
- compact state;
- metadata;
- alignment/padding;
- any runtime-required auxiliary tensors.

When over budget, demote only components for which lower precision has already been measured. Re-open the byte ledger, not the entire recipe.

### 11. Export and prove the exact artifact

The exporter/loader validation should:

- verify every expected source tensor;
- verify every emitted payload byte range;
- reject unsupported representation gaps;
- assert no compatible dense matrix survives as fallback;
- install all required corrections;
- validate unpacked/decoded graph parity where applicable;
- compute an immutable artifact checksum;
- run the exact runtime/backend intended for release.

A mechanically valid file is necessary but not sufficient.

### 12. Open a fresh sealed release gate

Only after representation, corrections, rescued rows, and byte allocation are frozen:

1. open the release gate;
2. run the full-precision teacher/reference;
3. identify teacher-correct, completed items;
4. run the exact packed artifact;
5. report conditional retention on teacher-correct items;
6. report every behavior family separately;
7. report malformed outputs, non-stops, over-abstention, and other failure modes explicitly.

Do not tune on the sealed result. If you change the model after seeing it, the gate is no longer sealed and a new release gate is required.

## Acceptance gates

A release candidate should not ship until all applicable gates pass:

### Mechanical gate

- artifact loads in the declared runtime;
- checksums and byte counts match;
- no undeclared dense fallback;
- all tensors have a declared representation;
- all required adapters/corrections are attached.

### Numerical diagnostic gate

- reconstruction/KL/token metrics are recorded;
- no unexplained numerical explosion;
- these metrics are marked diagnostic, not final quality claims.

### Behavioral gate

- exact free-running artifact is evaluated;
- teacher/reference misses are separated from retention denominator;
- category-level results are visible;
- stopping and schema validity are measured where relevant.

### Budget gate

- final physical file size is measured from disk;
- every non-core byte is accounted for;
- comparison uses the same decimal/GiB convention throughout.

## Failure modes to avoid

- Promoting candidates on perplexity alone.
- Evaluating dequantized fake weights and calling that an artifact result.
- Calibrating later layers only on pristine full-precision hidden states.
- Assigning precision from architecture folklore instead of measured recovery.
- Training repair on the teacher graph instead of the compressed graph.
- Using development-gate retention as the release claim.
- Hiding a weak behavior category inside an aggregate score.
- Counting only decoder payload bytes when quoting final size.
- Leaving a dense fallback path that makes the compact artifact look functional.
- Treating one checkpoint's precision map as universal.

## Experiment record

For each candidate, store at minimum:

```text
experiment_id:
source_revision:
source_checksum:
runtime_revision:
representation:
bit_budget_target:
physical_artifact_bytes:
calibration_split_hash:
development_gate_hash:
release_gate_hash:
proxy_metrics:
prefix_gate_results:
precision_overrides:
correction_payloads:
teacher_correct_count:
artifact_retained_count:
category_breakdown:
malformed_or_nonstop_count:
artifact_sha256:
notes:
```

## Reporting language

Prefer narrow, falsifiable claims.

Say:

- "conditional retention on teacher-correct gate items";
- "measured on this checkpoint and this runtime";
- "development gate" versus "sealed release gate";
- "physical artifact bytes";
- "proxy metric" versus "free-running behavior."

Avoid converting a contract-retention percentage into a claim about general intelligence, coding ability, or universal model quality.

## Source-derived BTL-3 preset summary

The source procedure's defining sequence is:

**freeze -> split -> replay packed prefix -> collect second moments -> fit low-bit blocks -> install -> short-prefix behavior gate -> bisect first behavioral failure -> measured precision promotion -> fresh downstream capture -> freeze decoder -> train bounded compressed-graph correction -> quantize vocabulary/head with measured rescue -> solve exact byte ledger -> export/byte-verify -> open a new sealed release gate.**

Preserve this order when reproducing that specific method. For new architectures, preserve the experimental discipline even when the representation changes.
