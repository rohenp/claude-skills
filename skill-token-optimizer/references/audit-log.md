# Skill Token Optimizer — Audit Log
> Append-only. One entry per optimization run. Never edit or delete prior entries.
> Format: see prose-source.md § Audit Log Convention.
> Compaction: see `audit-log-archive/` for older entries; current state in `preserved-sections.md`.

---

## 2026-05-01 [skill-token-optimizer] (re-encode tool-choice rule)
Action: skipped-trivial — pre-flight fired (diff = 1 ln on SKILL.md, L1 unchanged)
Trigger: 2026-04-30 #7 finalsite-committer-effectiveness audit was deferred to a later session because multi-Edit on a 600-line SKILL.md hit token-budget pressure. Codified the lesson as a step-3 sub-rule under Skill Creation & Edit Integration.
Rule added: prefer single `Write` for substantive re-encodes — net additive prose growth ≥10%, ≥3 new sections, or change touching >200 lines of SKILL.md. Use `Edit` only for narrow targeted changes (≤2 sections, ≤50 lines net) with clean uniqueness. Heuristic: if you'd need ≥4 sequential Edits to mirror prose, do one Write.
Files edited: `references/prose-source.md` (one Edit; 1 new bullet under Steps §3) → `SKILL.md` (one Edit; mirrored). The SKILL.md mirror was itself a clean ≤2-section ≤50-line change — the canonical case where Edit is the right tool, demonstrating the rule's own Edit-vs-Write boundary.
L1 changes: none — auto-trigger description already covers re-encode behavior.
Notes: rule is DNC-classified (behavioral instruction with judgment cues "≥10%", "≥3 sections", "≥4 Edits") — T10 gate would fire on any T9 attempt. No compression applied or warranted.

## 2026-04-30 [skill-token-optimizer] (prose-source-only edit rule)
Action: audit-only — growth-justified, behavioral DNC
Trigger: user reported a wasteful pattern where edits land in SKILL.md first, then get hand-copied to prose-source, then trigger the optimizer (which re-encodes prose into SKILL.md and discards the hand-compression). Tightened the workflow rule from "edit prose first, backport allowed" to "edit prose only; SKILL.md is a build artifact — never hand-edit it."
Before: SKILL ~1850 tok (247 ln) | prose-source ~3300 tok (488 ln) | L1 ~1010/1024
After:  SKILL ~1930 tok (251 ln) | prose-source ~3400 tok (497 ln) | L1 unchanged
Sections changed (3): top warning blockquote (line 15) | Skill Creation & Edit Integration § Steps | Workflow § prose-source-sync (renamed to "prose-source is the only edit surface").
Techniques: none — all 3 modified sections classify as DNC (behavioral triggers + Do-X-not-Y anti-pattern callouts). T10 gate fires (judgment cues, decoding-cost prose). ⊘T9 = 3.
L1 changes: none — auto-trigger description already implies the workflow.
Notes:
  - Net +80 tok (~4%) on SKILL.md, justified: rule prevents recurring hand-edit→backport→re-encode waste cycle.
  - Anti-pattern callout phrasing ("editing SKILL.md → copying diff into prose-source → triggering optimizer") preserved verbatim per Do-X-not-Y DNC rule — compressing would lose the failure-mode framing that makes the rule stick.
  - prose-source.md updated first; SKILL.md mirrored. Workflow now demonstrates its own rule.
  - Source-vs-build-artifact analogy added in prose-source for fluent-reader decoding; mirrored in SKILL.md as compact "don't edit the binary, edit the source and rebuild" pointer.

---

## 2026-04-30 [prompt-optimizer] (LLM-as-judge prompt architecture)
Action: audit-only — growth-justified
Trigger: user folded LLM-as-judge prompt patterns into prompt-optimizer (prose-source.md updated first, then SKILL.md compressed mirror): non-scoring list, tie-break, worked examples for weaker judges, self-judge ceiling, calibration vs gold subset.
Before: SKILL ~1700 tok (191 ln) | prose-source ~1750 tok (231 ln) | L1 ~1010/1024
After:  SKILL ~2150 tok (242 ln) | prose-source ~2300 tok (286 ln) | L1 unchanged
Techniques: T6 (instruction density) on the SKILL.md mirror — prose-source carries full template + calibration steps + post-rules, SKILL.md compresses to 5-row failure table + skeleton template + 5-bullet key rules.
⊘T9 rationale:
  - Failure-mode table — DNC: structural reference for diagnosis.
  - Judge template — DNC: exact runnable template; the structure IS the convention.
  - 5 key rules — judgment cues ("biggest determinant", "use sparingly"); ⊘T9.
  - Cross-link to rag-conventions § Evaluation — keeps prompt-optimizer focused on the prompt-design lens; RAG-eval-specific framing lives upstream.
L1 changes: none — at 99% utilization. Existing trigger "write a system prompt" + "agent system prompt" cover the new section. Considered adding "LLM-as-judge prompt" trigger; deferred — over-budget risk outweighs the marginal recall gain since RAG-eval users will hit the skill via rag-conventions cross-link.
Notes:
  - Bracket: domain-expert (1500–2500 tok). Post-edit 2150 tok stays in bracket.
  - prose-source.md is canonical for this skill; SKILL.md is the compressed mirror. Both updated.
  - Cross-references intentionally bidirectional with rag-conventions: rag-conventions § LLM-as-judge prompt design covers RAG-eval framing; prompt-optimizer § LLM-as-Judge Prompt Architecture covers prompt-engineering patterns. Each links to the other.
  - The judge template is the highest-leverage piece — concrete, fillable, with the non-scoring list slot explicit. Compressing further would lose the slot structure that makes it copy-paste-usable.

---

## 2026-04-30 [golang-rag-conventions] (existence-check key + local-LLM ops)
Action: audit-only — growth-justified
Trigger: user folded two learnings: (1) existence-check key must equal dedup UNIQUE key on idempotent generators; (2) local LLM stacks (Ollama / llama.cpp / vLLM) have a finite VRAM ceiling and require pre-warming / per-call client config / serialised heavy harnesses.
Before: ~2200 tok (293 ln) | L1 ~1010/1024
After:  ~2700 tok (366 ln) — **+23% growth** (justified, no compression) | L1 unchanged
Techniques: none (T1–T9 not applied)
⊘T9 rationale:
  - Existence-check key Go snippet — DNC: exact pattern; the snippet IS the convention.
  - "Discipline" 1-bullet — ≤3 lines; minimal-section gate.
  - Destructive regen vs in-place backfill — DNC: cross-link + decision rule; pointer to postgres-conventions.
  - Local LLM stacks 5-bullet ops list — judgment-heavy ("transient", "hint not guarantee"); ⊘T9.
  - Per-workload client Go snippet — DNC: exact runnable template w/ inline comments contrasting two timeouts.
  - Two new Pitfalls bullets — Core+short.
L1 changes: none — at 99% utilization (1010/1024); adding triggers risks over-budget. Existing "ingest", "embedding worker", "concurrency or batching" cover the new content.
Notes:
  - Bracket: domain-expert (1500–2500 tok). Post-edit 2700 tok is just over the upper bound. Pre-existing content was already at the high end; the additions push into low comprehensive.
  - prose-source.md does not exist for this skill. SKILL.md is canonical.
  - Per-workload client snippet is high-leverage — many engineers ship a single global Ollama client and silently break ingest under load. The two-snippet contrast is the convention.
  - Cross-link to postgres-conventions § Destructive vs backfill keeps the Go-side note minimal; the full decision flow lives in the DB skill.

---

## 2026-04-30 [postgres-conventions] (destructive vs backfill rule)
Action: audit-only — growth-justified
Trigger: user folded a learning from a destructive RAG migration: when a NOT NULL column can't be reliably reconstructed from existing rows, choose destructive + regen over fragile backfill heuristics. SKILL.md edit synced from references/prose-source.md.
Before: SKILL ~700 tok (101 ln) | prose-source ~750 tok (155 ln) | L1 ~880/1024
After:  SKILL ~810 tok (102 ln) | prose-source ~960 tok (170 ln) | L1 unchanged
Techniques: T6 (instruction density) on the SKILL.md mirror — the prose-source carries the full decision flow + production caveats, the SKILL.md mirror compresses to one bullet under Migration Discipline (the criterion + 3-branch decision flow inline).
⊘T9 rationale:
  - Decision flow (3 branches) — DNC: judgment cues ("if any path picks the wrong value... wrong even if rare"); the rule IS the criterion.
  - Production caveats — judgment-heavy, ≤3 lines each.
L1 changes: none — existing trigger "auditing destructive changes" covers the new rule.
Notes:
  - prose-source.md is canonical for this skill; SKILL.md is the compressed mirror. Both updated; prose carries the production-vs-dev distinction in full, mirror gives the decision flow inline.
  - Bracket: narrow/focused (500–800 tok) for SKILL.md → just inside upper bound at ~810 tok. Bracket: domain-expert lower for prose-source.
  - The "honest" framing in the criterion ("the rule is wrong even if rare") is judgment-load-bearing — drives the operator toward destructive paths in dev/pre-prod where regen is fast, vs nullable+lazy-fill in prod where reconstruction is impossible. Cutting the framing turns the rule into prescriptive advice instead of a decision principle.
  - Cross-link: postgres-chunking-conventions § Sidecar UNIQUE-key dimensions references this back as the destructive-fix path when widening UNIQUE.

---

## 2026-04-30 [postgres-vector-conventions] (multi-arm FTS UNION + stemming)
Action: audit-only — growth-justified
Trigger: user folded two learnings: (1) multi-arm hybrid FTS UNION pattern w/ `MAX(rank)` aggregation; (2) English snowball stemming asymmetries (marriage/marry → different lemmas) discoverable only by running `to_tsvector @@ plainto_tsquery`.
Before: ~3000 tok (255 ln) | L1 ~960/1024
After:  ~3450 tok (310 ln) — **+15% growth** (justified, no compression) | L1 unchanged
Techniques: none (T1–T9 not applied)
⊘T9 rationale:
  - Multi-arm UNION SQL — DNC: exact template; the runnable query IS the convention.
  - "Two non-obvious rules" 2-bullet list — ≤3 lines; minimal-section gate; behavioral.
  - Stemming asymmetries 6-row table — kept (T3 rule says ≥3 rows ok; the table IS the evidence). Could compress to 3 rows but loses the divorce/engage families which provide the "asymmetric across families" point.
  - "Two fixes, applied in order of leverage" — DNC: numbered procedure with rationale.
L1 changes: none — existing trigger "hybrid search Postgres" still pulls the skill in.
Notes:
  - Bracket: comprehensive (2500–4000 tok). Post-edit 3450 tok stays in bracket.
  - prose-source.md does not exist for this skill. SKILL.md is canonical.
  - Cross-link: rag-conventions § Eval metric artefact gates and § LLM-as-judge implicitly reference these patterns; not folded back as the abstraction direction is rag-conventions → postgres-vector-conventions (general → DB-specific).
  - The "verified counterexamples" table is the highest-leverage piece — moves the lesson from "stemming can mismatch" to "here are specific lemma pairs you'll hit". Compressing to a single example loses the family-asymmetry point.

---

## 2026-04-30 [postgres-chunking-conventions] (sidecar UNIQUE-key dimensions)
Action: audit-only — growth-justified
Trigger: user folded a learning from a Postgres RAG migration: when a sidecar carries a typed attribute (kind/version/source_type), the attribute must be in the UNIQUE — narrower UNIQUE makes the generator's iteration order an implicit load-bearing decision.
Before: ~1900 tok (233 ln) | L1 ~720/1024
After:  ~2200 tok (273 ln) — **+16% growth** (justified, no compression) | L1 unchanged
Techniques: none (T1–T9 not applied)
⊘T9 rationale:
  - 2 SQL templates (WRONG/RIGHT pair) — DNC: "Do X, not Y" pattern; exact syntax matters; cutting → brittle.
  - "Why this matters" 3-bullet symptoms — ≤3 lines; minimal-section gate.
  - "Discipline" 3-bullet rules — judgment cues ("typical", "honest", "fragile heuristic"); ⊘T9.
L1 changes: none — existing triggers ("chunk schema", "ingest pipeline Postgres") still cover sidecar-design queries; no new trigger phrase needed because the new section is a refinement of the canonical schema discussion.
Notes:
  - Bracket: domain-expert (1500–2500 tok). Post-edit 2200 tok stays within bracket.
  - prose-source.md does not exist for this skill. SKILL.md is canonical.
  - WRONG/RIGHT SQL pair is the highest-leverage piece — it makes the abstract rule ("UNIQUE must include every distinguishing dimension") concrete in code-example form. Compressing to one example loses the contrast that makes the rule memorable.
  - Cross-link: the new section references postgres-conventions § Destructive vs backfill (a forward reference to an upcoming addition in the same multi-skill update).

---

## 2026-04-30 [rag-conventions] (LLM-as-judge / eval-harness additions)
Action: audit-only — growth-justified
Trigger: user folded learnings from a multi-round RAG eval-harness project into the skill — four new subsections under Evaluation: LLM-as-judge prompt design, Single-shot variance & noise bands, Raw vs judged-only metrics, Artefact gates on metrics.
Before: ~3700 tok (265 ln) | L1 930/1024
After:  ~4400 tok (303 ln) — **+19% growth** (justified, no compression) | L1 930/1024 (unchanged)
Techniques: none (T1–T9 not applied)
⊘T9 rationale: every added subsection is DNC or T10-gated:
  - LLM-as-judge prompt design — behavioral patterns w/ heavy judgment cues ("when in doubt", "self-judge ceiling", "stubborn pedantry"); 4 bullets each ≤3 lines.
  - Single-shot variance & noise bands — 3-bullet metric examples, ≤3 lines each, T10 minimal-section gate fires.
  - Raw vs judged-only metrics — DNC: decision criteria, cutting → brittle edge cases.
  - Artefact gates on metrics — DNC: behavioral instruction; the 3 examples (speaker-hit / recall@K / faithfulness) are exactly the cases the rule must cover.
  - Considered T3 table-form on the 3-row noise-band list — bullets win per "<3 rows → bullets" rule.
L1 changes: none — Covers section already names "LLM-as-judge"; trigger "RAG eval" still pulls in. L1 at 91% utilization; adding more triggers risks over-budget.
Notes:
  - Bracket: comprehensive (2500–4000 tok). Post-edit 4400 tok is just over the upper bound. Pre-existing skill was already at upper end of domain-expert / lower end of comprehensive; the additions push it ~10% over the comprehensive ceiling.
  - Hot skill: any RAG work loads it; cache amortizes the size cost across many turns.
  - prose-source.md does not exist for rag-conventions (only references/lookup-tables.md). Convention "prose-source wins" doesn't apply when the skill never had one. SKILL.md is canonical for this skill.
  - Considered T1 (move LLM-as-judge prompt patterns to references/) — rejected: behavioral instruction is the highest-leverage content for in-context use; pulling to references/ requires an explicit fetch the model rarely thinks to do mid-design.
  - Future: if another 1–2 sections land before next audit, consider extracting the entire Evaluation section to references/eval.md and leaving a behavioral pointer + decision-order line in SKILL.md. Single threshold to revisit.

---

## 2026-04-27 [npm-security]
Action: new skill — full audit on creation
Trigger: user asked to extend / create a security skill so npm package security is thoroughly researched (pinning, freezing, cool-down windows, security-patch override, vetting, install scripts, provenance, audit tooling)

Sizes:
  Before (initial draft):  L1 1437 chars (raw) / ~1437 folded — **OVER 1024 hard limit**; L2 ~1887 tok (1415 wd, 222 ln total file)
  After:                   L1 1051 chars (raw) / 1021 folded — under limit; L2 ~1693 tok (1270 wd, 170 ln total file)
  Reduction: L1 −29% (over-limit → in-limit); L2 ~−10% words / ~−10% tokens

L1 changes (T8 + T8b):
  - Dropped "for the npm ecosystem" (redundant with `npm/pnpm/yarn/bun` package list)
  - Removed verbose "Use when adding/upgrading… or responding to a CVE / GHSA in JS/TS." preamble (trigger keywords already cover use cases)
  - Compressed "Covers:" entries: "Renovate/Dependabot cooldown config" → "Renovate/Dependabot config"; "npm provenance and `npm audit signatures`" → "npm provenance + `audit signatures`"; collapsed "(no ^/~/latest)" → "(no ^/~)"
  - skip-clauses tightened: "[Go module security, go.sum]" → "[Go modules]"; "[LLM/agent input handling, prompt injection]" → "[LLM input, prompt injection]"; "[Next.js routing, App Router patterns]" → "[Next.js routing]"; "[TS strict config, type narrowing]" → "[TS strict config]"
  - Trigger CSV: dropped "GHSA, CVE in node module" → kept (high-recall keywords); dropped "postinstall script" → "postinstall"

Techniques applied:
  - T1 (progressive-disclosure): Renovate JSON config (~16 ln) and Dependabot YAML config (~14 ln) moved to `references/automation-config.md`. SKILL.md replaced with one-line directive summary + pointer. Templates are reference material, not behavioral; behavioral instruction is "configure cool-down via Renovate or Dependabot" + "run one tool per repo".
  - T6 (instruction-density): Dropped preambles ["The lockfile is the source of truth for what installs.", "Install scripts run arbitrary code on every machine that installs the package. Primary attack vector for supply-chain compromise." → "Install scripts run arbitrary code on every install — primary supply-chain attack vector.", "Configure once:" → dropped, "Not every advisory is exploitable for a given project. Document every exception:" → dropped]. Removed "The window is non-negotiable… is not a security justification" (T5 within-skill duplicate — same rule appears in Anti-patterns).
  - T2 (bullet-compression): Lockfile frozen-install command list collapsed from one-bullet-per-tool to single-line CSV with `·` separators. Reduces 5 sub-bullets to 1.
  - T8/T8b (L1 description) — see L1 changes above.

⊘T9 (sections preserved from schema-encoding):
  - Cool-down table — already tight; <15 tok savings; rendering as table beats notation
  - Vetting checklist (numbered 1–6) — judgment cues throughout ("all must check out", "Any single check fails → no install without explicit owner sign-off"); decoding-cost would fail
  - Install Scripts allowlist semantics — DNC (decision criteria with native-module exception logic)
  - Anti-patterns — DNC (verbatim "Do X, not Y" pairs)
  - Security Patches Override numbered procedure — judgment cues ("A High CVE in unreachable code is not an emergency", "Many advisories are scoped to APIs the project never calls")

Notes:
  - New skill: prose-source.md created first (canonical). SKILL.md is the encoded version.
  - L1 was over the 1024-char hard limit on first draft (1437 chars folded). Three L1 compression passes brought it to 1021 — fits with 3 chars of headroom. Trigger-keyword recall preserved (all 15 lockfile/manifest filenames + tool keywords kept).
  - References created: `references/automation-config.md` (Renovate + Dependabot full templates with key-directive callouts).
  - Bracket: domain-expert (1500–2500 tok). Post-opt 1693 tok is in-bracket but above the 800–1200 ideal target. Further compression would require moving the Vetting checklist to references/, but it is the highest-frequency-use section (every new-dep decision) and belongs inline.
  - Behavioral content preserved verbatim: pinning rule, frozen-install command per tool, cool-down windows, security-patch override procedure, vetting checklist, install-script allowlist semantics, audit threshold, exception schema, monorepo override doc requirement, pre-publish hardening, anti-patterns.
  - Did NOT extend an existing skill: api-security is web-API-shape focused; coldfusion-security is CFML-specific; prompt-security is LLM-specific. npm supply chain is its own domain (specific to npm registry, lockfiles, install scripts, provenance) and matches the existing `<tech>-security` skill-naming pattern.



---

## 2026-04-29 [finalsite-docs]
Trigger: new skill created — persistence layer for finalsite-* assessment skill output, backing the /Users/rohen/code/finalsite/docs/ flat-file knowledge base
Before: ~1440 tok (135 ln) | L1 990/1024 (97% — tight margin)
After:  ~1397 tok (135 ln) — 3% reduction | L1 880/1024 (86% — healthy headroom)
Techniques: T8 (L1 preamble tighten — drop "Trigger phrases include", drop redundant articles, compress "the user", "any", "a"), T8b (already applied at draft — skip clause for finalsite-committer-effectiveness), T6 (Google Drive sync intro — drop preamble, "Use the" → "Use", tighten bullet rationales)
⊘T9: 5 sections preserved
L1 changes: tightened preamble & articles; preserved every trigger phrase verbatim; "skip:" clause kept; net -110 chars frees headroom for future trigger additions
Notes:
  - Bracket: domain-expert (1500–2500 tok target 800–1200). Post-opt 1397 tok is below upper bracket bound but above ideal target.
  - ⊘T9 sections (T10 gate fired): "When to use" (judgment cues "non-trivial", "worth preserving"), "Update protocol" (judgment cues "contradicts", "materially changed", "verify before"), "Reading protocol" (≤3 lines after frontmatter — Core+short), "What does NOT go in docs/" (DNC: anti-pattern list), "File conventions" (output template — frontmatter YAML must be exact).
  - Templates preserved (T7 keep): directory layout tree, frontmatter YAML, bootstrap-new-topic-file template — all are exact-substitutable templates, not illustrative.
  - Companion-skills section kept inline (10-skill list) — could T1 to references/, but list is short and load-frequency would equal trigger frequency; T5/T1 reversal would fire. Keep inline.
  - Did NOT extend existing skill: finalsite-* assessment skills produce findings in-context; finalsite-docs is the persistence layer with its own update/read/sync protocol. No overlap with finalsite-tech-inventory (calibration source) or finalsite-cto-engagement (synthesis lens).
  - prose-source.md created at references/prose-source.md and synced.
  - Auto-trigger storm: PostToolUse hook re-fires on every Edit; pre-flight 1 (trivial-skip) catches subsequent <3-line edits within the same audit run. Flagged for awareness — not a defect, but agents calling Edit repeatedly on a single skill should batch into a Write.


---

## 2026-04-29 [finalsite-team-mapping]
Trigger: new skill created — wraps Jira (Rovo MCP) + on-disk GitLab group enumeration into a quarterly refresh of `docs/people/ownership-map.md` (the Jira teams ↔ engineers ↔ repos map).
Before: ~1633 tok (141 ln) | L1 1119/1024 (109% — over hard limit)
After:  ~1345 tok (122 ln) — 17.6% reduction | L1 982/1024 (96% — under)
Techniques: T8 (L1 preamble tighten — drop "(1)..(5)" enumeration verbosity, "via Rovo MCP" → "Rovo MCP", "Pair with X for Y and Z for W" → "Pair: X (blast-radius), Z (code-output validation)"), T1 (Calibration anchors → references/calibration-anchors.md, leave 1-line pointer), T5 (When-NOT-to-use body section deleted — verbatim semantic dup of L1 `skip:` rules), T6 (Hard rules — drop "the whole point is", "If the table is long, the table is long" → "Enumerate every assignee, however long")
⊘T9: 4 sections preserved
L1 changes: 1119 → 982 chars; preserved all 6 trigger phrases; preserved skip rules; got under 1024 budget with 42-char headroom
Notes:
  - Bracket: domain-expert (1500–2500 tok target 800–1200). Post-opt 1345 tok is below upper bracket bound but above ideal target. Cold-skill rule applied: stop after T1+T6 per `cold (rare trigs) → drop "if over" steps; parse-cost > savings`.
  - Cold trigger: skill is intended for quarterly manual refresh, not per-edit. Aggressive T9 schema-encoding not warranted (decode-cost > savings for human-readable workflow).
  - ⊘T9 sections (T10 gate fired): Workflow steps 1–7 (judgment cues "whichever comes first", "when in doubt", "only when assignee names don't pattern-match"; numbered procedure that survives readability test), Hard rules (judgment-heavy: "patterns observed", "however long the table gets"), Step 7 output template (DNC: structure of resulting doc), Skill arguments (≤3 lines + key=value template).
  - prose-source.md created at references/prose-source.md (canonical) and synced verbatim with intent of SKILL.md.
  - references/calibration-anchors.md created with sister-skill cross-reference matrix (114 words, ~152 tok) — reference data, not behavioral.
  - Did NOT extend an existing skill: `finalsite-dependency-ownership-map` is repo→team blast-radius; `finalsite-committer-effectiveness` is per-engineer dossier; `atlassian-rovo` is generic MCP tool-using skill. This skill is the orchestrator for refreshing one specific durable artifact (`ownership-map.md`) and merits standalone identity.
  - Auto-trigger storm: PostToolUse hook re-fires on every Edit; treated subsequent edits within the same audit run as continuations (not fresh audits). Pre-flight 1 trivial-skip would fire on the next genuine sub-3-line edit.

---

## 2026-04-29 compaction — 12 entries archived; preserved-sections.md rebuilt
Older entries (2026-04-25 → 2026-04-27 prior to npm-security) moved to `audit-log-archive/2026-Q2.md`.

---

## 2026-04-29 [finalsite-team-mapping] (re-audit, tier expansion)
Action: audit-only — growth-justified
Trigger: user requested per-team contribution-tier categorization in `docs/people/ownership-map.md`; skill body extended to describe tier computation, leaderboard, and "tiers ≠ effectiveness" framing rule.
Before: ~1345 tok (122 ln) | L1 982/1024
After:  ~1801 tok (138 ln) — **+33% growth** (justified, no compression) | L1 982/1024 (unchanged)
Techniques: none (T1–T9 not applied)
⊘T9 rationale: every added section is DNC or T10-gated:
  - Step 4 tier-rule table — Core+short, exact definitions
  - "Tiers measure ticket volume only" callout — judgment-heavy ("A senior who mentors may sit in Tier C while shipping more value")
  - Step 6 leaderboard bullet — ≤3 lines, behavioral
  - Step 7 output template (10 sections including leaderboard + tier-distribution) — DNC: exact structure
  - New Hard Rule about labeling — judgment cues + anti-pattern preservation ("never label Tier C as 'low performer'"); the rule exists *exactly because* `docs/INDEX.md` policy was written to prevent volume↔performance conflation.
L1 changes: none
Notes:
  - Bracket: domain-expert (1500–2500 tok). Post-opt 1801 tok is now mid-bracket — the additions push past the ideal 800–1200 target but stay within the upper bound.
  - prose-source.md synced (added tier rules + the policy-conflation warning verbatim).
  - Doc sync (`docs/people/ownership-map.md`): tier column injected into all 23 team tables; methodology callout added near the top; per-team tier-distribution table appended to bus-factor section; cross-team leaderboard added; data caveats extended.
  - Cold-skill rule: parse-cost > savings; further compression of judgment-heavy framing would degrade the policy guard the user asked for.
  - preserved-sections.md updated separately for new ⊘T9 sections.


---

## 2026-05-06 [golang-conventions]
Action: audit-only — growth-justified
Trigger: user-driven sweep against Go 1.25 (Aug 2025) + Go 1.26 (Q1 2026) release notes; missing modern-stdlib idioms added.
Before: ~1148 tok (77 ln) | L1 ~360/1024 (35%)
After:  ~1634 tok (89 ln) — **+42% growth** (justified, no body compression) | L1 ~700/1024 (68%)
Techniques: T8 (L1 trigger-keyword expansion only) | ⊘T9: 8 sections preserved
L1 changes: added Covers keywords (slog/runtime/signal/tooling/go fix/govulncheck) + "Includes Go 1.25/1.26 additions:" enumeration (runtime.AddCleanup, slog.NewMultiHandler, slog.GroupAttrs, container-aware GOMAXPROCS, Green Tea GC, net/http.CrossOriginProtection, self-referential generics) — recall lift for queries that previously didn't activate this skill.
⊘T9 rationale (T10 gate fired):
  - Standard Library First table — already at T3 floor (1–3 word headers); 2 new rows (CSRF, Object cleanup) preserve format
  - Modern Stdlib bullets — list format already compact; converting to schema would lose syntactic clarity (decoding-cost fail)
  - Tooling section — judgment cues ("on every change", "on CI", "before review"); Core+short range
  - Concurrency & Performance new bullets (GOMAXPROCS dynamic, Green Tea GC, synctest pointer) — ≤3 lines each, Core+short
  - Standard Service Architecture, Error Handling, Naming & Readability, Package Design — all bullet-tight, ⊘T9
Notes:
  - Bracket: domain-expert (1500–2500 tok target 800–1200). Post-add 1634 tok in-bracket but above ideal target. Body at compression floor — further reduction would require moving Modern Stdlib bullets to references/, but they are highest-frequency-use behavioral content (every Go file touched).
  - prose-source.md updated first (canonical), then SKILL.md re-encoded — workflow followed.
  - New behavioral content preserved verbatim across prose+SKILL: runtime.AddCleanup signature, slog.NewMultiHandler/GroupAttrs intro, signal.NotifyContext cause-on-1.26, reflect iterators, self-referential generics, GOMAXPROCS dynamic guidance, Green Tea GC default, encoding/json/v2 forward-look, go fix modernizer, govulncheck.
  - L1 budget healthy (~324 chars headroom) — room for future Go 1.27 additions without compression.
  - Removed `glog` from "do not use" logger list (long-dead; noise).


---

## 2026-05-06 [golang-tester]
Action: audit-only — growth-justified
Trigger: same sweep as golang-conventions; testing/synctest (Go 1.25, the major test-primitive addition in two releases) was missing entirely.
Before: ~1052 tok (153 ln) | L1 ~370/1024 (36%)
After:  ~1715 tok (213 ln) — **+63% growth** (justified; largest gap of the two) | L1 ~530/1024 (52%)
Techniques: T8 (L1 trigger-keyword expansion) | ⊘T9: 6 sections preserved
L1 changes: added Covers keywords (testing/synctest, fuzz testing, t.ArtifactDir, t.Attr, t.Output, testing/cryptotest, AllocsPerRun parallel guard); added Trigger keywords (synctest, fuzz, t.ArtifactDir). Skill now activates on synctest/fuzz queries that previously bypassed it.
⊘T9 rationale (T10 gate fired):
  - synctest section — code block is exact-syntax template (T7 keep); surrounding prose has judgment cues ("whenever the test would otherwise need a real sleep, poll, or jittered timeout")
  - Fuzzing section — code block is template (T7 keep); 2-sentence prose Core+short
  - Crypto Determinism — 3 sentences, ≤3 lines, Core+short; deprecation line is compliance-flavored (DNC)
  - Allocation Assertions — 2 sentences, Core+short
  - Test Metadata — bullet pair, Core+short
  - Testdata `t.ArtifactDir()` addition — 2 sentences, Core+short
Notes:
  - Bracket move: narrow/focused (500–800) → domain-expert (1500–2500). Post-add 1715 tok in-bracket but above ideal target. Size jump is structural — 5 new sections covering Go 1.25/1.26 testing primitives.
  - prose-source.md updated first (canonical), then SKILL.md re-encoded — workflow followed.
  - synctest is the most consequential addition: prevents an entire class of test flakiness (time.Sleep / polling / jittered timeouts in concurrent tests). Worth the +20 lines.
  - Fuzzing section closes a pre-existing gap — fuzz target syntax has been stable since Go 1.18 but the skill never covered it; trust-boundary code (parsers/decoders/validators) is exactly where fuzz targets pay back.
  - Cryptotest section is migration-driven: Go 1.27 removes the cryptocustomrand fallback, so deterministic crypto tests must migrate now. Behavioral instruction with a deadline.
  - L1 budget healthy (~494 chars headroom).
  - Companion change in golang-conventions: added pointer to synctest under Concurrency & Performance ("see golang-tester") so engineers reading conventions route correctly.
