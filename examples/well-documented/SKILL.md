# Campaign Link Validator

Validates tracking links across a marketing campaign before launch.
Checks for the presence and correctness of UTM parameters, flags broken
or redirecting URLs, and produces a structured validation report suitable
for stakeholder sign-off.

Use this skill before any paid or owned campaign goes live. Do not use
it for post-launch performance analysis or for building UTM parameters
from scratch — it validates what already exists, it does not generate.

---

## When to use this skill

- Checking a batch of campaign URLs before launch
- Verifying UTM parameters are consistent across a campaign
- Producing a sign-off report for a campaign manager or media buyer
- Catching broken redirects before paid budget is committed

## When NOT to use this skill

- Generating UTM parameters for a new campaign — this skill validates,
  it does not create
- Post-launch UTM audits across historical data — use a reporting skill
- Checking non-campaign URLs (product pages, blog posts) — out of scope

---

## Instructions

### Step 1 — Receive inputs

The user provides one or both of:
- A list of URLs (pasted, uploaded as CSV, or described)
- A campaign brief or summary that references the URLs

If neither is provided, ask for a URL list before proceeding.

### Step 2 — Parse and validate UTM parameters

For each URL:

1. Extract all query parameters from the URL string
2. Check for required fields: `utm_source`, `utm_medium`, `utm_campaign`
   - Missing required field → BLOCKING finding
3. Check for recommended fields: `utm_term` (paid search), `utm_content`
   (display/social)
   - Missing recommended field where channel warrants it → ADVISORY finding
4. Check values for common errors:
   - Unencoded spaces in values → BLOCKING (indicates copy-paste error)
   - Inconsistent casing for the same parameter across URLs (e.g.
     `utm_campaign=Q2` on one URL and `utm_campaign=q2` on another) →
     ADVISORY
   - Unrecognized `utm_medium` value (anything outside: cpc, email,
     social, organic, referral, display, video, affiliate) → ADVISORY

### Step 3 — Check URL reachability

If web access is available, attempt to resolve each URL:
- 200 OK → pass
- 301/302 redirect → note destination, flag as ADVISORY if destination
  domain differs from expected
- 404 or unreachable → BLOCKING

If web access is not available, note all URLs as requiring manual
reachability verification and continue with parameter validation only.

### Step 4 — Produce the validation report

```
Campaign Link Validation Report
────────────────────────────────
Campaign: [name if available, otherwise "unnamed"]
Validated: [date]
URLs checked: [n]

BLOCKING ISSUES ([n])
  [URL or parameter] — [specific issue]

ADVISORY ITEMS ([n])
  [URL or parameter] — [specific issue]

URL RESULTS
  [url] — PASS | [n issues]
  [url] — PASS | [n issues]

RECOMMENDATION
  [One paragraph: is this campaign ready to launch from a link
  validation standpoint? What must be fixed vs. what is discretionary?]
```

---

## Output

A structured validation report as formatted markdown. If the user asks
for a CSV or spreadsheet format, produce a table instead with columns:
URL, Status, Blocking Issues, Advisory Items.

---

## Known limitations

- Reachability checks require web access — unavailable in offline sessions
- Does not validate that UTM values match the campaign brief (e.g. does
  not check that `utm_campaign` matches the campaign name) — it validates
  format and consistency, not semantic correctness
- Does not handle URL shorteners — shortened URLs should be expanded by
  the user before validation
