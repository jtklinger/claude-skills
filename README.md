# claude-skills

Claude Code skills I've written and use day to day. Each skill is a folder under
`skills/` containing a `SKILL.md` (frontmatter `name` + `description`, then the
instructions Claude follows when the skill is invoked).

## Skills

| Skill | What it does |
|---|---|
| [session-handoff](skills/session-handoff/SKILL.md) | Before clearing or ending a session, writes a handoff doc on the working branch, updates auto-memory, and (gated) proposes repo `CLAUDE.md` learnings, then prints a paste-ready resume prompt. |
| [remediate](skills/remediate/SKILL.md) | Fixes one drift finding filed as a GitHub issue by an automated nightly check: read-only recon, a plan that waits for go-ahead, branch → PR → deploy → verify, then closes the issue. Generic version; host and repo names from my setup removed. |
| [exporting-granola-transcripts](skills/exporting-granola-transcripts/SKILL.md) | Exports Granola.ai meeting summaries + full transcripts to markdown, verifying every transcript is complete and failing loudly instead of claiming partial success. |

## Install

Copy a skill folder into `~/.claude/skills/` (user-wide) or `<repo>/.claude/skills/`
(project-only). Claude Code picks it up on the next session; invoke with `/<name>` or
let the description trigger it.

```bash
cp -r skills/session-handoff ~/.claude/skills/
```

## License

MIT. See [LICENSE](LICENSE).
