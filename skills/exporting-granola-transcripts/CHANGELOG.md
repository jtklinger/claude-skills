# Changelog

All notable changes to the Exporting Granola Transcripts skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-02-11

### Added
- Initial release of exporting-granola-transcripts skill
- Verification checklist for transcript completeness
- Binary success criteria (ALL complete or FAIL)
- Red Flags section to prevent "partial success" claims
- Error handling with retry logic (up to 2x with exponential backoff)
- Rate limit handling (60s delays between requests)
- Common mistakes table documenting success bias
- Single-file markdown format enforcement
- Proper failure report format with root cause analysis

### Testing
- 5 baseline tests documented default agent behavior
- 3 GREEN/REFACTOR tests verified skill effectiveness
- Success scenario validated with complete data
- Failure scenario validated with incomplete data

### Known Issues
- Granola API rate limits may block large exports (by design)
- Rate limit timing may vary by account tier (not yet tested across tiers)

## [1.1.0] - 2026-02-12

### Added
- Interactive prompting for export location, date range, and meeting selection
- Fixed requirements section preventing format/structure questions
- Default values for common scenarios (last 30 days, ./granola_exports/)
- Example interaction showing user experience
- Guidance on when to skip prompts (if user provides all info upfront)
- Prominent warnings about non-negotiable format standards

### Changed
- Reorganized Interactive Prompting section for clarity
- Made "Do NOT ask about" requirements more visible

### Testing
- RED-GREEN-REFACTOR cycle validated prompting behavior
- Verified agents only ask 3 required questions
- Confirmed format requirements are enforced

## [Unreleased]

### Planned
- Adaptive rate limit detection
- Batch export with intelligent pacing
- Export progress tracking
- Account tier detection
