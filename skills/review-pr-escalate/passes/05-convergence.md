# Review Pass: Convergence

You are performing the **final holistic review** of a pull request. Four previous passes have examined this change through progressively broader lenses: code correctness, gaps and subtle issues, design fit, and architectural soundness. Fixes have been applied between passes.

Your job is to read the change one last time with everything in mind — code, design, and architecture together — and determine whether it has converged.

## Setup

1. Run `gh pr diff` (or `git diff main...HEAD` if no PR exists) to get the full diff as it stands now (after all previous fixes).
2. Run `gh pr view --json title,body,author` to understand the PR's stated intent.
3. Read any files that are central to the change in full.

## What to look for

### 1. Holistic coherence
- Considering all the fixes applied during previous passes, does the change still hold together as a whole?
- Did the individual fixes introduce any inconsistencies with each other?
- Is the final result clean and intentional, or does it feel like it was patched together?

### 2. Remaining concerns
- Is there anything that still bothers you about this change, even if you can't precisely articulate why? Surface it as a `thought`.
- Are there any lingering doubts about correctness, design, or approach?
- Does the change read like it was written by someone who understood the problem, or does it feel mechanical?

### 3. Regression check
- Could any of the fixes from previous passes have introduced new issues?
- Are the tests still coherent after modifications?
- Do the commit messages from fix rounds accurately describe what was changed?

### 4. Convergence determination
- **If you have no new findings** beyond what previous passes already surfaced: declare **Converged**.
- **If you have new findings**: report them normally. The change has not yet converged.

"No new findings" means no new `issue`, `todo`, or `rethink` items. Residual `suggestion`, `nitpick`, `note`, and `thought` items do NOT prevent convergence — note them but still declare convergence.

## Output format

| Indicator | Label | When to use | Blocks merge? |
|---|---|---|---|
| 🔴 | `issue` | A new defect found in this final pass | Yes |
| 🟠 | `rethink` | A new fundamental concern | Pauses for input |
| 🟡 | `suggestion` | A non-blocking improvement | Never blocks convergence |
| ⚪ | `thought` | A residual observation | Never blocks convergence |

### If converged:

```
## Review — Convergence Pass

> This change has been reviewed across five escalating passes (code, gaps, design, architecture, convergence). No new blocking or rethink findings were identified in this final holistic review.

### ⚪ Residual observations (optional)

**`file.py:42`** — **thought:** [anything worth noting but not blocking]

---

### Verdict

> 🏁 **Converged** — This change is about as good as we can make it. [1 sentence on overall quality.]
```

### If NOT converged:

```
## Review — Convergence Pass

> [What new concerns surfaced in this final pass?]

---

### 🔴 Blocking (or 🟠 Needs rethink)

[findings in standard format]

---

### Verdict

> 🔴 **Not converged** — [what still needs resolution]
```

Omit empty sections. If the change has converged, say so with confidence. If it hasn't, be specific about what remains.
