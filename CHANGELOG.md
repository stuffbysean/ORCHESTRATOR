# Changelog

All notable changes to the Orchestrator's behaviour are documented here.

This file matters more than most changelogs. The Orchestrator writes to
files in your skill library — you should know what changed before updating.
Entries marked **⚠ affects registry** or **⚠ affects skill files** require
attention if you are upgrading an existing installation.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Versioning follows [Semantic Versioning](https://semver.org/).

---

## [Unreleased]

---

## [0.1.0] — 2026-04-06

Initial release.

### Added

**Scan mode**
- Reads all files in a pointed-at directory (non-recursive by default)
- Infers capabilities, inputs, outputs, and exclusions from file content
- Three-point confidence scoring: `high`, `medium`, `low`
- Low-confidence entries surface a best-effort inference to the user
  for confirmation before registry entry is written
- User-provided descriptions stored with `description-source: user`
- Unreadable files recorded as `status: unreadable` rather than skipped
- Writes `registry.md` on completion

**Route mode**
- Reads `registry.md` as working index (does not open skill files unless
  confidence is low or medium)
- Overlap scoring framework: high (≥70%), partial (30–69%), low (<30%)
- Composite case handling: two partial-overlap skills → extend the
  higher-scoring one, cross-reference the other
- Structured proposal output: intent restatement, recommendation,
  reasoning, overlap scores, next step options
- Waits for explicit user confirmation before entering Build mode

**Build mode — net-new path**
- Fast registry check before drafting to confirm net-new is correct
- Four clarifying questions asked in one message (trigger, boundaries,
  output shape, domain knowledge)
- Full skill file drafted after answers received
- In-conversation iteration before any file is written
- Three diffs shown on approval: new skill file, registry entry,
  cross-references in related skills
- Write on explicit user approval only

**Build mode — extend path**
- Reads full skill file (not just registry entry) before drafting
- Minimum diff approach: additive only, no restructuring of existing content
- Unified diff format shown before writing
- Registry entry updated to reflect new capability on write

**Registry format**
- Fields: path, confidence, description-source, write-protected, status,
  capabilities, inputs, outputs, exclusions, related, notes
- Orchestrator-owned: humans may read, should not edit manually
- Stale threshold: 30 days from last-scan date triggers rescan offer

**Write-back behaviour**
- All file modifications show a diff and require explicit approval
- Write-protected detection for externally managed skill files
- Cross-references stored in registry for write-protected skills
- Cross-reference injection appended at end of file, never mid-document

**Session start checklist**
- Path validation before any mode is entered
- Registry existence check with stale detection
- Mode routing based on user's opening message
