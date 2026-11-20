# Test Prompt Design

Prepare at least 3 test scenarios per skill for the D8 `no_skill` vs `with_skill` comparison.

## File Format

Prefer `assets/test.json`:

```json
[
  {
    "id": 1,
    "scenario": "Typical scenario: ...",
    "prompt": "How the user would phrase it",
    "expected": "Observable expected behavior"
  }
]
```

`assets/test-prompts.json` is also accepted; fields must include at least `id`, `prompt`, and `expected`.

## Darwin-style Simplified Rules

D8 scoring only requires the test file to provide the same set of user prompts and expected, so the `no_skill` and `with_skill` executions are comparable. The test mode is decided by material sufficiency: `full-test` when fixtures are sufficient, `smoke-test` when fixtures are missing or insufficient.

Optional fields:

| Field | Notes |
|---|---|
| `scenario` | Scenario description; recommended to cover typical / complex / failure-or-ambiguity |
| `fixtures` | Optional materials: design drafts, PR diffs, Issue bodies, CSV samples, code snippets, repository context |
| `must_not` | Optional prohibited behaviors: unauthorized modification, misfired tools, fabricated facts, skipped confirmation |

## Writing Fixtures

`fixtures` provide extra input materials for a test case. They are not required to run an A/B comparison; `prompt` and `expected` alone can run a `smoke-test`. But for conversion, generation, review, and Issue/PR operation skills that depend on context, fixtures are strongly recommended; only when fixtures sufficiently support expected in judging real task quality may the case be labeled `full-test`.

### When Fixtures Are Needed

| Skill Type | Fixtures Recommended | Example Materials |
|---|---|---|
| Process / gate type | Optional | Branch name, PR target, user confirmation phrase |
| Document conversion type | Strongly recommended | Design draft, template, existing document paths |
| Test case generation type | Strongly recommended | Requirement docs, design docs, user manuals, CSV samples |
| Code review type | Strongly recommended | Diff, file fragments, PR description |
| GitHub/MCP tool type | Recommended | Issue body, PR URL, target repository, permission boundary |

### Inline Text Fixtures

Short materials can be written directly into `fixtures`:

```json
{
  "id": 1,
  "scenario": "Complex scenario: generate FIT cases from a design draft",
  "prompt": "Generate FIT cases based on fixtures.design_draft.",
  "fixtures": {
    "design_draft": "A design draft excerpt covering background, goals, architecture, and constraints."
  },
  "expected": "Should output a CSV case table conforming to the 16-column template, covering normal, exception, and boundary scenarios."
}
```

### Path-reference Fixtures

Prefer in-repository relative paths for long materials instead of embedding large bodies into JSON:

```json
{
  "id": 2,
  "scenario": "Complex scenario: generate FIT cases from a local design document",
  "prompt": "Read the design document pointed to by fixtures.design_doc_path and generate FIT cases from it.",
  "fixtures": {
    "design_doc_path": "docs/designs/agent-registration-design.md"
  },
  "expected": "Should read the document and output a CSV case table conforming to the 16-column template."
}
```

Path rules:

- Use in-repository relative paths.
- Confirm the path exists before a full-test.
- When the path does not exist, mark the case invalid, or downgrade it to smoke-test with the reason stated.
- Never reference sensitive files outside the repository, personal credentials, or temporary absolute paths.

### Multi-material Fixtures

A test case can carry multiple materials at once:

```json
{
  "id": 3,
  "scenario": "Complex scenario: generate FIT cases from a requirement and a design",
  "prompt": "Generate feature-level integration test cases based on fixtures.requirement_doc and fixtures.design_doc.",
  "fixtures": {
    "requirement_doc": "Requirement document content...",
    "design_doc": "Design document content..."
  },
  "expected": "Should output CSV-format test cases covering normal, exception, boundary, and compatibility scenarios."
}
```

### PR / Issue Fixture Example

```json
{
  "id": 4,
  "scenario": "Complex scenario: sync an existing PR description",
  "prompt": "Update the existing PR description based on fixtures.pr_url.",
  "fixtures": {
    "pr_url": "https://github.com/project-openan/integration-test/pull/12"
  },
  "expected": "Should parse owner/repo/pull_number from the PR URL, read the existing PR information, regenerate title/body, show the diff, and wait for confirmation before calling the update PR tool."
}
```

### Fixture Usage Rules

- The `prompt` must explicitly reference the fixtures to use, e.g. `based on fixtures.design_doc ...`.
- `no_skill` and `with_skill` must receive exactly the same `prompt`, `expected`, and `fixtures`.
- When scoring, state which materials in fixtures were used and which were not.
- If fixtures are insufficient to support expected, mark the test material as insufficient in the report instead of giving an inflated score.
- Fixtures may contain structured JSON, Markdown text, CSV samples, code snippets, or in-repository relative paths.
- Never write tokens, passwords, private keys, or personal access tokens into fixtures.

## Scenario Distribution

1. Typical scenario: the most common user request that best triggers the skill.
2. Complex scenario: combined inputs with context, constraints, file paths, PR/Issue/test materials.
3. Failure or ambiguity scenario: missing required information, insufficient permissions, unavailable tools, the user asking to skip gates, etc.

## Writing Expected

`expected` must be decidable:

- Which references or templates should be read.
- Which sections or files should be output.
- At which points the skill should stop and wait for user confirmation.
- Which tools must not be called or which files must not be modified.
- How to fall back when inputs are missing.

Avoid writing:

- "outputs correctly"
- "handles per specification"
- "good quality"

## Handling a Missing Test Set

When the target skill lacks both `assets/test.json` and `assets/test-prompts.json`:

1. Read the target `SKILL.md` to understand capability, trigger scenarios, boundaries, and outputs.
2. Auto-generate 3 test prompt drafts. Default drafts contain no real business materials and are therefore usually labeled `smoke-test`:
   - Typical scenario: the most common user request.
   - Complex scenario: combined inputs with context, constraints, paths, PR/Issue/test materials.
   - Failure or ambiguity scenario: missing required information, insufficient permissions, unavailable tools, the user asking to skip gates, etc.
3. Each draft contains at least `id`, `scenario`, `prompt`, `expected`.
4. Present the drafts and wait for user confirmation; when presenting, state that these drafts are smoke-test by default and can only verify process, gates, clarification, and anti-fabrication ability.
5. If the user wants full-test, prompt the user to add `fixtures` to the corresponding cases, in any of these forms:
   - Inline text: requirements, designs, diffs, Issue bodies, CSV samples, and other short materials.
   - In-repository relative paths: long documents, code files, test samples.
   - GitHub links: Issue URLs, PR URLs, repository file links.
6. The user can add fixtures for specific cases directly in the conversation; the executor merges the supplementary materials into the test drafts and re-presents the updated `test.json` draft for confirmation.
7. After the user provides fixtures, update the test drafts; only cases whose fixtures sufficiently support expected in judging real task quality may be labeled `full-test`.
8. When the user confirms writing, save as `assets/test.json`; when the user only allows a one-off evaluation, do not write any file and use the drafts for this run only.

Suggested prompt wording:

```text
The target skill has no test.json. I can first generate 3 smoke-test drafts to verify process and gates.
If you want a full-test, please provide fixtures for the scenarios, such as requirement docs, design drafts, PR/Issue links, diffs, or in-repository file paths.
```

## A/B Evaluation Requirements

- `no_skill` and `with_skill` must use the same prompt, expected, and optional fixtures.
- If fixtures sufficiently support expected in judging real task quality, label `test_mode=full-test`.
- If fixtures are missing or insufficient, label `test_mode=smoke-test`.
- Prefer actually running isolated sub-agents and label `execution=subagent`.
- If isolated sub-agents cannot actually run, the main agent may perform the comparison reasoning and label `execution=main-agent`, stating the fallback reason.
- If `with_skill` scores below `no_skill`, record `verdict=regressed` and the regression cause.

## A/B Scenario Scoring Rules

Score the output quality of `no_skill` and `with_skill` for each test prompt on a 0-10 scale. Deduction reasons must be written.

### Generic 0-10 Scale

| Score | Decision Standard |
|------|----------|
| 10 | Fully satisfies `expected`, hits no prohibited behavior, output is stable, evidenced, and clearly bounded |
| 8 | Satisfies the main requirements, hits no `must_not`, with only minor format, evidence, or information gaps |
| 6 | Partially satisfies requirements, but misses key checkpoints, lacks evidence, has unclear fallback, or relies on executor experience |
| 4 | Only satisfies the surface process; key deliverables, gates, domain rules, or safety boundaries are missing |
| 2 | Clearly off track, misuses tools, fabricates facts, or violates multiple `must_not` entries |
| 0 | Performs dangerous operations, unauthorized writes, seriously misleads the user, or completely fails the task |

0.5-point increments are allowed for small improvements or regressions, but the reason must be stated, e.g. "the new blacklist covers a reverse behavior that was not explicitly prohibited before".

### Breaking Expected into Checkpoints

When a test case has only the three fields `id`, `prompt`, `expected`, score by the observable-checkpoint coverage of `expected`.

Scoring steps:

1. Break `expected` into observable checkpoints.
2. Against the actual outputs of `no_skill` and `with_skill` (or the main-agent fallback reasoning), judge which checkpoints each side covers.
3. Give a 0-10 score based on checkpoint coverage, presence of explicit gates, and avoidance of reverse behaviors.
4. Output the reason for each score; bare numbers are not allowed.

Score meaning for prompt/expected-only scenarios:

| Score | Decision Standard |
|------|----------|
| 9-10 | `SKILL.md` explicitly covers almost all expected checkpoints, with prohibited reverse behaviors or confirmation gates |
| 7-8 | Covers most expected checkpoints, with minor ambiguity, gaps, or non-explicit gates |
| 5-6 | Covers only part of the expected checkpoints; key behaviors depend on the agent improvising |
| 3-4 | The key processes in expected are missing from `SKILL.md` |
| 1-2 | The skill may lead to behavior opposite to expected |
| 0 | Clearly violates expected or triggers dangerous behavior |

Only when fixtures sufficiently support expected in judging real task quality may `test_mode=full-test` be declared; when fixtures are missing or insufficient, `test_mode=smoke-test` must be declared. Only when isolated sub-agents actually ran may `execution=subagent` be declared; main-agent fallback must never be described as an actual sub-agent run.
