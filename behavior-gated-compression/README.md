# Behavior-Gated Compression

A failure-driven skill for aggressive LLM compression where **free-running deployed behavior** decides which candidate advances.

## Best for

- dense or agentic models;
- custom low-bit representations;
- models that look healthy under perplexity/token metrics but fail real tasks;
- localizing cumulative damage across a packed network;
- allocating measured precision islands;
- training corrections on the final compressed graph;
- proving a final packed artifact has no hidden dense fallback.

## Main workflow

1. Freeze model, tokenizer, runtime, evaluator, and byte ceiling.
2. Keep calibration/development/release data disjoint.
3. Calibrate each layer from the state produced by the already-packed prefix.
4. Gate short prefixes in free-running mode.
5. When behavior flips, bisect by module or interacting module group.
6. Promote precision only when the intervention restores the same emitted behavior.
7. Re-capture downstream activations after every accepted intervention.
8. Freeze the decoder before correction training.
9. Treat embeddings/output heads according to measured serialization sensitivity.
10. Solve the complete physical byte ledger.
11. Export one exact artifact and reject fallback paths.
12. Open a fresh sealed gate and report every behavior family separately.

See [`SKILL.md`](./SKILL.md) for the complete procedure, templates, stop conditions, and reporting rules.
