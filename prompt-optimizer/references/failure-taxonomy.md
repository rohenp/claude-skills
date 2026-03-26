# Prompt Failure Taxonomy

| Failure | Symptom | Fix |
|---|---|---|
| Under-constrained | Output varies wildly across runs | Add output format + exemplar |
| Over-explained | Long prompt, mediocre output | Strip rationale; add constraints |
| Role confusion | Generic, timid responses | Strong role declaration + anti-patterns |
| Context overload | Claude loses the thread | Move context to references; keep prompt lean |
| Format underspecified | Inconsistent structure | Specify format with template |
| Implicit audience | Mismatch in depth/tone | State the reader explicitly |
| Missing negative space | Claude hedges where you want conviction | Add "do not X" constraints |

## Quality Metrics

| Metric | How to measure |
|---|---|
| Token count | word_count ÷ 0.75 |
| Output consistency | Run 3×; variance? |
| Format compliance | Match on first try? |
| Failure rate | % runs needing follow-up |
| Instruction-to-constraint ratio | >30% constraints → probably over-specified; cut rationales |

Target: >90% consistency across 3 runs, no follow-up required.
