---
name: skill-token-optimizer
description: >
  Optimizes skill files for token efficiency without degrading instruction quality.
  Use for: audit/rewrite/compress skills, token cost reduction, context window management,
  new skill design, skill set redundancy analysis.
  Trigger: "optimize this skill", "reduce tokens", "compress skill", "audit token usage",
  "skills are getting expensive", creating a new skill, editing an existing skill, updating a skill.
  Auto-trigger: always run as a final step after any skill is created, edited, or updated.
  Pre-flight: trivial-change skip (diff<3 ln & L1 unchanged) | at-floor (last 2 runs <3%) → skip.
  skip: [prompt optimization, non-skill files] → prompt-optimizer
---

# Skill Token Optimizer
> Prose source: `references/prose-source.md` is the **only** edit surface. SKILL.md is a build artifact — never hand-edit it. Edits to SKILL.md are overwritten on the next audit. Workflow: edit prose → run optimizer → optimizer regenerates SKILL.md.

Maximize instruction clarity per token. Aim is **net positive value**, not minimum bytes — schema has parse cost; references/ cost a load when fetched; hot skills amortize via cache, cold skills don't.

## Token Architecture
```
L1: description/frontmatter → always loaded | 1024-char hard limit | optimize for trigger accuracy
L2: SKILL.md body          → loaded on trigger | target <500 lines | highest leverage
L3: references/            → on-demand only; never auto-loaded | unlimited
packaging: prose→references/prose-source.md | compressed→SKILL.md | shared-data→_shared/
L1 metric: L1_tokens ≈ chars÷4 | L1_budget_used = chars÷1024 (utilization, not tokens — distinct)
L2 <200 tok: audit L1 only — body at compression floor
cache amortization: hot skills (≥10 trigs/day) amortize via 5-min cache → aggressive ok
                    cold skills (rare) → schema parse-cost > savings; prefer prose, stop after T1+T6
_shared/: convention only — files load on-demand like references/ (no auto-load).
          Token win only if SKILL.md replaces inline copy with pointer AND pointer not followed.
```

---

## Notation (single source — referenced by classify+route and T9)

```
Subject grammar: s∈Set=membership  s=Type=type-equality  s.prop=property-access
Operator grammar: →=route  |=branch  []=group  &=and  !=not
                  ?(c):T|F=ternary (true=first after `:`; else-chain after first `|`)
                  X?(f)=inline-guard (postfix; skip on filter miss)
                  else:[]=else-group (disambiguate ternary false-branch from post-ternary `|`)
List format: `sym: purpose` keyed rows (never list ops inside [...|...] — `|` is itself an operator)
```

**Decoding-cost principle**: schema correct iff (a) operator grammar visible in same section AND (b) fluent reader decodes in <5s without scrolling. Else prose wins. T10 enforces this for body sections.

---

## Classification Taxonomy (5 categories)

```
Core         → behavioral, >3 lines, no judgment cues   → T9 candidate (T10-gated)
Core+short   → DNC match | template | ≤3 lines | judgment-heavy → preserve; STOP
Reference    → lookup | benchmarks | comp data           → T1 (move to references/)
Redundant    → verbatim/near-verbatim in 3+ skills       → T5 (with reversal check) | delete
Verbose      → prose w/ rationale|preamble|hedges        → T6 first, then re-classify
```

---

## Audit Protocol

**Pre-flight 1 — trivial-change skip**: `diff<3 ln & L1 unchanged → log Action:skipped-trivial; STOP`

**Pre-flight 2 — stopping criterion**: `last 2 entries each <3% reduction & no L1 issues → log Action:at-floor; STOP`

> audit-log: skip ⊘T9-preserved & DNC sections (named in Notes). If audit-log >200 ln → run compaction first.

**Measure first**: `L1=chars (≤1024) & ≈chars÷4 tok | L2=word_count÷0.75` | tokens_by_section | count:[headers,bullets,tables,code_blocks] | hot/cold tag

**L1 description audit** (run before body audit):
- Char count ≤1024? If over: convert sentences→CSV trigger phrases; drop narrative preamble
- Trigger recall: does description activate on all intended queries? Optimize for recall, not brevity
- Format: `Use for: [CSV]` | `Trigger: [CSV of literal phrases]` | no full sentences
- T8b scan: find "Do NOT use for X, use Y" sentences → convert to `skip: [X] → Y`
- T8 pass: tighten remaining L1 phrases; verbs-first; strip preamble not caught by T8b
- Dedup: remove trigger phrases with >80% semantic overlap

**L2 body — unified classify + route** (one pass; stop at first match; subjects/operators per Notation):

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

`⊘T9` = T10 gate fired; section preserved.

Behavioral defaults in 3+ skills → `_shared/behavioral-defaults.md` (subject to T5 reversal check)

---

## Compression Techniques (T1–T10 + T8b)

```
T1:  progressive-disclosure  → move lookup→references/; keep behavioral in SKILL.md [highest leverage]
                               T1 when: s.size>50tok | s.changes_frequently | s=lookup-only
                               REVERSAL: load_freq ≥ 80% of trigger_freq → pull back inline (T1 was wrong)
T2:  bullet-compression      → strip preamble; merge same-parent bullets; cut header-restate bullets
                               headers: shorten to ≤3 words; drop parentheticals
T3:  table-compression       → shorten headers 1-2 words; abbrev; rm identical cols; <3rows→bullets; >4cols→split|drop-lowest-value
T4:  section-consolidation   → merge thin same-theme sections; inline pipe lists for grouped metrics
                               pair with T2 on verbose sections; T4 fires when sections share a theme
T5:  shared-context          → extract repeated ctx→_shared/; replace with pointer line
                               [5sk×300tok×200/mo=300K wasted/mo — only if pointer not followed]
                               MECHANISM: _shared/ loads on-demand like references/; no auto-load
                               REVERSAL: shared file loaded ≥80% of trigger fires → inline was cheaper; revert
                               check T5 before T4 — if content is cross-skill, T5 (or accept dup), not T4
T6:  instruction-density     → drop ["you should","make sure to","always remember to"]; use imperatives
                               guard: never strip negation prefixes [Do NOT | never | avoid]
T7:  code-minimalism         → rm illustrative-only blocks; keep only:[exact syntax|real templates|structure-only-via-code]
                               illustrative=[...]/placeholder-only; exact=runnable|substitutable
T8:  description-tightening  [L1] → tighten remaining phrases after T8b; verbs-first; strip preamble
T8b: negative-trigger-compression [L1 ONLY] → "Do NOT use for X, use Y" → `skip: [X] → Y`
                               [L1 only; body neg-triggers=behavioral→preserve; T10 n/a]
                               NUMBERING: T8b is co-located variant of T8 (both L1-scope); not "T11" — do not renumber
T9:  schema-encoding         → replace conditional/procedural prose with compact notation
                               operators+subjects: see Notation section above (single source)
     apply to:[trigger conditions|decision routing|action lists|output structure|classify tables|workflow steps]
     do NOT apply to:[nuanced judgment|compliance language|decoding-cost principle fails]
     examples:
       behavioral & len>3 & !judgment_cues → T9
       repeated≥3 & !in_behavioral_defaults → T5
       ?(T10:saves≥15 & !judgment_cues):T9 | else:[T7?(code)|T3?(table>3r)]
       s=verbose & len>5 → T6 first; then ?(T10:saves≥15 & !judgment_cues):T9  [chain: never skip T6]
T10: confidence-gate [⊘T9]  → mandatory gate before every T9 call:
                               saves<15 tok → ⊘T9 (prose clearer)
                               judgment cues ["when","unless","except if","but only"] present → ⊘T9
                               section ≤3 lines → Core+short; ⊘T9
                               decoding-cost fails (grammar not in same section | decode >5s) → ⊘T9
                               report ⊘T9 count in audit output and log
```

For before/after examples → `references/prose-source.md`

---

## Skill Creation & Edit Integration

**Always run as a final step** after any skill is created, edited, or updated — subject to pre-flight skips.

Trigger conditions (auto-apply):
- New skill written → full audit before packaging/presenting
- Existing skill edited → re-audit changed sections + L1 (subject to trivial-change skip)
- Skill description updated → L1 audit only (T8b+T8 + recall check)

Steps:
1. **Edit `references/prose-source.md` only.** Never edit SKILL.md directly — it is regenerated from prose. Hand-edits to SKILL.md are overwritten on the next audit and waste effort.
2. Run pre-flight (trivial-skip + stopping criterion). If either fires, log and stop.
3. Re-encode SKILL.md from prose-source using compression techniques.
   - **Re-encode tool choice (mandatory)**: prefer single `Write` for substantive re-encodes — net additive prose growth ≥10%, ≥3 new sections, or change touching >200 lines of SKILL.md. Multi-`Edit` chains on large SKILL.md (>400 lines) are brittle: `old_string` uniqueness fails on similar schema/template lines, sequential edits drift state, accumulated diff cost can exhaust token budget mid-run. Use `Edit` only for narrow targeted changes (≤2 sections, ≤50 lines net) with clean uniqueness. **Heuristic**: if you'd need ≥4 sequential Edits to mirror prose, do one `Write`.
4. Run audit protocol on resulting SKILL.md
5. Apply techniques where warranted (never compress DNC sections)
6. Report: before/after token counts + techniques applied
7. Savings <5% & no L1 issues → note "already efficient"; skip rewrite

**Anti-pattern**: editing SKILL.md → copying diff into prose-source → triggering optimizer. The optimizer re-encodes prose into SKILL.md and discards your hand-compression. Always start in prose.

---

## Workflow

**Single skill:** `pre-flight → audit-log? → L1(T8b+T8) → L2 classify+route(T1 first; ?(T10)→T9) → recount → verify → sync prose-source → log`

**Set:** `redundancy(ref+behavioral) → _shared/(reversal-check) → optimize each → report → log`
redundancy detection: compare Defaults+pointer lines across skills; flag verbatim ≥10 tok matches

### prose-source is the only edit surface (mandatory)

All human edits land in `references/prose-source.md`. SKILL.md is a build artifact regenerated by the optimizer — never hand-edit it.
- New skill: write prose-source.md first; optimizer encodes SKILL.md from it
- Edit existing skill: edit prose-source only; run optimizer to regenerate SKILL.md
- Drift: if SKILL.md and prose-source differ, prose-source wins — re-encode and discard the SKILL.md hand-edit
- Pattern: source vs build artifact — don't edit the binary, edit the source and rebuild

### Audit Log Convention

```
## [YYYY-MM-DD] [skill-name]
Before: ~[N] tokens ([lines] lines)
After:  ~[N] tokens ([lines] lines) — [X]% reduction
Techniques: [list] | ⊘T9: [N] (sections preserved)
L1 changes: [description or "none"]
Notes: [preserved sections and reasons — guides future audits]
```

Rules: append-only | name ⊘T9-preserved sections in Notes | log `Action: audit-only` if no changes | log `Action: skipped-trivial` or `Action: at-floor` if pre-flight fired (no metrics block)

### Audit log compaction (audit-log >200 ln)

```
1. Keep latest 3 entries verbatim
2. Move older → references/audit-log-archive/YYYY-QN.md (quarterly)
3. Maintain references/preserved-sections.md as current-state index:
   `- [section name] — [reason: DNC | ⊘T9 (judgment cues|<15tok|minimal|decode-cost)]`
4. Append one-line note: `## YYYY-MM-DD compaction — N entries archived; preserved-sections.md rebuilt`
Future "check audit-log first": read preserved-sections.md + latest 3 entries only
```

Benchmarks (apply hot/cold modifier — cold skills stop after first techniques):
```
narrow/focused:  500–800 tok  → target <300–500  | first: T1→T6→T8b+T8 | if over (hot only): T3→T2
domain-expert:   1500–2500    → target 800–1200   | first: T1→T5→T6     | if over (hot only): ?(T10):T9→T3
comprehensive:   2500–4000    → target 1200–1800  | first: T1→T5→T6     | if over (hot only): ?(T10):T9 then T7?(code)
>2500 before T1: consider split — distinct audience | distinct trigger set | domain boundary
new skill: write prose-source.md first; target narrow/focused bracket (<500 tok)
hot/cold rule: cold (rare trigs) → drop "if over" steps; parse-cost > savings
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
- Any section where T10 gate fires (⊘T9: <15tok | judgment cues | decode-cost fails | minimal)
- Worked examples that are the sole demonstration of a technique (compressing loses training signal)

---

## Output Format

**Audit report:**
`Skill: [name] | hot/cold: [hot|cold|unknown] | L1: ~[N] tok | L2: ~[N] tok ([lines] lines) | top: [biggest-section]`
`L1: [chars]/1024 ([X]% budget) | recall: [ok/risk] | T8b: [N] | issues: [over-1024|low-recall|none]`
`Sections: [section → token count]`
`Classified: [N] Core | [N] Core+short | [N] Reference | [N] Redundant | [N] Verbose`
`⊘T9: [N] | Techniques: [list] | Post-opt: ~[N] tok ([X]%)`
`Pre-flight: [ran|skipped-trivial|at-floor] | Audit log: [appended | audit-only]`

**After rewrite:** `before/after count | techniques applied | ⊘T9 fired (sections preserved) | behavioral instructions confirmed intact | audit log entry appended`
