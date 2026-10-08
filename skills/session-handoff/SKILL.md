---
name: session-handoff
description: Use when the user is about to clear, end, or compact a session and wants in-flight work preserved — e.g. "prepare a handoff", "I'm about to clear", "wrap up this session", or /session-handoff.
---

# Session Handoff

Capture in-flight work so a zero-context session can resume it, update auto-memory, and (gated) promote repo learnings to the repo's CLAUDE.md. Run the four steps in order. Full design rationale: `projects-root/docs/superpowers/specs/2026-07-30-session-handoff-skill-design.md`.

## Step 1 — Handoff doc (skip only if nothing is in flight)

"Nothing in flight" = work merged, tree clean, no open branch. Otherwise, for **each** repo with in-flight work:

1. Must be on a working branch. On main with pending work → STOP and ask; never commit to main.
2. Commit any uncommitted WIP first (`wip:` prefix) so nothing lives only in the tree.
3. Write `docs/handoffs/YYYY-MM-DD-<topic>.md` from the template below. Every section is REQUIRED — a section with nothing to say gets an explicit "none".
4. Commit on the working branch and `git push -u origin <branch>`.
5. Check `docs/handoffs/` for docs describing finished work (PR merged, branch gone) — flag them to the user as stale; delete only on a yes.

### Template (all sections REQUIRED)

```markdown
# Handoff: <topic> — YYYY-MM-DD

## Resume here
- Repo: <name>, branch: <branch>, worktree: <path or "create one">
- PR: <URL + state, or "not yet opened">
- Verify state before continuing (command → expected output):
  - git log --oneline -3 → top commit <sha> <msg>
  - <repo gate command> → <expected status, as actually observed this session>
- If a check fails, the branch moved since this doc was written — re-read git log and reconcile before acting.

## Goal
<One or two sentences.>

## State
- Done and verified: <list>
- Half-done / untested: <list>

## Next steps (ordered, concrete)
1. <run X, expect Y, then do Z — never "finish the feature">

## Open questions / decisions pending
<Awaiting the user's input, or "none">

## Gotchas hit this session
<What the next session would rediscover the hard way, or "none">

## Cleanup
Delete this file on this branch once the work above is complete, before the PR merges.
```

Verification commands must record what was **actually observed this session** (real sha, real test counts) — not aspirations.

## Step 2 — Memory update

Follow the existing memory conventions (one fact per file, frontmatter, MEMORY.md index line, absolute dates, `[[links]]`):

- Update the topic file(s) for what this session touched; note in-flight state with a pointer to the handoff doc ("read that, not this summary").
- New durable gotchas/preferences → their own files.
- Light audit: fix anything this session proved stale (completed PENDINGs, changed versions). Nothing beyond that — heavy cleanup belongs to consolidate-memory.

## Step 3 — Repo CLAUDE.md (GATED)

Candidate learnings: durable, non-obvious, repo-level (gates, dev-server quirks, deploy gotchas). Exclude what belongs in memory (cross-repo, personal, operational) or is derivable from code/git history.

- None found → say so, move on (the common case).
- Found → present the proposed edit as a diff-style summary and **wait for an explicit yes before editing**. Recording the same fact in memory or the handoff doc does not substitute for, or authorize, the CLAUDE.md edit. Approved edits ride the working branch.

## Step 4 — Confirmation summary (REQUIRED format)

One final message containing, in order:

1. Handoff doc path(s) + commit sha(s)
2. Memory files created / updated / fixed
3. CLAUDE.md: what changed, or "no changes"
4. A paste-ready **resume prompt**, quoted for copying:

   > Resume from `docs/handoffs/<file>.md` on branch `<branch>` in the `<repo>` repo. Read the handoff doc first, run its verification checks, then continue from its next steps.

Multiple repos → the prompt lists each. Nothing in flight → replace the prompt with "nothing in flight".

## Worktree cleanup (when the session is inside a worktree)

A session can never remove the worktree it is standing in — git refuses and the OS holds the cwd locked. Never run `git worktree remove` on your own working directory.

- Work still in flight → the worktree must stay anyway; no cleanup, no warning.
- Work merged and the worktree is due for removal:
  - This session created it with EnterWorktree → call ExitWorktree with `action: "remove"` (use `"keep"` instead if anything uncommitted must survive).
  - Session was launched inside it (EnterWorktree never called) → do NOT attempt removal. Add a post-session line to the confirmation summary:

    > After closing this session, run from the main checkout: `git worktree remove <path>; git worktree prune` (or `/clean_gone`).

## Edge cases

| Situation | Do |
|---|---|
| Not in a git repo | Say the doc has no home; offer memory-only mode (steps 2–4) |
| Nothing in flight | Skip step 1; steps 2–4 still run |
| On main with pending work | Stop and ask before any commit |
| Multiple repos touched | One handoff doc per repo with in-flight work |
| Session inside a worktree | See Worktree cleanup above — never remove your own cwd |
