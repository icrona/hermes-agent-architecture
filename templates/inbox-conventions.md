# Inbox conventions

`inbox.md` is the shared capture file of a workdir. The box and the Mac both append to it. It is append-only, plain text and git-tracked.

## Line format

    - YYYY-MM-DD HH:MM · #tag[ #tag] · <text, one line, 200 chars max> · <source>

- Time is in the profile timezone `<IANA-timezone>`.
- Source is `telegram`, a URL, or `forwarded: <sender type>`. Never a person's name.
- One idea per line. No line breaks inside the text.

## Tag vocabulary

| Tag | Meaning |
|---|---|
| `#<topic-a>` | <what belongs here> |
| `#<topic-b>` | <what belongs here> |
| `#idea` | Something to consider later. |
| `#question` | Needs an answer from the Mac session. |
| `#research` | Needs a browser, a login or long research. The box stops and hands over. |
| `#other` | Fallback. Use one or two tags per line at most. |

## Strike-through

- `- ~~2026-01-01 09:00 · #idea · text · telegram~~ → <file that answers it>` means handled.
- A line is never deleted. The strike keeps the history and the diff readable.
- The side that does the work strikes the line, in the same commit as the result.
- The other side never edits a line it did not write, except to strike it after the work is done.

## Monthly archive

- Once a month, move struck lines older than 30 days to `inbox-archive.md`.
- Keep open lines in `inbox.md` whatever their age.
- One job does this: `<profile>-inbox-archive`, box side, one commit.

## Why a file beats a database here

- Both agents already have a shell and git. A file needs no client, no schema, no server.
- `git pull --rebase` resolves concurrent appends; two appended lines rarely conflict.
- `git log` is the audit trail. `tail` and `grep` are the query tools.
- The owner can read and fix it on a phone.
- The file costs a few hundred tokens to load when needed, and zero when not.
