# Sources

These skills were derived from two Bad Theory Labs technical reports supplied with the project.

## Behavior Before Perplexity

**Behavior Before Perplexity: A failure-driven recipe for compressing a 27B agentic model to 8.39 GB**  
Bad Theory Labs, July 2026.

Key ideas carried into the skill:

- candidate promotion based on free-running deployed behavior rather than perplexity or reconstruction metrics;
- calibration using the hidden states produced by the already-packed prefix;
- behavioral bisection to localize the first failing layer/module;
- precision promotion only when the intervention restores emitted behavior;
- low-rank repair trained against the exact compressed graph;
- exact physical byte accounting and artifact-level validation;
- a sealed release gate separate from development decisions.

## Range Before Representation

**Range Before Representation: Behavior-gated two-bit quantization of a 35B mixture-of-experts model into a released 9.96 GB GGUF**  
Bad Theory Labs, August 2026.

Key ideas carried into the skill:

- fixed-budget range-selection ablations;
- per-group clipping/range selection as a causal quantization variable;
- explicit testing of reconstruction-level cliffs;
- measuring whether damage is concentrated in experts or non-expert tensors;
- separating fake-quantization evidence from packed-artifact evidence;
- use of upstream-supported quantization paths where they preserve behavior;
- runtime residency and KV-cache headroom analysis;
- behavior-gated release validation of the exact artifact.

## Important limitation

The papers' numerical results are checkpoint- and configuration-specific. The skills preserve those values only as examples or proven presets from the source experiments. They do not claim universal two-bit thresholds, universal tensor-sensitivity maps, or universal superiority of one quantization family.
