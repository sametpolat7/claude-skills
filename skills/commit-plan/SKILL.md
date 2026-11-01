---
name: commit-plan
description: Splits every working-tree change (staged, unstaged and untracked) into commit-ready groups by relationship and layer, in any language or framework, with all test changes in one group; writes a title-only Conventional Commits message per group; and prints a runnable, dependency-ordered block that stages, commits and pushes each group on the current branch. Read-only — prints the plan and the commands, never stages, commits or pushes. Use when the user says "commit plan", "group the changes", "what belongs together", "summarize my changes", or asks how to split a large diff into commits.
allowed-tools: Bash(git diff *), Bash(git status *), Bash(git ls-files *), Bash(git branch --show-current), Bash(git rev-parse --show-toplevel), Read, Grep, Glob
---

# Commit Plan

## Context (auto-collected)

### Status

```!
git status
```

### Repository root

```!
git rev-parse --show-toplevel
```

### Current branch

```!
git branch --show-current
```

### Staged diff

```!
git diff --cached
```

### Unstaged diff

```!
git diff
```

### Untracked files (relative to the repository root)

```!
git ls-files --others --exclude-standard --full-name :/
```

### Tracked or staged paths that .gitignore matches

```!
git ls-files --cached --ignored --exclude-standard --full-name :/
```

## Task

From the diffs and the untracked list, group every changed file into commit-ready groups, write a title-only Conventional Commits message for each, and emit one runnable block that stages, commits, and pushes every group in dependency order. Investigate with the Read, Grep and Glob tools, not shell commands: everything git can tell you is collected above.

- If both diffs and the untracked list are empty, state that there are no changes and stop.
- If the current branch is empty (detached HEAD), state that there is no branch to push to and stop.
- If the status shows a merge, rebase, cherry-pick or revert in progress, or any unmerged path, state that it must be finished or aborted first and stop.

### Grouping rules

- **Relationship** — within a layer, files that must land together form one group. Test: "would committing these apart break the build or the feature?" This never overrides the layer split or the single test group: an intermediate commit in the sequence need not be independently green — the sequence as a whole must be.
- **Layer** — split a multi-layer change by layer: dependencies, build and CI (manifests, lockfiles, pipeline config), schema and migrations, models and types, core logic, interfaces (routes, controllers, handlers, API, CLI), UI (views, templates, components, styles), config and translations, docs.
- **Granularity** — within a layer, split further by resource or concern: two handlers serving different resources are two groups even when one feature touched both. Each group should read as one focused change a reviewer can hold in their head; when in doubt, split. **Exception — UI stays coarse:** one resource's view, template or component directory is one group.
- **All tests in exactly one group** — every changed test file and test-only support file (fixtures, factories, helpers), whatever its directory or naming convention (`test/`, `tests/`, `spec/`, `__tests__/`, `*_test.go`, `*.test.ts`, `test_*.py`, `*Test.java`, …), forms a single group: never mixed with the code it covers, and never split by resource, test type or directory. A source file that also holds its own inline tests (Rust `#[cfg(test)]` modules, doctests) is not a test file: it goes whole with its code group.
- **Untracked files** carry no diff — open the key ones with the Read tool at the repository root plus the listed path (entry points, not every template) to place them.
- **Moves and deletions** — a deleted path goes in the group whose change removed it, and a moved file's old and new paths go in the same group; `git add` on a deleted path stages the deletion.
- **Drive-by/unrelated edits** form their own group — mark the heading as unrelated.
- **.gitignore decides** — a path in the .gitignore list above that the staged diff shows as a new file was force-added past .gitignore: leave it out of every group (the opening `git reset` turns it back into an ignored file, and `git add` would refuse it and fail its whole group). Other paths in that list are tracked files and are grouped as usual.

### Order

Foundations first: a group that another group depends on comes before it. When nothing forces an order: dependencies, build and CI → schema and migrations → models and types → core logic → interfaces → UI → config and translations → docs → **the single test group last**.

### Message per group

Conventional Commits, in **English**, derived from that group's slice of the diff.

**The message consists solely of the title. This is a zero-tolerance requirement.** Exactly one `-m` per commit; never add a body, a footer, or a second `-m`. Write the title in the imperative mood, at most 72 characters. After the `type(scope):` prefix, start with a lowercase verb and keep every other word in its usual case: proper nouns, acronyms, file names and identifiers stay as written (`MIT`, `macOS`, `README.md`, `CSV`, `UserService`). Count each title's characters yourself before printing it (no command needed), and shorten any that runs over.

**Be specific.** Name the concrete resource, action, template, or key the group changes and what changed about it — pull the nouns from the diff, and use the full 72 characters when it buys precision. Generic subjects are banned: not `refactor(api): update handlers` but `refactor(api): move invoice PDF rendering out of the orders handler`; not `fix(ui): fix search` but `fix(ui): keep search filters when paging through results`.

Prefix = the group's dominant change: `feat` (new capability) · `fix` (bug) · `refactor` (restructure, no behavior change) · `perf` · `test` · `docs` · `style` (formatting) · `build` (deps/build) · `ci` · `chore` · `revert`.

Special cases: dependency bump → `build(deps): bump <pkg> from <old> to <new>`; revert → `revert: "<exact reverted subject>"`; the test group → `test`; a docs-only group → `docs`.

### Output

Exactly two parts, in English — no preamble, rationale prose, closing summary, or next steps.

**Part 1 — the groups.** Per group, in commit order: a heading (unit + layer) and its files as clickable markdown links relative to the repository root (link the directory instead when the group is staged as one).

**Part 2 — the runnable block.** One ` ```bash ` block:

- Open with `cd "$(git rev-parse --show-toplevel)" && git reset` as its own commented statement, before group 1. Every path in the block is relative to the repository root. Take paths from the diff headers and the untracked list, not from the status, whose paths are relative to the current directory.
- Then one statement per group, in the same order, headed `# N — <title>` and never chained to the next group, so each can run on its own: `git add <paths> && git commit -m '<title>' && git push origin <branch> || git reset -q`. `<branch>` is the branch collected above.
- **Stage explicit file paths, not directories.** Use a directory only when the group owns every changed file under it (a UI directory, or a test-only directory) and listing them would bloat the line.
- **Quote every `-m` with single quotes.** For a literal apostrophe, rewrite around it or escape it as `'\''`. Quote any path that contains spaces or shell metacharacters. Git prints non-ASCII paths as quoted octal escapes (`"d\303\266c.md"`); never copy that form into the block or the links — write the real characters.

### Example shape (collected branch: `feature/invoice-export`)

**1. Invoice exports table** (schema)
- [20261031120000_create_invoice_exports.sql](db/migrations/20261031120000_create_invoice_exports.sql)

**2. Invoice CSV export** (core logic)
- [csv_export.py](app/invoices/csv_export.py)

**3. Export endpoint** (interface)
- [routes.py](app/invoices/routes.py)

**4. Invoice list export button** (UI)
- [templates/invoices/](templates/invoices/)

**5. Tests — all test changes, one group** (tests)
- [test_csv_export.py](tests/invoices/test_csv_export.py)
- [test_export_endpoint.py](tests/api/test_export_endpoint.py)

```bash
# 0 — go to the repository root and unstage everything so each commit contains exactly its group
cd "$(git rev-parse --show-toplevel)" && git reset

# 1 — feat(db): add invoice_exports table for queued CSV exports
git add db/migrations/20261031120000_create_invoice_exports.sql && git commit -m 'feat(db): add invoice_exports table for queued CSV exports' && git push origin feature/invoice-export || git reset -q

# 2 — feat(invoices): build CSV export with one row per line item
git add app/invoices/csv_export.py && git commit -m 'feat(invoices): build CSV export with one row per line item' && git push origin feature/invoice-export || git reset -q

# 3 — feat(api): add admin-only POST /invoices/export endpoint
git add app/invoices/routes.py && git commit -m 'feat(api): add admin-only POST /invoices/export endpoint' && git push origin feature/invoice-export || git reset -q

# 4 — feat(ui): add export button and status badge to invoice list
git add templates/invoices/ && git commit -m 'feat(ui): add export button and status badge to invoice list' && git push origin feature/invoice-export || git reset -q

# 5 — test(invoices): cover CSV export rows and admin-only export endpoint
git add tests/invoices/test_csv_export.py tests/api/test_export_endpoint.py && git commit -m 'test(invoices): cover CSV export rows and admin-only export endpoint' && git push origin feature/invoice-export || git reset -q
```

Report only — print the groups and the block. **Never run `git add`, `git commit`, or `git push` yourself**; the user runs it.
