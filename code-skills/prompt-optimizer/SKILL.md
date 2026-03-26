---
name: prompt-optimizer
description: >
  Optimizes one-shot task prompts, system prompts, and multi-turn conversation starters for
  token efficiency and output quality. Use for: reducing prompt length without losing intent,
  improving output consistency, fixing underspecified prompts, structuring system prompts for
  cache efficiency, converting verbose instructions into compact high-signal prompts.
  Trigger: "optimize this prompt", "my prompt is too long", "Claude keeps misunderstanding X",
  "help me write a system prompt", "make this prompt cheaper", "prompt for my app".
  Do NOT use for skill file optimization — use skill-token-optimizer instead.
---

# Prompt Optimizer

Maximize Claude output quality per input token. Prompts are often one-shot, user-written, and suffer from over-explanation rather than structural bloat.

---

## Prompt Failure Taxonomy

Diagnose before optimizing:

| Failure | Symptom | Fix |
|---|---|---|
| Under-constrained | Output varies wildly across runs | Add output format + exemplar |
| Over-explained | Long prompt, mediocre output | Strip rationale; add constraints |
| Role confusion | Generic, timid responses | Strong role declaration + anti-patterns |
| Context overload | Claude loses the thread | Move context to references; keep prompt lean |
| Format underspecified | Inconsistent structure | Specify format with a template |
| Implicit audience | Mismatch in depth/tone | State the reader explicitly |
| Missing negative space | Claude hedges where you want conviction | Add "do not X" constraints |

---

## One-Shot Prompt Architecture

```
[ROLE] You are [specific role with domain + posture].
[TASK] [Verb] [object] [constraints].
[CONTEXT] [Only what changes the output. Nothing else.]
[FORMAT] [Exact structure. Include exemplar for non-obvious formats.]
[CONSTRAINTS] Do not [anti-pattern]. Do not [hedge behavior to suppress].
```

**Cut**: Explanation of why you want output | Background that doesn't change the answer | Politeness ("please", "could you") | Redundant restatements | "Think step by step" (add only if chain-of-thought genuinely helps).

**Keep**: Any constraint that would change the output if removed | Output format if non-obvious | Negative constraints that prevent the most common failure mode.

---

## System Prompt Architecture

System prompts load on every turn — token cost compounds. Structure for cache efficiency and behavioral precision.

### Layering (stable → variable)
```
Layer 1 (cache): Role + domain + persona + output format defaults
Layer 2 (cache): Standing constraints + anti-patterns + tone rules
Layer 3 (don't cache): Session context + user preferences + live data
```

Put layers 1–2 first, always. Anthropic prompt caching means identical prefixes cost ~10% of normal tokens on cache hits. Reordering can cut effective system prompt cost 40–60% on high-traffic apps.

### System Prompt Compression
- **Role**: One sentence. Domain + posture. No backstory.
- **Standing constraints**: Bullets, not paragraphs. Imperatives only.
- **Output format**: One exemplar > three paragraphs of description.
- **Anti-patterns**: "Do not X" is ~3 tokens. A paragraph explaining why X is bad is ~40 tokens. Cut the explanation.

### System Prompt Template
```
You are [role] for [product/context]. [One sentence on posture/goal].

Rules:
- [Constraint 1]
- [Constraint 2]
- Do not [anti-pattern]

Output format:
[Exemplar or schema]
```

---

## Semantic Density Techniques

**Concept pointers**: Replace explained concepts with named ones Claude knows. "Structure so categories don't overlap and everything is covered" → "Apply MECE structure." ~20 tokens saved per instance.

**Exemplar over description**: One concrete example of desired output > 100 words describing it.

**Constraint inversion**: Describe the failure mode instead of the desired quality. "Write with conviction and make clear recommendations" → "Do not hedge or present options without a recommendation."

**Output anchoring**: Specify the first word or phrase. "Start with the recommendation, not the analysis." Dramatically reduces preamble.

**Persona loading**: Strong role declaration activates implicit behaviors. "Senior SaaS CFO advising a PE-owned portfolio company" loads financial rigor, PE awareness, exit lens, directness. Explicit instructions overlapping the loaded persona are redundant tokens.

---

## Multi-Turn Conversation Design

- **System prompt**: Role + standing constraints + output format. Stable. Cache-optimized. No session context.
- **First user turn**: Task framing + session context. Keep lean.
- **Subsequent turns**: Just new information or instruction. Trust conversation history.

**Anti-pattern**: Restating the system prompt in every user turn. Defeats caching and bloats context.

---

## Prompt Quality Metrics

| Metric | How to measure |
|---|---|
| Token count | word_count ÷ 0.75 |
| Output consistency | Run 3×; how much does output vary? |
| Format compliance | Output match specified format on first try? |
| Failure rate | % runs requiring follow-up correction |
| Instruction-to-constraint ratio | >30% constraints → probably over-specified; cut rationales |

Target: output consistency >90% across 3 runs with no follow-up required.

---

## Defaults
Diagnose failure mode before rewriting | strip rationale before adding constraints | always specify output format | use exemplar for non-obvious formats | cache-order system prompts | test 3× before declaring done.

## Output Formats
- **Audit**: current prompt → failure taxonomy → token count → classified waste → rewrite plan
- **Optimized prompt**: new prompt + token delta + techniques applied + behavioral changes
- **System prompt**: layered structure with cache boundary marked + effective cost estimate
- **A/B comparison**: original vs. optimized side-by-side + predicted improvement per metric
