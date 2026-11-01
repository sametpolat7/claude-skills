---
name: write-rspec
description: >-
  Writes new RSpec specs for the changes on the current branch — uncommitted, committed, or both —
  and revises the existing specs tied to the changed files, guided by a bundled Testing Manifesto.
  Use in a Ruby project with an RSpec suite when the user asks to write, add, update, or revise
  tests/specs for their changes or diff — e.g. "write specs for my changes", "test this diff",
  "add specs for this", "update the specs", "cover this with tests". This is the follow-up once a
  change has been verified by hand, not part of its first implementation.
argument-hint: "[uncommitted | committed | hybrid]"
allowed-tools: Bash(git status *), Bash(git diff *), Bash(git log *), Bash(git ls-files *), Bash(git symbolic-ref --short refs/remotes/origin/HEAD), Read, Grep, Glob
---

# Write RSpec — new and revised specs for the current changes

Two concerns, held apart:
- **How to write a good test** → the **Manifesto** decides (quality and style; it outranks the suite's habits).
- **What to stay compatible with** → the existing suite decides (reuse its factories, traits, sequences and setup so new specs don't break neighbors).

Read before writing:
- [references/manifesto.md](references/manifesto.md) — the 11 principles; the authority on *how* and *what* to test.
- [references/templates.md](references/templates.md) — skeletons (service, model, request, factory) that embody the manifesto.

## Scope

Arguments: `$ARGUMENTS`

If the arguments or the user's request already name a scope, use it. Otherwise ask with
AskUserQuestion before anything else — "Which scope should the specs cover?" — and wait for the
answer, as it decides what Step 1 reads:
- **Uncommitted** — the working tree only: staged, unstaged, and untracked files vs `HEAD`.
- **Committed** — commits on this branch not yet in the base branch.
- **Hybrid** — both: every branch commit plus the current working tree.

Write specs once the change has been verified by hand, not during its first implementation. A
request for specs is that signal; if it is unclear, confirm first.

## Procedure

Open files with the Read tool, not `cat` or shell loops; the shell is for git, `bin/rails` tasks and the test commands.

### 1. Survey the changes (per the scope)
- **Committed / Hybrid:** resolve the base branch first — `git symbolic-ref --short refs/remotes/origin/HEAD`; ask if it fails or looks wrong.
- **Uncommitted:** `git status` + `git diff HEAD` + untracked (`git ls-files --others --exclude-standard`).
- **Committed:** `git diff <base>...HEAD`; skim `git log <base>..HEAD` for intent.
- **Hybrid:** `git diff --merge-base <base>` (committed + uncommitted) + untracked.
- Read untracked files in full.
- Read each changed or new `app`/`lib` file fully — for the **behavior it promises outward**, not its internals (#1).
- Note moves and deletions — relocated logic may carry (missing) coverage with it.

### 2. Learn the suite (compatibility, not style)
- No `spec/` directory, or no `rspec` in `Gemfile.lock`? Say the project has no RSpec suite and stop — don't set one up uninvited.
- `.rspec`, `spec/spec_helper.rb`, `spec/rails_helper.rb`, `spec/support/` — what is auto-required, how the database is isolated, which helpers are mixed in (FactoryBot syntax, shoulda-matchers, WebMock, job test modes).
- The factory definitions (`spec/factories/`, `spec/factories.rb`, or the paths in `FactoryBot.definition_file_paths`) — the factories, traits and sequences that already exist.
- For a request spec, an existing request spec in the same namespace — how that part of the app authenticates (a session `sign_in`, a token header).
- The suite's tooling is plumbing too: if it lacks FactoryBot, Faker or shoulda-matchers, work with what it has (fixtures, plain builders, literal values) and say so. Never add a gem or change the Gemfile or spec helpers without asking.

### 3. Map changes → specs
- Specs mirror `app/` and `lib/` (`app/services/x/y.rb` → `spec/services/x/y_spec.rb`); HTTP behavior goes in `spec/requests/` (`app/controllers/admin/reports_controller.rb` → `spec/requests/admin/reports_spec.rb`).
- Per changed unit: **write new**, **revise existing**, or **leave alone**.
- Flag **stale specs** — code gained behavior the spec never asserts (e.g. a method now writes a second table too); revising these matters as much as adding new ones.
- Share a short plan (files, layer, new vs revise) before a large batch.

### 4. Decide what earns a test (be ruthless — #2, #3)
- Test **our** logic only, never the framework's guarantees (`validates :presence`, plain associations). A `shoulda-matchers` one-liner is fine shorthand for *our* declared rules — don't pad with them.
- Every test must be able to **fail for a real defect**; else drop it.
- Pick the layer, integration-first (#7, #8):
  - **Service / PORO** → service spec, through the public method (`described_class.new(...).call`).
  - **Controller / HTTP** → **request spec** (`type: :request`), never a new controller spec. An existing controller spec stays: run it, fix only what the change breaks, and note it in the report.
  - **Model logic** (scopes, state methods, custom validations, callbacks with real effects) → model spec.
- Stub only true external boundaries (#7).

### 5. Write / revise the specs
Per the templates. Non-negotiables:
- Start each file the way the suite's files start: the same magic comment (e.g. `# frozen_string_literal: true`) and the same `require` line (e.g. `require 'rails_helper'`), or none if the suite relies on `.rspec`.
- **AAA**, clearly separated (#4).
- `it` names describe behavior (#5).
- **DAMP over DRY** (#6): a local helper beats scattered indirection.
- **Lean factories, rich traits** (#10): add a **trait** for a missing variation, not base attributes; create a lean factory only if none exists. Grep the suite for `factory :<name>` first and **never redefine a factory** (duplicate names break other specs). A new factory builds its own associations rather than leaning on seed rows (`Model.first`) or hard-coded ids.
- **Faker** for realistic data (#11); the suite's sequences for unique values.
- Make randomness in the code under test deterministic (`srand(42)`, or inject the random source).

### 6. Run & iterate
- Schema changed? Bring the test database up to date first (`bin/rails db:test:prepare`). Never rewrite `db/schema.rb` or `db/structure.sql` to make specs pass.
- Run: `bundle exec rspec spec/path/to/x_spec.rb`.
- Touched a **shared** factory, trait or helper? Also run its existing dependents, to prove you didn't break neighbors.
- Iterate until green. If a test can't pass because the code is wrong, **surface the defect** — don't bend the test to buggy behavior.

### 7. Final self-check
- Would it fail if the behavior broke? If not, delete it.
- Behavior, not implementation — would a legitimate refactor keep it green?
- Readable top-to-bottom without hunting across files?
- Report honestly: what you added or revised, the command run, and the pass/fail output.
