# Preserved Sections Index
> Current-state index of sections protected from compression across audited skills.
> Rebuilt 2026-04-29 during audit log compaction. Append entries as future audits flag preservation.

## skill-token-optimizer (self)
- Do-NOT-Compress list (10 bullets) — DNC: Core+short
- Output Format block — DNC: exact template
- Audit Log Convention entry format — DNC: exact template
- Benchmarks table (3 rows + technique-priority cols) — ⊘T9 (already minimal; <3 rows)
- Operator grammar table (Notation section) — ⊘T9 (exact template; load-bearing)
- Subject grammar table (Notation section) — ⊘T9 (exact template; 3 rows)
- T9 operator set with examples — DNC: behavioral schema definition
- Classify+route unified table — ⊘T9 (judgment routing; do not split back)
- Decoding-cost principle — DNC: methodology gate (judgment-heavy)
- Hot/cold cache amortization rule — DNC: cost model
- T1 reversal heuristic (load_freq ≥80%) — DNC: technique guard
- T5 mechanism + reversal (shared loaded ≥80%) — DNC: technique guard
- L1 metric formulas (chars÷4 vs chars÷1024) — DNC: definition
- Pre-flight 1 (trivial-change skip) — DNC: stopping criterion
- Pre-flight 2 (at-floor stopping) — DNC: stopping criterion
- Audit log compaction protocol — DNC: this protocol's own definition
- Classification taxonomy (5 categories) — DNC: definition
- T8b numbering note — DNC: behavioral instruction
- s=other fallthrough row — DNC: safety net

## prompt-optimizer
- Failure Taxonomy table (8 rows) — DNC: behavioral routing
- Claude-Specific Patterns sub-bullets (XML / prefilling / system-vs-turn / model-aware / few-shot / tool-use) — DNC: judgment-heavy techniques
- One-Shot template — DNC: exact output template
- System Prompt template — DNC: exact output template
- Agent template — DNC: exact output template
- Caching mechanics (4 bullets: read 0.1×, write 1.25×, 5-min vs 1-hr, breakpoint placement) — ⊘T9 (each <15 tok savings)
- Concept-pointers cautionary examples (MECE/BLUF/GROW) — DNC: sole demonstration of safety rule
- Token Estimation bullets (code/JSON ÷3, CJK ÷1.5) — DNC: lookup used per-trigger
- Skill File Detection ("if ambiguous, ask") — ⊘T9: judgment cues

## golang-conventions
- Modern Stdlib First bullet (`wg.Go(fn) over wg.Add(1)+defer Done()`) — DNC: idiom contrast vs Concurrency&Performance bullet
- All Modern Stdlib bullets (~17 lines) — preserve as bullets for legibility (Go 1.26 idioms)

## golang-tester
- t.Context, b.Loop, errors.AsType, wg.Go test patterns — DNC: Go 1.26 verbatim per user request

## postgres-conventions
- tern Migrations section (~25 lines: command syntax + safety rules) — DNC: imperative defaults
- errors.AsType for *pgconn.PgError — DNC: error-mapping idiom

## docker-dev
- tern make targets (3 entries) — DNC: imperative defaults
- multi-stage Dockerfile.dev layer cache table — DNC: Core+short reference

## api-conventions
- errors.AsType note in Sentry section — DNC: idiom

## service-design
- cmp.Or Config example (~13 lines) — DNC: behavioral demonstration of Go 1.26 idiom

## api-security, api-performance, observability
- All sections preserved verbatim per user audit-only request 2026-04-25

## nextjs-conventions (rewritten 2026-04-26 for Next.js 16)
- proxy.ts code template — DNC: exact replacement for middleware.ts
- Async Request APIs example block — DNC: shows mandatory `await` on cookies/headers/params/searchParams
- "use cache" directive code block — DNC: shows cacheTag + cacheLife usage
- Invalidation block (updateTag / revalidateTag / refresh) — DNC: 3 primitives, behavioral routing
- Typed Go API Client template — DNC: zod-validated response pattern
- Server Action template — DNC: 'use server' + Go API delegation pattern
- React Compiler "do NOT manually wrap" rule — DNC: judgment-heavy anti-pattern guidance
- Migration notes (15→16) — DNC: behavioral migration checklist

## rag-conventions (T1'd 2026-04-26)
- Pipeline diagram (6 stages + observability arrow) — DNC: load-bearing visual
- Sizing rules (4 bullets, atomic-units rule) — DNC: behavioral
- Mandatory metadata list — DNC: behavioral; chunk schema spec
- Operational embedding rules (6 bullets: Normalize/Pin/Re-embed/Batch/Cache/Truncate) — DNC: imperatives
- Quantization rules (4 bullets) — DNC: numeric defaults
- RRF formula `k=60` block — DNC: production default
- Context construction (7 bullets) — DNC: behavioral
- Evaluation 4-section block (Golden set / Retrieval / Generation / Online / Don't ship) — DNC: behavioral test discipline
- Production hardening 5-section block — DNC: behavioral
- Pitfalls (10 bullets) — DNC: anti-pattern list
- Decision order (10-step audit checklist) — DNC: workflow

## coldfusion-security (T1'd 2026-04-26)
- Enterprise context framing (3 paragraphs) — DNC: blast-radius framing
- Vulnerability index table (12 rows: severity + summary + audit grep) — DNC: triage-level reference; do not compress further (each row already minimal)
- CVE patch table (3 rows) — DNC: live security data
- Java deserialization / Network hardening / Enterprise governance sections — DNC: behavioral
- Quick-reference checklists (3) — DNC: PR/audit checklists
- Output Format block — DNC: exact template

## npm-security (added 2026-04-27)
- Anti-patterns block (verbatim "Do X, not Y" pairs) — DNC: token-efficient pairs
- Security Patches Override numbered procedure — ⊘T9 (judgment cues: "high CVE in unreachable code is not an emergency", "scoped to APIs the project never calls")
- Vetting checklist — kept inline (highest-frequency-use; load_freq would equal trigger_freq → T1 reversal)
- Pinning rule — DNC: imperative default
- Frozen-install command per tool — DNC: exact syntax
- Cool-down windows (7d/14d/30d) — DNC: numeric defaults
- Install-script lockdown (`ignore-scripts`, `onlyBuiltDependencies`, `trustedDependencies`) — DNC: exact directives
- Audit threshold + exception schema — DNC: behavioral

## finalsite-docs (added 2026-04-29)
- "When to use" — ⊘T9 (judgment cues: "non-trivial", "worth preserving")
- "Update protocol" — ⊘T9 (judgment cues: "contradicts", "materially changed", "verify before")
- "Reading protocol" — Core+short (≤3 lines after frontmatter)
- "What does NOT go in docs/" — DNC: anti-pattern list
- "File conventions" — DNC: output template (frontmatter YAML must be exact)
- Templates (T7 keep): directory layout tree, frontmatter YAML, bootstrap-new-topic-file — exact-substitutable
- Companion-skills section (10-skill list) — kept inline (T1 reversal would fire; load_freq=trigger_freq)

## finalsite-team-mapping (added 2026-04-29; tier-expanded 2026-04-29)
- Workflow steps 1–7 — ⊘T9 (judgment cues: "whichever comes first", "when in doubt", "only when assignee names don't pattern-match")
- Hard rules — ⊘T9 (judgment-heavy: "patterns observed", "however long the table gets")
- **Hard rule about Tier labeling** — DNC: anti-pattern guard ("never label Tier C as 'low performer'"); judgment-heavy + policy-load-bearing per `docs/INDEX.md`
- Step 7 output template (10-item numbered structure incl. tier-distribution + leaderboard) — DNC: exact template
- Skill arguments — Core+short (≤3 lines + key=value template)
- Repo→team baseline mapping table — DNC: lookup data; primary value of skill
- Issuetype classification buckets (bug/qa/feat/unk) — DNC: behavioral routing
- **Tier rule table (A/B/C with rank/share thresholds)** — DNC: exact rule definitions, Core+short
- **"Tiers measure ticket volume only" methodology callout** — DNC: judgment-heavy ("a senior who mentors may sit in Tier C while shipping more value than Tier A"); the framing exists to prevent volume→performance conflation per `docs/INDEX.md` policy

## Future runs
When auditing any skill, check this index BEFORE applying T6/T7/T9 to a section. If listed here, preserve.
