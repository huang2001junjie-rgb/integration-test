---
name: skill-governor
description: OpenAN skill governance and quality assessment. Reviews and scores the skills of the integration-test repository, designs test prompts, and produces governance reports; after explicit user confirmation, performs controlled single-point optimization on the lowest-scoring dimension. Use when the user asks to evaluate or govern a skill, design test.json/test-prompts, score against the rubric, or optimize a skill in a controlled way.
---

# Skill Governor

A dedicated governance workflow for the skills of the `integration-test` repository. Read-only evaluation by default; never modifies files unless the user explicitly requests and confirms a specific patch.

## Modes

| Mode | Trigger | Modifies files | Output |
|------|---------|----------------|------|
| `review` | Evaluate, score, governance report, quality check | No | Score table, lowest dimension, evidence, improvement suggestions |
| `optimize` | Optimize a skill, modify per the lowest dimension | Yes, requires user confirmation | Optimization proposal, diff, re-review result, keep/revert recommendation |

When no mode is specified, use `review`.

## Inputs

| Input | Required | Notes |
|------|----------|-------|
| `skills_root` | No | Defaults to `skills/` |
| `target_skill` | No | A single skill name; when omitted, scan all skills |
| `mode` | No | `review`, `optimize` |
| `max_rounds` | No | Defaults to 3 in optimize mode |
| `test_file` | No | Prefer `assets/test.json`, then `assets/test-prompts.json` |

## Mandatory References

- `references/openan-skill-rubric.md`: scoring dimensions, weights, and decision rules. Must be read before scoring.
- `references/runtime-neutrality.md`: runtime-neutrality red-flag scan. Must be read when evaluating GitHub/MCP/local-tool skills.
- `references/test-prompt-design.md`: test prompt design rules. Must be read when the test set is missing or additional tests are needed.
- `templates/skill-governance-report.md`: used when producing the governance report.
- Community specifications: must read this skill's bundled `references/coding-specification.md` and `references/test-specification.md`; if the target repository has more specific `AGENTS.md`, coding standards, test standards, or related templates, read them as supplementary evidence.
- If a specification file references other documents not shipped with this skill, only the explicit clauses already read may be used; never assume the referenced file exists, and never treat its absence as the target skill violating the specification.

## Workflow

### 1. Discovery and Modeling

Execute:

- Scan `skills_root/*/SKILL.md`, or read `target_skill/SKILL.md`.
- Read `assets/test.json`; if absent, read `assets/test-prompts.json`.
- If the test file is missing, read the target `SKILL.md` and auto-generate 3 test prompt drafts (typical, complex, failure/ambiguity), then present them to the user for confirmation.
- When the user confirms writing, save as `assets/test.json`; when the user only allows a one-off evaluation, do not write any file and use the drafts for this run only.

Output:

- Evaluation scope
- Test set status of each skill; if drafts were auto-generated, record whether they were written or used only temporarily
- List of invalid skill directories

### 2. Review Scoring

Execute:

- Score against the 10-dimension rubric in `openan-skill-rubric.md`.
- Identify the community specification types the target skill touches, read the applicable standards or templates, and build a "specification requirement -> skill evidence" mapping.
- For each dimension give a 1-10 score, evidence, and the reason for deductions.
- For each test prompt run a Darwin-style A/B effectiveness check: compare `no_skill` and `with_skill`, label `test_mode=full-test/smoke-test` by material sufficiency, and label `execution=subagent/main-agent` by execution method.
- Run the runtime-neutrality scan and record red-flag hits.

D3-specific requirements:

- D3 only evaluates community specifications with explicit documented basis; never infer undocumented community processes.
- D3 evidence must contain the specification source, the specific requirement, and the target skill evidence at the same time.
- When no applicable specification is found, D3 must be marked `evidence-insufficient`; never score from model common sense or describe the skill as conforming to community process.

Output:

- Total score
- Full D1-D10 dimension scores
- Lowest dimension
- Key evidence
- Improvement suggestions
- A/B effectiveness conclusion, full-test / smoke-test case counts, subagent / main-agent execution counts, average delta
- Risk level

Gates:

- **STOP: review mode must not modify files.**
- **STOP: review mode must output the D1-D10 score breakdown of every skill in the conversation report.**
- **STOP: review mode must not write a results file by default; only when the user explicitly requests export may `artifact/skill-governor-results.csv` or `artifact/skill-governor-results.jsonl` be written.**
- **STOP: when the test set is missing, never fabricate D8 results**; first generate test prompt drafts and obtain user confirmation, or explicitly label the run as a temporary smoke-test only.

### 2.5 Darwin-style A/B Effectiveness Verification

Purpose:

- Verify that the target skill actually improves task output, not just that `SKILL.md` is well written.

Execute:

1. For each test prompt run the `no_skill` baseline: do not provide, read, or follow the target skill; complete the task from the prompt and expected only.
2. Run the `with_skill` candidate: provide the target `SKILL.md`, require reading and following the target skill, then complete the task from the same prompt and expected.
3. Both executions must use the same prompt, expected, and optional fixtures; the only difference allowed is whether the target skill is loaded.
4. After both results return, or after the main agent finishes both-side reasoning in fallback, score each side against the observable checkpoints of expected and the optional must_not.
5. Compute `delta = with_skill_score - no_skill_score`.
6. Give a `verdict`: `improved`, `neutral`, or `regressed`.
7. Label `test_mode` by fixture sufficiency:
   - `full-test`: the test case provides fixtures, and the fixtures are sufficient for expected to judge real task quality.
   - `smoke-test`: the test case has no fixtures, or fixtures are insufficient for expected; only process, gates, clarification, and anti-fabrication ability are verified.
8. Label `execution` by execution method:
   - `subagent`: two isolated sub-agents actually ran.
   - `main-agent`: when the platform does not support sub-agents, times out, or lacks tools, the main agent performs the `no_skill` / `with_skill` comparison reasoning.
9. Output the scoring rationale; when a test case has only `id/prompt/expected`, explain which checkpoints expected was broken into.

#### Test Mode Strategy

- Prefer `full-test` by default: if the test case includes fixtures and the fixtures sufficiently support the expected judgment, label `test_mode=full-test`.
- If the test case has no fixtures, or fixtures are insufficient for expected, label `test_mode=smoke-test`.
- A `smoke-test` can still actually launch `no_skill` / `with_skill` sub-agents; it is not downgraded because no agent was started, but because real materials are missing.
- Both `full-test` and `smoke-test` prefer two isolated sub-agents; if the platform does not support sub-agents, times out, or lacks tools, a main-agent fallback is allowed but must be labeled `execution=main-agent`.
- Before batch skill evaluation, present the estimated execution count and wait for confirmation; estimated sub-agent count = `skill count x prompt count x 2`.
- Never call a `smoke-test` a `full-test`; never call `execution=main-agent` an actual sub-agent run.

Decision:

- `improved`: `delta > 0` for most test prompts, with no safety, tool-boundary, or runtime regression.
- `neutral`: output structure is more stable but task quality shows no clear improvement, or positive and negative changes cancel out.
- `regressed`: `with_skill_score < no_skill_score`, or side effects are introduced such as unauthorized operations, excessive process, wrong formats, or misfired tools.

Gates:

- **STOP: never call a `smoke-test` a `full-test`.**
- **STOP: never call `execution=main-agent` an actual sub-agent run.**
- **STOP: if `with_skill` scores below `no_skill`, never recommend keeping directly; output a regression diagnosis and one minimal fix direction.**
- **STOP: D8 only compares `no_skill` vs `with_skill`; `old_skill` vs `new_skill` is only used for the optimize regression check.**
- **STOP: A/B scores must not be bare numbers; expected checkpoint coverage or deduction reasons are required.**

### 3. Optimize: Controlled Modification

Execute:

1. Re-review the current skill; record the baseline score, lowest dimension, and A/B result.
2. Present the "optimization proposal": current lowest dimension, evidence, one single-point modification plan, intended location, expected benefit, risk, and verification method.
3. If the A/B result is `regressed` or `neutral`, the proposal must target the A/B failure points first, not merely add content to raise documentation scores.
4. Wait for explicit user confirmation.
5. Apply the confirmed patch to the target skill.
6. Show the diff.
7. Re-review with the test prompts.
8. Re-run the D8 A/B effectiveness check (`no_skill` vs `with_skill`), and run the optimization regression check (`old_skill` vs `new_skill`).
9. If both the rubric score and the optimization regression check improve, recommend keep; if any key item drops or does not improve, recommend stop or revert.

Multi-round optimization loop:

- `optimize` enters a controlled loop of at most 3 rounds by default; unless the user explicitly says "one round only", never end automatically after the first re-review.
- Each round must re-review the current skill and record the round baseline score.
- Each round may only select the current lowest dimension; on ties, prefer D5/D6/D7/D10 safety, boundary, and runtime-risk dimensions.
- Each round may design only one modification point, and it must directly address the current lowest dimension or the A/B regression point.
- Before each patch, present the "optimization proposal" and wait for user confirmation.
- After each patch, show the diff, the re-reviewed score of the target dimension, its score change, and the regression check.
- `optimize` only outputs the before/after scores of the dimension modified in the current round by default; never repeat the full D1-D10 table unless the user explicitly asks.
- After each patch, show the D8 A/B scores: `no_skill` vs `with_skill`; also show the optimization regression check: `old_skill` vs `new_skill`.
- After each round, let the user choose: continue to the next round, stop and keep the current changes, or revert this round.
- If a round brings no improvement to the target dimension, recommend stop or revert; never expand the modification scope to keep trying.
- If a round raises the documentation score but degrades the A/B result, judge it as a risky modification first and recommend stop or revert.

Gates:

- **STOP: each round changes one skill, one lowest dimension, one modification point.**
- **STOP: before each patch, present the "optimization proposal" and wait for user confirmation.**
- **STOP: run at most `max_rounds` rounds; stop at the limit and wait for the user to decide.**
- **STOP: final keep or revert must be confirmed by the user.**

## Scoring Output

Batch evaluation uses this table:

```markdown
| Skill | Total | D1 | D2 | D3 | D4 | D5 | D6 | D7 | D8 | D9 | D10 | Lowest Dimension | Evidence | Suggestion |
|---|--:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---|---|---|
```

A/B effectiveness verification uses:

```markdown
| Skill | Scenario | Test Mode | Execution | no_skill | with_skill | Delta | Verdict | Reason |
|---|---|---|---|---:|---:|---:|---|---|
```

Before/after optimization A/B comparison uses:

```markdown
| Skill | Scenario | Test Mode | Execution | Old Skill | New Skill | Delta | Verdict | Reason |
|---|---|---|---|---:|---:|---:|---|---|
```

Single-skill re-review uses:

```markdown
## <skill-name> Re-review

- Total score before: <score>
- Total score after: <score>
- Target dimension: <D# name>
- Target dimension before: <score>
- Target dimension after: <score>
- Target dimension change: <after - before>
- Lowest dimension before: <dimension, optional>
- Lowest dimension after: <dimension, optional>
- Test mode: smoke-test/full-test
- Execution: subagent/main-agent
- A/B score change: <new_with_skill - old_with_skill or with_skill - no_skill>
- full-test case count: <N>
- smoke-test case count: <N>
- Decision: keep/revert/waiting for user confirmation
```

## Anti-pattern Blacklist

- Never modify files in review mode.
- Never optimize multiple skills at once.
- Never add redundant paragraphs just to raise scores.
- Never write a diff to disk without user confirmation.
- Never skip the confirmation gate before GitHub/MCP calls.
- Never treat a dimension 8 score without test prompts as a real measured high score.
- Never silently skip runtime red-flag hits.
- Never celebrate a rising `SKILL.md` score while ignoring A/B output quality regression.
- Never present smoke-test results as full-test.
- Never present main-agent fallback as an actual sub-agent run.
- Never output A/B scores without expected checkpoints or deduction reasons.

## Exception Handling

| Scenario | Handling |
|----------|----------|
| `SKILL.md` missing | Mark as an invalid skill, continue evaluating other directories |
| Both `assets/test.json` and `assets/test-prompts.json` missing | Read the target `SKILL.md`, auto-generate 3 test prompt drafts, present them for confirmation; never write files before confirmation |
| Test JSON parse failure | Report the file path and parse error; suggest fixing the test file first |
| Sub-agent unavailable or timed out | Use main-agent fallback and explain why; never label `execution=subagent` |
| Test case has only `id/prompt/expected` | Break expected into checkpoints before scoring |
| A/B shows with-skill is worse | Mark `regressed`, output a regression diagnosis; the optimize "optimization proposal" may only make a single-point fix around the regression cause |
| Score drops after optimize | Show the diff and the dropped dimensions, recommend revert, wait for user confirmation |
| Unrelated changes already exist in the workspace | Never roll back or overwrite; only report the diff related to the target skill |

## Quality Gates

- [ ] Default mode is `review`
- [ ] `review` outputs the D1-D10 score breakdown of every skill
- [ ] `review` writes no results file by default, unless the user explicitly requests export
- [ ] Every score has evidence
- [ ] Lowest dimension is explicit
- [ ] The "optimization proposal" only targets the lowest dimension or A/B regression points
- [ ] User confirmation before modification
- [ ] Diff and the round's target-dimension re-review shown after modification
- [ ] A/B test mode, execution method, delta, and verdict are output
- [ ] A/B scores include expected checkpoint coverage or deduction reasons
- [ ] After each round, offer continue, stop-and-keep, or revert options
- [ ] Final keep or revert is confirmed by the user
