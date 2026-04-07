# Examples

This directory contains a minimal fake skill library for testing and
demonstrating the Orchestrator. It has three skills, each designed to
exercise a different part of the inference and routing logic.

---

## How to use these examples

Point the Orchestrator at this directory in a Claude Code session:

```
Read ORCHESTRATOR.md. My skill library is at examples/.
```

Then try the scenarios below to see each mode in action.

---

## The three skills

### `well-documented/` — high-confidence inference

A fully written skill file with clear sections, explicit capabilities,
and documented exclusions. The Orchestrator should infer this one
correctly without asking for confirmation.

**Use this to test:**
- Scan mode producing a `confidence: high` registry entry
- Route mode matching an intent against well-defined capabilities
- Extend path producing a clean targeted diff

**Try routing against it:**
```
I want to validate that all URLs in a campaign have correct tracking
parameters before the campaign goes live.
```

---

### `terse/` — low-confidence inference

A minimal skill file: a name, a one-line description, and a few
bullet points. The kind of thing someone writes at 11pm and commits
without filling it out. The Orchestrator should flag this as
`confidence: low` and ask you to confirm its inference before
writing the registry entry.

**Use this to test:**
- Scan mode surfacing a best-effort inference for user confirmation
- The confirmation dialogue and `description-source: user` field
- How a low-confidence entry affects Route mode scoring

**What the Orchestrator should say during scan:**
```
I read `terse/SKILL.md` and my best guess is that it [inference].
Is that accurate, or can you give me a one-line description of
what it does?
```

---

### `external/` — read-only write-back

A skill file marked as externally managed via a comment in its header.
Simulates a skill pulled from another repo or installed as a submodule
that you don't want the Orchestrator to touch.

**Use this to test:**
- Write-protected detection during scan
- `write-protected: true` in the registry entry
- Cross-references stored in the registry rather than injected into
  the file when a related skill is built

**Note:** The Orchestrator detects write-protection from the file
header comment. In a real installation, a file would be write-protected
because it lives in a read-only path (e.g. a submodule) or because the
user marked it as such. This example simulates that with a comment.

---

## Running a full end-to-end test

To exercise all three modes in sequence:

**1. Scan**
```
Read ORCHESTRATOR.md. My skill library is at examples/. Please scan
and build the registry.
```
Confirm the terse skill inference when prompted.

**2. Route**
```
I want to build something that checks whether campaign tracking links
are set up correctly before launch.
```
The Orchestrator should recommend extending `well-documented` (high
overlap) and note `external` as a related skill.

**3. Build — extend path**
```
Yes, extend it.
```
Review the diff. Approve or suggest changes.

**4. Build — net-new path**
Start a fresh session and try:
```
I want to create a skill for automatically generating a post-campaign
performance summary from raw data exports.
```
Answer the clarifying questions. Review the draft. Approve.

---

## After testing

Delete `examples/registry.md` to reset to a clean state for the next
test run. The registry is gitignored in this repo so test runs don't
pollute the commit history.
