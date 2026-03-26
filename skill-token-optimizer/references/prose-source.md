# Skill Token Optimizer — Prose Source
> This is the human-readable reference version. The loaded SKILL.md uses schema-encoded compression (T9).
> Edit this file first when making changes, then re-encode SKILL.md from it.
> Last updated: 2026-04-26 v4 (decoding-cost principle, hot/cold cache amortization, T1/T5 reversal heuristic, _shared/ mechanism documented, L1 char-vs-token clarified, stopping criterion, audit-log compaction protocol, trivial-change skip, classification taxonomy made explicit, T8b numbering note, grammar hoisted to single Notation section)

---

## Purpose

Maximize instruction clarity per token. Bloated skills are a recurring tax on every inference call.

The aim is **net positive value**, not minimum bytes. Compression has costs — schema notation has cognitive parse cost; references/ files cost a load when they fire; hot skills benefit from prompt-cache amortization while cold skills don't. The optimizer must reason about both sides.

---

## Token Architecture

Three levels — optimize each separately:

- **Level 1: Description (frontmatter)** — always in context; **1024-char hard limit**; optimize for trigger accuracy.
- **Level 2: SKILL.md body** — loaded when triggered; target <500 lines; highest leverage.
- **Level 3: references/** — on-demand only; never auto-loaded; loaded *only* when the model explicitly fetches.

### L1 metric — char vs token

L1 has a 1024-character hard limit (enforced by the harness). Token cost is approximately `chars ÷ 4` for English. These are separate measurements:

- `L1_tokens ≈ chars ÷ 4` — the cost on each call
- `L1_budget_used = chars ÷ 1024` — fraction of the 1024-char ceiling consumed (0–1)

Don't conflate. Earlier audit logs used `chars÷1024` as a token estimate — that is wrong; it's a budget-utilization ratio.

### L2 lower bound

Bodies under 200 tokens are at the compression floor — likely all Core+short already. Audit L1 only; don't re-compress L2.

### Prompt-cache amortization (hot vs cold skills)

Skills load into the system-prompt cache (5-min TTL on Anthropic API). Stable skill content is amortized across many calls within the cache window. So:

- **Hot skills** (≥10 triggers/day or always-loaded): aggressive optimization is justified — every byte saved compounds across many calls. Hot skills also benefit most from *stability* — frequent edits bust the cache.
- **Cold skills** (rare triggers): aggressive optimization is theatre. Schema-encoding a cold skill saves negligible cost while adding parse friction every time it does fire. Leave verbose for clarity.

Estimate hotness by: trigger frequency × user count × cache-bust rate. If you don't know, assume cold and prefer prose.

---

## Packaging Convention (prose-source pattern)

Every skill stores its human-readable source in `references/prose-source.md`. This is the canonical edit target — written in plain prose, no compression. The `SKILL.md` body is derived from it using the compression techniques below.

**Workflow**: always edit `prose-source.md` first → re-encode `SKILL.md` from it. **Never edit `SKILL.md` directly.** Editing SKILL.md first and then "backporting" to prose wastes work — the optimizer re-encodes SKILL.md from prose anyway, so any compression you hand-apply to SKILL.md is discarded on the next audit. Prose is the input; SKILL.md is the output. Treat them like source code and a build artifact: change the source, then rebuild.

Shared reference files (lookup data, competitor lists, browser targets, pool defaults) live in `_shared/` and are loaded on-demand via explicit pointer. Non-trivial shared files also carry a `prose-source.md` as their edit target.

### `_shared/` loading mechanism (clarified)

`_shared/` is a **convention**, not a special loader. Files there behave exactly like per-skill `references/` — they load only when the model explicitly fetches them via Read tool. There is no auto-loading.

What `_shared/` actually buys you:
- A single source of truth for cross-skill content (no manual sync between per-skill copies).
- Discoverability — files are grouped at one path.

What it does NOT buy you:
- Automatic deduplication of in-context tokens. If three skills each contain a 50-line copy of "browser targets", placing one canonical copy in `_shared/` saves bytes on disk and sync overhead — but does NOT reduce in-context tokens unless the SKILL.md bodies replace the inlined content with a pointer line. Then the savings are real *only when the pointer is not followed*.

Use `_shared/` when: content is identical across 3+ skills AND the content is reference (lookup data) not behavioral (instructions).

---

## Notation (single source of truth — referenced by classify+route and T9)

This section is the canonical operator and subject grammar. Both the classify+route table and T9 schema-encoding refer back to it instead of redefining.

### Operator grammar

| Symbol | Name | Meaning |
|---|---|---|
| `→` | route | if X, then Y |
| `\|` | branch | alternative / or |
| `[]` | group | explicit grouping |
| `k:v` | key:value | typed property |
| `&` | conjunction | both must hold |
| `!X` | negation | must not hold |
| `?(c):T\|F` | ternary gate (prefix) | true-branch = first operand after `:`; else-chain starts after first `\|` |
| `X?(f)` | inline guard (postfix) | apply X only when filter f holds; no defined false-branch; skip on filter miss |
| `else:[]` | else-group | explicitly groups false-branch alternatives in a ternary; prevents ambiguity with post-ternary `\|` chains |

**Operator list format:** use `sym: purpose` keyed rows. Never list operators inside `[...|...]` with `|` as separator — `|` is itself an operator and creates a parsing collision.

### Subject grammar (for classify table rows)

| Notation | Meaning | Example |
|---|---|---|
| `s∈Set` | set membership | `s∈Do-NOT-Compress` |
| `s=Type` | type equality | `s=reference`, `s=behavioral` |
| `s.prop` | property access | `s.repeated≥3` |

Use `∈` for named lists/sets. Use `=` for type categories. Use `.prop` for measurable attributes.

### Decoding-cost principle

Schema notation saves storage tokens but Claude pays a parse cost on every trigger. Schema is correct iff:

1. The operator grammar is visible **in the same section** (or via a single-section reference like this one), AND
2. A fluent reader can decode it in **<5 seconds without scrolling**.

If either fails, prose wins. T10's "judgment cues → ⊘T9" is one instance of this principle; the principle is the general rule.

---

## Classification Taxonomy (5 categories, defined once)

Every section gets exactly one classification. Each classification has one routing decision:

| Category | Definition | Route |
|---|---|---|
| **Core** | Behavioral instructions, >3 lines, no judgment cues | T9 candidate (T10-gated) |
| **Core+short** | Behavioral or template content where compression would harm clarity (≤3 lines, output templates, anti-pattern lists, default lists, DNC matches, judgment-heavy) | preserve as-is; STOP |
| **Reference** | Lookup data, benchmarks, comparison tables — non-behavioral | T1 (move to references/) |
| **Redundant** | Content appearing verbatim or near-verbatim in 3+ skills | T5 (extract to `_shared/` or pointer to CLAUDE.md) |
| **Verbose** | Prose with rationale, preamble, hedges; behavioral content buried in narrative | T6 first, then re-classify |

The classify+route table below is the operational form of this taxonomy.

---

## Audit Protocol

### Pre-flight 1 — Trivial-change skip

If the SKILL.md diff since the last audit-log entry is <3 lines AND the L1 description is unchanged: **skip the audit**. Log `Action: skipped-trivial` (one line, no metrics block).

A 1-line typo fix should not trigger a full audit cycle.

### Pre-flight 2 — Stopping criterion

If the last 2 audit-log entries for this skill each yielded <3% reduction AND no L1 issues were flagged: **halt**. Log `Action: at-floor (last 2 runs <3%)`. No further passes until skill content is externally edited.

This prevents the "7 back-to-back diminishing-returns runs in one day" pattern visible in older logs.

### Check audit-log first

Before classifying any section, read `references/audit-log.md`. Sections that were explicitly preserved by T10 gate or Do-NOT-Compress are named in the Notes field of past entries. Do not re-classify these sections — treat them as already assigned Core+short and skip them.

If audit-log.md exceeds 200 lines, run the **audit-log compaction** step (defined under Optimization Workflow) before reading.

### L1 Description Audit (run this first)

L1 is always loaded — it burns tokens on every call, not just when the skill triggers. Audit it before the body.

1. **Char count**: ≤1024 characters? If over: convert narrative sentences to CSV trigger phrases; drop preamble.
2. **Trigger recall**: Does the description activate on all intended queries? Over-tightening hurts recall. Optimize for accuracy, not brevity.
3. **Format check**: Should follow `Use for: [CSV list]` | `Trigger: [CSV of literal phrases]` | `Auto-trigger: [conditions]` — no full sentences needed.
4. **Negative trigger audit (T8b)**: Scan for "Do NOT use for X, use Y instead" sentences. Convert to `skip: [X] → Y` notation. L1-only — body negative triggers are behavioral instructions and must NOT be compressed.
5. **Description tightening (T8)**: After T8b, tighten remaining L1 phrases. Pack as comma-separated lists; verbs-first ordering; strip narrative preamble not caught by T8b. Optimize for recall, not completeness.
6. **Dedup**: Remove trigger phrases with >80% semantic overlap (e.g. "compress skill" ≈ "optimize this skill").

### L2 Body Audit — Unified Classify + Route

Each row is a match condition; stop at the first match. Subject and operator grammar are defined in the **Notation** section above.

```
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

Flag behavioral defaults that appear verbatim (or near-verbatim) in 3 or more skills → candidate for `_shared/behavioral-defaults.md` (subject to T5 reversal heuristic — see T5).

---

## Compression Techniques

**T1. Progressive Disclosure** — highest leverage *when applicable*. Move lookup data to `references/` and replace with a pointer. Keep behavioral guidance in SKILL.md; move factual lookup data out.

*Reversal heuristic (T1 correctness check)*: T1 is correct only when (a) the moved content is non-behavioral AND (b) the reference is loaded less often than the skill triggers. If a reference is read on ≥80% of trigger fires, you've made things worse — pull it back into SKILL.md. Track via grep on conversation transcripts when measurable; otherwise apply judgment: lookup tables, benchmarks, before/after libraries are good T1 candidates; one-paragraph instructions are not.

T1 when: `s.size>50tok | s.changes_frequently | s=lookup-only`.

**T2. Bullet Compression** — strip preamble ("Are X doing Y?", "This is important because..."); merge bullets sharing a parent concept; cut bullets that restate their header. Headers: shorten to ≤3 words; drop parentheticals.

**T3. Table Compression** — shorten headers to 1–2 words; use abbreviations; remove identical-value columns; replace star ratings with numbers; <3-row tables → bullets; >4-col tables → split or drop lowest-value column.

**T4. Section Consolidation** — merge thin sections sharing a behavioral theme; use inline pipe-separated lists for grouped metrics. Pairs with T2 on verbose sections; T4 fires when sections share a theme.

**T5. Shared Context Files** — extract repeated context to `_shared/` (or to a CLAUDE.md section). Each skill replaces with a single pointer line.

*Mechanism reality check*: `_shared/` files load on-demand only — they are NOT auto-loaded into every skill activation. T5 only saves in-context tokens when (a) the SKILL.md bodies replace inlined content with a pointer AND (b) the pointer is not followed on every trigger. If models routinely fetch the shared file every time, T5 is a sync convenience but not a token win.

*Reversal heuristic*: same as T1 — if the shared file is loaded ≥80% of trigger fires across the consuming skills, the inline copies were cheaper. In that case, prefer keeping content inline and using T6/T9 for compression instead.

Cost calc (only valid if pointer not followed): 5 skills × 300 shared tokens × 200 loads/month = 300K wasted tokens/month. Behavioral defaults appearing in 3+ skills are candidates, but apply the reversal heuristic before extracting.

Check T5 *before* T4 — if content is cross-skill, T5 (or accept duplication) is the choice, not T4 within one skill.

**T6. Instruction Density** — drop "you should", "make sure to", "always remember to". Use imperative fragments. Before (~40 tokens): "When analyzing product questions, you should always make sure to clearly identify who the buyer is..." After (~12 tokens): "Always distinguish buyer vs. user." Guard: never strip negation prefixes [Do NOT | never | avoid].

**T7. Code Block Minimalism** — remove illustrative-only blocks; replace with inline examples or cut. Keep code blocks only when: exact syntax matters, it's a real template, or structure can't be conveyed inline. Illustrative = `[...]` / placeholder-only blocks; exact = runnable | substitutable.

**T8. Description Tightening** *(L1)* — runs after T8b on remaining L1 phrases. Pack as comma-separated lists; verbs-first; strip narrative preamble not caught by T8b. Optimize for recall, not completeness.

**T8b. Negative Trigger Compression** *(L1 only)* — convert "Do NOT use for X, use Y instead" sentences in the description (frontmatter) to compact `skip: [X] → Y` notation. This technique applies *only* to L1 descriptions — negative trigger clauses in the SKILL.md body are behavioral instructions and must NOT be compressed. T10 gate does not apply to T8b; savings are structural, not judgment-dependent.

*Numbering note*: T8b is intentionally inserted between T8 and T9 because it shares L1-only scope with T8. It is not "T11" by sequence — the suffix marks it as a co-located variant. Audit-log entries from 2026-03-26 onward use this naming; do not renumber.

**T9. Schema Encoding** — replace conditional/procedural prose with compact notation. Operators and subject grammar are defined in the **Notation** section above; do not re-list them here.

*Apply to:* trigger conditions, decision routing, action lists, output structure, classify tables, workflow steps.
*Do not apply to:* nuanced judgment calls, compliance language, or any instruction where the **decoding-cost principle** fails (operator grammar not visible in same section, or decode time >5s).

Examples:

```
behavioral & len>3 & !judgment_cues → T9
repeated≥3 & !in_behavioral_defaults → T5
?(T10:saves≥15 & !judgment_cues):T9 | else:[T7?(code)|T3?(table>3r)]
s=verbose & len>5 → T6 first; then ?(T10:saves≥15 & !judgment_cues):T9  [chain: never skip T6]
```

**T10. Confidence Gate** — a final check applied before every T9 call, protecting sections from over-compression. Before encoding a Core section with T9, ask:
- Would this save fewer than 15 tokens? → Skip T9; prose is clearer.
- Does the section contain judgment cues ("when", "unless", "except if", "but only")? → Skip T9; schema notation loses the nuance.
- Is the section already 3 lines or fewer? → Assign Core+short; skip all compression.
- Does the **decoding-cost principle** fail (operator grammar not in same section, decode >5s)? → Skip T9.

T10 is not an optional polish step — it is a mandatory gate. Every T9 application requires a T10 check first.

**⊘T9 shorthand**: When T10 fires and a section is preserved, record this as `⊘T9` in the audit log Techniques field rather than writing "T10 gate fired" in full. The count of ⊘T9 events replaces the old "T10 gates fired: N" phrasing.

---

## Technique Selection and Priority

The unified classify+route table handles routing. Priority ordering within each bracket:

| Skill size | Typical tokens | Target | First techniques | If still over |
|---|---|---|---|---|
| Narrow/focused | 500–800 | <300–500 | T1 → T6 → T8b | T3 → T2 |
| Domain-expert | 1,500–2,500 | 800–1,200 | T1 → T5 → T6 | ?(T10):T9 → T3 |
| Comprehensive | 2,500–4,000 | 1,200–1,800 | T1 → T5 → T6 | ?(T10):T9 then T7?(code) |

**Hot/cold modifier** (apply to all rows): for **cold skills** (rare triggers), drop the "If still over" column and stop after first techniques — schema-encoding a cold skill costs more in parse friction than it saves. Reserve aggressive ?(T10):T9 for **hot skills** that fire frequently enough to amortize the parse cost.

**Global ordering**: T1 first — removes the most tokens without any compression risk. Then T8b on L1. Then ?(T10) gate before every T9 call. Then T6 on verbose sections. Then T2/T3/T4 structural tightening.

Run L1 description audit (including T8b) before the body — L1 fires on every call.

`>2500 tokens before T1`: consider split — skill may be too broad. Split on: distinct audience | distinct trigger set | domain boundary.

`new skill`: write prose-source.md first; target narrow/focused bracket (<500 tok).

---

## Skill Creation & Edit Integration

This optimizer should run automatically as a final step after any skill is created, edited, or updated — even when the user did not explicitly request optimization.

**When to auto-trigger:**
- A new skill has been written → run a full audit before packaging or presenting the skill.
- An existing skill has been edited → re-audit the changed sections plus the L1 description, **subject to the trivial-change skip** (see Audit Protocol pre-flight).
- Only the skill description was updated → run L1 audit only (T8b + T8 pass, recall check).

**Steps:**
1. Make all edits to `references/prose-source.md`. Do NOT touch `SKILL.md` directly — it is regenerated from prose. Hand-edits to SKILL.md will be overwritten on the next audit and waste effort.
2. Run pre-flight: trivial-change skip and stopping criterion. If either fires, log and stop.
3. Re-encode `SKILL.md` from prose-source using the compression techniques.
   - **Re-encode tool choice (mandatory)**: prefer a single `Write` for substantive re-encodes — net additive prose growth ≥10%, ≥3 new sections, or any change touching >200 lines of SKILL.md. Multi-`Edit` chains on large SKILL.md files (>400 lines) are brittle: `old_string` uniqueness fails on similar-looking schema/template lines, sequential edits drift state, and accumulated diff cost can exhaust the token budget mid-run (the 2026-04-30 #7 `finalsite-committer-effectiveness` audit deferred to a later session for exactly this reason). Use `Edit` only for narrow targeted changes (≤2 sections, ≤50 lines net) where uniqueness is clean. **Heuristic**: if you'd need ≥4 sequential Edits to mirror prose-source, do one `Write` instead.
4. Run the audit protocol on the resulting SKILL.md.
5. Apply techniques where warranted — never compress Do-NOT-Compress sections.
6. Report before/after token counts and techniques applied.
7. If savings are less than 5% and there are no L1 issues, note "already efficient" and skip the rewrite.

**Anti-pattern (do not do this)**: edit SKILL.md → copy diff into prose-source → trigger optimizer. The optimizer will re-encode prose into SKILL.md and discard your hand-compression. Always start in prose.

This integration requirement is also reflected in the L1 description with an explicit `Auto-trigger` line.

---

## Optimization Workflow

**Single skill**: `pre-flight (trivial skip + stopping criterion) → check audit-log → L1 audit (T8b scan) → L2 unified classify+route (T1 first, ?(T10) before T9) → recount → verify behavioral instructions intact → sync prose-source → package → append entry to references/audit-log.md`

**Skill set**: Audit all for redundancy (both reference content AND behavioral defaults) → create `_shared/` entries (with reversal-heuristic check) → optimize individually → report total reduction → append all entries to audit logs. Redundancy detection: compare Defaults+pointer lines across skills; flag verbatim ≥10-token matches.

### prose-source is the only edit surface (mandatory)

**Rule**: all human edits land in `references/prose-source.md`. `SKILL.md` is a build artifact regenerated by the optimizer. Do not edit SKILL.md directly under any circumstance — not for "minor fixes", not for "small tweaks", not for "I'll backport later". The next audit will overwrite hand-edits and the round trip wastes time.

- **New skill**: write `references/prose-source.md` first. The optimizer re-encodes it into SKILL.md as part of the create flow.
- **Editing an existing skill**: edit prose-source only. Then run the optimizer to regenerate SKILL.md.
- **Optimization run**: techniques are applied during the prose→SKILL.md re-encode. prose-source itself stays in plain prose.
- **Drift check**: if you discover SKILL.md and prose-source have diverged (e.g. someone hand-edited SKILL.md), prose-source wins — re-encode and discard the SKILL.md hand-edit. Do not "merge" by copying SKILL.md changes into prose; investigate why the divergence happened and re-do the edit in prose.

This is the same source-vs-build-artifact pattern as compiled code: you don't edit the binary, you edit the source and rebuild.

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
- If pre-flight fired, log as `Action: skipped-trivial` or `Action: at-floor (last 2 runs <3%)` with no metrics block.

### Audit log compaction

When `audit-log.md` exceeds 200 lines, compact:

1. Keep the latest 3 entries verbatim.
2. Move older entries to `references/audit-log-archive/YYYY-QN.md` (quarterly archive).
3. Maintain `references/preserved-sections.md` as a current-state index: one bullet per preserved section, format `- [section name] — [reason: DNC | ⊘T9 (judgment cues | <15tok | minimal)]`. This index replaces the need to scan all old entries on each "check audit-log first" step.
4. Append a one-line compaction note to `audit-log.md`: `## YYYY-MM-DD compaction — N entries archived to YYYY-QN.md; preserved-sections.md rebuilt.`

Future "check audit-log first" reads: open `preserved-sections.md` (small, fast) plus the latest 3 entries. Do not re-read the archive unless investigating a specific historical decision.

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
- **Negative trigger clauses in SKILL.md body** — behavioral instructions; T8b applies to L1 only
- **Any section where ⊘T9 fires** — saves <15 tokens, judgment cues present, decoding-cost principle fails, or section already minimal
- **Worked examples that are the sole demonstration of a technique** — compressing loses training signal

---

## Output Format

**Audit report:**
```
Skill: [name] | hot/cold: [hot|cold|unknown] | Current: ~[N] tokens ([lines] lines)
L1: [chars]/1024 ([X]% budget) | ~[N] tok | trigger recall: [ok/risk] | T8b: [N] | issues: [list or none]
Sections: [section → token count]
Classified: [N] Core | [N] Core+short | [N] Reference | [N] Redundant | [N] Verbose
⊘T9: [N sections preserved] | Techniques: [list] | Estimated post-opt: ~[N] tokens ([X]% reduction)
Pre-flight: [ran | skipped-trivial | at-floor] | Audit log: [appended | skipped]
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

The `else:[]` grouping makes clear that `T7?(code)` and `T3?(table>3r)` are both false-branch alternatives, not post-ternary routing steps.

---

### T10 — Confidence Gate firing (⊘T9 — section preserved)

**Section under review** (3 lines, behavioral):
> Diagnose the real constraint before prescribing. Stated constraints are often not the actual constraint — the real one is usually one layer beneath. Name it explicitly before recommending.

**T10 check:**
- Savings if T9 applied: ~8 tokens (below 15-token threshold)
- Contains judgment cues: "often", "usually", "one layer beneath"
- Section is 3 lines → Core+short

**Result**: ⊘T9. Section preserved as-is.

---

### T1 reversal — when progressive disclosure is wrong

**Section moved to references/sql-error-codes.md** (200 tokens of pgconn error code mappings).

**Observed**: across 30 trigger fires of postgres-conventions, the reference was loaded 28 times (93%). Net cost: SKILL.md shrank by 200 tok but model paid 28× the reference load instead of 28× of the (cached) SKILL.md inclusion.

**Action**: pull back into SKILL.md inline. T1 was wrong for this content.

**Heuristic**: load frequency ≥80% of trigger frequency → reverse T1.

---

### Hot vs cold optimization — cold skill left verbose

**Skill**: `coldfusion-architecture` — triggers <1×/week.

**Body**: 1,800 tokens, mix of prose and tables. Comprehensive bracket per benchmark.

**Decision**: stop after T1 + T6. Skip ?(T10):T9 entirely. The schema-encoding would save ~150 tok per call but the skill fires so rarely that lifetime savings (≤7,800 tok/year) are dwarfed by the cumulative parse-friction cost across the team.

**Logged as**: `cold-skill — T9 deferred per hot/cold modifier`.

---

### Trivial-change skip example

**Diff**: typo fix in one bullet of golang-conventions (one line, "interfance" → "interface").

**Pre-flight result**: diff <3 lines, L1 unchanged → `Action: skipped-trivial`.

**Log entry**: `## 2026-04-26 golang-conventions — Action: skipped-trivial (1-line typo fix; L1 unchanged)`.

No audit cycle, no rewrite. Saves 5+ minutes of optimizer noise per trivial edit.

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
