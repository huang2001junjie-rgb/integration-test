# Runtime Neutrality

OpenAN skills should target generic runtime environments that support Agent Skills, not bind to a single agent product.

## Red-Flag Scan

Check `SKILL.md` for the following patterns:

```text
in Claude Code
Claude Code skill
Claude Code users
Cursor only
in Codex
~/.claude/skills/
/plugin install
```

Record hits as D10 risks. If the skill is explicitly a single-runtime dedicated tool, state the exception reason in the report.

## Allowed

- Mentioning a tool as an example while also providing a generic alternative.
- Referencing skill names that already exist in the current repository.
- Describing fallbacks such as "if the runtime lacks a tool, the main agent executes serially".

## Neutrality of GitHub MCP Skills

GitHub MCP is a business capability dependency, not runtime binding. When evaluating, focus on:

- Whether reading the schema is required before calling MCP.
- Whether parameters follow the MCP descriptor.
- Whether MCP failure preserves the in-conversation conclusion or provides a manual path.
- Whether remote write actions are placed after user confirmation.

## Report Format

```markdown
- Runtime neutrality: pass / risk / fail
- Red-flag hits: <pattern> at <path>
- Impact: <why this misleads other runtimes>
- Suggestion: <replace with a generic expression or add a fallback>
```
