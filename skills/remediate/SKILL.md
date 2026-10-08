---
name: remediate
description: Fix one drift or research finding filed as a GitHub issue by an automated nightly check — e.g. "/remediate infra#718", "remediate 474", "fix the finding in issue 718". Read-only recon, plan, wait for go-ahead, then branch → PR → deploy, and close the loop in the ledger and the issue.
---

# Remediate a drift finding

An automated nightly check only *reports* drift: it files a GitHub issue (label
`agent-research`) in the repo that owns the affected host and notifies the operator.
This skill is the operator's separate "fix it" ask. One issue per invocation.

Argument: an issue reference — `<repo>#<n>`, a bare number (use the current repo), or an
issue URL. Each host belongs to exactly one infra repo. If this session's repo is not the
issue's repo, STOP and say which checkout to open instead — never fix a host from a repo
that does not own it.

## Step 1 — Recon (read-only, no go-ahead needed)

1. `gh issue view <n> --comments` — read the finding, its evidence, and the runbook section
   or watch-list row it cites. Note any linked issues (known lags, deliberate holds).
2. `git fetch && git status` — confirm the checkout is current; pull `main` if behind.
3. Read the cited section of the repo's drift-check runbook (and its open-findings ledger)
   or deployed-versions doc. Check the do-not-flag / deliberate-hold lists: if the finding
   is a known hold, the fix is a docs change (or closing the issue), not a deploy.
4. Re-verify the evidence live, read-only (`ssh <host> '...'` with status/inspect/version
   commands only). Drift that has already cleared → comment on the issue, close it, stop.
5. For version bumps: read the upstream release notes between deployed and target, and
   note breaking changes, migrations, and the rollback path (previous image tag, config
   backup).

## Step 2 — Plan, then WAIT

Post one plain-language plan and stop for the operator's explicit go-ahead. It must state:

- what changes, on which host(s), and the exact files in the repo
- the deploy command(s) and rollback command(s)
- the docs that move in the same PR (versions-doc row, ledger entry, runbook baseline)
- risk and blast radius, including anything that serializes on a shared target
  (one checkout deploys to a given host at a time)

No change to any server or repo file before the yes. If the operator answers with a change
of scope, re-plan; do not partially proceed.

## Step 3 — Change (after go-ahead)

1. Branch from `main`: `fix/…` or `chore/…` with the issue number in the slug.
2. Make the repo change (unit file, config, script, versions doc, ledger). Minimum diff;
   touch only what the issue needs.
3. Run the repo's gates (its `CLAUDE.md` lists them). Commit with a single-line summary
   that ends `(#<n>)`. Push: `git push -u origin <branch>`.
4. Open the PR with `gh pr create`: body = Summary + Test plan + `Closes #<n>`. The PR
   title becomes the squash commit and changelog line — write it as one.

## Step 4 — Deploy and verify (after go-ahead, per the repo's deploy procedure)

- Infra repos deploy from the branch to the live host and verify against the fleet;
  app repos deploy on merge. Follow the repo's documented procedure, never an ad-hoc one.
- Verify with the same command the runbook uses for that check, and paste the output.
- If verification fails: roll back with the command from Step 2, report what happened,
  leave the PR open with a comment. Do not retry a different fix without a new plan.

## Step 5 — Close the loop

1. Comment on the issue with the verification output and the PR link.
2. Merge only on the operator's explicit go-ahead (`gh pr merge --squash --delete-branch`),
   then `git checkout main && git pull`.
3. State in the final message: issue, PR, host(s) changed, verification evidence, and that
   the next nightly run is the confirmation the fix held — it should report the row
   CURRENT / the ledger item resolved, not re-report it.

## Never

- Fix drift that the issue does not cover ("while I'm here" changes belong in a new issue).
- Edit the runbook's baseline or the versions doc without the matching deploy in the same PR.
- Act on a host the current repo does not own.
- Treat "the fix is in the PR" as done — done means verified live and the issue closed.
