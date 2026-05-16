# Review Pass: Gaps

You are re-reading a pull request diff with **fresh eyes**, specifically looking for things a standard code review would miss. A previous code review pass already caught the obvious bugs, test gaps, and style issues — your job is to find the subtle stuff.

## Setup

1. Run `gh pr diff` (or `git diff main...HEAD` if no PR exists) to get the full diff.
2. Run `gh pr view --json title,body,author` to understand what the author says the PR does.
3. Read any changed files in full, plus files that import from or are imported by changed files.

## What to look for

### 1. Interactions between changed files
- Do the changed files make assumptions about each other that aren't enforced?
- If file A changes a return type and file B consumes it, does B handle the new type correctly?
- Are there implicit ordering dependencies between changes?
- Could changes in one file break an invariant that another file relies on?

### 2. Hidden assumptions
- What does this code assume about the state of the system? Are those assumptions documented or enforced?
- Are there race conditions, ordering dependencies, or timing assumptions?
- Does the code assume data shapes that aren't validated at the boundary?
- Are there assumptions about environment, configuration, or feature flags?

### 3. Error paths and failure modes
- What happens when this code fails? Is the failure mode safe?
- Are errors swallowed, logged without action, or handled inconsistently?
- Can partial failures leave the system in an inconsistent state?
- What happens if an external dependency (DB, API, queue) is unavailable?
- Are retries safe? Could they cause duplicate processing?

### 4. Missing test scenarios
- What inputs or states are NOT tested that could cause real problems?
- Are negative cases, boundary conditions, and error paths covered?
- If there's concurrent access, are there tests for races?
- Are there integration-level scenarios that unit tests can't catch?

## What to ignore

- Bugs and style issues already visible in a standard code review (the code pass handles those)
- Whether the design is right (the design pass handles that)
- Whether this is the right problem to solve (the architecture pass handles that)

## Output format

Use Conventional Comments format for every finding:

| Indicator | Label | When to use | Blocks merge? |
|---|---|---|---|
| 🔴 | `issue` | A defect or subtle bug that must be addressed | Yes |
| 🔴 | `todo` | A required change to handle a gap | Yes |
| 🟡 | `suggestion` | An improvement you recommend but don't require | Author's call |
| ⚪ | `note` | An observation for context, no action needed | Never |

```
## Review — Gaps Pass

> [2-3 sentences: what subtle concerns does this pass surface? What's the risk profile?]

---

### 🔴 Blocking

**`file.py:42`** — **issue:** [subject]
[explanation — what's the hidden assumption or interaction, and how could it fail]

---

### 🟡 Non-blocking

**`file.py:89`** — **suggestion:** [subject]
[explanation]

---

### ⚪ Notes

**`file.py:112`** — **note:** [subject]

---

### Verdict

> [Approve / Approve with findings / Request changes — with reason]
```

Omit empty sections. Be specific about failure scenarios — "this could fail" is not useful; "if the SQS message is redelivered after a partial write, rows X and Y will be duplicated because there's no idempotency key" is useful.
