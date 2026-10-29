# AGENTS.md, shared contract for <workdir-name>

This repo is shared by two agents: the Hermes agent on the box and a Claude Code session on the Mac.
Both read this file at the start of every turn. Each file below has one owner section. Do not edit the other side's lines.

## Git rule
- Run `git pull --rebase` before writing anything.
- Run `git add -A && git commit -m "<what changed>" && git push` after writing.
- Never commit `.env*`, tokens or anything under `<ignored-folder>/`.

## PII rule
Never write these into any file in this repo:
- <document numbers>
- <booking references>
- <card digits>
- <full names of third parties>
- <list more as needed>

## File map
| File | Purpose | Written by |
|---|---|---|
| `inbox.md` | Captured ideas and questions, append-only. | both |
| `<topic>/<item>.md` | One file per tracked item. | Mac |
| `<topic>/<item>.facts.yaml` | Machine-readable facts for one item. | box |
| `journal.md` | What the box did and when, append-only. | box |
| `checklist.md` | Open actions with due dates. | both |
| `decisions.md` | Dated one-line decisions. | Mac |
| `handoff.md` | Work passed to the other side. The taker removes the line. | both |
| `scripts/` | Helper scripts, stdlib only. | Mac |

## Line formats
Keep them exact. Scripts parse them.

    inbox:     - YYYY-MM-DD HH:MM · #tag · text · source
    journal:   - YYYY-MM-DD · <task, 80 chars max> · <branch or -> · <link or -> · <result>
    checklist: - [ ] YYYY-MM-DD · <action> · <owner: box|Mac|human> · <item>

A handled inbox line is wrapped in `~~...~~`. It is never deleted. See [inbox conventions](inbox-conventions.md).

## Helper scripts
<!-- Run with `python3 -I <path> <args>`. One line per script: path, arguments, what it prints. -->
- `<scripts/example.py> <arg>`: <what it prints>

## Cron job names
<!-- Box only. Name, schedule in the profile timezone, one-line purpose. -->
- `<profile>-<job>`: <schedule>, <purpose>
