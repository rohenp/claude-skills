# Skill Token Optimizer — Prose Source
> This is the human-readable reference version. The loaded SKILL.md uses schema-encoded compression (T9).
> Edit this file first when making changes, then re-encode SKILL.md from it.

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

Measure before rewriting:
1. Total tokens (word_count ÷ 0.75) + tokens by section
2. Count: headers, bullets, tables, code blocks
3. Flag reference-only content (lookup tables, competitor lists, benchmarks) → move to `references/`
4. Flag content repeated across skills → move to `_shared/`
5. Flag prose that restates what structure already implies

Classify each section: **Core** (behavioral — keep) | **Reference** (lookup — move out) | **Redundant** (delete) | **Verbose** (compress)

---

## Compression Techniques

**T1. Progressive Disclosure** — highest leverage. Move lookup data to `references/` and replace with a pointer. Keep behavioral guidance in SKILL.md; move factual lookup data out.

**T2. Bullet Compression** — strip preamble ("Are X doing Y?", "This is important because..."); merge bullets sharing a parent concept; cut bullets that restate their header.

**T3. Table Compression** — shorten headers to 1–2 words; use abbreviations; remove identical-value columns; replace star ratings with numbers; <3-row tables → bullets.

**T4. Section Consolidation** — merge thin sections sharing a behavioral theme; use inline pipe-separated lists for grouped metrics.

**T5. Shared Context Files** — extract repeated context (operating mandate, company background) to `_shared/`. Each skill replaces with a single pointer line. 5 skills × 300 shared tokens × 200 loads/month = 300K wasted tokens/month.

**T6. Instruction Density** — drop "you should", "make sure to", "always remember to". Use imperative fragments. Before (~40 tokens): "When analyzing product questions, you should always make sure to clearly identify who the buyer is..." After (~12 tokens): "Always distinguish buyer vs. user."

**T7. Code Block Minimalism** — remove illustrative-only blocks; replace with inline examples or cut. Keep code blocks only when: exact syntax matters, it's a real template, or structure can't be conveyed inline.

**T8. Description Tightening** — pack trigger phrases as comma-separated lists; full sentences waste characters. Optimize for accurate triggering, not completeness.

**T9. Schema Encoding** — replace conditional/procedural prose with compact notation. Use: arrow routing (`→`), pipe branching (`|`), bracket grouping (`[a,b,c]`), colon typing (`key:value`). Apply to: trigger conditions, decision routing, action lists, output structure. *Do not apply to*: nuanced judgment calls, compliance language, or any instruction where the schema would require more tokens to decode than it saves.

---

## Optimization Workflow

**Single skill**: Audit → Classify → Apply (start with T1 for max impact) → Recount → Verify behavioral instructions intact → Package

**Skill set**: Audit all for cross-skill redundancy → Create `_shared/` → Optimize individually → Report total reduction

### Benchmarks

| Skill type       | Typical tokens | Target tokens |
|------------------|----------------|---------------|
| Narrow/focused   | 500–800        | <300–500      |
| Domain expert    | 1,500–2,500    | 800–1,200     |
| Comprehensive    | 2,500–4,000    | 1,200–1,800   |

---

## What NOT to Compress

- Behavioral triggers — instructions that change what Claude does
- Output format templates — preserve exact structure when it matters
- "Do X, not Y" pairs — token-efficient per unit of behavioral guidance
- Decision criteria — cutting creates brittle behavior on novel cases
- Compliance/legal specifics — precision matters; do not paraphrase

---

## Output Format

**Audit report:**
`Skill: [name] | Current: ~[N] tokens ([lines] lines)`
`Sections: [section → token count]`
`Classified: [N] Core | [N] Reference | [N] Redundant | [N] Verbose`
`Techniques: [list] | Estimated post-optimization: ~[N] tokens ([X]% reduction)`

**After rewrite:** before/after count | techniques applied | content that couldn't compress without quality loss | behavioral instructions confirmed intact
