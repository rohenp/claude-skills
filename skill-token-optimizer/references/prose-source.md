# Skill Token Optimizer — Prose Source
> This is the human-readable reference version. The loaded SKILL.md uses schema-encoded compression (T9).
> Edit this file first when making changes, then re-encode SKILL.md from it.
> Last updated: 2026-03-26

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

### L1 Description Audit (run this first)

L1 is always loaded — it burns tokens on every call, not just when the skill triggers. Audit it before the body.

1. **Char count**: Is it ≤1024 characters? If over: convert narrative sentences to CSV trigger phrases; drop preamble.
2. **Trigger recall**: Does the description activate on all intended queries? Over-tightening hurts recall. Optimize for accuracy, not brevity.
3. **Format check**: Should follow `Use for: [CSV list]` | `Trigger: [CSV of literal phrases]` — no full sentences needed.

### L2 Body Audit — Closed-Loop Classification

The classification step is a closed loop: check the Do-NOT-Compress list first, before applying any technique. A section that matches is assigned `Core+short` immediately and receives no further processing.

**Classification order (apply in sequence, stop at first match):**

1. Does the section match the Do-NOT-Compress list? → **Core+short — STOP. Apply no technique.**
2. Is it reference-only content (lookup tables, competitor lists, benchmarks, historical data)? → **Reference** → T1 (move to `references/`)
3. Does the same text appear verbatim or near-verbatim in 3+ skills? → **Redundant** → T5 or delete
4. Is it prose that restates what the structure already implies? → **Redundant** → delete
5. Is it behavioral AND already 3 lines or fewer? → **Core+short — STOP. Apply no technique.**
6. Is it behavioral AND more than 3 lines? → **Core+long** → route to technique selection
7. Is it verbose prose (rationale, preamble, "you should" hedges)? → **Verbose** → T6 first

Flag behavioral defaults that appear verbatim (or near-verbatim) in 3 or more skills → candidate for `_shared/behavioral-defaults.md`.

---

## Compression Techniques

**T1. Progressive Disclosure** — highest leverage. Move lookup data to `references/` and replace with a pointer. Keep behavioral guidance in SKILL.md; move factual lookup data out.

**T2. Bullet Compression** — strip preamble ("Are X doing Y?", "This is important because..."); merge bullets sharing a parent concept; cut bullets that restate their header.

**T3. Table Compression** — shorten headers to 1–2 words; use abbreviations; remove identical-value columns; replace star ratings with numbers; <3-row tables → bullets.

**T4. Section Consolidation** — merge thin sections sharing a behavioral theme; use inline pipe-separated lists for grouped metrics.

**T5. Shared Context Files** — extract repeated context (operating mandate, company background) to `_shared/`. Each skill replaces with a single pointer line. 5 skills × 300 shared tokens × 200 loads/month = 300K wasted tokens/month.

This also applies to **behavioral defaults**: if a Defaults section bullet appears in 3+ skills (e.g. "flag intersections with the other domain immediately"), extract to `_shared/behavioral-defaults.md`. Each skill replaces with a pointer + any skill-specific overrides.

**T6. Instruction Density** — drop "you should", "make sure to", "always remember to". Use imperative fragments. Before (~40 tokens): "When analyzing product questions, you should always make sure to clearly identify who the buyer is..." After (~12 tokens): "Always distinguish buyer vs. user."

**T7. Code Block Minimalism** — remove illustrative-only blocks; replace with inline examples or cut. Keep code blocks only when: exact syntax matters, it's a real template, or structure can't be conveyed inline.

**T8. Description Tightening** — pack trigger phrases as comma-separated lists; verbs-first ordering; full sentences waste characters. Optimize for accurate triggering and recall, not completeness.

**T9. Schema Encoding** — replace conditional/procedural prose with compact notation. Use: arrow routing (`→`), pipe branching (`|`), bracket grouping (`[a,b,c]`), colon typing (`key:value`). Apply to: trigger conditions, decision routing, action lists, output structure. *Do not apply to*: nuanced judgment calls, compliance language, or any instruction where the schema would require more tokens to decode than it saves.

**T10. Confidence Gate** — a final check applied before every T9 call, protecting sections from over-compression. Before encoding a Core section with T9, ask:
- Would this save fewer than 15 tokens? → Skip T9; prose is clearer.
- Does the section contain judgment cues ("when", "unless", "except if", "but only")? → Skip T9; schema notation loses the nuance.
- Is the section already 3 lines or fewer? → Assign Core+short; skip all compression.

T10 is not an optional polish step — it is a mandatory gate. Every T9 application requires a T10 check first. Report how many sections T10 preserved in the audit output.

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

**Application order:**
1. T1 first — removes the most tokens without any compression risk.
2. T10 gate — check before every T9 call.
3. T6 on remaining verbose sections.
4. T2/T3/T4 for structural tightening.

Run L1 description audit before the body — L1 fires on every call.

---

## Optimization Workflow

**Single skill**: L1 audit → L2 classify (Do-NOT-Compress check first) → route techniques (T1 first, T10 gate before T9) → recount → verify behavioral instructions intact → package

**Skill set**: Audit all for redundancy (both reference content AND behavioral defaults) → create `_shared/` entries → optimize individually → report total reduction

### Benchmarks

| Skill type       | Typical tokens | Target tokens |
|------------------|----------------|---------------|
| Narrow/focused   | 500–800        | <300–500      |
| Domain expert    | 1,500–2,500    | 800–1,200     |
| Comprehensive    | 2,500–4,000    | 1,200–1,800   |

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
- **Any section where T10 gate fires** — savings < 15 tokens, judgment cues present, or section already minimal

---

## Output Format

**Audit report:**
```
Skill: [name] | Current: ~[N] tokens ([lines] lines)
L1: [char count]/1024 | trigger recall: [ok/risk] | issues: [list or none]
Sections: [section → token count]
Classified: [N] Core | [N] Core+short | [N] Reference | [N] Redundant | [N] Verbose
T10 gates fired: [N sections skipped] | Techniques: [list] | Estimated post-opt: ~[N] tokens ([X]% reduction)
```

**After rewrite:** before/after count | techniques applied | T10 gates fired (sections preserved, with reasons) | behavioral instructions confirmed intact

---

## Before/After Examples

### T6 — Instruction Density

**Before** (~40 tokens):
> When analyzing product questions, you should always make sure to clearly identify who the buyer is, as this is different from the user of the product, and it matters for the recommendation.

**After** (~12 tokens):
> Always distinguish buyer vs. user vs. champion.

---

### T9 — Schema Encoding

**Before** (~35 tokens):
> If the section contains only reference material like benchmarks or tables of historical data, you should move it to the references folder and replace it with a pointer line in the SKILL.md.

**After** (~12 tokens):
> `Reference → T1 (move to references/)`

---

### T10 — Confidence Gate firing (section preserved)

**Section under review** (3 lines, behavioral):
> Diagnose the real constraint before prescribing. Stated constraints are often not the actual constraint — the real one is usually one layer beneath. Name it explicitly before recommending.

**T10 check:**
- Savings if T9 applied: ~8 tokens (below 15-token threshold)
- Contains judgment cues: "often", "usually", "one layer beneath"
- Section is 3 lines → Core+short

**Result**: T10 gate fires. Section preserved as-is. Reported as "1 T10 gate: judgment cues + minimal savings."

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
