# Prompt Optimizer — Prose Source
> Human-readable source. Edit here first, then re-encode SKILL.md.
> Last updated: 2026-04-26

## Purpose

Maximize Claude output quality per input token. Covers one-shot prompts, system prompts, multi-turn conversation starters, and agentic system prompts. Prompts typically suffer from over-explanation rather than structural bloat.

## Skill File Detection

Before optimizing, check whether the input is a skill file or a prompt. If the input contains any of the following, treat it as a skill file and redirect to skill-token-optimizer rather than proceeding:

- YAML frontmatter (`---` at the top, with `name:` or `description:` fields)
- A filename or path ending in `SKILL.md`
- References to `references/prose-source.md`, `_shared/`, or L1/L2/L3 architecture language
- Instructions structured as "when triggered, do X" (skill behavioral framing rather than task framing)

When redirecting, say: "This looks like a skill file rather than a prompt — the skill-token-optimizer is the right tool for this. Want me to use that instead?"

If ambiguous (e.g., a pasted block with no frontmatter that could be either), ask: "Is this a skill file or a prompt you're using to call Claude?"

## Iteration Workflow

The skill operates as a loop, not a one-shot rewrite:

1. **Diagnose** — match symptoms to the failure taxonomy. Don't rewrite without a named failure mode.
2. **Choose architecture** — one-shot, system, or agent. Mismatched architecture is itself a common failure mode.
3. **Rewrite** — apply the relevant template and density techniques.
4. **Test 3×** — run at `temperature: 1.0` (or three manual runs). Fail if outputs differ in structure, recommendation, or first-token category. Variance type → diagnosis:
   - Structure varies → format underspecified
   - Recommendation varies → role/persona underspecified
   - First-token varies → no output anchor
5. **Iterate** — fix one failure mode per pass; re-test.

## Failure Taxonomy

Diagnose before optimizing.

| Failure | Symptom | Fix |
|---|---|---|
| Under-constrained | Output varies wildly across runs | Add output format + exemplar |
| Over-explained | Long prompt, mediocre output | Strip rationale; add constraints |
| Role confusion | Generic, timid responses | Strong role declaration + anti-patterns |
| Context overload | Claude loses the thread | Move bulk context to the Files API or a separate user turn; keep instructions lean |
| Format underspecified | Inconsistent structure | Specify format with a template or prefill |
| Implicit audience | Mismatch in depth/tone | State the reader explicitly |
| Missing negative space | Claude hedges where you want conviction | Add "do not X" constraints |
| Wrong architecture | Reactive prompt used for autonomous loop, or vice versa | Switch to agent template if sequential tool calls are required |

## Claude-Specific Patterns

These exploit features Claude is trained on or that the Anthropic API exposes. Use them before generic prompt-engineering tricks.

### XML structural tags
Claude is trained to attend to XML-style tags as section delimiters. They cost a few tokens but improve attention on the right span and reduce instruction-context bleed.

```
<instructions>
You are a senior SaaS CFO. Recommend, do not summarize.
</instructions>

<context>
{rev_data}
</context>

<example>
Recommendation: Cut sales hiring 30%. Rationale: CAC payback exceeds 24 months.
</example>
```

Use when context, instructions, and examples are all present in one prompt. Skip for trivial 1–2 line prompts.

### Assistant prefilling
The Anthropic API accepts a partial `assistant` turn that Claude continues from. This is the cleanest way to enforce output format, eliminate preamble, and skip refusals on benign tasks.

- JSON-only output: prefill `{`
- Skip preamble: prefill `Recommendation:`
- Force XML structure: prefill `<analysis>`

This is more reliable than instructing "respond with only JSON" — instructions can be ignored, prefills cannot.

### System param vs. first user turn
- **System param**: stable role, standing constraints, format defaults. Cached by default. Strong instruction-following.
- **First user turn**: task-specific context, dynamic data, examples that change per request. Cached only if explicitly marked.
- **Anti-pattern**: putting per-request data in the system param defeats caching across users.

### Model-aware prompting
Different models tolerate different prompt density:
- **Haiku 4.5** — weaker persona loading; needs explicit constraints, exemplars, and stopping conditions. Add ~30% more scaffolding than for larger models.
- **Sonnet 4.6** — handles concept pointers and persona loading well. Default target.
- **Opus 4.7** — strong implicit reasoning; over-specification can degrade output. Lean on persona + terminal goal; cut step-by-step instructions unless the domain requires them.
- **Extended thinking** (Opus/Sonnet) — for tasks where reasoning steps matter, enable thinking instead of writing "think step by step" into the prompt. Thinking tokens are not user-visible and don't bloat the response.

### Few-shot examples
- 1 example: enforce format. 3+ examples: teach a pattern. More than 5: diminishing returns and high token cost.
- Order matters — Claude weights later examples more heavily. Put the canonical example last.
- Label format consistently across examples (e.g., always `Input: ... / Output: ...`).
- Wrap examples in `<example>` tags so Claude doesn't confuse them with the live request.

### Tool use (agent prompts)
For prompts driving the Anthropic tool-use API:
- `tool_choice: "auto"` — default, Claude decides. Most flexible.
- `tool_choice: {"type": "tool", "name": "X"}` — force a specific tool. Use when the first action is deterministic.
- `tool_choice: "any"` — force *some* tool but not which. Useful for "must take an action" loops.
- Parallel tool calls: enabled by default; disable when later calls depend on earlier results.
- Tool descriptions are part of the system prompt budget — write them with the same density rules.

## One-Shot Prompt Architecture

```
[ROLE] You are [specific role with domain + posture].
[TASK] [Verb] [object] [constraints].
[CONTEXT] [Only what changes the output. Nothing else.]
[FORMAT] [Exact structure. Include exemplar for non-obvious formats.]
[CONSTRAINTS] Do not [anti-pattern]. Do not [hedge behavior to suppress].
```

Cut: explanation of why you want the output | background that doesn't change the answer | politeness ("please", "could you") | redundant restatements | "think step by step" (use extended thinking instead if available).

Keep: any constraint that would change the output if removed | output format if non-obvious | negative constraints that prevent the most common failure mode.

## System Prompt Architecture

System prompts load on every turn — token cost compounds. Structure for cache efficiency and behavioral precision.

### Layering (stable → variable)
- Layer 1 (cache): Role + domain + persona + output format defaults
- Layer 2 (cache): Standing constraints + anti-patterns + tone rules
- Layer 3 (don't cache): Session context + user preferences + live data

Put layers 1–2 first, always. Mark the cache breakpoint with `cache_control: {"type": "ephemeral"}` on the last content block of layer 2 in the system array.

### Caching mechanics
- **Read cost**: cached prefix tokens cost 0.1× the normal input rate (90% savings on cache hits).
- **Write cost**: the request that establishes a cache entry costs 1.25× the normal rate. Caching is only net-positive after ~2 reuses of that prefix.
- **TTL**: default 5 minutes, rolling — refreshes on each hit. The `extended-cache-ttl-2025-04-11` beta enables a 1-hour TTL at 2× write cost — worth it for low-traffic system prompts that would otherwise miss the 5-min window.
- **Breakpoints**: up to 4 `cache_control` markers per request. Place at the end of stable spans.

### System Prompt Compression
- **Role**: One sentence. Domain + posture. No backstory.
- **Standing constraints**: Bullets, not paragraphs. Imperatives only.
- **Output format**: One exemplar > three paragraphs of description.
- **Anti-patterns**: "Do not X" is ~3 tokens. A paragraph explaining why X is bad is ~40 tokens. Cut the explanation.

### System Prompt Template
```
You are [role] for [product/context]. [One sentence on posture/goal].

Rules:
- [Constraint 1]
- [Constraint 2]
- Do not [anti-pattern]

Output format:
[Exemplar or schema]
```

### Multi-turn
System prompt = stable role + constraints + format, no session context. First user turn = task + session context. Subsequent turns = new info only — trust history. Anti-pattern: restating system prompt content in each user turn defeats caching and bloats the conversation.

## Semantic Density Techniques

**Concept pointers**: Replace explained concepts with named ones Claude knows. "Structure so categories don't overlap and everything is covered" → "Apply MECE structure." ~20 tokens saved per instance. Safety rule: only use for concepts with a single canonical meaning. "MECE" is safe; "BLUF" is regional (military/business but not universal); "GROW model" loads coaching context (wrong domain in a finance prompt). When in doubt, define inline.

**Exemplar over description**: One concrete example of desired output > 100 words describing it.

**Constraint inversion**: Describe the failure mode instead of the desired quality. "Write with conviction and make clear recommendations" → "Do not hedge or present options without a recommendation."

**Output anchoring (prefill)**: The most reliable form is API-level assistant prefill (see Claude-Specific Patterns). When prefill isn't available (Workbench-style UIs without prefill support), instruct: "Start with the recommendation, not the analysis."

**Persona loading**: Strong role declaration activates implicit behaviors. "Senior SaaS CFO advising a PE-owned portfolio company" loads financial rigor, PE awareness, exit lens, directness. Explicit instructions overlapping the loaded persona are redundant tokens.

## Agent System Prompt Architecture

Agent prompts have unique failure modes: tool misuse, goal drift, unnecessary loops, and unsafe autonomous actions. Optimize for behavioral precision over token savings — wrong agent behavior costs more than extra tokens.

### Failure Modes (Agent-Specific)

| Failure | Symptom | Fix |
|---|---|---|
| Goal drift | Agent pursues sub-goals, loses the original task | Restate terminal goal + stopping condition |
| Tool overuse | Agent calls tools when it already has the answer | Add "Only use tools when X" constraint |
| Unsafe autonomy | Agent takes irreversible actions without checking | Add explicit confirmation gates |
| Loop traps | Agent retries the same failing tool call indefinitely | Add max-retry + fallback instruction |
| Under-scoped tools | Agent can't complete task; halts or hallucinates | Audit tool list against task requirements |
| Role confusion | Agent behaves as a chatbot, not an autonomous actor | Strong agentic role declaration + `tool_choice` tuning |

### Agent Prompt Template

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

### Agent-Specific Optimization Rules

- **Terminal goal clarity**: State the end state, not the process. "Return a JSON summary of all open tasks" not "look through tasks and summarize them."
- **Stopping conditions are required**: Agents without explicit stopping conditions over-run or loop. Every agent prompt must specify when done.
- **Tool scoping**: List exactly which tools may be used and under what conditions. Unrestricted tool access leads to creative misuse. Pair with `tool_choice` and parallel-call settings (see Claude-Specific Patterns → Tool use).
- **Confirmation gates**: For irreversible actions (delete, send, publish, pay), require an explicit check-in before acting.
- **Fallback chains**: "If the tool returns an error, [specific fallback]." Without this, agents loop or hallucinate.

### Agent vs. System Prompt Distinction

Use agent architecture when the task requires multiple tool calls in sequence, Claude must make decisions between steps without human input, and there is a clear terminal goal (not an ongoing assistant role). Use standard system prompt architecture when Claude is responding to user turns reactively with no autonomous tool loop required.

## LLM-as-Judge Prompt Architecture

A judge prompt asks a model to score a candidate output against a reference (ground truth, rubric, or both) and emit a structured verdict. Failure modes are different from chat or agent prompts: judges silently demote correct answers on non-scoring details, flap on borderline cases, and drift toward the curator's exact phrasing. Optimize for **score validity**, not output style.

### Judge-specific failure modes

| Failure | Symptom | Fix |
|---|---|---|
| Phrasing pedantry | Demotes for title spelling, citation format, paraphrase | Explicit non-scoring list (see below) |
| Borderline flapping | Verdict swings between runs on same input | Tie-break heuristic + concrete examples |
| Note-mimicry | Recites curator's note as the rationale; ignores rules that contradict it | Worked examples that contradict notes; rules-then-examples ordering |
| Self-judge ceiling | When the judge IS the gen model, residual error plateaus ~0.85–0.90 | External judge OR self-consistency (N-sample majority) |
| Verbose rationale | Long explanations crowd out the verdict | Length cap + JSON schema with one-sentence rationale field |

### Judge prompt template

```
You are a strict evaluator of [domain]. You will be given:
  - INPUT: the original task / question / source
  - REFERENCE: the canonical correct answer + context
  - CANDIDATE: the model's output to score

Score the MAIN factual claim of CANDIDATE against REFERENCE. The following are NEVER part of the score:
  - [list of non-scoring dimensions specific to the domain — e.g. citation numbers, exact titles, phrasing, additional accurate context, which specific part of a multi-step event is named]

Verdicts:
- "correct": main claim matches REFERENCE. When in doubt between "correct" and "partial", prefer "correct".
- "partial": right broader entity but a substantive secondary fact is wrong. Use sparingly — non-scoring differences above are not partials.
- "wrong": main claim conflicts with REFERENCE.

Worked examples — these are CORRECT:
  - [concrete example demonstrating non-scoring rule #1]
  - [concrete example demonstrating non-scoring rule #2]
  - [concrete example demonstrating non-scoring rule #3]

Output JSON: {"verdict": "correct" | "partial" | "wrong", "rationale": "<= 200 chars"}
```

### Judge optimization rules

- **Non-scoring list is load-bearing**. Smaller / weaker judges silently demote on every difference unless told what doesn't count. The list is the single biggest determinant of judge accuracy.
- **Worked examples beat rules** for weaker judges. 2–3 concrete CORRECT-flagged examples teach the non-scoring rules better than a longer rules block. Place examples *after* the rules section so they reinforce, not replace.
- **Tie-break heuristic** ("when in doubt, prefer correct" or its inverse, depending on which error is costlier) collapses borderline flapping. Pick the direction that matches the cost asymmetry of the use case.
- **Same-model judging** (judge is the gen model) has a hard accuracy ceiling around 0.85–0.90. Past that, prompt iteration alone can't move the number; you need an external judge (stronger model) or majority voting (3–5 samples, mode wins).
- **Verdict shape**: prefer JSON-mode / `response_schema` over inline format strings. Length-cap the rationale (one sentence, ≤200 chars) — long rationales correlate with verdict drift.
- **Don't pass the curator's note verbatim into the rationale prompt** unless the rules explicitly contradict it. Otherwise the judge mimics note phrasing instead of applying rules.

### Judge eval & calibration

A judge prompt is itself an artifact under test. Calibrate against a gold-labeled subset:

1. Hand-score 30–50 candidate outputs as the ground-truth oracle.
2. Run the judge prompt against the same set.
3. Confusion matrix vs. oracle. Target: ≥0.85 agreement on `correct`, ≥0.70 on `partial`, ≤5% false `correct`.
4. Iterate the prompt; re-run. Each prompt change gets a calibration round.

Pin the noise band: ±5 pts single-shot, ±2 pts with N=3 majority. Don't react to deltas inside the band.

## Prompt Quality Metrics

| Metric | How to measure |
|---|---|
| Token count | English prose: `word_count ÷ 0.75`. Code/JSON/XML: `char_count ÷ 3` (denser than prose). CJK: `char_count ÷ 1.5`. Anthropic Workbench shows exact counts. |
| Output consistency | Run 3× at `temperature: 1.0`; compare structure, recommendation, first-token category |
| Format compliance | Output match specified format on first try? |
| Failure rate | % runs requiring follow-up correction |

Target: output consistency >90% across 3 runs with no follow-up required.

## Defaults
Detect skill file before optimizing — redirect if needed | diagnose failure mode before rewriting | strip rationale before adding constraints | always specify output format | use exemplar for non-obvious formats | cache-order system prompts and mark the breakpoint | for agents: confirm stopping condition + safety gates present | test 3× at `temperature: 1.0` before declaring done.

## Output Formats
- **Audit**: current prompt → failure taxonomy match → token count → classified waste → rewrite plan
- **Optimized prompt**: new prompt + token delta + techniques applied + behavioral changes
- **System prompt**: layered structure with cache breakpoint marked + read/write cost estimate
- **Agent prompt**: failure mode diagnosis → architecture template filled → gates + fallbacks verified
- **A/B comparison**: original vs. optimized side-by-side + predicted improvement per metric
