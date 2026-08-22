# MoE Range Before Representation

A behavior-gated skill for extreme post-training quantization of routed mixture-of-experts models.

## Best for

- ~2-bit expert-weight targets;
- deciding whether a custom representation is actually necessary;
- endpoint-vs-clipped range selection experiments;
- binary/ternary/four-level expert studies;
- locating damage in experts versus embeddings/heads/non-expert tensors;
- importance-aware upstream GGUF quantization;
- deployment residency and KV-cache planning.

## Main workflow

1. Establish the teacher-correct behavioral denominator.
2. Keep simulated quantization and packed-artifact evidence separate.
3. Hold budget/grouping/layer specification constant while changing one range-selection decision.
4. Test reconstruction-level count directly rather than relying on total bpw.
5. Localize functional damage by tensor family.
6. Allocate precision according to measured sensitivity.
7. Try stock/upstream-supported importance-aware formats before inventing a private codec when behavior supports that route.
8. Measure the exact packed release separately.
9. Report file residency and realistic KV/workspace headroom.
10. Keep all checkpoint-specific thresholds local to the experiment.

See [`SKILL.md`](./SKILL.md) for the complete experimental protocol and reporting templates.
