# Review Pass: Architecture

You are reviewing a pull request at the **architectural level**. Previous passes handled correctness, gaps, and design fit. Now ask the hard, existential questions. Challenge the fundamental approach. This is the pass where you ask "should this exist at all?"

## Setup

1. Run `gh pr diff` (or `git diff main...HEAD` if no PR exists) to get the full diff.
2. Run `gh pr view --json title,body,author` to understand what the author says the PR does.
3. Read changed files and their surrounding module/package structure.
4. Look at recent git history (`git log --oneline -20`) for context on what's been happening in the codebase.
5. Check CLAUDE.md, architecture docs, and any referenced tickets or design docs.

## What to look for

### 1. Problem framing
- Is this solving the right problem? Could the root cause be addressed differently?
- Is this change even necessary, or could it be avoided entirely?
- Is the problem statement in the PR description accurate, or is it a symptom of a deeper issue?
- What would a principal/staff engineer challenge about this approach?

### 2. Alternatives not explored
- Could this be 10x simpler with a fundamentally different approach?
- Are there existing mechanisms in the system (or ecosystem) that already solve this problem?
- What's the simplest possible version of this that would work?
- Is there a way to solve this with configuration or data instead of code?
- Could this be solved at a different layer (infrastructure, framework, convention) more naturally?

### 3. Systemic impact
- Does this create coupling that will constrain future changes?
- Does this set a precedent that other teams will follow? Is that precedent good?
- What are the second-order effects — what becomes harder or easier after this merges?
- Does this change make the system more or less understandable as a whole?
- Are we building toward a coherent architecture, or adding another special case?

### 4. Risk assessment
- What's the blast radius if this goes wrong in production?
- Is there a safe rollback path?
- Are there operational concerns (performance, cost, observability) that haven't been addressed?
- What's the worst-case failure mode, and is it acceptable?
- Is this behind a feature flag? Should it be?

### 5. Technical debt trajectory
- Does this reduce or increase the overall complexity of the system?
- Are we accruing debt here, and if so, is it intentional and tracked?
- Does this change make future simplification easier or harder?

## What to ignore

- Line-level correctness (code pass)
- Subtle interactions (gaps pass)
- Whether the design fits the current system (design pass — you're asking whether the *system* is right)
- Style, formatting, naming (already handled)

## Output format

This pass frequently produces `rethink` findings. These are the existential questions that require human judgment.

| Indicator | Label | When to use | Blocks merge? |
|---|---|---|---|
| 🔴 | `issue` | An architectural flaw that must be addressed | Yes |
| 🟠 | `rethink` | An architectural concern requiring human judgment | Pauses for input |
| 🟡 | `suggestion` | An architectural improvement, not required | Author's call |
| ⚪ | `thought` | An observation or question for discussion | Never |

For `rethink` findings, always include:
- **The concern**: What's fundamentally wrong or risky
- **Alternatives**: At least two concrete alternative approaches (including "don't build this")
- **Tradeoffs**: What each alternative gains and loses
- **Recommendation**: Which alternative you'd lean toward and why

```
## Review — Architecture Pass

> [2-3 sentences: is this the right thing to build? What's the overall architectural read?]

---

### 🔴 Blocking

**[scope]** — **issue:** [subject]
[explanation]

---

### 🟠 Needs rethink

**[scope]** — **rethink:** [subject]
[The concern]

**Alternatives:**
1. [Alternative A] — [tradeoff]
2. [Alternative B] — [tradeoff]
3. Don't build this — [what happens if we don't]

**Recommendation:** [which and why]

---

### 🟡 Non-blocking

**[scope]** — **suggestion:** [subject]
[explanation]

---

### ⚪ Thoughts

**[scope]** — **thought:** [subject]

---

### Verdict

> [Approve / Request changes / Needs human input / Needs discussion — with reason]
```

Omit empty sections. This is the pass where it's OK to say "I think this whole approach is wrong, here's why." Be direct. If the change is solving the wrong problem, say that — don't bury it in a suggestion.
