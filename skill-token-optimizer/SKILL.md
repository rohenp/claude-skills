---
name: skill-token-optimizer
description: >
  Optimizes skill files for token efficiency without degrading instruction quality.
  Use for: audit/rewrite/compress skills, token cost reduction, context window management,
  new skill design, skill set redundancy analysis. Trigger: "optimize this skill",
  "reduce tokens", "compress skill", "audit token usage", "skills are getting expensive".
---

# Skill Token Optimizer
> Prose source: `references/prose-source.md` — edit there first, then re-encode here.

Maximize instruction clarity per token.

## Token Architecture
```
L1: description/frontmatter → always loaded | ≤1024 chars | optimize for trigger accuracy
L2: SKILL.md body          → loaded on trigger | target <500 lines | highest leverage
L3: references/            → on-demand only | unlimited | never auto-loaded
packaging: prose→references/prose-source.md | compressed→SKILL.md | shared-data→_shared/references/
```

---

## Audit Protocol

**Measure first**: `total_tokens=(word_count÷0.75)` | tokens_by_section | count:[headers,bullets,tables,code_blocks]

**L1 description audit** (run before body audit):
- Char count ≤1024? If over: convert sentences→CSV trigger phrases; drop narrative preamble
- Trigger recall: does description activate on all intended queries? Over-tightening → missed triggers; optimize for recall, not brevity
- Format: `Use for: [CSV]` | `Trigger: [CSV of literal phrases]` | no full sentences

**L2 body classification** — classify each section first, then route to technique:

```
classify(section):
  IF matches Do-NOT-Compress list (see below) → Core+short; STOP — apply no technique
  IF reference-only (lookup tables, benchmarks, competitor data) → Reference → T1
  IF appears verbatim/near-verbatim in 3+ skills → Redundant → T5 or delete
  IF prose restating structure → Redundant → delete
  IF behavioral, already ≤3 lines → Core+short; STOP
  IF behavioral, >3 lines → Core+long → route to technique selection
  IF verbose prose (rationale, preamble, hedges) → Verbose → T6 first
```

Behavioral defaults in 3+ skills → extract to `_shared/behavioral-defaults.md`

---

## Compression Techniques (T1–T10)

```
T1:  progressive-disclosure  → move lookup→references/; keep behavioral in SKILL.md [highest leverage]
T2:  bullet-compression      → strip preamble; merge same-parent bullets; cut header-restate bullets
T3:  table-compression       → shorten headers 1-2 words; abbrev; rm identical cols; <3rows→bullets
T4:  section-consolidation   → merge thin same-theme sections; inline pipe lists for grouped metrics
T5:  shared-context          → extract repeated ctx→_shared/; replace with pointer line
                               (e.g. 5skills × 300tok × 200loads/mo = 300K wasted tok/mo)
                               also applies to behavioral defaults repeated verbatim across skills
T6:  instruction-density     → drop ["you should","make sure to","always remember to"]; use imperatives
T7:  code-minimalism         → rm illustrative-only blocks; keep only:[exact syntax|real templates|structure-only-via-code]
T8:  description-tightening  → triggers as CSV not sentences; verbs-first; optimize for recall not completeness
T9:  schema-encoding         → replace conditional/procedural prose with compact notation
                               notation:[→routing | |branching | [grouping] | key:value typing]
                               apply to:[trigger conditions|decision routing|action lists|output structure]
                               do NOT apply to:[nuanced judgment|compliance language|decoding costs > savings]
T10: confidence-gate         → before applying T9 to any Core section, check:
                               savings < 15 tokens → skip T9; prose is clearer
                               section contains judgment cues ("when","unless","except if") → skip T9
                               section already ≤3 lines → Core+short; skip all compression
                               apply T10 as final gate before every T9 call
```

For before/after examples → `references/prose-source.md`

---

## Technique Selection Routing

```
classify(section) → technique:
  Reference    → T1 (move to references/)
  Redundant    → delete | or T5 if shared value across skills
  Verbose      → T6 first | then [T10 gate] T9 if decision-tree shape | then T2/T4
  Core+long    → [T10 gate] T9 if decision-tree | T7 if code block | T3 if table >3 rows
  Core+short   → STOP — preserve; compression risk > savings
```

**Ordering**: T1 before all others (removes most tokens, zero risk) → T10 gate before every T9 call → T6 on remaining verbose sections → T2/T3/T4 for structural tightening.

Run L1 description audit first — L1 burns tokens on every call, not just when skill triggers.

---

## Workflow

**Single skill:** `L1 audit → L2 classify (apply Do-NOT-Compress check first) → route techniques (T1 first, T10 gate before T9) → recount → verify behavioral intact → package`

**Skill set:** `audit all for redundancy (reference + behavioral) → create _shared/ entries → optimize individually → report total reduction`

Benchmarks:
```
narrow/focused:  500–800 tok  → target <300–500
domain-expert:   1500–2500    → target 800–1200
comprehensive:   2500–4000    → target 1200–1800
```

---

## Do NOT Compress

These patterns are exempt from all compression techniques. The classify step must check this list first and assign Core+short — no technique is applied.

- Behavioral triggers (change what Claude does)
- Output format templates (exact structure matters)
- "Do X, not Y" pairs (token-efficient per guidance unit)
- Decision criteria (cutting → brittle edge-case behavior)
- Compliance/legal specifics (precision > brevity)
- Sections already ≤3 lines of behavioral instruction (Core+short)
- Description triggers: optimize for recall — over-tightening reduces activation accuracy
- Any section where T10 gate fires (savings < 15 tok, judgment cues present, or already minimal)

---

## Output Format

**Audit report:**
`Skill: [name] | Current: ~[N] tokens ([lines] lines)`
`L1: [char count]/1024 | trigger recall: [ok/risk] | issues: [list or none]`
`Sections: [section → token count]`
`Classified: [N] Core | [N] Core+short | [N] Reference | [N] Redundant | [N] Verbose`
`T10 gates fired: [N sections skipped] | Techniques: [list] | Estimated post-opt: ~[N] tokens ([X]% reduction)`

**After rewrite:** `before/after count | techniques applied | T10 gates fired (sections preserved) | behavioral instructions confirmed intact`
