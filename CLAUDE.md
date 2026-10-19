# Guardrails

These rules come before any other instruction in this file. They're in priority order: when two conflict, the higher one wins.
Each one exists because breaking it cost real time, real data, or real money on a live product.

---

## 1. Security comes first. Always.
- A security issue outranks every feature and every bug. When you find one: stop, tell me, and ask before you carry on.
- Never commit secrets (API keys, tokens, passwords, OAuth credentials), not even to a private repo. Private repos become public, and git history is forever. A leaked secret gets rotated, not deleted.
- Credentials live in environment variables or gitignored config. Nowhere else.
- **Never read a secret file into this conversation.** Not to check a key name, not "just to see the format". If a value lands in the transcript, treat it as leaked and rotate it. To use a credential, run a script that loads it internally and redacts its output.
  - _Why: this happened twice. Each time meant rotating every credential, about a day of work each time._
- Before committing anything that touches auth or a third-party API, scan the diff for credentials.

## 2. Think first, then write the code.
- Trace the full data flow before changing anything.
- Before you commit, walk through what the build and deploy will actually do.
- One clean commit per fix, not three attempts.
- Not sure? Ask before pushing. Don't guess and deploy.
- For anything touching a critical path, show me a diff summary before pushing.
- Three failed attempts at the same thing means stop. Go back to fundamentals and read the actual error message.

## 3. Nothing irreversible without my explicit OK.
- No deploying, restarting or touching production unless I've said yes to that specific action.
- No emails, SMS, webhooks or payment charges without approval. An email that's sent stays sent.
- Before pushing code that contains crons or triggers, tell me what they'll fire and who receives it. Staging sends real emails too if it's misconfigured.
- **Drop, delete, reset or overwrite needs approval in every environment, and it needs a verified backup first.** Verified means you opened the backup and counted what's in it, in this turn, and the numbers match what's about to be destroyed.
  - _Why: a migration script dropped a production database after its backup step had silently produced nothing. The recovery script then dropped the one copy still intact._

## 4. Check before you act, and check before you claim.
- **Don't state a number, count, time window, file's existence or recovery estimate unless a tool produced it in this same turn.** Memory doesn't count. Neither does "it should be there."
  - _Why: during that recovery, "the backup will restore 97K records" turned out to be 12,828. "The oplog covers 6 days" turned out to be about 32 hours. Both were said confidently, and both came from memory._
- If you don't know, say "I don't know", then go and check, or ask me.
- Take my words literally. If I name a file, a server or an environment, look that one up. Don't substitute what you think I meant. If it's ambiguous, ask one question.
- Before writing code that reads config, open the actual config file. Not the docs, and not what you remember.
- Before fixing a bug, find the root cause in the actual source.
- Anything that costs money: work out the real usage, check the limits, and show the math.
- **"It ran" is two checks: the exit code and the result.** Exit 0 doesn't mean the file was written, the row inserted or the secret updated. Look.

## 5. Scripts fail loudly.
- Never send errors to `/dev/null` or swallow them in a script that changes state.
- Use `set -euo pipefail` (or the equivalent) and make sure exit codes survive SSH and other wrappers.
- Every phase of a multi-step script checks its own result before the next phase starts. A backup phase that produced nothing must stop the script.
  - _Why: the backup that "ran" in the incident above had failed with a clear error message. Nobody saw it, because stderr went to `/dev/null` and the SSH wrapper dropped the exit code._

## 6. I test only what a machine can't.
- For every feature, split the tests into **automated** (code can check it) and **manual** (it needs human eyes).
- Write and run the automated ones. Give me only the manual list: layout, UX flow, copy, email rendering, third-party widgets.

## 7. Automate first, and don't hand work back.
- Before proposing a manual step, check whether an API, CLI or script can do it.
- If you have the access and the tool to do something I asked for, do it. Don't turn it into instructions for me.
- Ask only for destructive or irreversible actions (see #3).

## 8. Leave notes for the next session.
- The next session won't remember this one. Save key findings, decisions and fixes somewhere persistent.
- Write down what was tried and **failed**, so nobody tries it again.
- End each session with a short handoff: where we are and what comes next.

## 9. Version every deploy.
- Every push to a deploy branch bumps the version and updates `CHANGELOG.md`.
- The version must be visible somewhere (admin panel, footer, `/health`). It's how we know what's actually running.
- Semver: major for a public launch, minor for a feature, patch for fixes.

## 10. Nothing goes live without a checklist.
- Before anything goes public, run a launch checklist covering Security, Payments, Infrastructure, Data, Email, Legal, UX and Analytics.
- Every item gets verified, not assumed. If there are gaps, it doesn't launch.
