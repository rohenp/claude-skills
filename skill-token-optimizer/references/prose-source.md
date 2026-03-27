# Skill Token Optimizer — Prose Source
> This is the human-readable reference version. The loaded SKILL.md uses schema-encoded compression (T9).
> Edit this file first when making changes, then re-encode SKILL.md from it.
> Last updated: 2026-03-26 v2 (encoding review: operator grammar formalized, inline guard X?(f), subject grammar, else:[] precedence, fallthrough row, T4/T8 routing, judgment-cues sync, benchmark consistency)

---

## Purpose

Maximize instruction clarity per token. Bloated skills are a recurring tax on every inference call.

---

## Token Architecture

Three levels — optimize each separately:

- **Level 1: Description (frontmatter)** — always in context; ≤1024 chars; optimize for trigger accuracy
- **Level 2: SKILL.md body** — loaded when triggered; target <500 lines; highest leverage
- **Level 3: references/** — on-demand only; unlimited; never auto-loaded

## Packaging Convention (prose-source pattern)

Every skill stores its human-readable source in `references/prose-source.md`. This is the canonical edit target — written in plain prose, no compression. The `SKILL.md` body is derived from it using the compression techniques below.

**Workflow**: edit `prose-source.md` first → re-encode `SKILL.md`. Never edit `SKILL.md` directly except for minor behavioral fixes; always backport changes to the prose source.

Shared reference files (lookup data, competitor lists, PE context, architecture principles) live in `_shared/references/` and are loaded on-demand. Non-trivial shared files also carry a `prose-source.md` as their edit target.

---

## Audit Protocol

### Check audit-log first

Before classifying any section, read `references/audit-log.md`. Sections that were explicitly preserved by T10 gate or Do-NOT-Compress are named in the Notes field of past entries. Do not re-classify these sections — treat them as already assigned Core+short and skip them.

This prevents repeated work and keeps T10 gate decisions stable across runs.

### L1 Description Audit (run this first)

L1 is always loaded — it burns tokens on every call, not just when the skill triggers. Audit it before the body.

1. **Char count**: Is it ≤1024 characters? If over: convert narrative sentences to CSV trigger phrases; drop preamble.
2. **Trigger recall**: Does the description activate on all intended queries? Over-tightening hurts recall. Optimize for accuracy, not brevity.
3. **Format check**: Should follow `Use for: [CSV list]` | `Trigger: [CSV of literal phrases]` — no full sentences needed.
4. **Negative trigger audit (T8b)**: Scan for "Do NOT use for X, use Y instead" sentences. Convert to `skip: [X] → Y` notation. These are L1-only — negative triggers in the body are behavioral instructions and must NOT be compressed.
5. **Description tightening (T8)**: After T8b, tighten remaining L1 phrases. Pack as comma-separated lists; verbs-first ordering; strip any narrative preamble not caught by T8b. Optimize for recall, not completeness.

### L2 Body Audit — Unified Classify + Route

Previously, classification and routing were two separate steps with two separate code blocks. They are now merged into a single-pass decision table. Each row is a match condition; stop at the first match. This eliminates the context-switch between classification and routing and saves ~15 tokens in the SKILL.md body.

**Unified table (one pass; stop at first match):**

```
Subject grammar: s∈Set=membership  s=Type=type-equality  s.prop=property-access
Operator grammar: →=route  |=branch  []=group  &=and  !=not  ?(c):T|F=ternary  X?(f)=inline-guard  else:[]=else-group

s∈Do-NOT-Compress                          → Core+short; STOP
s=reference [lookup|benchmarks|comp data]  → T1 (move to references/)
s.repeated≥3 & !behavioral                → T5 | delete
s=prose restating structure               → delete
s=behavioral & len≤3                      → Core+short; STOP
s=behavioral & len>3                      → ?(T10:saves≥15 & !judgment_cues):T9 | else:[T7?(code) | T3?(table>3r)]
s=verbose [rationale|preamble|hedges]     → T6 → ?(T10:saves≥15 & !judgment_cues):T9 | else:[T2 | T4]
s=other                                   → Core+short; STOP  (safe fallthrough)
```

The `?(condition):true | false` ternary notation is explained in the T9 section below.

Flag behavioral defaults that appear verbatim (or near-verbatim) in 3 or more skills → candidate for `_shared/behavioral-defaults.md`.

---

## Compression Techniques

**T1. Progressive Disclosure** — highest leverage. Move lookup data to `references/` and replace with a pointer. Keep behavioral guidance in SKILL.md; move factual lookup data out.

**T2. Bullet Compression** — strip preamble ("Are X doing Y?", "This is important because..."); merge bullets sharing a parent concept; cut bullets that restate their header.

**T3. Table Compression** — shorten headers to 1–2 words; use abbreviations; remove identical-value columns; replace star ratings with numbers; <3-row tables → bullets.

**T4. Section Consolidation** — merge thin sections sharing a behavioral theme; use inline pipe-separated lists for grouped metrics. Fires as a companion to T2 on verbose sections; triggers when two or more thin sections share a theme and merging loses no behavioral content.

**T5. Shared Context Files** — extract repeated context (operating mandate, company background) to `_shared/`. Each skill replaces with a single pointer line. 5 skills × 300 shared tokens × 200 loads/month = 300K wasted tokens/month.

This also applies to **behavioral defaults**: if a Defaults section bullet appears in 3+ skills (e.g. "flag intersections with the other domain immediately"), extract to `_shared/behavioral-defaults.md`. Each skill replaces with a pointer + any skill-specific overrides.

**T6. Instruction Density** — drop "you should", "make sure to", "always remember to". Use imperative fragments. Before (~40 tokens): "When analyzing product questions, you should always make sure to clearly identify who the buyer is..." After (~12 tokens): "Always distinguish buyer vs. user."

**T7. Code Block Minimalism** — remove illustrative-only blocks; replace with inline examples or cut. Keep code blocks only when: exact syntax matters, it's a real template, or structure can't be conveyed inline.

**T8. Description Tightening** *(L1)* — runs after T8b on remaining L1 phrases. Pack as comma-separated lists; verbs-first; strip narrative preamble not caught by T8b. Optimize for recall, not completeness.

**T8b. Negative Trigger Compression** *(L1 only)* — convert "Do NOT use for X, use Y instead" sentences in the description (frontmatter) to compact `skip: [X] → Y` notation. This technique applies *only* to L1 descriptions — negative trigger clauses in the SKILL.md body are behavioral instructions and must NOT be compressed. T10 gate does not apply to T8b; savings are structural, not judgment-dependent.

**T9. Schema Encoding** — replace conditional/procedural prose with compact notation.

**Complete operator grammar (formal reference):**

| Symbol | Name | Meaning |
|---|---|---|
| `→` | route | if X, then Y |
| `|` | branch | alternative / or |
| `[]` | group | explicit grouping |
| `k:v` | key:value | typed property |
| `&` | conjunction | both must hold |
| `!X` | negation | must not hold |
| `?(c):T|F` | ternary gate (prefix) | true-branch = first operand after `:`; else-chain starts after first `|` |
| `X?(f)` | inline guard (postfix) | apply X only when filter f holds; no defined false-branch; skip on filter miss |
| `else:[]` | else-group | explicitly groups false-branch alternatives in a ternary; prevents ambiguity with post-ternary `|` chains |

**Operator list format:** use `sym: purpose` keyed rows. Never list operators inside `[...|...]` with `|` as separator — `|` is itself an operator and creates a parsing collision.

**Subject grammar for classify table rows:**

| Notation | Meaning | Example |
|---|---|---|
| `s∈Set` | set membership | `s∈Do-NOT-Compress` |
| `s=Type` | type equality | `s=reference`, `s=behavioral` |
| `s.prop` | property access | `s.repeated≥3` |

Use `∈` for named lists/sets. Use `=` for type categories. Use `.prop` for measurable attributes.

**Ternary precedence rule:** in `?(c):T | else:[A|B]`, the true-branch is everything between `:` and the first top-level `|`. Mark false-branches with `else:[]` when there are multiple alternatives. Without `else:[]`, a chain of `|` after the true-branch is ambiguous.

**Inline guard `X?(f)`:** postfix, distinct from prefix ternary. Applies technique X only when section type matches filter. No false-branch — if filter fails, term is skipped and next `|` branch evaluated. Use for type-restricted techniques: `T7?(code)`, `T3?(table>3r)`.

*Apply to:* trigger conditions, decision routing, action lists, output structure.
*Do not apply to:* nuanced judgment calls, compliance language, or any instruction where decoding the schema costs more tokens than were saved.

**T10. Confidence Gate** — a final check applied before every T9 call, protecting sections from over-compression. Before encoding a Core section with T9, ask:
- Would this save fewer than 15 tokens? → Skip T9; prose is clearer.
- Does the section contain judgment cues ("when", "unless", "except if", "but only")? → Skip T9; schema notation loses the nuance.
- Is the section already 3 lines or fewer? → Assign Core+short; skip all compression.

T10 is not an optional polish step — it is a mandatory gate. Every T9 application requires a T10 check first.

**⊘T9 shorthand**: When T10 fires and a section is preserved, record this as `⊘T9` in the audit log Techniques field rather than writing "T10 gate fired" in full. This applies in the SKILL.md audit output format and in the audit-log.md entry. The count of ⊘T9 events replaces the old "T10 gates fired: N" phrasing.

---

## Technique Selection and Priority

The unified classify+route table handles routing. Priority ordering within each bracket:

| Skill size | Typical tokens | Target | First techniques | If still over |
|---|---|---|---|---|
| Narrow/focused | 500–800 | <300–500 | T1 → T6 → T8b | T3 → T2 |
| Domain-expert | 1,500–2,500 | 800–1,200 | T1 → T5 → T6 | ?(T10):T9 → T3 |
| Comprehensive | 2,500–4,000 | 1,200–1,800 | T1 → T5 → T6 | ?(T10):T9 then T7?(code) |

**Global ordering**: T1 first — removes the most tokens without any compression risk. Then T8b on L1. Then ?(T10) gate before every T9 call. Then T6 on verbose sections. Then T2/T3/T4 structural tightening.

Run L1 description audit (including T8b) before the body — L1 fires on every call.

---

## Optimization Workflow

**Single skill**: `check audit-log → L1 audit (T8b scan) → L2 unified classify+route (T1 first, ?(T10) before T9) → recount → verify behavioral instructions intact → package → append entry to references/audit-log.md`

**Skill set**: Audit all for redundancy (both reference content AND behavioral defaults) → create `_shared/` entries → optimize individually → report total reduction → append all entries to audit logs

### Audit Log Convention

Every optimization run appends one entry to `references/audit-log.md` in the affected skill's folder. This prevents re-classifying already-audited sections on future runs and provides a change history.

**Entry format:**
```
## [YYYY-MM-DD] [skill-name]
Before: ~[N] tokens ([lines] lines)
After:  ~[N] tokens ([lines] lines) — [X]% reduction
Techniques: [T1, T6, ...] | ⊘T9: [N] (sections preserved)
L1 changes: [description or "none"]
Notes: [anything worth knowing for future audits]
```

**Rules:**
- Append only — never edit or delete prior entries.
- If a section was explicitly preserved (⊘T9 or Do-NOT-Compress), name it in Notes so future runs skip re-classifying it.
- If no optimization was performed (audit only, no changes), log as `Action: audit-only`.

---

## Do NOT Compress

These patterns are exempt from all compression techniques. The classify step must check this list first and assign `Core+short` — no technique is applied, no T9 encoding, no bullet merging, no prose shortening.

- **Behavioral triggers** — instructions that change what Claude does
- **Output format templates** — preserve exact structure when it matters
- **"Do X, not Y" pairs** — token-efficient per unit of behavioral guidance
- **Decision criteria** — cutting creates brittle behavior on novel cases
- **Compliance/legal specifics** — precision matters; do not paraphrase
- **Sections already ≤3 lines of behavioral instruction** — assigned Core+short immediately
- **Description triggers** — optimize for trigger recall; over-tightening descriptions reduces skill activation accuracy
- **Negative trigger clauses in SKILL.md body** — these are behavioral instructions; T8b applies to L1 only
- **Any section where ⊘T9 fires** — saves < 15 tokens, judgment cues present, or section already minimal

---

## Output Format

**Audit report:**
```
Skill: [name] | Current: ~[N] tokens ([lines] lines)
L1: [char count]/1024 | trigger recall: [ok/risk] | T8b: [N negative triggers found/compressed] | issues: [list or none]
Sections: [section → token count]
Classified: [N] Core | [N] Core+short | [N] Reference | [N] Redundant | [N] Verbose
⊘T9: [N sections preserved] | Techniques: [list] | Estimated post-opt: ~[N] tokens ([X]% reduction)
Audit log: [appended | skipped — audit only]
```

**After rewrite:** before/after count | techniques applied | ⊘T9 fired (sections preserved, with reasons) | behavioral instructions confirmed intact | audit log entry appended

---

## Before/After Examples

### T6 — Instruction Density

**Before** (~40 tokens):
> When analyzing product questions, you should always make sure to clearly identify who the buyer is, as this is different from the user of the product, and it matters for the recommendation.

**After** (~12 tokens):
> Always distinguish buyer vs. user vs. champion.

---

### T8b — Negative Trigger Compression (L1 description only)

**Before** (~22 tokens, in description frontmatter):
> Do NOT use for skill file optimization — use skill-token-optimizer instead.

**After** (~8 tokens):
> `skip: [skill files] → skill-token-optimizer`

---

### T9 — Schema Encoding

**Before** (~35 tokens):
> If the section contains only reference material like benchmarks or tables of historical data, you should move it to the references folder and replace it with a pointer line in the SKILL.md.

**After** (~12 tokens):
> `Reference → T1 (move to references/)`

---

### T9 extended — Conjunction & negation operators

**Before** (~28 tokens):
> Apply T9 only when the section is behavioral and longer than three lines, but only if it does not contain any judgment cues.

**After** (~10 tokens):
> `behavioral & len>3 & !judgment_cues → T9`

---

### T9 extended — Ternary gate operator ?(cond):T|F

**Before** (~38 tokens):
> If the T10 gate clears — meaning the compression would save at least 15 tokens and there are no judgment cues — apply T9. Otherwise, if it's a code block, apply T7. If it's a table with more than 3 rows, apply T3.

**After** (~18 tokens):
> `?(T10:saves≥15 & !judgment_cues):T9 | else:[T7?(code) | T3?(table>3r)]`

The `else:[]` grouping makes clear that `T7?(code)` and `T3?(table>3r)` are both false-branch alternatives, not post-ternary routing steps. Without `else:[]`, a reader cannot tell where the ternary ends and sequential `|` branching begins.

The ternary gate is most useful when both true and false branches have non-trivial actions. When the false-branch is just "skip", plain `→` routing is cleaner.

---

### T10 — Confidence Gate firing (⊘T9 — section preserved)

**Section under review** (3 lines, behavioral):
> Diagnose the real constraint before prescribing. Stated constraints are often not the actual constraint — the real one is usually one layer beneath. Name it explicitly before recommending.

**T10 check:**
- Savings if T9 applied: ~8 tokens (below 15-token threshold)
- Contains judgment cues: "often", "usually", "one layer beneath"
- Section is 3 lines → Core+short

**Result**: ⊘T9. Section preserved as-is. Logged as `⊘T9: 1 (judgment cues + minimal savings)`.

---

### T10 — Confidence Gate not firing (T9 proceeds)

**Section under review** (decision routing, 6 lines):
> If the section is reference-only, move it to references/. If it is redundant across skills, extract it to _shared/. If it is verbose prose, apply T6 first. If it is a core behavioral section that is already short, preserve it.

**T10 check:**
- Savings if T9 applied: ~28 tokens (above threshold)
- No judgment cues — pure routing logic
- Section is 6 lines (not minimal)

**Result**: T10 gate does not fire. T9 applied:
```
Reference → T1 | Redundant → T5 | Verbose → T6 | Core+short → preserve
```

---

### Unified Classify+Route — single-pass example

**Before** (two separate code blocks, ~45 tokens across both):
```
classify(section):
  IF matches Do-NOT-Compress list → Core+short; STOP
  IF reference-only → Reference → T1
  IF behavioral, already ≤3 lines → Core+short; STOP
  IF behavioral, >3 lines → Core+long → route to technique selection

classify(section) → technique:
  Reference    → T1
  Core+long    → [T10 gate] T9 if decision-tree | T7 if code block
  Core+short   → STOP
```

**After** (one unified table, ~30 tokens):
```
Subject grammar: s∈Set=membership  s=Type=type-equality  s.prop=property-access

s∈Do-NOT-Compress      → Core+short; STOP
s=reference            → T1
s=behavioral & len≤3   → Core+short; STOP
s=behavioral & len>3   → ?(T10:saves≥15 & !judgment_cues):T9 | else:[T7?(code) | T3?(table>3r)]
s=other                → Core+short; STOP
```

---

### Audit Log Example (with ⊘T9 shorthand)

**`references/audit-log.md` entry:**
```
## 2026-03-26 engineering-strategy
Before: ~1070 tokens (106 lines)
After:  ~820 tokens (84 lines) — 23% reduction
Techniques: T6, T2 | ⊘T9: 3 (velocity-metrics table, DD simulation checklist, defaults)
L1 changes: none
Notes: Velocity metrics table is Core+short — do not compress. DD simulation checklist contains judgment cues ("comfortable with"), ⊘T9. Defaults bullets all ≤3 lines, preserve.
```
