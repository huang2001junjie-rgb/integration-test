# Test Specification

This specification is the top-level test standard of the `integration-test` repository, governing the authoring, naming, classification, automation, and review of test cases. Mandatory items are stated directly; detailed rules and examples are referenced by path from `skills/test-cases-reviewer/references/` and `docs/` to avoid duplication.

> **Core principles**: atomicity, repeatability, traceability, completeness, independence.

## 1. Scope and Referenced Documents

### 1.1 Audience

- All feature test cases in the repository (functional tests, feature-level integration tests (FIT), DFX tests, and SIT)
- Feature owners authoring cases, reviewers reviewing cases, and automation engineers onboarding cases
- Used together with the `test-cases-reviewer` skill: this specification is the basis before review; the reference documents are the detailed rules during review

### 1.2 Referenced Documents (mandatory; the latest version in the repository prevails)

| Document | Purpose | Path |
|----------|---------|------|
| Governance baseline | Process, quality gates, roles, environment variable contract | [docs/OpenAN Integration Testing Process and Framework.md](docs/OpenAN%20Integration%20Testing%20Process%20and%20Framework.md) |
| Test case template | 16-column header and field rules | [docs/templates/testcases-template-fields.md](docs/templates/testcases-template-fields.md) and [docs/templates/testcases-template.csv](docs/templates/testcases-template.csv) |
| Test case review checklist | Universal mandatory/suggested self-check items | [skills/test-cases-reviewer/references/test-case-review-checklist.md](skills/test-cases-reviewer/references/test-case-review-checklist.md) |
| FIT authoring guide | FIT-specific rules | [skills/test-cases-reviewer/references/fit-case-authoring-guide.md](skills/test-cases-reviewer/references/fit-case-authoring-guide.md) |
| DFX authoring guide | Core checkpoints of the 7 DFX dimensions | [skills/test-cases-reviewer/references/dfx-case-authoring-guide.md](skills/test-cases-reviewer/references/dfx-case-authoring-guide.md) |
| FIT/DFX attribution criteria | Actionable attribution and cross-table deduplication (single source of truth) | [skills/test-cases-reviewer/references/fit-dfx-scenario-attribution-criteria.md](skills/test-cases-reviewer/references/fit-dfx-scenario-attribution-criteria.md) |
| test-cases-reviewer SKILL | Review process, decision rules, report template | [skills/test-cases-reviewer/SKILL.md](skills/test-cases-reviewer/SKILL.md) |

## 2. Test Types and Applicability

### 2.1 Type List

| Case Type | Applicable Scenario | Authoring Guide |
|----------|---------------------|-----------------|
| Functional Test | Single interface/function-point verification; every specified function must be covered by a case | Checklist only |
| Feature-level Integration Test (FIT) | Interfaces and collaboration between modules inside a feature, verified end to end in a real environment; **only** business functional correctness (deployment, main path, functional config change, business state transition, business-level negative cases) | Checklist + FIT guide + attribution criteria |
| DFX Test | Non-functional specialties (testability, diagnosability, serviceability, reliability, performance, security, compatibility); must not copy FIT business main paths | Checklist + DFX guide + attribution criteria |
| System Test (SIT) | Distribution-level multi-feature combination scenarios on RC versions | Checklist + framework document |

### 2.2 Classification Decision Priority

The classification decision of a CSV file (used to select the applicable guide during review) follows this priority:

1. **Explicit user statement** ("review FIT cases", "DFX cases", "full set" in the conversation)
2. **Filename keywords**: containing `-fit-test-cases` / `FIT` (and not `DFX`) -> FIT; containing `-dfx-test-cases` / `DFX` / `_DFT_` / `_DFD_` / `_DFS_` / `_DFR_` / `_DFP_` / `_DFNS_` / `_DFC_` -> DFX
3. **The `Case Type` column and the `dfx_type` key in Notes**: when mixed, **select the guide per row**; record the file as "full set"
4. **Undeterminable** -> treat as the full set and consult both the FIT and DFX guides

### 2.3 FIT vs DFX Scenario Attribution (mandatory)

Judge by the verification target, not by the triggering operation. Full rules: [fit-dfx-scenario-attribution-criteria.md](skills/test-cases-reviewer/references/fit-dfx-scenario-attribution-criteria.md).

- Any case whose verification target is a non-functional design capability - probes / logs / metrics / self-healing / performance / security / compatibility - belongs to **DFX**
- Any case whose verification target is business functional correctness belongs to **FIT**
- The same trigger may appear once in each table, but the assertion focus must differ; the same assertion focus in both tables is duplication and fails the review
- FIT must not contain process-kill self-healing, resource-shortage degradation, or dependency-failure recovery; DFX must not copy FIT with a business result as its only assertion

`test-cases-generator` must perform cross-CSV boundary checks and deduplication in the FIT+DFX full pipeline and include the result in the pass/fail decision.

## 3. Test Case Structure and Template

### 3.1 Field Definition (16 columns, fixed order)

The header must exactly match [testcases-template-fields.md](docs/templates/testcases-template-fields.md); **column count, order, and semantics are all immutable**:

```text
Milestone,Case Name,Case ID,Module,Component Deps,Requirement Link,Case Type,Priority,Precondition,Steps,Expected Result,Automatable,Automated Test,Dev Owner,Test Owner,Notes
```

The CSV is saved in UTF-8; cells containing line breaks must be wrapped in double quotes. All content is in English.

### 3.2 Required-field Rules

| Field | Required | Notes |
|------|------|------|
| Milestone / Case Name / Case ID / Module / Component Deps / Case Type / Priority / Precondition / Steps / Expected Result / Automatable / Dev Owner / Test Owner | Mandatory | Empty values fail the review |
| Requirement Link | Mandatory for P0/P1 | Used for the requirement traceability matrix and coverage statistics |
| Automated Test | Mandatory (after automation lands) | Not checked at the review stage (see section 10 exclusions) |
| Notes | Optional | Content is not checked at the review stage, except the orchestration key conventions (see section 3.4) |

### 3.3 File Naming

| Case Class | Convention | Example |
|----------|-----------|-----------------|
| FIT | Filename contains `-fit-test-cases` or `FIT` | `registry-center-agentcard-fit-test-cases.csv` |
| DFX | Filename contains `-dfx-test-cases`, `DFX`, or a `_DFx_` segment | `registry-center-agentcard-dfx-test-cases.csv` |
| Full set | Contains the version and feature/component, with no single-type suffix | `agent-registration-test-cases.csv` |

> Keep directory hierarchy clear and naming accurate when archiving into `docs/plans/<version>/`.

### 3.4 Notes Column Orchestration Keys

Orchestration keys are written as `key:value` pairs in the `Notes` column, separated by `;`, consumed downstream by `e2e-test-code-gen`:

| Key | Meaning |
|-----|---------|
| `language_profile` | Target language/framework profile, e.g. `python/pytest`, `java/junit5`, `js/vitest`, `shell/bats` |
| `component_deps` | Component dependency declaration; must be consistent with the `Component Deps` column |
| `protocol_primitive` | A2A-T protocol primitives involved (Task-T / Negotiation-T / Notification-T / Authorization-T); `none` for non-protocol cases |
| `execution_strategy` | e.g. `smoke` / `full` / `slow` |
| `environment` | e.g. `docker-compose` |
| `dfx_type` | Required for DFX rows; one of DFT/DFD/DFS/DFR/DFP/DFNS/DFC |

`Component Deps` is the standalone 5th column and **must not** be written into Notes.

## 4. Case Design Methods

The 7 methods are detailed in [fit-case-authoring-guide.md](skills/test-cases-reviewer/references/fit-case-authoring-guide.md). Complex scenarios **must** note the method used via `design_method` in Notes.

| Design Method | Applicable Scenario |
|---------------|---------------------|
| Equivalence partitioning | Input has clear valid/invalid ranges |
| Boundary value analysis | Input has numeric or length limits |
| Orthogonal arrays | Multiple independent factors, each with multiple values |
| State transition | The object under test has states and transitions between them |
| Decision table | Multiple condition combinations produce different actions |
| Error guessing | Experience-based supplementary exception scenarios |
| Scenario method | Business flows with multiple steps and branches |

**Method selection guide**:

| Scenario Feature | Recommended Method |
|------------------|--------------------|
| Interface parameter validation | Equivalence partitioning + boundary values |
| Multi-factor environment combination (OS/arch/version) | Orthogonal arrays |
| State change flows | State transition |
| Business end-to-end flows | Scenario method |
| Complex conditional logic | Decision table |
| Known high-risk modules | Error guessing |

## 5. General Authoring Rules (mandatory quick reference)

The following are hard gates that fail the review. Detailed rule descriptions and typical problems: [test-case-review-checklist.md](skills/test-cases-reviewer/references/test-case-review-checklist.md).

### 5.1 Case Name

- Expresses the test purpose/intent; concise, unambiguous
- Verb-object phrase; without the Case ID prefix; in English
- Suggested length: 50 characters or fewer

**Examples**: `Register a valid Agent successfully` / `Query an Agent with an invalid ID returns 404`

### 5.2 Case ID

- Globally unique within the product scope (duplicates fail every involved case)
- Wrapped in square brackets; FIT: `[FIT-<feature>-<topic>-<3-digit seq>]`; DFX: `[DFX-<feature>-<topic>-<3-digit seq>]`; SIT: `[SIT-<scenario>-<number>]`
- Character set: English letters, digits, `_`, `-`, square brackets; no spaces, `#`, `/`, or parentheses

### 5.3 Milestone

- `<YY>-<season>` format (e.g. `26-winter`), matching the version directory under `docs/plans/` and consistent with the test plan

### 5.4 Priority

| Level | Definition |
|-------|------------|
| P0 | Feature acceptance cases; install/uninstall success, core functional positive main path; must be automatable (`Yes`) with Requirement Link filled |
| P1 | Basic feature functionality cases covering the vast majority of requirement specifications; same P0 requirements on automation and requirement links |
| P2 | Complex scenarios and business-level exception/negative cases (the FIT table holds no DFX cases; DFX specialties go into the DFX table) |

### 5.5 Precondition

- States the concrete test environment, data configuration, and current system state (software/hardware/network)
- **Explicit environment elements**: e.g. which images the registry contains, node roles, namespaces, cluster scale
- Numbered items (`1. ` / `2. `); semicolons to separate multiple items
- Must not contain test steps such as logging in or opening pages

### 5.6 Steps

- Numbered steps (`1. `); one action per step; wrap in-cell line breaks in CSV double quotes
- Each step is executable and unambiguous
- A step needing a result check ends with `see result N` (N is the expected-result number)
- Multiple consecutive independent steps must not share one number
- State input data, operation entry, and test parameters (suggested)
- The last step is a cleanup action

### 5.7 Expected Result

- Numbered results (`1. `)
- **Vague wording is prohibited**: `normal`, `success`, `as expected`, `no error`, and other non-objectively-decidable phrases
- Must be concrete data, response messages, UI changes, status codes, log keywords, metric names, or Pod/CR status fields
- Numbers correspond to the step numbers (suggested)
- LLM-related cases use deterministic assertions (status codes / structure / state fields) and never assert model-generated text

### 5.8 Automatable

- Must be evaluated and filled with `Yes` / `No`
- P0/P1 cases must be `Yes`
- Non-automatable cases state the reason and are covered manually

### 5.9 Owners

- `Dev Owner` or `Test Owner` empty -> fail
- Note it specially when automation is done by someone other than the feature owner

## 6. FIT-specific Rules

Applies to cases reviewed as FIT. Details: [fit-case-authoring-guide.md](skills/test-cases-reviewer/references/fit-case-authoring-guide.md).

| Attribute | Requirement | Key Points |
|-----------|-------------|------------|
| Case ID | Mandatory | `[FIT-<feature>-<topic>-<3-digit seq>]` |
| Precondition | Mandatory | Environment elements concrete enough to reproduce (image lists, node roles, namespaces, etc.) |
| Steps/Expected | Mandatory | Business integration scenarios executable; business assertions objectively decidable |
| Design method | Mandatory | Complex scenarios note the method via `design_method` in Notes (equivalence / boundary / orthogonal / state transition / decision table / error guessing / scenario) |

**Design key points**:

1. Identify all internal interfaces from the module interaction diagram
2. Cover normal data flows and key exception data flows
3. Verify data transfer, format conversion, and exception propagation between modules

## 7. DFX-specific Rules

Applies to DFX rows (carrying the `dfx_type` key in Notes). Details: [dfx-case-authoring-guide.md](skills/test-cases-reviewer/references/dfx-case-authoring-guide.md).

### 7.1 The Seven DFX Dimensions

| Dimension | Abbrev. | Full Name | Core Verification Target |
|-----------|---------|-----------|--------------------------|
| Design for Testability | DFT | Testability | Easy to automate, externally probeable, state verifiable without intrusion |
| Design for Diagnosability | DFD | Diagnosability | Exception scenarios output enough observability to locate root causes fast, reducing MTTR |
| Design for Serviceability | DFS | Serviceability | Fits cloud-native standard operations: deploy, upgrade, rollback, config change, scale |
| Design for Reliability | DFR | Reliability | Tolerance, self-healing, and isolation under abnormal environments, resource fluctuation, process or network failures |
| Design for Performance | DFP | Performance | Controlled resource overhead, stable long runs, no avalanche under high load, no resource leaks |
| Design for Security | DFNS | Security | Authentication, encryption, audit, injection prevention, least privilege, no high-risk dependency vulnerabilities |
| Design for Compatibility | DFC | Compatibility | Compatibility across versions, environments, architectures, and dependency combinations |

### 7.2 Field Requirements

| Attribute | Requirement | Key Points |
|-----------|-------------|------------|
| Case ID | Mandatory | `[DFX-<feature>-<topic>-<3-digit seq>]` |
| `dfx_type` in Notes | Mandatory | One of DFT/DFD/DFS/DFR/DFP/DFNS/DFC |
| Steps | Mandatory | Concrete commands, interface paths, parameters, observation windows; non-executable descriptions prohibited |
| Expected Result | Mandatory | HTTP status codes, log levels and keywords, metric names, Pod/CR status fields, and other objectively decidable items; `normal` / `no error` prohibited |
| Priority | Suggested | DFX is mostly P2; core observability/serviceability acceptance may be P0/P1 |
| Scenario boundary | Mandatory (semantic) | Against the attribution criteria: no duplication of FIT business main paths; invalid input only asserts diagnosability; config change only asserts operability/testability mechanisms |

### 7.3 Dimension Coverage Requirements

- Every applicable DFX dimension must have corresponding cases
- Inapplicable DFX dimensions **must** state the reason (in the test plan or Notes)
- DFX rows must not repeat FIT business main paths

### 7.4 Core Checkpoints per Dimension (excerpt)

| Dimension | Core Checkpoints (excerpt) |
|-----------|----------------------------|
| DFT | Standardized health probes; structured observability output; dynamic config takes effect; start/run/exit states externally recognizable |
| DFD | Log levels INFO/WARN/ERROR; exceptions have dedicated error logs with context; no silent failures; exception counters and status metrics exist |
| DFS | Deployed from K8s standard resources; rolling upgrade without interruption; externalized config with hot update; version rollback; horizontal scaling without state conflicts |
| DFR | Process crash auto-restart; fault isolation; network jitter retry/circuit breaking; degradation on resource shortage; start failure retryable |
| DFP | CPU/memory stable without leaks; high concurrency non-blocking; no connection/handle/goroutine leaks; stable processing latency; resource caps |
| DFNS | Identity authentication and access authorization; sensitive data encryption; audit logs for key operations; input validation against injection; least privilege; no high-risk dependency vulnerabilities |
| DFC | Interface backward compatibility; multi-version dependency compatibility; consistent behavior across OS/architectures; old/new version coexistence during upgrade/rollback; config change compatibility policy |

> Generic standard case examples per dimension: [dfx-case-authoring-guide.md](skills/test-cases-reviewer/references/dfx-case-authoring-guide.md).

## 8. Requirement and Scenario Coverage

### 8.1 Requirement-case Tracking

- Every requirement has at least one corresponding case; the requirement-case traceability matrix is complete with no omitted requirements
- `Requirement Link` is mandatory for P0/P1 cases; requirement coverage = (approved requirements with corresponding executed cases) / (approved requirements), targeting 100%

### 8.2 Scenario Coverage

Covers normal scenarios, exception scenarios, and functional interaction scenarios; case goals are clear, and:

- Functional tests and FIT cover business paths
- DFX dimensions match the `dfx_type` declared in Notes (full-set tables must cover every FIT and DFX type present)

### 8.3 Quality Metrics

| Metric | Target | Notes |
|--------|--------|-------|
| P0/P1 automation rate | 100% | Verified at handover acceptance (framework section 4.3.1) |
| TRR entry | P0/P1 pass rate = 100%; P2 pass rate >= 80% | Framework section 8.1 |
| FIT exit | Overall FIT pass rate >= 95%; failed cases archived as issues and labeled `skip-temporarily` | Framework section 8.2 |
| Integration test API coverage | >= 80% (counted by OpenAPI paths) | Framework section 8.4 |

### 8.4 Defect Linkage

- 100% of failed cases are linked to an issue

## 9. Automation Rules

### 9.1 Automation Assessment

- The `Automatable` column must be filled (`Yes` / `No`) when cases are authored
- Non-automatable cases state the reason and are covered manually
- Periodically revisit non-automated cases

### 9.2 Automation Landing

- After automation code is done, the `Automated Test` column is backfilled as `<test file>::<test function>` or `test-<scenario>.spec.ts`
- Recommended stacks: backend integration with pytest; frontend UI with Playwright (TypeScript); Java repositories with JUnit 5 + Maven; shell/deployment repositories with bats
- Test function naming: `test_<action>_<expected_result>`; test files: `test_<topic>.py`; associated with the manual case via the Case ID label
- Every automated case carries the label dimensions `feature` / `environment` / `components` / `priority` / `execution` (plus `skip-temporarily` when applicable); no `sleep` synchronization, no order dependencies, explicit assertions, complete teardown cleanup

### 9.3 Typical Scenarios Unsuitable for Automation

- Hardware-related operations
- Subjective UI experience assessment
- One-off verifications

## 10. Review and Quality Gates

### 10.1 Review Entry

Review process, decision rules, and report template: [test-cases-reviewer SKILL.md](skills/test-cases-reviewer/SKILL.md). Not expanded here.

Invocation example:

```text
/test-cases-reviewer

Review registry-center-agentcard-dfx-test-cases.csv
Case class: DFX
Target milestone: 26-winter
```

### 10.2 Review Scope Exclusions

The following columns stay in the 16-column header but are **excluded** from mandatory/suggested judging; empty values or format issues there fail nothing:

| Column | Notes |
|--------|-------|
| Automated Test | Backfilled by development after automation lands; not checked at the review stage |
| Notes | Content is not checked, except the orchestration key conventions of section 3.4 |

### 10.3 Decision Standard

| Checklist Requirement Level | Review Conclusion |
|-----------------------------|-------------------|
| **Mandatory** item unsatisfied | Fail |
| **Suggested** item unsatisfied | Remediation suggestion (a case still passing all mandatory items is marked "pass with suggestions") |

### 10.4 Re-review Mechanism

- After fixes, rerun `test-cases-reviewer` and compare against the previous report number
- Template non-compliance must be fixed at the table-structure level before per-case review
- P0-related or blocking items are fixed first

## 11. Appendix

### 11.1 Test Case Template Example

See [testcases-template.csv](docs/templates/testcases-template.csv) and [test-case-template.md](skills/test-cases-reviewer/references/test-case-template.md).

### 11.2 Review Report Structure

See the "Output: Markdown review report" section of [test-cases-reviewer SKILL.md](skills/test-cases-reviewer/SKILL.md): review summary, template and file-level conclusions, failed/to-be-fixed items, remediation suggestions, passed case list, statistics and conclusion.

### 11.3 Glossary

| Term | Meaning |
|------|---------|
| FIT | Feature-level Integration Test |
| SIT | System Integration Test, run by the Integration Testing WG on RC versions |
| DFX | Design for X, non-functional design across the full lifecycle |
| DFT/DFD/DFS/DFR/DFP/DFNS/DFC | The 7 DFX dimension abbreviations, see section 7.1 |
| TRR | Test Readiness Review, the acceptance checkpoint before entering FIT |
| WG | Working Group, the collaborative unit of the OpenAN community |
| MTTR | Mean Time To Repair |
