# claude-skills

[Claude Code](https://code.claude.com/docs/en/overview) skills for everyday development work.

## Skills

| Skill | What it does | How to use |
|---|---|---|
| [`commit-plan`](skills/commit-plan/SKILL.md) | Splits every uncommitted change into commit-ready groups, writes a Conventional Commits title for each, and prints a ready-to-run block that commits and pushes them in dependency order. It only reads: it never stages, commits or pushes. Works with any language. | `/commit-plan`, or ask for a "commit plan" |
| [`write-rspec`](skills/write-rspec/SKILL.md) | Writes new RSpec specs and revises existing ones for the changes on your branch, following a bundled testing manifesto. It runs them and reports a bug in the code instead of bending a test around it. Needs a Ruby project with an RSpec suite. | `/write-rspec [uncommitted \| committed \| hybrid]` |

## Install

You need Claude Code and git.

Clone the repository and link each skill into your personal skills folder:

```bash
git clone https://github.com/sametpolat7/claude-skills.git ~/claude-skills
mkdir -p ~/.claude/skills && for d in ~/claude-skills/skills/*/; do ln -s "${d%/}" ~/.claude/skills/; done
```

Start a new Claude Code session and type `/` to find the skills. Because they are links, `git pull` in the clone keeps them up to date.

To install a single skill, copy its folder from `skills/` into `~/.claude/skills/` instead.

## Compatibility

These are [Claude Code skills](https://code.claude.com/docs/en/skills). Some use frontmatter fields that only Claude Code supports, such as `argument-hint` and `disable-model-invocation`, so claude.ai and the Skills API reject those skills on upload.

## License

[MIT](LICENSE)
