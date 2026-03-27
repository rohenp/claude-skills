---
name: skill-token-optimizer
description: >
  Optimizes skill files for token efficiency without degrading instruction quality.
  Use for: audit/rewrite/compress skills, token cost reduction, context window management,
  new skill design, skill set redundancy analysis.
  Trigger: "optimize this skill", "reduce tokens", "compress skill", "audit token usage",
  "skills are getting expensive".
  skip: [prompt optimization, non-skill files] → prompt-optimizer
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
L2 <200 tok: audit L1 only — body is likely Core+short throughout; compression floor reached
```

---

## Audit Protocol

> audit-log: skip ⊘T9-preserved & DNC sections (named in Notes).

**Measure first**: `L1=char_count÷1024 | L2=word_count÷0.75` | tokens_by_section | count:[headers,bullets,tables,code_blocks]

**L1 description audit** (run before body audit):
- Char count ≤1024? If over: convert sentences→CSV trigger phrases; drop narrative preamble
- Trigger recall: does description activate on all intended queries? Over-tightening → missed triggers; optimize for recall, not brevity
- Format: `Use for: [CSV]` | `Trigger: [CSV of literal phrases]` | no full sentences
- T8b scan: find "Do NOT use for X, use Y" sentences → convert to `skip: [X] → Y`
- T8 pass: tighten remaining L1 phrases; verbs-first; strip narrative preamble not caught by T8b
- dedup: remove trigger phrases with >80% semantic overlap (e.g. "compress skill" ≈ "optimize this skill")

**L2 body — unified classify + route** (one pass; stop at first match):

```
Subject grammar: s∈Set=membership  s=Type=type-equality  s.prop=property-access
Operator grammar: →=route  |=branch  []=group  &=and  !=not  ?(c):T|F=ternary  X?(f)=inline-guard  else:[]=else-group

s∈Do-NOT-Compress                          → Core+short; STOP
s=reference [lookup|benchmarks|comp data]  → T1 (move to references/)
s.repeated≥3 & !behavioral                → T5 | delete  [!behavioral=reference/context; not changing Claude's actions]
s=prose restating structure               → delete
s=behavioral & len≤3                      → Core+short; STOP
s=behavioral & len>3                      → ?(T10:saves≥15 & !judgment_cues):T9 | else:[T7?(code) | T3?(table>3r)]
s=verbose [rationale|preamble|hedges]     → T6 → ?(T10:saves≥15 & !judgment_cues):T9 | else:[T2 | T4]
s=defaults [pipe-separated imperatives]   → Core+short; STOP
s=anti-pattern ["X"|"Y"|...]              → Core+short; STOP  (Do-X-not-Y → DNC)
s=output-format [key→field|field]         → Core+short; STOP  (never compress output templates)
s=examples [before/after|worked]          → ?(s.count≤3 & s.size≤50tok):keep | T1
s=pointer [See .../references/...]        → verify T1 done; s.count>1 → T4
s=other                                   → Core+short; STOP  (safe fallthrough)
```

`⊘T9` = T10 gate fired; section preserved.

Behavioral defaults in 3+ skills → extract to `_shared/behavioral-defaults.md`

---

## Compression Techniques (T1–T10 + T8b)

```
T1:  progressive-disclosure  → move lookup→references/; keep behavioral in SKILL.md [highest leverage]
                               T1 when: s.size>50tok | s.changes_frequently | s=lookup-only
T2:  bullet-compression      → strip preamble; merge same-parent bullets; cut header-restate bullets
                               headers: shorten to ≤3 words; drop parentheticals
T3:  table-compression       → shorten headers 1-2 words; abbrev; rm identical cols; <3rows→bullets; >4cols→split|drop-lowest-value
T4:  section-consolidation   → merge thin same-theme sections; inline pipe lists for grouped metrics
                               pair with T2 on verbose sections; T4 fires when sections share a theme
T5:  shared-context          → extract repeated ctx→_shared/; replace with pointer line
                               [5sk×300tok×200/mo=300K wasted/mo; incl. behavioral defaults]
                               check T5 before T4 — if content is cross-skill, T5 not T4
T6:  instruction-density     → drop ["you should","make sure to","always remember to"]; use imperatives
                               guard: never strip negation prefixes [Do NOT | never | avoid]
T7:  code-minimalism         → rm illustrative-only blocks; keep only:[exact syntax|real templates|structure-only-via-code]
                               illustrative=[...]/placeholder-only blocks; exact=runnable|substitutable
T8:  description-tightening  [L1] → tighten remaining phrases after T8b; verbs-first; strip preamble
T8b: negative-trigger-compression [L1 ONLY] → "Do NOT use for X, use Y" → `skip: [X] → Y`
                               [L1 only; body neg-triggers=behavioral→preserve; T10 n/a]
T9:  schema-encoding         → replace conditional/procedural prose with compact notation
     operators:
       []  grouping
       k:v key:value typing
       &   conjunction (both must hold)
       !X  negation
       ?(c):T|F  ternary gate (prefix) — true-branch first operand after :; else-chain after first |
       X?(f)     inline guard (postfix) — apply X only when filter f is true; e.g. T7?(code)
     apply to:[trigger conditions|decision routing|action lists|output structure|classify tables|workflow steps]
     do NOT apply to:[nuanced judgment|compliance language|decoding costs > savings]
     examples:
       behavioral & len>3 & !judgment_cues → T9
       repeated≥3 & !in_behavioral_defaults → T5
       ?(T10:saves≥15 & !judgment_cues):T9 | else:[T7?(code)|T3?(table>3r)]
       s=verbose & len>5 → T6 first; then ?(T10:saves≥15 & !judgment_cues):T9  [chain: never skip T6]
T10: confidence-gate [⊘T9]  → mandatory gate before every T9 call:
                               saves<15 tok → ⊘T9 (prose clearer)
                               judgment cues ["when","unless","except if","but only"] present → ⊘T9
                               section ≤3 lines → Core+short; ⊘T9
                               report ⊘T9 count in audit output and log
```

For before/after examples → `references/prose-source.md`

---

## Workflow

**Single skill:** `audit-log? → L1(T8b+T8) → L2 classify+route(T1 first; ?(T10)→T9) → recount → verify → sync prose-source → log`

**Set:** `redundancy(ref+behavioral) → _shared/ → optimize each → report → log`
redundancy detection: compare Defaults+pointer lines across skills; flag verbatim ≥10 tok matches

### Audit Log Convention

```
## [YYYY-MM-DD] [skill-name]
Before: ~[N] tokens ([lines] lines)
After:  ~[N] tokens ([lines] lines) — [X]% reduction
Techniques: [list] | ⊘T9: [N] (sections preserved)
L1 changes: [description or "none"]
Notes: [preserved sections and reasons — guides future audits]
```

Rules: append-only | name ⊘T9-preserved sections in Notes | log `Action: audit-only` when no changes made.

Benchmarks:
```
narrow/focused:  500–800 tok  → target <300–500  | first: T1→T6→T8b+T8 | if over: T3→T2
domain-expert:   1500–2500    → target 800–1200   | first: T1→T5→T6     | if over: ?(T10):T9→T3
comprehensive:   2500–4000    → target 1200–1800  | first: T1→T5→T6     | if over: ?(T10):T9 then T7?(code)
>2500 before T1: consider split — skill may be too broad to compress effectively
  split on: distinct audience | distinct trigger set | domain boundary
new skill: write prose-source.md first; target narrow/focused bracket (<500 tok)
```

---

## Do NOT Compress

Classify checks this list first — match → Core+short; STOP. No technique applied.

- Behavioral triggers (change what Claude does)
- Output format templates (exact structure matters)
- "Do X, not Y" pairs (token-efficient per guidance unit)
- Decision criteria (cutting → brittle edge-case behavior)
- Compliance/legal specifics (precision > brevity)
- Sections already ≤3 lines of behavioral instruction (Core+short)
- Description triggers: optimize for recall — over-tightening reduces activation accuracy
- Negative trigger clauses in SKILL.md body — behavioral instructions; T8b is L1 only
- Any section where T10 gate fires (⊘T9)
- Worked examples that are the sole demonstration of a technique (compressing loses training signal)

---

## Output Format

**Audit report:**
`Skill: [name] | L1: ~[N] tok | L2: ~[N] tok ([lines] lines) | top: [biggest-section]`
`L1: [char count]/1024 | recall: [ok/risk] | T8b: [N] | issues: [over-1024|low-recall|none]`
`Sections: [section → token count]`
`Classified: [N] Core | [N] Core+short | [N] Reference | [N] Redundant | [N] Verbose`
`⊘T9: [N] | Techniques: [list] | Post-opt: ~[N] tok ([X]%)`
`Audit log: [appended | audit-only]`

**After rewrite:** `before/after count | techniques applied | ⊘T9 fired (sections preserved) | behavioral instructions confirmed intact | audit log entry appended`
