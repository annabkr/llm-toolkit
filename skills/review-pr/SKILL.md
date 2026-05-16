---
name: review-pr
description: >-
  Review a pull request and produce structured, clearly labeled findings.
  Use when asked to review a PR, critique a change, or give feedback on code.
compatibility: Works on all platforms. Requires git CLI and gh CLI on PATH.
metadata:
  author: annabkr
  version: "0.1.0"
---

# PR Review

Reviews a pull request with ruthless critical rigor and produces structured, labeled findings.

---

## Prerequisites

- Git CLI and `gh` CLI installed and authenticated
- Current working directory in a git repository

---

## Step 1: Load the change

If a PR number or branch was provided, fetch the diff with `gh pr diff <number>` or `git diff main...<branch>`. If no argument was given, run `gh pr view --json baseRefName,number` to detect an open PR for the current branch, then fetch its diff. Also read `gh pr view --json title,body,author` to understand what the author says the PR does.

---

## Step 2: High-level pass — design and intent

Before producing any output, answer these questions:

1. **Does this PR make sense?** Does the stated intent in the description match what the diff actually does?
2. **Should it exist?** Is the problem being solved the right problem? Is this change even necessary?
3. **Does the design fit the system?** Does the code integrate soundly with existing architecture, naming conventions, and ownership boundaries?
4. **Is scope appropriate?** PRs over ~400 lines become hard to review effectively and increase risk. Over 1,000 lines is almost always too large. If the PR is too large, flag it before continuing — detailed line-level feedback may be invalidated if a split is needed.

If there is a fundamental design problem, communicate it clearly before continuing.

---

## Step 3: Code review pass — priority order

Work through the diff in this priority order. Higher priorities warrant blocking feedback; lower priorities are usually nitpicks:

### 1. Correctness and functionality
- Does the code do what it's supposed to do?
- Are there edge cases, off-by-one errors, null/None handling gaps, or concurrency issues?

### 2. Tests
- Are tests present for new behavior?
- Do they actually validate the right thing — would they fail if the code broke?
- Are they testing logic or just coverage lines? Tests must fail when the code they test has a bug.
- Do the tests and any inline documentation explain the intent well enough that someone unfamiliar with this change could understand, debug, and extend it?

### 3. Design and structure
- Is this more complex than necessary? Guard against over-engineering — future problems should be solved when they arrive.
- Does the abstraction introduced fit the existing codebase — or does it add a new pattern that conflicts?
- Is there duplication that should be extracted?
- Is the PR doing one coherent thing, or multiple things that should be separate PRs?

### 4. Clarity and maintainability
- Could someone unfamiliar with this code maintain it in 6 months?
- Are variable, function, and class names clear without being verbose?
- Do comments explain *why*, not just *what*? Are they still accurate? Flag comments that restate what the code already clearly says — they add noise without value.
- Is any logic unclear enough that it requires multiple passes to understand? That's a signal it needs clarification in the code itself.
- Are helper functions pulling their weight? Flag thin wrappers that have a single call site and add an abstraction layer without adding clarity.

### 5. Documentation and contracts
- If public-facing behavior changed (API contracts, config, CLI), is documentation updated?
- Are deprecations, migrations, or breaking changes noted?

### 6. Style and formatting
- Does it follow team conventions? Style issues not covered by a linter get a `nitpick` label.
- Do not block on personal stylistic preferences. If a style point comes up repeatedly, that's a signal to add a linter rule.

---

## Step 4: Compile findings

### Finding format — Conventional Comments

Label every finding using the [Conventional Comments](https://conventionalcomments.org/) format: `**label [decorations]:** subject`. This makes blocking vs. non-blocking explicit.

| Indicator | Label | When to use | Blocks merge? |
|---|---|---|---|
| 🔴 | `issue` | A defect, bug, or design problem that must be addressed | Yes |
| 🔴 | `todo` | A small required change (typo, missing null check, etc.) | Yes |
| 🟡 | `suggestion` | An improvement you recommend but don't require | Author's call |
| ⚪ | `nitpick` | Trivial, preference-based; completely optional | Never |
| ⚪ | `thought` | Something that came up while reading — not a request | Never |
| ⚪ | `note` | An observation for context, no action needed | Never |

Add `(blocking)` or `(non-blocking)` decorations when the default might be ambiguous. Example: **`suggestion (blocking):`** means you're recommending it strongly enough to block on it.

### Finding guidelines

- **Be ruthlessly direct.** State what is wrong and why. Don't hedge, soften, or qualify problems — vague feedback doesn't get acted on and wastes the review.
  - Instead of: "This might be a bit complex."
  - Write: "**issue:** This function handles three unrelated concerns. Split it."

- **Always explain the why.** A label without reasoning is just an assertion. Cite the specific design principle, convention, or correctness issue. Feedback without reasoning can be ignored; feedback with reasoning cannot.
  - Instead of: "**issue:** Don't do this."
  - Write: "**issue:** This adds a new abstraction with a single call site. Abstractions need at least two consumers to justify their existence — inline it."

- **When something diverges from an established pattern, flag the divergence but don't assume it's a bug.** If the PR description doesn't explain the deviation, use `note` rather than `issue` — intent can't be determined from the diff alone.
  - Write: "**note:** The rest of the API uses cursor pagination. If offset was chosen intentionally here, the PR description should say why."

- **Provide a concrete alternative when proposing a change.** An example makes the feedback unambiguous and verifiable.

- **Flag the pattern once.** If the same problem appears in multiple places, identify all instances but explain the issue only once.

---

## Step 5: Output the review

Print the review to the terminal using this format. Use the emoji indicators from the label table to color-code each finding — they render visually in Claude Code's terminal output and make severity immediately scannable.

```
## PR Review — [PR title]

> [2-4 sentences. What does this PR do? Is the approach sound? What's the overall read?
> If there are fundamental issues, state them here before the findings.]

---

### 🔴 Blocking

**`file.py:42`** — **issue:** [subject]
[explanation — one or two sentences. Include a code example if proposing an alternative.]

**`file.py:67`** — **todo:** [subject]
[explanation]

---

### 🟡 Non-blocking

**`file.py:89`** — **suggestion:** [subject]
[explanation]

---

### ⚪ Notes & nitpicks

**`file.py:112`** — **nitpick:** [subject]

**`file.py:130`** — **thought:** [subject]

---

### Verdict

> 🔴 **Request changes** — [brief reason: what must be resolved]
```

Omit any section that has no findings. If there are no blocking issues, start with 🟡. The verdict line should use one of:

| | Verdict | When |
|---|---|---|
| ✅ | **Approve** | Ready to merge; all findings non-blocking or resolved |
| 💬 | **Approve with findings** | Ready to merge; remaining findings are optional |
| 🔴 | **Request changes** | One or more blocking issues must be resolved |
| ❓ | **Needs discussion** | Fundamental design question must be settled first |

---

## Rules

- **Block only on things that genuinely degrade code health or introduce bugs.** Nitpicking style and speculative future concerns are not blocking issues. Approve when the code definitely improves the overall code health of the system, even if it isn't perfect.
- **Label every finding.** An unlabeled finding leaves the author guessing whether it blocks merge.
- **Be firm on principles, flexible on implementation.** Require that a problem be solved while giving the author latitude in how to solve it.
- **Never expand scope into unrelated code.** If you notice a problem outside the PR's scope, file it as a `note` rather than blocking this PR on it.
- **Don't accept untracked promises.** Require fixes now, or flag them as needing a tracked issue — a promise in a PR description is not a commitment.

---

## Reference: What you're optimizing for

The goal is ruthlessly accurate, complete feedback — not comfortable feedback. Be critical. Find real problems and say exactly what they are and why they matter. Block on things that degrade code health or introduce bugs; don't block on trivia. Every finding must be specific enough that the author knows exactly what to fix.
