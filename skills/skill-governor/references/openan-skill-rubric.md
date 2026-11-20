# OpenAN Skill Rubric

Score across 10 dimensions. Each dimension scores 1-10; the total is the weighted average, out of 100.

| # | Dimension | Weight | Decision Focus |
|---|-----------|--------|----------------|
| D1 | Frontmatter and trigger quality | 8 | `name` is compliant; `description` states capability, trigger scenarios, boundaries, and keywords |
| D2 | Workflow clarity | 12 | Stage order, inputs/outputs, execution actions, and state transitions are clear |
| D3 | OpenAN community specification compliance | 10 | Whether the skill conforms to explicit requirements in repository standards, coding standards, test standards, and related templates |
| D4 | Deliverable format stability | 10 | Output files, tables, templates, fields, paths, and naming conventions are stable |
| D5 | Human-in-the-loop checkpoints | 10 | STOP, confirmation, parameter display, or diff display before high-risk actions |
| D6 | Failure modes and fallback | 12 | Explicit if-then branches; recoverable from missing inputs, tool failures, permission failures, conflicts |
| D7 | GitHub MCP / local tool boundaries | 10 | Read the schema before calling MCP; never guess parameters; clear boundary between local commands and remote writes |
| D8 | Skill A/B actual effectiveness | 14 | Scores the output-quality gain of `no_skill` vs `with_skill`, judging whether the skill itself brings real improvement; `test.json` / `test-prompts.json` are only sources of prompts and expected, not direct evidence for a high score |
| D9 | Anti-examples and risk blacklist | 8 | Explicit "do not" list; dangerous actions, anti-patterns, and misuse scenarios are enumerated |
| D10 | Runtime neutrality | 6 | Not bound to a single agent runtime; tool capabilities have generic descriptions or fallbacks |

## Scoring Rules

- 9-10: rules are complete, directly executable, and failure paths are clear.
- 7-8: mostly usable, with only small gaps or insufficiently explicit expression.
- 5-6: works but relies on executor experience; missing checkpoints or fallbacks.
- 3-4: vague process, missing key gates, easy to misuse.
- 1-2: cannot be triggered or executed stably; may cause wrong tool calls or wrong deliverables.

## D3 Scoring Rules

D3 only evaluates community specifications with explicit documented basis; never infer undocumented community processes. Before evaluating, read this skill's bundled `references/coding-specification.md` and `references/test-specification.md`; standards and templates in the target repository serve only as supplementary evidence.

Every D3 conclusion must provide all of the following:

- The specification source and section;
- The specific requirement in the specification;
- The target skill's file path and section;
- The target skill's coverage of the requirement: compliant, partially compliant, or non-compliant.
- If a document linked from the specification is unavailable, note "referenced material unavailable" and judge only by the explicit clauses in the specification file at hand.

When no applicable community specification is found, D3 must be marked `evidence-insufficient`; never infer from model common sense, and never describe the skill as conforming to community process.

## D8 Scoring Rules

D8 measures the skill's **actual effectiveness gain** in test scenarios, not whether `test.json` itself is well written.

Scoring only compares `no_skill` vs `with_skill`: after using the target skill, are output quality, safety boundaries, and process stability clearly better than without the skill.

The role of `test.json` / `test-prompts.json` is to provide prompts and expected so that `no_skill` and `with_skill` are comparable on the same task. The mere existence of a test set cannot raise D8; if A/B shows `with_skill` is not better than `no_skill`, D8 must not score high.

`old_skill` vs `new_skill` may be used in optimize mode to judge whether the current round improved the skill, but it is an optimization regression check, not the defining source of D8.

Suggested decisions:

- 9-10: in most test scenarios `with_skill > no_skill`, with no safety, tool-boundary, or runtime regression; preferably supported by full-test outputs with sufficient fixtures.
- 7-8: most scenarios show positive gain, but mainly from smoke-test, or full-test ran via main-agent fallback, or only workflow quality is covered.
- 5-6: output structure is more stable, but task-quality gain is unclear, or test scenarios are insufficient to prove real capability improvement.
- 3-4: A/B results are mostly neutral, key scenarios lack gain, or side effects such as excessive process or misfired tools are introduced.
- 1-2: `with_skill` is clearly worse than `no_skill`.

## Lowest Dimension Selection

Only the current lowest dimension is selected for optimization. On ties:

1. First fix D5, D6, D7: safety and process-boundary issues.
2. Then fix D8 A/B effectiveness, test-scenario evidence, or task-quality gain issues.
3. Finally fix D1, D2, D4, D9, D10: expression and structure issues.

## Evidence Requirements

Every deduction must cite verifiable evidence:

- File path
- Section title or keyword
- The specific missing item
- The corresponding test prompt scenario

Never write only "slightly weak" or "needs strengthening".
