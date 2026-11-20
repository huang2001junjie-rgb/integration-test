# Coding Specification

## General Principles

- Prefer clear, simple, maintainable code.
- Keep changes minimally necessary when modifying code; no unrelated refactoring, formatting, or file moves.
- New code must match the existing style, directory boundaries, and dependency direction of its module.
- Public APIs, configuration, error semantics, data formats, and command-line arguments are compatibility boundaries; impact must be assessed before modification.

## Team Conventions

- New files go into existing directories with matching responsibility first; never create new top-level directories casually for a single feature.
- When adding a documentation page, check the menu entry at the same time.
- Documentation site page filenames use a numeric prefix plus kebab-case; skill directories and names use kebab-case.
- `skills/*/SKILL.md` must contain the `name` and `description` frontmatter: `name` exactly matches the directory name; `description` is a single-line, non-wrapping, complete string (`>-` / `|` folding is prohibited).
- Never commit temporary scripts, debug output, cache files, or build artifacts, unless the project explicitly agrees to commit them.
- Never write real tokens, keys, or personal information into code, documents, logs, or test snapshots.

## Security Rules


## General Coding Rules

<!-- The following is in principle not expanded in detail in `AGENTS.md`; keep it only when the repository has explicit preferences, historical pitfalls, or unified constraints: -->

## Reserved Section

<!-- - Repository-specific architecture boundaries:
- Repository-specific API / configuration constraints:
- Repository-specific engineering gates:
- Repository-specific security red lines: -->
