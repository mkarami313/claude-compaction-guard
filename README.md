# compaction-guard

Keep goals, instructions, decisions and next steps alive across Claude Code context compaction.

A [Claude skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) for Claude Code — Python 3 standard library only, no dependencies. Part of the [mkarami313/Claude-skill](https://github.com/mkarami313/Claude-skill) collection.

Compaction keeps a summary and drops the rest — and the summary predictably loses your exact instructions, why decisions were made, which files were mid-change, and what was next. `compaction-guard` keeps that in a small state file and puts it back into context automatically after every compaction.

**Use it when** sessions get long, when Claude "forgets" instructions after `/compact` or auto-compact, or before a big multi-step task.

## How it works
| Layer | What it does |
|---|---|
| `NOW.md` | A ≤60-line state file Claude keeps current: goal, your instructions verbatim, decisions with reasons, in-progress files, next steps. Updated with small edits at milestones. |
| PreCompact hook | Writes `snapshot.md` from the transcript (first request, last user messages, files changed, recent failures — secrets redacted) and appends short "what to keep" instructions to the compaction prompt. Never blocks compaction. |
| SessionStart (`compact`) hook | Re-injects `NOW.md` + snapshot (kept under Claude Code's ~10k limit) right after compaction, so the next turn starts with the real state. |

## Setup
```bash
python3 skills/compaction-guard/scripts/guard.py install          # this project (.claude/settings.json)
python3 skills/compaction-guard/scripts/guard.py install --user   # all projects (~/.claude/settings.json)
python3 skills/compaction-guard/scripts/guard.py status           # what would be re-injected
```
Existing hooks are kept and a `.bak` is written; running it twice is safe; `uninstall` removes only its own hooks; `--on-clear` also re-injects after `/clear`. Restart Claude Code after installing.

## What's inside
```
skills/compaction-guard/
├── SKILL.md                 NOW.md format and update rules, what to do after compaction
└── scripts/
    └── guard.py             install / uninstall / precompact / sessionstart / status
```

## Installation

### Claude Code — plugin marketplace (recommended)
```
/plugin marketplace add mkarami313/claude-compaction-guard
/plugin install compaction-guard@claude-compaction-guard
```
Update later with `/plugin marketplace update claude-compaction-guard`.

### Claude Code — copy the skill folder
```bash
git clone https://github.com/mkarami313/claude-compaction-guard
cp -r claude-compaction-guard/skills/compaction-guard <your-repo>/.claude/skills/     # this project only
cp -r claude-compaction-guard/skills/compaction-guard ~/.claude/skills/               # all your projects
```

### Claude.ai / Claude desktop app
Zip the folder so that `compaction-guard/SKILL.md` is at the root of the archive and upload it in Claude's skills settings:
```bash
cd claude-compaction-guard/skills && zip -r compaction-guard.zip compaction-guard
```

## Testing
`tests/compaction-guard/scenarios.md` lists pressure scenarios. After changing the skill, walk through each one with the new `SKILL.md` (or give it to a fresh Claude session) and check the expected behaviour still follows.

## Contributing
Issues and PRs welcome. Keep `SKILL.md` lean (long material → `references/`, code → `scripts/`), scripts stdlib-only with LF endings, and add a scenario for every behaviour you change.

## License
[MIT](LICENSE)
