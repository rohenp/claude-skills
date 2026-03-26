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
- Trigger recall: does description activate on all intended queries? Over-tightening descriptions → missed triggers; optimize for recall, not brevity
- Format: `Use for: [CSV]` | `Trigger: [CSV of literal phrases]` | no full sentences

**L2 body classification** — flag → action:
- reference-only content (lookup tables, benchmarks) → move to `references/`
- cross-skill repetition → move to `_shared/`
- prose restating structure → delete
- behavioral defaults appearing in 3+ skills → extract to `_shared/behavioral-defaults.md`

Classify each section: `Core`(behavioral→keep) | `Reference`(lookup→move) | `Redundant`(delete) | `Verbose`(compress) | `Core+short`(already efficient→preserve)

---

## Compression Techniques (T1–T9)

```
T1: progressive-disclosure  → move lookup→references/; keep behavioral in SKILL.md [highest leverage]
T2: bullet-compression      → strip preamble; merge same-parent bullets; cut header-restate bullets
T3: table-compression       → shorten headers 1-2 words; abbrev; rm identical cols; <3rows→bullets
T4: section-consolidation   → merge thin same-theme sections; inline pipe lists for grouped metrics
T5: shared-context          → extract repeated ctx→_shared/; replace with pointer line
                              (e.g. 5skills × 300tok × 200loads/mo = 300K wasted tok/mo)
                              also applies to behavioral defaults repeated verbatim across skills
T6: instruction-density     → drop ["you should","make sure to","always remember to"]; use imperatives
T7: code-minimalism         → rm illustrative-only blocks; keep only:[exact syntax|real templates|structure-only-via-code]
T8: description-tightening  → triggers as CSV not sentences; verbs-first; optimize for recall not completeness
T9: schema-encoding         → replace conditional/procedural prose with compact notation
                              notation:[→routing | |branching | [grouping] | key:value typing]
                              apply to:[trigger conditions|decision routing|action lists|output structure]
                              do NOT apply to:[nuanced judgment|compliance language|decoding costs > savings]
```

For before/after examples → `references/prose-source.md`

---

## Technique Selection Routing

```
classify(section) → technique:
  Reference    → T1 (move to references/)
  Redundant    → delete
  Verbose      → T6 first | then T9 if decision-tree shape | then T2/T4
  Core+long    → T9 if decision-tree | T7 if code block | T3 if table >3 rows
  Core+short   → preserve (already efficient; compression risk > savings)
```

Apply T1 before all others — removes most tokens without compression risk.
Run L1 description audit first — L1 burns tokens on every call, not just when skill triggers.

---

## Workflow

**Single skill:** `L1 audit → L2 audit → classify → apply(T1 first) → recount → verify behavioral intact → package`

**Skill set:** `audit all for redundancy (reference + behavioral) → create _shared/ → optimize individually → report total reduction`

Benchmarks:
```
narrow/focused:  500–800 tok  → target <300–500
domain-expert:   1500–2500    → target 800–1200
comprehensive:   2500–4000    → target 1200–1800
```

---

## Do NOT Compress
- Behavioral triggers (change what Claude does)
- Output format templates (exact structure matters)
- "Do X, not Y" pairs (token-efficient per guidance unit)
- Decision criteria (cutting → brittle edge-case behavior)
- Compliance/legal specifics (precision > brevity)
- `Core+short` sections (already efficient; over-compression degrades quality)
- Description triggers: optimize for trigger recall — over-tightening reduces skill activation accuracy

---

## Output Format

**Audit report:**
`Skill: [name] | Current: ~[N] tokens ([lines] lines)`
`L1: [char count]/1024 | trigger recall: [ok/risk] | issues: [list or none]`
`Sections: [section → token count]`
`Classified: [N] Core | [N] Core+short | [N] Reference | [N] Redundant | [N] Verbose`
`Techniques: [list] | Estimated post-optimization: ~[N] tokens ([X]% reduction)`

**After rewrite:** `before/after count | techniques applied | content that couldn't compress | behavioral instructions confirmed intact`
