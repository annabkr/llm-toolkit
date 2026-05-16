---
name: babysit-ci
description: >-
  Monitor a PR for CI failures and bot review comments, fix issues, and push.
  Use when you want automated CI babysitting until a PR is green.
compatibility: Requires git CLI and gh CLI on PATH. Must be in a git repo with push access.
metadata:
  author: annabkr
  version: "0.2.0"
---

# Babysit CI

Monitors a PR for CI failures and bot review comments (CursorBot, etc.), diagnoses issues, applies fixes, and pushes — repeating until all checks pass and no unresolved bot comments remain.

---

## Prerequisites

- Git CLI and `gh` CLI installed and authenticated
- Current working directory in a git repository with a remote branch
- Push access to the remote branch

---

## Step 1: Identify the branch and PR

If a branch name or PR number was provided as an argument, use it. Otherwise, detect the current branch with `git branch --show-current` and check for an open PR with `gh pr view --json number,headRefName`.

---

## Step 2: Check PR health

Run these in parallel:

1. **CI status:** `gh run list --branch <branch> --limit 3 --json databaseId,status,conclusion,name`
2. **Bot review comments:** `gh api repos/{owner}/{repo}/pulls/{N}/comments --jq '.[] | {user: .user.login, path: .path, line: .line, body: .body}'` — look for comments from bots (CursorBot, copilot, dependabot, etc.) that flag bugs or request changes.
3. **Review state:** `gh pr view <number> --json reviews --jq '.reviews[] | {author: .author.login, state: .state}'` — check for bot reviews with `CHANGES_REQUESTED`.

### Done condition

If **all CI checks passed** AND **no unresolved bot comments** remain, report success and stop. If this skill is running in a loop, signal that the loop should be killed.

### Otherwise

- If CI runs are **in progress**, wait and check back.
- If any runs **failed**, proceed to Step 3 (CI failures).
- If bot comments flag issues, proceed to Step 3b (bot comments).

---

## Step 3: Diagnose failures

For each failed run:

1. Get the failed job logs: `gh run view <run_id> --log-failed`
2. Identify the failure category:

| Category | Signals | Typical fix |
|---|---|---|
| **Test failure** | `FAILED`, `AssertionError`, test file paths in output | Read the failing test, understand the assertion, fix code or test |
| **Lint / format** | `ruff`, `eslint`, `prettier`, hook names in output | Run the formatter/linter locally, commit the result |
| **Type check** | `mypy`, `pyright`, type error messages | Fix the type annotation or code |
| **Build failure** | `ImportError`, `ModuleNotFoundError`, compilation errors | Fix imports, dependencies, or build config |
| **Generated file drift** | "out of date", "regenerate", `make` targets in error | Run the generation command, commit the result |
| **Flaky / infra** | Timeout, network error, service unavailable | Re-run the workflow — don't change code |

---

## Step 3b: Address bot review comments

For each unresolved bot comment:

1. **Read the comment** to understand what the bot is flagging (bug report, style issue, security concern, etc.).
2. **Read the referenced file and line** to see the actual code in context.
3. **Assess validity.** Bots produce false positives. If the comment is wrong or not applicable, note it in your report to the user — don't change code to satisfy an incorrect bot.
4. **If valid**, treat it like a CI failure: understand the issue, make the minimal fix, and proceed to Step 4.

---

## Step 4: Apply the fix

Based on the diagnosis:

1. **Read the relevant source files** to understand context before making changes.
2. **Make the minimal fix.** Don't refactor, don't improve surrounding code. Fix exactly what CI is complaining about.
3. **Run the failing check locally** if possible (e.g., `pytest <path>`, `make openapi`, `ruff check`) to verify the fix before pushing.
4. **Stage, commit, and push.** Use a descriptive commit message that references what CI check failed and what was fixed.

### Fix guidelines

- **Test assertion mismatch** (e.g., `assert 5 == 4`): Understand *why* the count changed. If it's because you added something new (a flag, an endpoint, a field), update the test to account for it. Don't just change the number without understanding the cause.
- **Generated file drift**: Find the generation command (often in a `Makefile` or CI script), run it, and commit the result.
- **Lint/format failures**: Run the tool, stage the reformatted files, commit.
- **Flaky failures**: Use `gh run rerun <run_id> --failed` to retry. Don't change code for infrastructure flakiness.

---

## Step 5: Report back

After making any change — before pushing or continuing — **stop and report to the user**:

1. **What failed:** Which CI check or bot comment, the specific error.
2. **What you changed:** Which files, what the fix was, and why.
3. **What you dismissed** (if any): Bot comments you assessed as false positives, with reasoning.
4. **What you're about to do:** Push, re-run, or ask for guidance.

Wait for the user to acknowledge before pushing. Do not silently fix and push multiple issues in a row.

---

## Step 6: Verify and loop

After pushing:

1. Check that the push succeeded.
2. Poll for the new CI run: `gh run list --branch <branch> --limit 1 --json databaseId,status,conclusion`.
3. If CI **passes**, report success and stop.
4. If CI **fails again**, go back to Step 3.

### Circuit breaker

Stop and ask the user if:

- You've made **3 fix attempts** without CI passing — you may be chasing the wrong problem.
- The failure is in code you didn't write or touch — it may be a pre-existing issue on the base branch.
- The failure requires **interactive credentials**, **environment variables**, or **infrastructure access** you don't have.
- The fix would require a **non-trivial design change** — that's a human decision, not a CI fix.

---

## Rules

- **Minimal fixes only.** The goal is green CI, not code improvement. Don't expand scope.
- **Understand before fixing.** Read the error, read the code, then fix. Don't blindly adjust numbers or suppress errors.
- **Never skip or disable checks.** Don't add `# noqa`, `--no-verify`, `type: ignore`, or similar suppressions unless the check is genuinely wrong.
- **Don't retry indefinitely.** Flaky tests get one retry. If it fails twice, it's not flaky.
- **Commit each fix separately** with a clear message so the PR history shows what CI issues were resolved.
- **Report what you did.** After each fix cycle, tell the user what failed, what you changed, and whether CI is now green.

---

## Example invocations

```
/babysit-ci
/babysit-ci worktree/proj-2257
/babysit-ci 359
```

### Combining with /loop

This skill works well with `/loop` for hands-off monitoring:

```
/loop 1m /babysit-ci 359
```

The skill will automatically signal to kill the loop once all checks pass and no bot comments remain.
