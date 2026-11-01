# claude-skills

[![Release](https://img.shields.io/github/v/release/sametpolat7/claude-skills)](https://github.com/sametpolat7/claude-skills/releases/latest)
[![License: MIT](https://img.shields.io/github/license/sametpolat7/claude-skills)](LICENSE)

Three [Claude Code](https://code.claude.com/docs/en/overview) skills for everyday development work: planning commits, writing RSpec specs, and cleaning up code comments.

A skill is a folder of instructions that Claude Code follows when you type its name as a slash command.

## Skills

| Skill | What it does |
|---|---|
| [commit-plan](#commit-plan) | Groups your uncommitted changes into commits and prints the commands to make them. |
| [write-rspec](#write-rspec) | Writes and updates RSpec specs for the changes on your branch. |
| [prune-comments](#prune-comments) | Removes comments that add nothing and trims the rest. |

### commit-plan

Splits every uncommitted change into focused groups, writes a [Conventional Commits](https://www.conventionalcommits.org) title for each, and prints a ready-to-run block that commits and pushes them in dependency order. It only reads your repository; you run the block. Works with any language.

```text
/commit-plan
```

### write-rspec

Writes new RSpec specs and updates existing ones for the changes on your branch, following a bundled [testing manifesto](skills/write-rspec/references/manifesto.md). It runs the specs and reports a bug in the code instead of bending a test around it. Requires a Ruby project with an RSpec suite.

```text
/write-rspec [uncommitted | committed | hybrid]
```

The argument picks which changes to cover: uncommitted work, commits on your branch, or both. Without it, the skill asks.

### prune-comments

Removes comments that carry no information, shortens the rest to what the code can't say, and lists the ones that need your attention, such as contradictions, unfinished code and security issues. It never changes code or comments that tools read. Works with any language, and runs only when you call it.

```text
/prune-comments [path | glob | git-range] [--hard] [--dry-run]
```

Without a target, it covers the whole project. `--dry-run` previews the changes, and `--hard` applies the strictest rules.

## Installation

You need Claude Code and git.

```bash
git clone https://github.com/sametpolat7/claude-skills.git ~/claude-skills
mkdir -p ~/.claude/skills
for d in ~/claude-skills/skills/*/; do ln -s "${d%/}" ~/.claude/skills/; done
```

Start a new Claude Code session and type `/` to see the skills.

- **Update:** run `git pull` in `~/claude-skills`. The linked skills update right away.
- **One skill only:** copy its folder from `skills/` into `~/.claude/skills/`.

## Compatibility

These skills are built for [Claude Code](https://code.claude.com/docs/en/skills). `write-rspec` and `prune-comments` use Claude Code frontmatter fields (`argument-hint`, `disable-model-invocation`) that claude.ai and the Skills API don't accept on upload.

## License

[MIT](LICENSE) © Samet Polat
