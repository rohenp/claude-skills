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
> Prose source: `references/prose-source.md` — edit there first, then re-encode here.

Maximize Claude output quality per input token. Prompts suffer from over-explanation, not structural bloat.

---

## Skill File Detection

Before optimizing, check if the input is a skill file. Redirect to skill-token-optimizer if input contains any of:
- YAML frontmatter (`---` at top with `name:` or `description:` fields)
- Filename/path ending in `SKILL.md`
- References to `references/prose-source.md`, `_shared/`, or L1/L2/L3 architecture
- Behavioral framing: "when triggered, do X"

Redirect message: *"This looks like a skill file — skill-token-optimizer is the right tool. Want me to use that instead?"*

If ambiguous: ask *"Is this a skill file or a prompt you're using to call Claude?"*

---

## Prompt Failure Taxonomy

| Failure | Symptom | Fix |
|---|---|---|
| Under-constrained | Output varies wildly across runs | Add output format + exemplar |
| Over-explained | Long prompt, mediocre output | Strip rationale; add constraints |
| Role confusion | Generic, timid responses | Strong role declaration + anti-patterns |
| Context overload | Claude loses the thread | Move context to references; keep prompt lean |
| Format underspecified | Inconsistent structure | Specify format with template |
| Implicit audience | Mismatch in depth/tone | State the reader explicitly |
| Missing negative space | Claude hedges where you want conviction | Add "do not X" constraints |

---

## One-Shot Prompt Architecture

```
[ROLE] You are [specific role with domain + posture].
[TASK] [Verb] [object] [constraints].
[CONTEXT] [Only what changes the output. Nothing else.]
[FORMAT] [Exact structure. Exemplar for non-obvious formats.]
[CONSTRAINTS] Do not [anti-pattern]. Do not [hedge behavior to suppress].
```

**Cut**: rationale for why you want output | background that doesn't change answer | politeness | redundant restatements | "think step by step" (add only if chain-of-thought genuinely helps).

**Keep**: any constraint that changes output if removed | output format if non-obvious | negative constraints that prevent the most common failure mode.

---

## System Prompt Architecture

Token cost compounds — structure for cache efficiency.

```
Layer 1 (cache): Role + domain + persona + output format defaults
Layer 2 (cache): Standing constraints + anti-patterns + tone rules
Layer 3 (no cache): Session context + user preferences + live data
```

Layers 1–2 first, always. Cache hits cost ~10% of normal tokens — reordering can cut effective cost 40–60% on high-traffic apps.

**Compression rules**: Role → one sentence, domain + posture, no backstory | Constraints → bullets, imperatives only | Format → one exemplar > three paragraphs | Anti-patterns → "Do not X" (~3 tok) not a paragraph explaining why (~40 tok).

**Template**:
```
You are [role] for [product/context]. [One sentence on posture/goal].
Rules:
- [Constraint]
- Do not [anti-pattern]
Output format:
[Exemplar or schema]
```

---

## Semantic Density Techniques

**Concept pointers**: Replace explained concepts with named ones. "Structure so categories don't overlap and everything is covered" → "Apply MECE structure."

**Exemplar over description**: One concrete output example > 100 words describing it.

**Constraint inversion**: Describe the failure mode. "Write with conviction" → "Do not hedge or present options without a recommendation."

**Output anchoring**: "Start with the recommendation, not the analysis." Eliminates preamble.

**Persona loading**: Strong role declaration activates implicit behaviors — don't redundantly restate what the persona already loads.

---

## Multi-Turn Design
- System prompt: role + standing constraints + format. Stable. Cache-optimized. No session context.
- First user turn: task framing + session context. Lean.
- Subsequent turns: new info or instruction only. Trust history.

Anti-pattern: restating the system prompt in every user turn — defeats caching, bloats context.

---

## Prompt Quality Metrics
| Metric | How to measure |
|---|---|
| Token count | word_count ÷ 0.75 |
| Output consistency | Run 3×; variance? |
| Format compliance | Match on first try? |
| Failure rate | % runs needing follow-up |
| Instruction-to-constraint ratio | >30% constraints → probably over-specified |

Target: >90% consistency across 3 runs, no follow-up required.

---

## Defaults
Detect skill file first — redirect if needed | diagnose failure mode before rewriting | strip rationale before adding constraints | always specify output format | exemplar for non-obvious formats | cache-order system prompts | test 3× before done.

## Output Formats
- **Audit**: current prompt → failure taxonomy → token count → classified waste → rewrite plan
- **Optimized prompt**: new prompt + token delta + techniques applied + behavioral changes
- **System prompt**: layered structure with cache boundary marked + effective cost estimate
- **A/B comparison**: original vs. optimized side-by-side + predicted improvement per metric
