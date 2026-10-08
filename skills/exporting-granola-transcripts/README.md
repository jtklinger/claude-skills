# Exporting Granola Transcripts - Claude Code Skill

A skill for Claude Code that ensures complete, verified exports of [Granola.ai](https://granola.ai) meeting transcripts with proper error handling and failure reporting.

## The Problem

Granola's API can return truncated transcripts without clear errors. By default, AI agents:
- Accept incomplete data and claim success
- Don't verify transcript completeness
- Bury warnings instead of failing loudly
- Report "partial success" as acceptable

This leads to incomplete exports that mislead users and cause real harm when used for analysis or review.

## The Solution

This skill enforces strict verification and binary success criteria:
- **Success = ALL transcripts complete**, not "most" or "some"
- Validates each transcript for completeness (character count, natural endings, no cutoffs)
- Retries on failure with exponential backoff
- **Fails loudly** when data is incomplete after retries
- Prevents "partial success" claims with explicit Red Flags

## Installation

### For Claude Code Users

1. Copy the skill to your skills directory:
```bash
cp -r exporting-granola-transcripts ~/.claude/skills/
```

2. The skill will be automatically available in Claude Code sessions

### For Other Claude Environments

Place `SKILL.md` in your agent's skills directory according to your platform's documentation.

## Usage

The skill is automatically invoked when you ask Claude to export Granola transcripts:

```
"Export my Granola meetings from August to December with summaries and full transcripts"
```

Claude will:
1. Fetch meetings in the specified date range
2. Export transcripts one-at-a-time (sequential processing)
3. Verify each transcript for completeness
4. Retry failed transcripts up to 2x with delays
5. Report clear SUCCESS or FAILURE

### Success Example

```
✅ EXPORT SUCCESS

All 5 transcripts exported completely:
- 2025-12-12_Technical_Interview.md ✓
- 2025-11-24_Legal_Deposition.md ✓
- ...

Files saved to: /path/to/export/
```

### Failure Example

```
❌ EXPORT FAILED

Reason: 2 of 5 transcripts incomplete after retries
- Meeting "Legal Deposition": Cuts off mid-sentence
- Meeting "Interview X": Ends abruptly at 8,500 chars

Action needed: Contact Granola support with meeting IDs
```

## Features

### Verification Checklist
- ✅ Character count validation (>10k for interviews/depositions)
- ✅ Natural conversation ending detection
- ✅ Mid-sentence cutoff detection
- ✅ Abrupt stop detection

### Error Handling
- 🔄 Automatic retries (up to 2x per transcript)
- ⏱️ Rate limit handling (60s delays between requests)
- 📊 Clear failure reporting with root cause analysis
- 🚫 Prevents "partial success" claims

### File Format
Each meeting exported to single markdown file:
```markdown
# [Meeting Title]

**Date:** [timestamp]
**Meeting ID:** [UUID]
**Participants:** [list]

---

## Summary
[AI-generated summary]

---

## Full Verbatim Transcript
[Complete transcript]
```

## Development

### Testing Methodology

This skill was developed using Test-Driven Development (TDD) for documentation:

**RED Phase:** 5 baseline tests documented agent behavior without skill
- Discovered: Success bias, incomplete data acceptance, no verification
- Key finding: Agents claim "Export Complete" with 40%+ failure rates

**GREEN Phase:** Skill written to address baseline failures
- Enforces binary success criteria
- Provides verification checklist
- Shows proper failure formats

**REFACTOR Phase:** 2 iterations to close loopholes
- Added Red Flags section to prevent "partial success" claims
- Enforced single-file format requirement
- Re-tested until bulletproof

### Running Tests

To verify the skill works in your environment:

```bash
# Test failure scenario (rate limits will likely block)
"Export all my Granola meetings from the past year"

# Test success scenario (single meeting)
"Export just my most recent Granola meeting"
```

Expected behavior:
- ❌ Multi-meeting: Likely FAILS due to rate limits (proper behavior)
- ✅ Single meeting: Should SUCCEED with complete data

## Requirements

- Claude Code with Granola MCP server configured
- Granola.ai account with recorded meetings
- MCP tools: `mcp__granola__list_meetings`, `mcp__granola__get_meetings`, `mcp__granola__get_meeting_transcript`

## Known Limitations

- **Granola API rate limits**: The API enforces strict rate limits (approximately 60 seconds between transcript requests). Large exports may fail due to rate limiting.
- **Account tier differences**: Rate limits may vary between free/personal and business accounts
- **No parallel processing**: Exports are intentionally sequential to respect rate limits

## Contributing

Contributions welcome! This skill is designed to be:
- **Concise**: ~500 words (optimized for AI context)
- **Tested**: All changes should follow TDD methodology (RED-GREEN-REFACTOR)
- **Specific**: Addresses real baseline failures, not hypothetical issues

### Improvement Ideas
- [ ] Adaptive rate limit detection
- [ ] Batch export with intelligent pacing
- [ ] Export progress tracking for large batches
- [ ] Account tier detection

## License

MIT License - See [LICENSE](LICENSE) file

## Credits

Developed using the [writing-skills](https://github.com/anthropics/claude-code/tree/main/skills/writing-skills) TDD methodology for Claude Code skills.

## Support

- **Issues**: Report problems via GitHub Issues
- **Granola API**: For API-specific issues, contact [Granola support](https://granola.ai)
- **Claude Code**: For platform issues, see [Claude Code documentation](https://github.com/anthropics/claude-code)
