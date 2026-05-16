---
name: review-pr-escalate
description: >-
  Escalating multi-pass PR review: code, gaps, design, architecture, then convergence.
  Each pass uses a fresh agent with a clean context window. Fixes blocking issues between passes.
  Use when the user wants review-until-converged with progressively broader lenses.
  Not for quick reviews (review-pr) or parallel specialist agents (review-pr-deep).
compatibility: Works on all platforms. Requires git CLI and gh CLI on PATH.
metadata:
  author: annabkr
  version: "0.1.0"
---

# PR Review (escalate)

Runs escalating review agents against a PR, automatically fixing issues and progressively widening the review lens until the change converges.

Each review pass launches a **fresh agent with a clean context window**. The agent sees only its pass-specific instructions and the code — not findings from other passes. This prevents bias and ensures each lens gives an honest, independent read.

Pass prompt files live in `passes/` next to this skill.

---

## The escalating review model

A single code review pass is never sufficient. Each pass uses a different lens, progressively broader:

| Round | Pass | Agent prompt | Lens |
|-------|------|-------------|------|
| 1 | `code` | `passes/01-code.md` | In-the-small: line-level bugs and quality |
| 2 | `gaps` | `passes/02-gaps.md` | In-the-small: what the first pass missed |
| 3 | `design` | `passes/03-design.md` | In-the-large: does this design belong here? |
| 4 | `architecture` | `passes/04-architecture.md` | In-the-large: is this the right thing to build? |
| 5 | `convergence` | `passes/05-convergence.md` | Full picture: is this as good as we can make it? |

Read pass files from this skill's directory (e.g. `skills/review-pr-escalate/passes/01-code.md` when installed from llm-toolkit).

Trust in the output is earned through this iteration. The first thing generated is a draft; the point at which the review declares convergence is the first point at which you can moderately trust the result.

---

## Instructions

1. **Read** the agent prompt file for the current pass (starting with `passes/01-code.md`).
2. **Launch a new agent** using the Agent tool:
   - `subagent_type`: `"general-purpose"`
   - `description`: `"Review PR — code pass"` (or whichever pass)
   - `prompt`: The contents of the agent prompt file, with the PR target appended
3. **Relay** the agent's findings to the user.
4. **Process findings** from the current iteration:
   - **`issue` and `todo` findings**: Read the relevant file(s), apply the fix directly in the code, and commit with a clear message.
   - **`rethink` findings**: Do NOT auto-fix. Pause and present the finding to the user with the alternatives and tradeoffs. Wait for the user's decision before proceeding. The user may:
     - Choose an alternative (apply it and continue)
     - Dismiss the concern (mark it resolved and continue)
     - Ask for more analysis (provide it, then re-present the decision)
   - **Non-blocking findings** (`suggestion`, `nitpick`, `note`, `thought`): Record but do not fix or iterate on these.
5. **Re-run the same pass** by launching a new agent with the same pass prompt (up to 3 iterations per pass). Each iteration gets a fresh agent that sees the code as it stands after fixes.
6. Repeat steps 4-5 until:
   - The pass produces no new blocking or `rethink` findings, or
   - The intra-pass iteration cap of **3 iterations** is reached.
7. **Advance to the next pass**: read the next agent prompt file and launch a new agent.
8. **Stop** when:
   - The verdict is **Converged** (pass 5 found no new issues), or
   - All 5 passes are complete, or
   - The user explicitly stops the loop.

---

## Agent isolation rules

- **Never include findings from previous passes in an agent's prompt.** Each agent gets only its pass instructions + the PR target. The clean context window is the whole point.
- **Never share one agent's output with another agent.** Cross-pass state lives only in the orchestrator (this conversation).
- **Fixes happen in the orchestrator, not agents.** Agents are read-only reviewers. You (the orchestrator) apply fixes between agent runs.
- **Each iteration within a pass also gets a fresh agent.** Don't use SendMessage to continue a previous agent — launch a new one so it re-reads the code after your fixes.

---

## Pass progression

- Always run passes in order: code → gaps → design → architecture → convergence.
- Within a pass, iterate up to **3 times**. After fixing blocking issues, launch a fresh agent for the same pass — the second (and third) iteration often catches things the first missed.
- If a pass produces no findings on its first iteration, still advance to the next pass. The broader lens may surface things the narrower lens cannot see.
- If the intra-pass cap is reached with findings still unresolved, record them as carryover and advance. The next pass's broader lens may reframe or subsume them.

---

## Handling `rethink` findings

`rethink` findings appear in passes 3+ (design, architecture). They represent concerns that require human judgment — a different approach, a scope change, or an architectural decision that can't be resolved by editing a few lines.

When you encounter a `rethink` finding:

1. **Present it clearly.** Show the concern, the alternatives, and the tradeoffs.
2. **Wait for a decision.** Do not guess what the user wants. Do not auto-fix.
3. **Apply the decision.** Once the user decides, implement their choice (which may involve significant refactoring) and commit.
4. **Continue the loop.** Advance to the next pass after all findings in the current round are resolved.

If the user's decision on a `rethink` finding requires substantial rework, apply the rework and then **restart from pass 1** (`code`) on the reworked code — launch fresh agents for each pass. The earlier passes need to re-validate the new code.

---

## Between-pass state tracking

Maintain a running log of findings across all passes and iterations. Before each agent launch, output a brief status:

```
### Pass N: <name> (iteration M/3)

Previous passes:
- Pass 1 (code): 3 iterations — 4 blocking fixed, 2 non-blocking noted
- Pass 2 (gaps): 1 iteration — clean on first run
- Pass 3 (design): 2 iterations — 1 blocking fixed, 1 rethink resolved by user

Launching pass N agent (iteration M)...
```

This helps the user track progress toward convergence.

---

## Convergence

The loop has converged when pass 5 (`convergence`) produces no new findings and the verdict is **Converged**. At that point, output:

```
### Review converged after 5 passes

This change has been reviewed across five escalating passes, each with a fresh agent:
1. **Code** (N iterations) — [summary of findings and fixes]
2. **Gaps** (N iterations) — [summary of findings and fixes]
3. **Design** (N iterations) — [summary of findings and fixes]
4. **Architecture** (N iterations) — [summary of findings and fixes]
5. **Convergence** (1 iteration) — no new findings

Total: X blocking issues fixed across Y agent runs, Z rethink decisions made, W non-blocking findings noted.

The change is about as good as we can make it.
```

---

## Early convergence

If passes 3, 4, or 5 produce no new findings, this is a signal that convergence may be happening early. Continue to the next pass anyway — the broader lens may still find something. Only declare convergence after pass 5 explicitly says so.

---

## If convergence is not reached

If all 5 passes complete but pass 5 still has new findings, output:

```
### Did not converge after 5 passes

Remaining findings from pass 5:
- [list findings]

Recommendation: [assess whether another cycle would help or if the remaining issues are fundamental]
```

---

## Rules

- **Each agent gets a clean context window.** This is non-negotiable. Do not inject prior findings.
- Each pass uses a different lens. Do not re-raise findings from previous passes in the orchestrator's fix decisions.
- Fix all blocking findings before re-running the same pass or advancing to the next. Do not advance mid-fix.
- Non-blocking findings (`suggestion`, `nitpick`, `note`, `thought`) are never a reason to continue iterating. Do not fix them unless the user explicitly requests it.
- Do not promote a non-blocking finding to blocking in a later pass.
- `rethink` findings always pause for human input. Never auto-resolve them.
- Each fix should be a separate commit. Do not batch unrelated fixes into one commit.
- If a `rethink` decision triggers substantial rework, restart from pass 1 with fresh agents.
