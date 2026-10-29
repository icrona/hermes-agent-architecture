---
name: <skill-name>
description: <One sentence. What it does and the single file it writes.>
triggers:
  - "/<command> <args>"
  - "<natural phrase that should start this skill>"
version: 0.1.0
---

# <Skill name>

<One paragraph: what this skill does, which file it writes, and that it stays cheap.>

## When to use
- The owner sends `/<command>`, or a message that clearly asks for this.
- Do not use for <neighbouring task>. Use `<other-skill>` instead.

## Inputs
- `<arg1>`: <meaning, format>
- `<arg2>` (optional): <meaning, default>
- Source: `telegram`, a URL, or `forwarded`. Never a person's name.

## Procedure
1. `cd <workdir> && git pull --rebase -q`. Done when there is no error.
2. Read `AGENTS.md` for the line format and the PII rule.
3. <Build the content. Say which tool or script, and what a good result looks like.>
4. <Write it to the one target file, in the exact line format.>
5. `git add <file> && git commit -qm "<skill>: <3 words>" && git push -q`.
6. Reply on Telegram with one line that points at the file written, for example `saved to <file> · <first 60 chars>`.

## Verification
- `tail -1 <file>` shows the new line in the exact format.
- `git log -1` shows the commit and `git status` is clean.
- The Telegram reply is one line.

## Hand-off rule
If the task needs a browser, a login or more than <N> minutes of research, append a `#research` line to `inbox.md` and stop. The Mac session does that work.

## Never
- Never put <secrets, document numbers, card digits> into any file.
- Never book, pay, submit a form or log in to a third-party site.
- Never delete a line from an append-only file.
- Never push to a default branch of a project repo.
- Never expand scope past the one file this skill owns.
