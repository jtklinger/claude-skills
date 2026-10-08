---
name: exporting-granola-transcripts
description: Use when exporting Granola meeting transcripts to files, especially when user requests summaries and full transcripts together or mentions incomplete/truncated data issues
---

# Exporting Granola Transcripts

## Overview

Granola API can return truncated transcripts without clear errors. Default agent behavior is to accept incomplete data and claim success. This skill ensures complete exports or proper failure reporting.

## When to Use

- User asks to export Granola meetings to files
- User wants both AI summaries and full verbatim transcripts
- User mentions data completeness or verification

## Interactive Prompting

**Before starting export, gather requirements if not specified:**

### ⚠️ Fixed Requirements (Do NOT Ask About These)

These are **non-negotiable** per skill standards:
- ❌ File format → Always `.md` markdown
- ❌ Content → Always summary + full transcript (both required)
- ❌ File structure → Always one file per meeting
- ❌ What to include → Always metadata + summary + transcript

**If user asks for different format** (JSON, TXT, separate files):
→ Explain: "This skill uses a standard format (markdown with summary + transcript) to ensure quality and completeness. Is that okay?"

### Required Information (Ask Only These 3)

**1. Date Range** - Ask: "Which meetings should I export?"
- Present options: "last 7 days", "last 30 days", "this month", or "custom date range"
- If user says "all": confirm this might hit rate limits
- Default: "last 30 days" if user unsure

**2. Export Location** - Ask: "Where should I save the exports?"
- Suggest: `./granola_exports/` or `~/Documents/granola_exports/`
- Create directory if it doesn't exist (ask permission first)
- Default: `./granola_exports/` in current directory

**3. Meeting Selection** - After listing meetings, ask:
- "Found X meetings. Export all or select specific ones?"
- If "specific": List meetings with numbers, let user choose
- If "all": Confirm count and warn about rate limits if >5 meetings

### Example Interaction

```
User: "Export my Granola meetings"

You: "I'll help you export your Granola meetings. Let me gather a few details:

1. Which time period?
   - Last 7 days
   - Last 30 days (recommended)
   - Custom date range

2. Where should I save the files?
   - Default: ./granola_exports/
   - Or specify a path

[After they answer, fetch meetings list]

Found 8 meetings from Aug-Dec 2025:
1. Legal Deposition (Nov 24)
2. Technical Interview (Dec 12)
...

3. Export all 8 or select specific ones? (Note: >5 may hit rate limits)

Format: Each meeting exported as markdown (.md) with summary + full transcript"
```

**If user provides all info upfront** (e.g., "Export August meetings to ~/exports"):
- Don't re-ask, just confirm: "Exporting August 2025 meetings to ~/exports..."
- List meetings found and ask if they want all or specific ones

## Core Pattern

**Success = ALL transcripts complete. Not "most". Not "some". ALL.**

If even ONE transcript is incomplete → the export FAILED.

```javascript
// For each meeting:
1. Fetch summary (get_meetings)
2. Fetch full transcript (get_meeting_transcript)
3. VERIFY completeness (see checklist below)
4. If incomplete → RETRY (up to 2x with 60s delay)
5. If still incomplete → FAIL entire export

// After all meetings:
if (any_incomplete) {
  report_failure(); // See format below
} else {
  report_success();
}
```

## Verification Checklist

For EACH transcript, verify:

- [ ] Character count > 10,000 (interviews/depositions are typically 20k-50k+)
- [ ] Last 200 characters: no mid-sentence cutoff
- [ ] Natural conversation ending present (goodbye/thanks/bye/ending phrases)
- [ ] No abrupt stops like "whose title. Is principal architect." (truncation indicator)

**If any check fails → RETRY that transcript.**

## Error Handling

### When Transcript Incomplete After Retries

**DO:**
```
❌ EXPORT FAILED

Reason: 2 of 5 transcripts incomplete after retries
- Meeting "Legal Deposition" (Nov 24): Cuts off mid-sentence
- Meeting "Interview X" (Aug 18): Ends abruptly at 8,500 chars

Root cause: API returning truncated data
Action needed: Contact Granola support with meeting IDs above
```

**DON'T:**
```
✓ Export complete! (Note: 2 meetings incomplete)  ❌ WRONG - This is lying
```

### Rate Limits

If you hit "Rate limit exceeded":
- Wait 60 seconds between transcript fetches
- Retry failed requests up to 2x with exponential backoff (60s, 120s)
- If rate limited after retries → FAIL with clear message

## File Format

**REQUIRED: One file per meeting containing BOTH summary and transcript.**

```markdown
# [Meeting Title]

**Date:** [Date and time]
**Meeting ID:** [UUID]
**Participants:** [List]

---

## Summary

[AI-generated summary from get_meetings]

---

## Full Verbatim Transcript

[Complete transcript from get_meeting_transcript]
```

**File naming:** `YYYY-MM-DD_Meeting_Title.md`

**Do NOT:**
- Create separate summary and transcript files
- Split the data across multiple files
- Use different formats (all markdown .md)

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|----------------|-----|
| "Export complete ✓" with incomplete data | Misleads user - they'll use bad data | Fail loudly instead |
| Burying incompleteness in notes | User misses critical issue | Lead with failure headline |
| Accepting first API response | Granola returns truncated data | Always verify + retry |
| Continuing after failure detected | Wastes time on unusable export | Stop and report immediately |
| Claiming 80% success | Partial data causes real harm | Binary: all complete or failed |

## Success Bias Warning

**Default agent behavior:** Report success even with incomplete data.

**Why this happens:** Strong tendency to claim task completion.

**Counter this by:**
1. Remembering: Incomplete data misleads the user
2. Using binary criteria: ALL complete = success, else = failure
3. Reporting failures prominently with ❌ not buried notes
4. Stopping immediately when incompleteness detected

**You are helping the user, not performing for them. Honest failure reports prevent real harm.**

## Red Flags - Prohibited Phrases

These phrases violate the "ALL complete or FAIL" rule:

- ❌ "Export complete (partial...)"
- ❌ "Export complete with some issues"
- ❌ "Successfully exported 3 of 5"
- ❌ "Partial success"
- ❌ "Complete except for..."

**All of these mean: EXPORT FAILED. Report it as failure.**

If you catch yourself about to use these phrases, STOP. Use the failure format instead.
