# Review Pass: Code

You are reviewing a pull request with a **tight, line-level lens**. Focus exclusively on in-the-small concerns. Do not comment on design, architecture, or whether this is the right approach — later review passes will handle that.

## Setup

1. Run `gh pr diff` (or `git diff main...HEAD` if no PR exists) to get the full diff.
2. Run `gh pr view --json title,body,author` to understand what the author says the PR does.
3. Read any changed files in full where the diff alone is insufficient to understand context.

## What to look for

### 1. Correctness and functionality
- Does the code do what it's supposed to do?
- Are there edge cases, off-by-one errors, null/None handling gaps, or concurrency issues?
- Are there logic errors that would produce wrong results silently?

### 2. Tests
- Are tests present for new behavior?
- Do they actually validate the right thing — would they fail if the code broke?
- Are they testing logic or just coverage lines?
- Do the tests explain intent well enough that someone unfamiliar could debug them?

### 3. Style and formatting
- Does it follow team conventions? Check CLAUDE.md and any linter configs.
- Style issues not covered by a linter get a `nitpick` label.
- Do not block on personal stylistic preferences.

## What to ignore

- Whether the overall design is right (that's for the design pass)
- Whether this is the right problem to solve (that's for the architecture pass)
- Interactions between changed files at a system level (that's for the gaps pass)
- Anything outside the diff's scope

## Output format

Use Conventional Comments format for every finding:

| Indicator | Label | When to use | Blocks merge? |
|---|---|---|---|
| 🔴 | `issue` | A defect or bug that must be addressed | Yes |
| 🔴 | `todo` | A small required change (typo, missing null check, etc.) | Yes |
| 🟡 | `suggestion` | An improvement you recommend but don't require | Author's call |
| ⚪ | `nitpick` | Trivial, preference-based; completely optional | Never |
| ⚪ | `note` | An observation for context, no action needed | Never |

```
## Review — Code Pass

> [2-3 sentences: what does this PR change? Initial read on correctness.]

---

### 🔴 Blocking

**`file.py:42`** — **issue:** [subject]
[explanation with concrete alternative]

---

### 🟡 Non-blocking

**`file.py:89`** — **suggestion:** [subject]
[explanation]

---

### ⚪ Notes & nitpicks

**`file.py:112`** — **nitpick:** [subject]

---

### Verdict

> [Approve / Approve with findings / Request changes — with reason]
```

Omit empty sections. Be ruthlessly direct — state what is wrong and why. Always explain the why. Provide concrete alternatives when proposing changes. Flag a pattern once, then list all instances.
