# Deploy contract

`deploy.sh <profile>` turns a profile source folder into a live profile on the box. These are its six steps as pseudocode. Run it as the agent user, from the checkout of the setup repo.

## Steps

1. Create the profile if it is missing.

        if profile <name> not in `hermes profile list`:
            hermes profile create <name>
        assert dir ~/.hermes/profiles/<name> exists

2. Deep-merge the config, with a backup.

        new = load(profiles/<name>/config.yaml)
        cur = load(live config.yaml) or {}
        copy live config.yaml -> config.yaml.bak-deploy
        merged = deep_merge(cur, new)        # repo keys win, other keys stay
        write(live config.yaml, merged)
        hermes -p <name> config check

   2b. Optional gateway routes, merged into the default profile.

        if profiles/<name>/gateway.yaml exists:
            text = read(gateway.yaml)         # contains __VAR__ placeholders
            missing = vars in text not set in profile .env
            if missing: print "not merged, set <missing>"; skip
            else: fill placeholders from .env; deep_merge into ~/.hermes/config.yaml; back up first

3. Copy SOUL, seed memories, sync skills.

        copy SOUL.md -> profile home (overwrite)
        for each file in memories/:
            if file is USER.md and live copy exists and has no "TODO": keep it
            else: copy with mode 600
        for each category in skills/:
            rsync -a --delete skills/<cat>/ -> profile skills/<cat>/

4. Seed the workdir, init git, commit.

        workdir = config.terminal.cwd           # fail if unset
        rsync -a --ignore-existing workspace-seed/ -> workdir
        if workdir has no .git:
            git init -b main
            git config user.name  "hermes-<name>"
            git config user.email "hermes-<name>@localhost"
        if working tree dirty: git add -A && git commit -m "deploy: seed/update workspace"

5. Apply cron lines idempotently.

        for each line in cron.txt (skip blanks and # comments):
            name = value of --name
            if name in `hermes -p <name> cron list`: skip
            else: hermes -p <name> cron create <line>

6. Report.

        hermes -p <name> skills list
        hermes -p <name> cron list

## Invariants

- Never touches `.env`. Secrets are created by hand on the box.
- Safe to re-run. A second run changes nothing unless the source changed.
- Repo wins for SOUL, skills and config keys. The box wins for keys it wrote and for a filled-in USER.md.
- Changes to `config.yaml` or gateway routes need a gateway restart. Skills, SOUL and cron do not.
- A failed config check stops the run before anything else is copied.

## Why idempotent

The box is re-deployed many times a week. Skills change often, the owner edits from the Mac, and a half-applied deploy must be fixable by running it again. Idempotence also lets any session on the box re-run a deploy without asking what state it is in.
