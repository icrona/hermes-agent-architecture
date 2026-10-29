# Workflows

Four flows cover everything that happens in this system: deploy a change, research hand-off, engineering hand-off and scheduled jobs. Each flow moves Markdown through git. The owner's phone triggers the hand-offs. The memory model behind them is in [Shared memory](03-shared-memory.md). The profiles are described in [Profiles](02-profiles.md).

```mermaid
flowchart LR
    Owner["Owner<br/>phone"]:::owner
    Mac["Mac<br/>Claude Code"]:::mac
    GH["Private repos"]:::github
    Box["Box<br/>Hermes profiles"]:::box
    Owner -->|A change request| Mac
    Mac -->|A push| GH
    GH -->|A deploy.sh| Box
    Owner -->|B and C notes| Box
    Box -->|B and C push| GH
    GH -->|B and C pull| Mac
    Box -->|D cron output| Owner
    classDef box fill:#dbeafe,stroke:#1d4ed8,color:#1e3a8a
    classDef mac fill:#dcfce7,stroke:#15803d,color:#14532d
    classDef github fill:#ede9fe,stroke:#6d28d9,color:#3b0764
    classDef owner fill:#fef3c7,stroke:#b45309,color:#78350f
    classDef external fill:#ccfbf1,stroke:#0f766e,color:#134e4a
    classDef secret fill:#fee2e2,stroke:#b91c1c,color:#7f1d1d,stroke-dasharray:4 2
```

## A. Change a profile (Mac to box)

```mermaid
sequenceDiagram
    participant Mac as 🟩 Mac (Claude Code)
    participant GH as 🟪 GitHub
    participant Bx as 🟦 Box (Hermes)
    Mac->>Mac: edit profile source, test scripts
    Mac->>GH: commit and push main
    Bx->>GH: pull (10-minute cron or by hand)
    Bx->>Bx: deploy.sh <profile>
    opt config changed
        Bx->>Bx: restart the gateway
    end
    Mac->>GH: append status-log entry
```

Steps:

1. Edit the profile source on the Mac. Test scripts with Python 3.12.
2. Commit and push to `main`.
3. The box pulls. The 10-minute cron does it, or an operator runs `git pull` by hand.
4. Run `deploy.sh <profile>`. It is idempotent. It creates the profile if missing, deep-merges `config.yaml`, copies SOUL and the USER seed, syncs skills with delete, seeds workdir files that do not exist, and applies the cron list by job name.
5. Restart the gateway only if config changed.
6. Append a status-log entry. Both machines append to it.

The contract between the repo and the script is in [the deploy contract](../templates/deploy-contract.md).

Observed cost: every distinct system prompt rewrites the cache prefix. Over 7 days one profile saw 9 distinct prompts and another 8. Cache writes were 49 to 81% of cost. Deploys are therefore batched.

## B. Research hand-off (box to Mac to box)

```mermaid
sequenceDiagram
    participant Owner as 🟨 Owner (phone)
    participant Bx as 🟦 Box (Hermes)
    participant GH as 🟪 GitHub
    participant Mac as 🟩 Mac (Claude Code)
    Owner->>Bx: /note <topic>
    Bx->>GH: pull --rebase, append inbox line, push
    Bx-->>Owner: one-line ack
    Mac->>GH: pull --rebase at session start
    Mac->>Mac: handle every unstruck inbox line
    Mac->>GH: write per-item files, strike lines, push
    Owner->>Bx: pastes "research is pushed"
    Bx->>GH: pull
    Bx-->>Owner: answers from files
```

The Mac holds browser logins and a better model. The box captures in one turn and answers later from files.

Observed over five days:

| Measure | Value |
|---|---|
| Inbox lines | 15 |
| Handled | 11 |
| Plans completed | 2 |
| Rejected pushes | 2, recovered by the pull-then-push rule |
| Conflicts | 0 |

## C. Engineering hand-off (both directions)

```mermaid
sequenceDiagram
    participant Owner as 🟨 Owner (phone)
    participant Bx as 🟦 Box (Hermes)
    participant GH as 🟪 GitHub
    participant Mac as 🟩 Mac (Claude Code)
    Owner->>Bx: /task <small change>
    Bx->>GH: push hermes/* branch, open PR
    Bx->>GH: update Facts and Journal in repo card
    Bx->>GH: add "→ Mac" line for a spike
    Mac->>GH: pull, take the line, remove it
    Mac->>GH: push mac/* branch, open PR
    Mac->>GH: write Decisions, add "→ box" line
    Owner->>GH: review and merge from phone
```

Rules:

- The box does PR-sized tasks on `hermes/*` branches.
- The Mac does spikes and migrations on `mac/*` branches.
- Neither pushes to a default branch. The owner merges.
- Each side edits only the card sections it owns, per [Shared memory](03-shared-memory.md).

Cron jobs on the box: a weekly discover job, a weekday PR digest, a weekly health check and a monthly audit. A dependency job is defined and paused.

Observed: about 20 PRs were opened or merged in two Mac hand-off runs.

## D. Scheduled jobs

A **cron job** in Hermes is a stored schedule that runs a skill and delivers the result to chat.

```mermaid
flowchart TB
    F["cron list in the repo<br/>one line per job"]:::github
    D["deploy.sh applies by name"]:::box
    S["Hermes scheduler"]:::box
    TZ["Expressions evaluated in<br/>profile timezone"]:::box
    J["Job runs a skill<br/>in the workdir"]:::box
    T["Telegram message<br/>to the owner"]:::owner
    F --> D --> S --> TZ --> J --> T
    classDef box fill:#dbeafe,stroke:#1d4ed8,color:#1e3a8a
    classDef mac fill:#dcfce7,stroke:#15803d,color:#14532d
    classDef github fill:#ede9fe,stroke:#6d28d9,color:#3b0764
    classDef owner fill:#fef3c7,stroke:#b45309,color:#78350f
    classDef external fill:#ccfbf1,stroke:#0f766e,color:#134e4a
    classDef secret fill:#fee2e2,stroke:#b91c1c,color:#7f1d1d,stroke-dasharray:4 2
```

Apply by name makes a redeploy skip a job that already exists instead of duplicating it. To change a job, delete it on the box and redeploy. See [the cron example](../templates/cron.example.txt).

The timezone lesson: Hermes evaluates cron expressions in the profile timezone, so a schedule pre-converted to UTC fires at the wrong hour. Details in [Lessons](08-lessons.md).

## E. Keeping clones fresh

| Direction | Mechanism | Latency |
|---|---|---|
| Mac to box | `git pull --rebase` at the start of each skill procedure | Immediate when a skill runs |
| Mac to box | Crontab fast-forward of three clones, ff-only, skips dirty or diverged clones and logs | 10 minutes at most |
| Box to Mac | `git pull --rebase` at the start of each Mac session | Next Mac session |
| "New work exists" | Owner pastes a line into the bot, or starts a Mac session | Manual |

The owner is the message bus. Nothing notifies the other machine on push. The 10-minute cron removes the stale-answer case when the paste is forgotten.

## F. Observability

| Tool | What it gives |
|---|---|
| Token audit script, read-only | Per-profile calls, list cost, prompt size, memory fill, prompt-hash churn |
| Token audit skill | The same audit on request, with thresholds and flags |
| Dated baselines | A measurement file per audit, from 2026-10-17 |
| Status log | Newest entry first, appended by both machines |
| Sync log on the box | One line per fast-forward or skip |

Baselines: about $3.5 list per day for one active profile on 2026-10-17, about $10.7 list per week for all profiles on 2026-10-25.

## Sources and dates

Workflows observed from 2026-10-17 to 2026-10-29. Hand-off counts from 2026-10-20 to 2026-10-25. Token audits on 2026-10-17 and 2026-10-25.
