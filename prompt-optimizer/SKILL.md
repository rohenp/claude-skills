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

Redirect to skill-token-optimizer if input contains any of: YAML frontmatter (`---` with `name:`/`description:`) | path ending in `SKILL.md` | references to `_shared/` or L1/L2/L3 architecture | behavioral framing ("when triggered, do X").

Redirect: *"This looks like a skill file — skill-token-optimizer is the right tool. Want me to use that instead?"* If ambiguous: *"Is this a skill file or a prompt you're using to call Claude?"*

---

## Diagnosis

Diagnose failure mode before rewriting. Failure taxonomy → `references/failure-taxonomy.md`

Common failure modes: under-constrained (output varies) | over-explained (long prompt, mediocre output) | role confusion (generic responses) | format underspecified (inconsistent structure) | missing negative space (Claude hedges).

---

## One-Shot Prompt Architecture

```
[ROLE] You are [specific role with domain + posture].
[TASK] [Verb] [object] [constraints].
[CONTEXT] [Only what changes the output. Nothing else.]
[FORMAT] [Exact structure. Exemplar for non-obvious formats.]
[CONSTRAINTS] Do not [anti-pattern]. Do not [hedge behavior to suppress].
```

**Cut**: rationale | background that doesn't change the answer | politeness | redundant restatements | "think step by step" (add only if CoT genuinely helps).
**Keep**: any constraint that changes output if removed | output format if non-obvious | negative constraints preventing the most common failure mode.

---

## System Prompt Architecture

Token cost compounds — structure for cache efficiency. Layers 1–2 first, always. Cache hits ~10% of normal tokens; reordering cuts effective cost 40–60% on high-traffic apps.

```
Layer 1 (cache): Role + domain + persona + output format defaults
Layer 2 (cache): Standing constraints + anti-patterns + tone rules
Layer 3 (no cache): Session context + user preferences + live data
```

**Rules**: Role → one sentence, domain + posture, no backstory | Constraints → bullets, imperatives only | Format → one exemplar > three paragraphs of description | Anti-patterns → "Do not X" (~3 tok) not an explanation paragraph (~40 tok).

**Multi-turn**: System prompt = stable role + constraints + format, no session context. First turn = task + session context. Subsequent turns = new info only — trust history. Anti-pattern: restating system prompt each turn defeats caching.

**Template**:
```
You are [role] for [product/context]. [One sentence posture/goal].
Rules:
- [Constraint]
- Do not [anti-pattern]
Output format: [Exemplar or schema]
```

---

## Semantic Density Techniques

- **Concept pointers**: Replace explained concepts with named ones Claude knows (e.g., "MECE structure" not "categories that don't overlap and cover everything")
- **Exemplar over description**: One concrete output example > 100 words describing it
- **Constraint inversion**: Describe the failure mode, not the desired quality ("Do not hedge" not "write with conviction")
- **Output anchoring**: Specify the first word/phrase to eliminate preamble
- **Persona loading**: Strong role declaration activates implicit behaviors — don't restate what the persona already loads

---

## Defaults
Detect skill file first — redirect if needed | diagnose failure mode before rewriting | strip rationale before adding constraints | always specify output format | exemplar for non-obvious formats | cache-order system prompts | test 3× before done.

## Output Formats
- **Audit**: current prompt → failure taxonomy → token count → classified waste → rewrite plan
- **Optimized prompt**: new prompt + token delta + techniques applied + behavioral changes
- **System prompt**: layered structure with cache boundary marked + effective cost estimate
- **A/B comparison**: original vs. optimized side-by-side + predicted improvement per metric
