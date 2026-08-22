---
name: moe-range-before-representation
description: Behavior-gated workflow for ultra-low-bit mixture-of-experts quantization. Use to compare range-selection methods at fixed storage, test reconstruction-level cliffs, localize damage to expert versus non-expert tensors, build importance-aware stock-format releases, and validate exact MoE artifacts.
---

# MoE Range Before Representation

Use this skill for aggressive post-training compression of routed mixture-of-experts language models, especially when the target is around two bits per weight and the goal is a practical artifact rather than a novel codec.

This workflow is derived from the BTL-4 "Range Before Representation" study. Its measured thresholds apply to that checkpoint and tested configurations. Do not universalize them without fresh experiments.

## Core principle

At extreme precision, **the usable reconstruction range and the number of representable expert-weight levels may matter more than the nominal representation family or total bits-per-weight.**

The shipping decision is still made by free-running behavior of the exact artifact.

## Use this skill when

- quantizing a large MoE to approximately 2-bit expert weights;
- deciding whether a custom codec is necessary;
- choosing endpoint scaling versus clipping/importance-aware range allocation;
- testing binary, ternary, or four-level expert representations;
- deciding whether to protect embeddings/output head or expert tensors;
- building IQ2_XXS or another upstream-supported GGUF path with an importance matrix;
- estimating residency and KV-cache headroom for local deployment;
- separating fake-quantization evidence from packed-release evidence.

## Required inputs

Collect:

1. frozen MoE checkpoint and architecture description;
2. number of layers;
3. expert count per layer;
4. experts routed per token;
5. active parameters per token;
6. full parameter count;
7. exact target file-size or bpw budget;
8. calibration corpus;
9. behavior gate with category labels;
10. exact runtime and quantizer revisions;
11. packing format and tensor-type support matrix.

Also record whether optional towers or prediction heads are included. File-size comparisons are meaningless if artifact scope differs.

## Evidence classes

Keep these labels separate in every notebook, table, and claim:

### Simulated/fake quantization

Weights are quantized and immediately dequantized in memory before generation. This isolates quantization choices but does **not** prove that a packed artifact behaves the same way.

### Packed artifact

The exact file intended for deployment is loaded through its real runtime and evaluated free-running.

Never report simulated retention as if it were packed-release retention.

## Workflow

### 1. Establish the teacher-correct behavior denominator

Run the reference model first.

For every gate item, record:

- category;
- whether the teacher is correct;
- whether the teacher finishes;
- expected behavior;
- exact scoring rule.

Compute retention only over teacher-correct, completed items. Also publish raw counts so a small denominator cannot hide sensitivity to a few items.

### 2. Hold storage constant and ablate one quantization decision at a time

When comparing quantization methods, keep constant as many variables as possible:

- nominal bits per weight;
- group size;
- layer/tensor specification;
- non-expert precision;
- calibration/evaluation data;
- generation settings.

Change one decision. Then run the same free-running gate.

If multiple factors change, label the configuration as confounded and do not attribute the difference to one factor.

### 3. Compare endpoint range selection with error-minimizing clipping

A simple endpoint quantizer maps the group's extrema to available codes. At very low bit width, one outlier can stretch the interval and force most values onto too few useful codes.

A clipping search instead evaluates candidate intervals and chooses the one with the lowest per-group weight error.

Operational test:

1. fix group size and code count;
2. generate candidate clip intervals;
3. fake-quantize/dequantize each group;
4. measure per-group MSE;
5. select the best interval;
6. run the exact same free-running behavior gate;
7. compare both overall retention and behavior-family retention.

Do not infer behavioral superiority solely from lower weight MSE; the behavior gate remains decisive.

### 4. Test reconstruction-level count explicitly

Do not use total bpw as a proxy for usable expert precision.

Construct controlled candidates that vary expert reconstruction levels, for example:

- four-level / nominal 2-bit experts;
- three-level / ternary experts;
- two-level / binary experts.

Measure behavior at each level count.

If a higher-bpw binary build loses far more behavior than a lower-bpw four-level build, record a **level-count cliff**. Do not describe it as a universal lower bound; describe it as a checkpoint- and route-specific cliff.

### 5. Localize where quantization damage lives

Test precision protection hypotheses directly.

At minimum compare interventions such as:

- quantized head + embedding, reference experts;
- reference head + embedding, quantized experts;
- selected higher-bit non-expert matrices;
- full-precision router and normalization tensors;
- alternative expert group sizes or importance weighting.

Use the behavior gate to decide where precision buys back function.

For MoE models, pay special attention to experts because they can dominate total parameters and routing determines which expert knowledge a token consults. But treat that as a hypothesis to measure, not a universal law.

### 6. Allocate precision according to measured sensitivity

Build a tensor-class budget table:

```text
class                 representation     nominal_bpw     bytes     behavior evidence
experts               ...                ...             ...       ...
attention/non-expert  ...                ...             ...       ...
router                 ...                ...             ...       ...
normalization          ...                ...             ...       ...
embedding              ...                ...             ...       ...
output head            ...                ...             ...       ...
```

Do not import dense-model "precision islands" unchanged into a routed model.

### 7. Prefer an upstream-supported artifact when it preserves the target behavior

If the controlled study suggests that range/importance allocation is the dominant factor, test whether an upstream quantization path can capture enough of that benefit.

A stock-format route can be preferable when it provides:

- existing runtime support;
- no private packed codec;
- no custom decode operator;
- simpler distribution;
- lower maintenance burden.

For a llama.cpp-style path, test an importance-matrix build using a calibration corpus representative of deployment. Record the corpus composition and number of chunks.

Do not claim equivalence between a custom simulated quantizer and an upstream packed quantizer just because their retention numbers are close.

### 8. Measure the exact release independently

After packing:

1. record exact file bytes from disk;
2. compute whole-artifact bpw using the declared model scope;
3. load the exact file in the intended runtime;
4. run the same gate;
5. compare packed-artifact results to teacher-correct items;
6. compare against the simulation only as a separate experiment.

A close result supports the deployment hypothesis. It does not prove the quantizers are mathematically equivalent.

### 9. Report artifact scope and compatibility precisely

For each release state:

- text-only versus multimodal;
- optional heads enabled/disabled;
- exact architecture name expected by runtime;
- required tensor types;
- minimum tested runtime revision;
- whether the format is upstream/standard versus custom;
- exact file bytes and GiB;
- checksum.

"Standard GGUF" means the representation is standard. It does not guarantee that every historical runtime can load the architecture or IQ types.

### 10. Estimate residency separately from arithmetic activity

MoE active parameters per token and file residency describe different resources.

Record:

- total resident weight bytes;
- active parameters per token;
- runtime workspace;
- KV-cache bytes per token for the selected cache type;
- context-window target;
- safety headroom.

A compact MoE can have low arithmetic per token while still requiring all quantized expert weights to remain resident.

Use measured runtime memory to set usable context. Do not assume the architectural maximum context fits simply because the weights fit.

## Acceptance gates

### Controlled-study gate

- one variable changed at a time, or confounds explicitly labeled;
- simulated quantization identified as simulation;
- group size, bpw, level count, and non-expert precision recorded;
- behavior results include raw counts and categories.

### Sensitivity gate

- head/embedding protection tested rather than assumed;
- expert damage measured directly;
- router/norm precision declared;
- precision allocation justified by observed functional sensitivity.

### Artifact gate

- exact file bytes measured;
- exact runtime build recorded;
- exact packed artifact evaluated;
- no simulated result substituted for packed result;
- checksum and compatibility requirements recorded.

### Claim gate

- no universal sub-two-bit claim from one checkpoint;
- no claim of quantizer novelty when using existing methods;
- no claim of equivalence between simulated and release quantizers;
- no claim that aggregate retention equals general model capability.

## Failure modes to avoid

- Comparing two range-selection methods while also changing group size or non-expert precision.
- Treating bpw alone as a quality predictor.
- Assuming binary/ternary degradation will be smooth.
- Protecting embedding/head because a prior dense model needed it.
- Ignoring expert tensors because only a few experts are active per token.
- Calling a fake-quantization result a release result.
- Shipping a custom codec before testing whether upstream importance-aware quantization is sufficient.
- Quoting maximum architectural context without KV-cache and workspace accounting.
- Using community popularity as scientific evidence.

## Suggested experiment matrix

Start with a compact matrix that isolates the important variables:

```text
ID  expert levels  expert group  range method        non-expert  simulated?  packed?  artifact bytes  retention
A   4              fixed         endpoint min/max    fixed       yes         no       n/a             ...
B   4              same          MSE clip search     same        yes         no       n/a             ...
C   3              same          best measured       same        yes         no       n/a             ...
D   2              same          best measured       same        yes         no       n/a             ...
E   4              release path  importance-aware    release     no          yes      ...             ...
```

Add further rows only when they answer a specific causal question.

## Behavior gate design

A small first-party gate can be useful when it measures deployment contracts, but disclose its limits.

Useful categories include:

- short-form factual recall;
- grounded extraction from supplied text;
- false-premise rejection;
- task-specific routing/tool contracts when relevant.

For every category publish:

- total prompts;
- teacher-correct count;
- artifact-retained count;
- rate;
- failure IDs or failure taxonomy.

## Reporting language

Prefer:

- "four reconstruction levels retained behavior on this checkpoint";
- "binary experts collapsed in the tested post-training configurations";
- "range-selection ablation at fixed budget";
- "exact release uses upstream tensor types plus an importance matrix";
- "simulation and release are separate experiments";
- "precision was allocated according to measured functional sensitivity."

Avoid:

- "two bits always works";
- "binary MoE is impossible";
- "heads never matter";
- "importance matrices are equivalent to MSE clipping";
- "standard GGUF runs everywhere."

## Source-derived BTL-4 preset summary

The source study's decision chain is:

**define a teacher-correct behavior gate -> hold budget constant -> compare endpoint versus MSE-selected group ranges -> explicitly vary reconstruction-level count -> test where damage lives -> reject inherited dense-model precision rules when evidence disagrees -> test an upstream importance-aware packing route -> measure the exact packed artifact separately -> report residency/KV requirements -> keep simulation and release claims separate.**

Preserve that causal discipline even if the model, quantizer, and exact bit allocation differ.
