---
name: prompt-optimizer
description: >
  Optimizes one-shot, system, multi-turn, and agent prompts for token efficiency and output quality.
  Covers Claude-specific patterns: XML tags, prefilling, prompt caching, model selection, tool use.
  Use for: shrinking prompts, improving output consistency, fixing under-specified prompts,
  agent prompt design, fixing goal drift / loops / unsafe autonomy / tool misuse.
  Trigger: "optimize this prompt", "prompt too long", "Claude misunderstanding X", "write a system prompt",
  "agent system prompt", "agent looping", "agent unsafe", "make this prompt cheaper".
  skip: [skill files] → skill-token-optimizer
---

# Prompt Optimizer
> Prose source: `references/prose-source.md` — edit there first, then re-encode here.

Maximize Claude output quality per input token. Prompts suffer from over-explanation, not structural bloat.

---

## Skill File Detection

Redirect to skill-token-optimizer if input contains any of: YAML frontmatter (`---` with `name:`/`description:`) | path ending in `SKILL.md` | references to `_shared/` or L1/L2/L3 architecture | behavioral framing ("when triggered, do X").

Redirect: *"This looks like a skill file — skill-token-optimizer is the right tool. Want me to use that instead?"* If ambiguous: *"Is this a skill file or a prompt you're using to call Claude?"*

---

## Iteration Workflow

1. **Diagnose** — match symptoms to taxonomy. No rewrite without a named failure mode.
2. **Choose architecture** — one-shot / system / agent. Wrong architecture is itself a failure mode.
3. **Rewrite** — apply template + density techniques.
4. **Test 3×** at `temperature: 1.0`. Variance type → diagnosis: structure → format underspecified | recommendation → persona underspecified | first-token → no output anchor.
5. **Iterate** — fix one failure mode per pass.

---

## Failure Taxonomy

| Failure | Symptom | Fix |
|---|---|---|
| Under-constrained | Output varies wildly | Add format + exemplar |
| Over-explained | Long prompt, mediocre output | Strip rationale; add constraints |
| Role confusion | Generic, timid | Strong role + anti-patterns |
| Context overload | Loses the thread | Bulk context → Files API or separate user turn |
| Format underspecified | Inconsistent structure | Template or prefill |
| Implicit audience | Wrong depth/tone | State the reader |
| Missing negative space | Hedges | "Do not X" constraints |
| Wrong architecture | Reactive prompt for agentic loop, or vice versa | Switch architecture |

---

## Claude-Specific Patterns

Use these before generic prompt-engineering tricks.

**XML structural tags**: Claude is trained to attend to `<instructions>`, `<context>`, `<example>`. Improves attention on the right span; reduces instruction-context bleed. Skip for trivial 1–2 line prompts.

**Assistant prefilling**: API accepts a partial `assistant` turn Claude continues from. More reliable than instructing format. Examples: prefill `{` for JSON-only | `Recommendation:` to skip preamble | `<analysis>` to force XML.

**System param vs. first user turn**: System = stable role + standing constraints + format defaults (cached by default). First user turn = task + session context. Anti-pattern: per-request data in system param defeats caching across users.

**Model-aware prompting**:
- **Haiku 4.5** — weaker persona loading; add ~30% more scaffolding (explicit constraints, exemplars, stopping conditions).
- **Sonnet 4.6** — handles concept pointers + persona loading well. Default target.
- **Opus 4.7** — strong implicit reasoning; over-specification degrades output. Lean on persona + terminal goal; cut step-by-step. Thinking content omitted from responses by default; opt in with `thinking: {type: "enabled", display: "summarized"}` to surface progress on long tasks.
- **Extended thinking** (Opus/Sonnet) — enable instead of "think step by step." Thinking tokens don't bloat response. Cached thinking carries 1-hour TTL.
- **Server-side context compaction** (Anthropic, 2026): for agent loops past ~150K tokens the API summarizes earlier turns server-side. Don't write your own summarizer — opt in via API parameter; reserve budget for the compaction prompt.
- **Structured outputs**: prefer `response_schema` / JSON-mode parameter over prefilling `{`. Server-side schema enforcement is more reliable.

**Few-shot examples**: 1 = enforce format, 3+ = teach a pattern, >5 = diminishing returns. Order matters — canonical example last. Wrap in `<example>` tags. Use consistent labels (e.g., `Input: ... / Output: ...`).

**Tool use** (agent prompts): `tool_choice: "auto"` (default, flexible) | `{"type": "tool", "name": "X"}` (force specific) | `"any"` (force *some* tool). Disable parallel calls when later calls depend on earlier results. Tool descriptions count toward system prompt budget — apply density rules.

---

## One-Shot Prompt Architecture

```
[ROLE] You are [specific role with domain + posture].
[TASK] [Verb] [object] [constraints].
[CONTEXT] [Only what changes the output. Nothing else.]
[FORMAT] [Exact structure. Exemplar for non-obvious formats.]
[CONSTRAINTS] Do not [anti-pattern]. Do not [hedge behavior to suppress].
```

**Cut**: rationale | background that doesn't change the answer | politeness | redundant restatements | "think step by step" (use extended thinking instead if available).
**Keep**: any constraint that changes output if removed | output format if non-obvious | negative constraints preventing the most common failure mode.

---

## System Prompt Architecture

Token cost compounds. Structure for cache efficiency.

```
Layer 1 (cache): Role + domain + persona + output format defaults
Layer 2 (cache): Standing constraints + anti-patterns + tone rules
Layer 3 (no cache): Session context + user preferences + live data
```

Mark cache breakpoint with `cache_control: {"type": "ephemeral"}` on the last content block of layer 2.

**Caching mechanics**:
- Read cost: 0.1× input rate (90% savings on hits).
- Write cost: 1.25× on the establishing request; net-positive after ~2 reuses.
- TTL: 5 min default (rolling — refreshes on hit). **1-hour TTL** is GA (2× write cost) — worth it for low-traffic system prompts missing the 5-min window, and long-running agent loops where system prompt + tool defs stay stable.
- Up to 4 breakpoints per request; place at end of stable spans.
- **Automatic prompt caching** (Anthropic, Q1 2026): API opportunistically caches eligible prefixes without explicit `cache_control` markers on many models. Treat manual breakpoints as a hint improving hit rate, not a requirement; verify via `usage.cache_read_input_tokens`.
- **Workspace isolation**: cache entries scoped per Anthropic workspace (Feb 2026). Cross-workspace prompts can't share — relevant for multi-tenant orgs splitting prod/staging.

**Rules**: Role → one sentence, domain + posture, no backstory | Constraints → bullets, imperatives only | Format → one exemplar > three paragraphs | Anti-patterns → "Do not X" (~3 tok) not paragraphs (~40 tok).

**Multi-turn**: System = stable role + constraints + format. First turn = task + session context. Later turns = new info only. Anti-pattern: restating system content per turn defeats caching.

**Template**:
```
You are [role] for [product/context]. [One sentence posture/goal].
Rules:
- [Constraint]
- Do not [anti-pattern]
Output format: [Exemplar or schema]
```

---

## Agent System Prompt Architecture

Agent prompts fail differently — optimize for behavioral precision first, tokens second. Wrong agent behavior costs more than extra tokens.

**Agent-specific failure modes:**

| Failure | Symptom | Fix |
|---|---|---|
| Goal drift | Sub-goals displace original task | Terminal goal + stopping condition |
| Tool overuse | Calls tools when answer is known | "Only call tools when [condition]" |
| Unsafe autonomy | Irreversible actions without checking | Explicit confirmation gates |
| Loop traps | Retries same failing call | Max-retry + fallback |
| Under-scoped tools | Halts or hallucinates | Audit tool list vs. requirements |
| Role confusion | Behaves as chatbot | Agentic role + `tool_choice` tuning |

**Agent template:**
```
You are [agent role] for [context]. Goal: [terminal goal]. Stop after completing it.
Tools: [list]. Only call a tool when [specific condition].
Loop: 1) Observe [what] 2) Plan next action 3) Act 4) Verify success.
Done when: [exact completion state].
Gates: Before [irreversible action], state what you're about to do and wait for approval.
Fallback: If [failure], [recovery].
Budget: No more than [N] tool calls before reporting back.
Format: [schema]
```

**Key rules:**
- **Terminal goal**: end state, not process. "Return JSON of open tasks" not "look through tasks and summarize."
- **Stopping condition required**: omitting → over-run or loops.
- **Tool scoping**: list which tools and under what conditions. Pair with `tool_choice` (see Claude-Specific Patterns).
- **Confirmation gates**: before delete/send/publish/pay.
- **Fallback chains**: "If tool returns error, [specific recovery]."

**Agent vs. system prompt**: Agent architecture when sequential tool calls are required, Claude decides between steps without human input, and there's a clear terminal goal. Standard system prompt when Claude responds reactively with no autonomous loop.

---

## LLM-as-Judge Prompt Architecture

A judge prompt scores a candidate output against a reference and emits a structured verdict. Optimize for **score validity**, not output style.

**Judge-specific failure modes:**

| Failure | Symptom | Fix |
|---|---|---|
| Phrasing pedantry | Demotes for title spelling, citations, paraphrase | Explicit non-scoring list |
| Borderline flapping | Verdict swings between runs | Tie-break heuristic + worked examples |
| Note-mimicry | Rationale recites curator's note; ignores rules | Worked examples that contradict notes |
| Self-judge ceiling | Same model judges itself; plateaus ~0.85–0.90 | External judge OR N-sample majority |
| Verbose rationale | Long explanations crowd out the verdict | Length cap + JSON schema |

**Judge template:**
```
You are a strict evaluator of [domain].
Score the MAIN factual claim of CANDIDATE against REFERENCE.

NEVER part of the score:
  - [non-scoring dimension #1, e.g. exact titles]
  - [non-scoring dimension #2, e.g. additional accurate context]
  - [non-scoring dimension #3, e.g. which specific part of an arc]

Verdicts:
- "correct": main claim matches. When in doubt, prefer "correct".
- "partial": right entity, substantive secondary fact wrong. Use sparingly.
- "wrong": main claim conflicts with REFERENCE.

Worked examples — these are CORRECT:
  - [example contradicting non-scoring rule #1]
  - [example contradicting non-scoring rule #2]

Output JSON: {"verdict": "correct"|"partial"|"wrong", "rationale": "≤200 chars"}
```

**Key rules:**
- **Non-scoring list is load-bearing** — biggest single determinant of judge accuracy.
- **Worked examples beat rules** for weaker judges. Place AFTER rules to reinforce.
- **Tie-break heuristic** ("when in doubt, prefer X") collapses borderline flapping.
- **Self-judge ceiling**: ~0.85–0.90 is the floor; past that needs external judge or majority voting (3–5 samples, mode wins). Pin a noise band: ±5 pts single-shot, ±2 pts with N=3.
- **Calibrate against a gold-labeled subset** of 30–50 outputs hand-scored as ground-truth oracle. Confusion matrix vs. oracle every prompt change.

For RAG-eval-specific judge framing (recall@K vs answer_correct@1, retrieval-vs-generation attribution) → `rag-conventions` § Evaluation.

---

## Semantic Density Techniques

- **Concept pointers**: Replace explained concepts with named ones (e.g., "MECE structure" not "categories that don't overlap and cover everything"). Safety rule: only for concepts with a single canonical meaning. "MECE" is safe; "BLUF" is regional; "GROW model" loads coaching context (wrong domain in a finance prompt). When in doubt, define inline.
- **Exemplar over description**: One concrete output example > 100 words describing it.
- **Constraint inversion**: Describe the failure mode, not the desired quality ("Do not hedge" not "write with conviction").
- **Output anchoring**: API prefill is the reliable form (see Claude-Specific Patterns). When prefill isn't available, instruct: "Start with the recommendation, not the analysis."
- **Persona loading**: Strong role declaration activates implicit behaviors — don't restate what the persona already loads.

---

## Token Estimation
- English prose: `word_count ÷ 0.75`
- Code/JSON/XML: `char_count ÷ 3` (denser than prose; word-based estimates undercount)
- CJK: `char_count ÷ 1.5`
- Anthropic Workbench shows exact counts.

---

## Defaults
Detect skill file → redirect if needed | diagnose failure mode before rewriting | strip rationale before adding constraints | always specify output format | exemplar for non-obvious formats | cache-order system prompts and mark the breakpoint | for agents: confirm stopping condition + safety gates | test 3× at `temperature: 1.0` before done.

## Output Formats
- **Audit**: current prompt → failure taxonomy match → token count → classified waste → rewrite plan
- **Optimized prompt**: new prompt + token delta + techniques applied + behavioral changes
- **System prompt**: layered structure with cache breakpoint marked + read/write cost estimate
- **Agent prompt**: failure mode diagnosis → architecture template filled → gates + fallbacks verified
- **A/B comparison**: original vs. optimized side-by-side + predicted improvement per metric
