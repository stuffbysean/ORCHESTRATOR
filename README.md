# skill-orchestrator

A skill-agnostic orchestrator for Claude Code skill libraries. Drop it into
any skill library and Claude gains the ability to scan what exists, route
intent to the right skill, and collaboratively build new ones — without
enforcing any schema on the skills it manages.

---

## What it does

The Orchestrator gives Claude three behaviours when working inside a skill
library:

| Mode | Trigger | What happens |
|---|---|---|
| **Scan** | First run, or explicit request | Reads every file in the library, infers capabilities, builds `registry.md` |
| **Route** | "I want to do X" | Scores intent against the registry, proposes extend vs. net-new with reasoning |
| **Build** | After routing, or "I want to create a skill for X" | Collaboratively drafts the skill file, shows diffs, writes on approval |

The Orchestrator works with skills in any format — it reads what it finds
and infers what it can. It never enforces a schema on skills it didn't create.
It never modifies any file without showing you a diff first.

---

## Requirements

- Claude Code (any recent version)
- A skill library directory on your local filesystem

---

## Installation

### Option A — Copy into your skill library

```bash
cp ORCHESTRATOR.md /path/to/your/skill-library/
```

Then tell Claude to read it at the start of each session (see Usage below).

### Option B — Reference from a central location

If you maintain multiple skill libraries, keep one copy of `ORCHESTRATOR.md`
and reference it by path in each session.

```bash
# Example: keep it at ~/skills/orchestrator/ORCHESTRATOR.md
# Reference it at the start of any skill library session
```

### Option C — Import as a git submodule

```bash
cd /path/to/your/skill-library
git submodule add https://github.com/[your-handle]/skill-orchestrator orchestrator
```

Then reference `orchestrator/ORCHESTRATOR.md` in your session start.

---

## Usage

### Starting a session

At the start of any Claude Code session where you want the Orchestrator
active, say:

```
Read orchestrator/ORCHESTRATOR.md. My skill library is at [path].
```

Claude will check for an existing registry and either load it or offer
to scan.

### First run — building the registry

On first run, no `registry.md` exists. Claude will enter Scan mode
automatically:

```
Read orchestrator/ORCHESTRATOR.md. My skill library is at ~/my-skills.
```

Claude scans every file in `~/my-skills`, infers capabilities, and
asks you to confirm any entries it's uncertain about:

```
I read `lead-enrichment/SKILL.md` and my best guess is that it takes
an inbound lead record and enriches it with company data from an
external source, then scores it against an ICP. Is that accurate, or
can you give me a one-line description of what it does?
```

Once all entries are resolved, it writes `registry.md` and confirms
the library is ready.

### Routing an intent

State what you want to accomplish in plain language:

```
I want to automatically flag campaigns that are missing UTM parameters
before they go live.
```

Claude scores your intent against the registry and responds with a
structured proposal:

```
Intent: Flag campaigns missing UTM parameters before launch

Recommendation: EXTEND campaign-qa

Reasoning: campaign-qa already covers pre-launch validation including
URL checking. UTM validation is an additive capability that fits
naturally within its existing scope.

Overlap scores:
  campaign-qa: 74% — 3 of 4 capabilities match; UTM validation not
               currently covered but not excluded
  attribution:  12% — handles post-launch attribution, not pre-launch QA

Capability to add: UTM parameter validation across a batch of campaign URLs
Estimated change: additive

Proceed? (extend / net-new / neither)
```

### Building a net-new skill

You can trigger Build mode directly without routing first:

```
I want to create a skill for scoring content briefs against our ICP.
```

Claude runs a fast registry check, confirms net-new is the right path,
then asks four targeted clarifying questions before drafting anything:

```
Before I draft, a few things I need from you:

1. Trigger: What does the user provide to invoke this skill? A brief
   document, a URL, a pasted paragraph?

2. Boundaries: What are 2-3 things this skill should explicitly NOT do?

3. Output shape: What does done look like — a score, a structured
   assessment, a set of suggested edits?

4. Domain knowledge: What ICP criteria should it score against? Are
   there thresholds or weightings I should know about?
```

After you answer, Claude drafts the complete skill file, iterates with
you until it's right, then shows diffs for the new skill file, the
registry entry, and any cross-references before writing anything.

### Extending an existing skill

After a Route recommendation to extend:

```
Yes, extend campaign-qa.
```

Claude reads the full skill file, drafts the minimum diff needed, and
shows it to you:

```diff
--- a/campaign-qa/SKILL.md
+++ b/campaign-qa/SKILL.md
@@ -38,6 +38,22 @@ Check that the base URL resolves or flag for manual verification.

+### Module 2 — UTM validation
+
+For each URL in the input list, check for the presence of required
+UTM fields: utm_source, utm_medium, utm_campaign. Flag utm_term and
+utm_content as advisory if absent.
+
+Check values for common errors:
+- Spaces in values (indicates a copy-paste error)
+- Inconsistent casing across URLs for the same campaign
+- Unrecognized utm_medium values
+
```

Approve, and Claude writes the updated file, updates the registry, and
injects cross-references into related skills.

---

## The registry

`registry.md` is created and maintained entirely by the Orchestrator.
You can read it — it's plain markdown — but you should not edit it
manually. The Orchestrator will overwrite manual changes on the next run.

Each entry looks like this:

```markdown
## campaign-qa

- **path**: campaign-qa/SKILL.md
- **confidence**: high
- **description-source**: inferred
- **write-protected**: false
- **status**: active
- **capabilities**:
  - Pre-launch campaign checklist generation
  - Audience segment sanity checking
  - Creative spec conformance review
  - Launch-readiness scoring
- **inputs**: Campaign brief, URL list, creative spec
- **outputs**: Structured QA report with pass/fail per check and readiness score
- **exclusions**: Post-launch analysis, campaign building
- **related**: attribution, content-brief
- **notes**: —
```

See `registry.schema.md` for a description of every field.

---

## Working with external or read-only skills

If a skill file is externally managed (e.g. pulled from another repo,
installed as a submodule, or otherwise not yours to edit), the
Orchestrator detects this and marks it `write-protected: true` in the
registry. Cross-references that would normally be injected into the file
are stored in the registry entry instead, so the relationship is
preserved without touching the source.

---

## Repository structure

```
skill-orchestrator/
├── ORCHESTRATOR.md        ← Claude reads this at session start
├── README.md              ← this file
├── CHANGELOG.md           ← version history
├── registry.schema.md     ← field reference for registry.md
└── examples/
    ├── README.md          ← how to use the examples
    ├── well-documented/
    │   └── SKILL.md       ← high-confidence inference example
    ├── terse/
    │   └── SKILL.md       ← low-confidence, triggers user confirmation
    └── external/
        └── SKILL.md       ← read-only, cross-refs stored in registry
```

---

## Versioning

This repo follows semantic versioning. Breaking changes to the
Orchestrator's behaviour (mode triggers, registry format, diff format)
increment the minor version. Additions and clarifications increment
the patch version.

Check `CHANGELOG.md` before updating — the Orchestrator modifies files
in your skill library, so you want to know what changed.

---

## Contributing

See `CONTRIBUTING.md`.

---

## License

MIT
