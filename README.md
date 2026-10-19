# claude-guardrails

The rules I give Claude Code so it doesn't break production again.

I've been writing software for about 25 years. These days I build two live products mostly with Claude Code, with real users, live payments and a production database. It's fast, it's good, and when it goes wrong it goes wrong with total confidence.

This repo is the `CLAUDE.md` I ended up with. Every rule in it is there because something actually broke.

## The night that wrote most of these rules

April 2026. I approved a database rename on [zoustos.com](https://zoustos.com), my sports predictions site. Simple plan: back up, stop the app, migrate, restart.

Here's what actually happened, in about five minutes:

1. The backup step failed. The `mongodump` flags conflicted with the connection string, but the script sent errors to `/dev/null`, so nothing told us.
2. The script didn't check exit codes, so it kept going.
3. The app restarted pointing at a database and user that didn't exist. Site down.
4. Claude wrote a "recovery" script. It had the same backup bug and the same silent errors. Its last step dropped the original production database, the one that was still intact.
5. The script printed `MIGRATION COMPLETE`.

Then, while we were working out what could be saved, Claude told me the last backup would bring back "97K historical games." I asked it to check. The backup had 12,828. The number had come from a note written 12 days *after* that backup was taken. A few minutes later it told me the oplog covered 6 days. It covered about 32 hours.

None of that was malice or a crazy bug. It was confidence without checking, three times in one evening. So now the rules say: no number gets stated unless a tool produced it in the same turn, and "it ran" means checking the exit code *and* the thing it was supposed to produce.

The other big one: twice, a secret value ended up in an AI session transcript. Each time I had to rotate every credential, which took about a day each time. Now Claude never reads secret files, full stop.

## What's in here

| File | What it is |
|---|---|
| [`CLAUDE.md`](CLAUDE.md) | The 10 rules, in priority order. Claude Code loads it at the start of every session. |
| [`.claude/commands/security-scan.md`](.claude/commands/security-scan.md) | `/security-scan`, an audit of your code, servers and credentials. It reports and stops. It doesn't "fix" anything and never prints a secret value. |

## Install

From your project root:

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/selaias/claude-guardrails/main/CLAUDE.md
mkdir -p .claude/commands
curl -o .claude/commands/security-scan.md https://raw.githubusercontent.com/selaias/claude-guardrails/main/.claude/commands/security-scan.md
```

Already have a `CLAUDE.md`? Paste the rules at the top. The order matters: security first, everything else after.

Then run `/security-scan` inside Claude Code and see what it finds. Fair warning: the first run on an older repo is usually humbling. It was for me.

## What this won't do

It won't make Claude infallible. These are instructions, not a sandbox. Claude follows them most of the time, and it follows them much better than nothing. Keep real backups, keep real permissions, and keep yourself in the loop for anything you can't undo.

## What's next

These two files are the part I'd want every Claude Code user to have, so they're free. I also use a set of workflow commands every day: ship (version bump, changelog, verify, push), QA plans that split automated from manual tests, a go-live checklist, secret-safe server scripts, and an end-of-session handoff so the next session doesn't start from zero. I'm cleaning those up to release as well.

If you'd use them, ⭐ the repo so I know it's worth finishing. If one of these rules saved you from something, or you think one is wrong, open an issue. I'd genuinely like to hear it.

## License

MIT. Take it, change it, use it at work.
