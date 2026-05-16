---
name: review-pr
description: >-
  Multi-lens PR review that runs parallel agents (general, security, integration),
  checks for prior reviews, and produces structured, deduplicated findings.
  Use when asked to review a PR, critique a change, or give feedback on code.
compatibility: Works on all platforms. Requires git CLI and gh CLI on PATH.
metadata:
  author: annabkr
  version: "0.2.0"
---

# PR Review

Reviews a pull request using multiple focused review agents in parallel, then merges their findings into a single structured review.

---

## Prerequisites

- Git CLI and `gh` CLI installed and authenticated
- Current working directory in a git repository

---

## Step 1: Load the change

Run these in parallel:

1. **Fetch the diff.** If a PR number or URL was provided, use `gh pr diff <number>`. If a branch was provided, use `git diff main...<branch>`. If no argument was given, run `gh pr view --json baseRefName,number` to detect an open PR for the current branch, then fetch its diff.
2. **Fetch PR metadata:** `gh pr view <number> --json title,body,author,additions,deletions,changedFiles,baseRefName,headRefName`
3. **Fetch existing review comments:** `gh api repos/{owner}/{repo}/pulls/{N}/comments --jq '.[] | {user: .user.login, path: .path, line: .line, body: .body}'`
4. **Fetch review state:** `gh pr view <number> --json reviews --jq '.reviews[] | {author: .author.login, state: .state, body: .body}'`

---

## Step 2: Assess prior review state

Before doing any review work, check:

- **Have other reviewers already left feedback?** Read their comments. If issues were raised and subsequently fixed (look for "Fixed in..." replies, or check if the referenced code has changed since the comment), note them as resolved — do not re-discover already-addressed issues.
- **Is this a re-review?** If reviews exist with state `CHANGES_REQUESTED`, focus on whether the requested changes were made rather than doing a full fresh review.
- **What's still open?** Identify unresolved threads — these are higher priority than new findings since someone already flagged them.

---

## Step 3: High-level pass — design and intent

Before launching agents, answer these questions yourself:

1. **Does this PR make sense?** Does the stated intent in the description match what the diff actually does?
2. **Should it exist?** Is the problem being solved the right problem? Is this change even necessary?
3. **Does the design fit the system?** Does the code integrate soundly with existing architecture, naming conventions, and ownership boundaries?
4. **Is scope appropriate?** PRs over ~400 lines become hard to review effectively and increase risk. Over 1,000 lines is almost always too large. If the PR is too large, flag it before continuing — detailed line-level feedback may be invalidated if a split is needed.

If there is a fundamental design problem, communicate it clearly and skip to the output step — don't waste agent runs on a PR that needs to be rethought.

---

## Step 4: Classify the PR and select lenses

Inspect the diff to determine which review agents to run:

| Signal | Classification | Agents to run |
|---|---|---|
| `.py`, `.ts`, `.go`, etc. changed | Code PR | general, security, integration |
| Only `.md`, `SKILL.md`, prompt files | Prompt/config PR | general, security, integration |
| Only docs, README, comments | Documentation PR | general, integration |
| Mix of code + config | Mixed PR | general, security, integration |
| Test files only | Test PR | general only |

**Small PR shortcut:** If the PR is under ~50 lines and touches a single file, skip the parallel agents and do a single-pass review yourself using the review checklist in the Appendix. Use judgment — small PRs don't need three agents.

---

## Step 5: Launch review agents

Spawn agents **in parallel** using the Agent tool. Each agent receives the full diff and PR metadata. Read the corresponding agent file and include its contents as the agent's instructions.

### Agent 1: General review

**Always runs.** Read `agents/general-reviewer.md` and use it as the agent prompt. Include the full diff and PR metadata.

### Agent 2: Security review

**Runs for Code, Prompt/config, and Mixed PRs.** Read `agents/security-reviewer.md` and use it as the agent prompt. Include the full diff and PR metadata.

### Agent 3: Integration review

**Runs for Code, Prompt/config, Mixed, and Documentation PRs.** Read `agents/integration-reviewer.md` and use it as the agent prompt. Include the full diff and PR metadata.

Do not summarize the diff when passing it to agents — they need the full context to produce accurate findings.

---

## Step 6: Merge and deduplicate

When all agents return:

1. **Deduplicate.** If two agents flag the same line or the same underlying issue, keep the more specific finding. Note when multiple lenses independently flagged the same problem — consensus across lenses is a confidence signal.
2. **Reconcile severity.** If agents disagree on severity for the same issue, use the higher severity.
3. **Combine with your own findings** from Step 3 (design/intent issues) and Step 2 (unresolved prior review threads).
4. **Merge into the output format** below.

---

## Step 7: Output the review

Use Conventional Comments format for all findings: `**label [decorations]:** subject`.

| Indicator | Label | When to use | Blocks merge? |
|---|---|---|---|
| 🔴 | `issue` | A defect, bug, or design problem that must be addressed | Yes |
| 🔴 | `todo` | A small required change (typo, missing null check, etc.) | Yes |
| 🟡 | `suggestion` | An improvement you recommend but don't require | Author's call |
| ⚪ | `nitpick` | Trivial, preference-based; completely optional | Never |
| ⚪ | `thought` | Something that came up while reading — not a request | Never |
| ⚪ | `note` | An observation for context, no action needed | Never |

Add `(blocking)` or `(non-blocking)` decorations when the default might be ambiguous.

### Finding guidelines

- **Be ruthlessly direct.** State what is wrong and why. Don't hedge, soften, or qualify.
- **Always explain the why.** Feedback without reasoning can be ignored; feedback with reasoning cannot.
- **When something diverges from an established pattern, flag the divergence but don't assume it's a bug.** Use `note` if intent is unclear from the diff alone.
- **Provide a concrete alternative** when proposing a change.
- **Flag the pattern once.** If the same issue repeats, list all instances but explain once.

### Output template

```
## PR Review — [PR title]

> [2-4 sentences. What does this PR do? Is the approach sound? What's the overall read?
> If there are fundamental issues, state them here before the findings.]

*Reviewed with: general, security, integration lenses*

---

### Prior review state
[Brief summary of existing reviews, what's been addressed, what remains open.
Omit this section entirely if this is the first review.]

---

### 🔴 Blocking

**`file.py:42`** — **issue:** [subject]
[explanation — one or two sentences. Include a code example if proposing an alternative.]

---

### 🟡 Non-blocking

**`file.py:89`** — **suggestion:** [subject]
[explanation]

---

### ⚪ Notes & nitpicks

**`file.py:112`** — **nitpick:** [subject]

---

### Verdict

> 🔴 **Request changes** — [brief reason: what must be resolved]
```

Omit any section that has no findings. The verdict line should use one of:

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
- **Don't invent findings to justify running an agent.** If a lens returns nothing, that's useful signal. Include "No [security/integration] issues identified" in the output.

---

## Appendix: Single-pass review checklist

When the small PR shortcut applies (Step 4), use this checklist instead of launching agents:

### Correctness
- Does the code do what it's supposed to do?
- Edge cases, null/None handling, boundary conditions?
- If the diff references file paths or cross-file dependencies — verify they exist.

### Security
- Does this consume external or user-controlled input? How is it validated?
- For AI/LLM code: prompt injection risk? Unbounded context?
- Secrets handled safely?

### Tests
- Present for new behavior? Would they fail if the code broke?

### Design
- More complex than necessary?
- System integration: new things registered in discovery mechanisms?

### Clarity
- Maintainable in 6 months? Names clear? Comments explain *why*?

### Non-code PRs
- Internal consistency, referenced resources exist, discovery mechanisms updated.
- For LLM-consumed content: injection risks, context exhaustion, missing guardrails.
