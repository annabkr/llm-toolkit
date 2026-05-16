# review-pr-deep converge

Use this prompt to run `/review-pr-deep` iteratively against a PR, automatically fixing blocking issues until the PR converges.

## Instructions

1. Run `/review-pr-deep` on the target PR and record all `issue` and `todo` findings from the output.
2. If the verdict is **Approve** or **Approve with findings**, stop. The PR is ready.
3. Otherwise, for each blocking (`issue` or `todo`) finding:
   - Read the relevant file(s)
   - Apply the fix directly in the code
   - Commit the fix with a clear message describing what was changed and why
4. Once all blocking findings from the current round are fixed, re-fetch the diff and run `/review-pr-deep` again.
5. On each subsequent round, check that all blocking findings from the **previous round** are resolved. Only raise a finding as unresolved if the underlying problem is still present — do not re-raise it if the fix addresses the problem, even imperfectly.
6. Stop when:
   - The verdict is **Approve** or **Approve with findings**, or
   - The round cap of **5 rounds** is reached.

## If the round cap is reached without convergence

Output a **Could not converge** verdict listing the blocking findings that remain unresolved after 5 rounds. Do not continue iterating.

## Rules

- Fix all blocking findings before re-running the review. Do not re-run the review mid-fix.
- Non-blocking findings (`suggestion`, `nitpick`, `note`, `thought`) are never a reason to continue iterating. Do not fix them unless explicitly instructed.
- Do not promote a non-blocking finding to blocking in a later round.
- Do not find new problems with how a previous finding was fixed. If the underlying issue is resolved, it passes.
- Each fix should be a separate commit. Do not batch unrelated fixes into one commit.
