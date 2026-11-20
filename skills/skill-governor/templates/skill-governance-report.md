# Skill Governance Report

## Summary

| Item | Value |
|------|-------|
| Evaluation scope | `<skills_root or target_skill>` |
| Mode | `review / optimize` |
| Test mode | `smoke-test / full-test / mixed` |
| Execution | `subagent / main-agent / mixed` |
| full-test case count | `<N>` |
| smoke-test case count | `<N>` |
| subagent execution count | `<N>` |
| main-agent fallback count | `<N>` |
| Average delta | `<with_skill - no_skill average>` |
| Skills evaluated | `<N>` |
| Average score | `<score>` |

## Score Summary

| Skill | Total | D1 | D2 | D3 | D4 | D5 | D6 | D7 | D8 | D9 | D10 | Lowest Dimension | Evidence | Suggestion |
|---|--:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---|---|---|
| `<skill>` | `<score>` |  |  |  |  |  |  |  |  |  |  | `<D# name>` | `<path/section evidence>` | `<one focused action>` |

Export note:

```text
Review mode writes no results file by default. Only when the user explicitly requests export are artifact/skill-governor-results.csv or artifact/skill-governor-results.jsonl written.
```

## Dimension Details

### `<skill-name>`

| Dimension | Score | Evidence | Notes |
|---|---:|---|---|
| D1 Frontmatter |  |  |  |
| D2 Workflow |  |  |  |
| D3 Community Specification Compliance |  |  |  |
| D4 Deliverable Format |  |  |  |
| D5 Human Checkpoints |  |  |  |
| D6 Failure Fallback |  |  |  |
| D7 Tool Boundary |  |  |  |
| D8 Test Performance |  |  |  |
| D9 Risk Blacklist |  |  |  |
| D10 Runtime Neutrality |  |  |  |

Lowest dimension: `<dimension>`

Suggested single-point action:

```text
<one lowest-dimension improvement>
```

## Optimization Round Results

Only used in `optimize` mode. The full D1-D10 score table is not repeated unless the user explicitly asks.

| Skill | Round | Target Dimension | Before | After | Change | Evidence | Conclusion |
|---|---:|---|---:|---:|---:|---|---|
| `<skill>` |  | `<D# name>` |  |  |  | `<diff or section evidence>` | `improved/neutral/regressed` |

## Runtime Neutrality

| Skill | Status | Evidence | Suggestion |
|---|---|---|---|

## Test Scenario Coverage

| Skill | Test File | Case Count | Gaps |
|---|---|---:|---|

## A/B Effectiveness Verification

| Skill | Scenario | Test Mode | Execution | no_skill | with_skill | Delta | Verdict | Reason |
|---|---|---|---|---:|---:|---:|---|---|
| `<skill>` | `<case id>` | `smoke-test/full-test` | `subagent/main-agent` |  |  |  | `improved/neutral/regressed` |  |

## Before/After Optimization A/B Comparison

Only used in `optimize` mode, comparing the same test scenario across the skill before and after modification.

| Skill | Scenario | Test Mode | Execution | Old Skill | New Skill | Delta | Verdict | Reason |
|---|---|---|---|---:|---:|---:|---|---|
| `<skill>` | `<case id>` | `smoke-test/full-test` | `subagent/main-agent` |  |  |  | `improved/neutral/regressed` |  |

Expected checkpoints:

```text
<List observable checkpoints derived from expected and note which side covered them.>
```

Regression diagnosis:

```text
<Only required when verdict is regressed or when with_skill introduces safety/tool/runtime risk.>
```

## Follow-up Actions

1. `<highest priority action>`
2. `<next action>`
3. `<next action>`
