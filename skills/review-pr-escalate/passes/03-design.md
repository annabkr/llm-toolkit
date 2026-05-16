# Review Pass: Design

You are reviewing a pull request at the **design level**. Previous passes already handled line-level correctness and subtle gaps. Step back and evaluate whether the design itself is sound — does this change fit the system it's being added to?

## Setup

1. Run `gh pr diff` (or `git diff main...HEAD` if no PR exists) to get the full diff.
2. Run `gh pr view --json title,body,author` to understand what the author says the PR does.
3. Read changed files in full. Also read neighboring files in the same packages/modules to understand the existing patterns and conventions.
4. Check CLAUDE.md and any architecture docs for documented conventions.

## What to look for

### 1. Fit with the system
- Does this change integrate soundly with existing architecture, naming conventions, and ownership boundaries?
- Does it follow established patterns in the codebase, or introduce a new pattern? If new, is the new pattern justified by the problem, or is it just a different way to do what existing patterns already handle?
- Would someone unfamiliar with this PR understand where to find and modify this code in 6 months?
- Does it respect existing bounded contexts or does it blur responsibilities between modules?

### 2. Abstraction level
- Are abstractions at the right level? Too abstract (premature generalization with one consumer) or too concrete (will need immediate rework)?
- Are helper functions and classes pulling their weight, or are they thin wrappers with a single call site?
- Is there duplication that should be extracted, or extraction that should be inlined?
- Does the public API of new types/functions reveal the right amount of detail?

### 3. Scope and cohesion
- Is this PR doing one coherent thing, or multiple things that should be separate PRs?
- Are there changes included that aren't necessary for the stated goal?
- Is the PR size appropriate? Over ~400 lines is hard to review; over 1,000 is almost always too large.

### 4. Maintenance burden
- What ongoing maintenance does this change create?
- Does it introduce complexity that will need to be understood by every future reader?
- Are there simpler alternatives that achieve the same goal with less cognitive overhead?
- Will this age well, or does it depend on assumptions that are likely to change soon?

### 5. Documentation and contracts
- If public-facing behavior changed (API contracts, config, CLI), is documentation updated?
- Are deprecations, migrations, or breaking changes noted?
- Are new abstractions documented well enough that the next person can use them correctly?

## What to ignore

- Line-level bugs and correctness (the code pass handles those)
- Subtle interactions and hidden assumptions (the gaps pass handles those)
- Whether this is the right problem to solve at all (the architecture pass handles that)

## Output format

This pass may produce `rethink` findings — design concerns that require human judgment rather than a localized code fix.

| Indicator | Label | When to use | Blocks merge? |
|---|---|---|---|
| 🔴 | `issue` | A design flaw that must be fixed | Yes |
| 🟠 | `rethink` | A design concern requiring human judgment | Pauses for input |
| 🟡 | `suggestion` | A design improvement, not required | Author's call |
| ⚪ | `note` | A design observation for context | Never |

For `rethink` findings, always include:
- **The concern**: What's wrong or risky about the current design
- **Alternatives**: At least two concrete alternative approaches
- **Tradeoffs**: What each alternative gains and loses

```
## Review — Design Pass

> [2-3 sentences: is the design sound? Does it fit the system?]

---

### 🔴 Blocking

**`file.py:42`** — **issue:** [subject]
[explanation with concrete alternative]

---

### 🟠 Needs rethink

**`module/`** — **rethink:** [subject]
[The concern]

**Alternatives:**
1. [Alternative A] — [tradeoff]
2. [Alternative B] — [tradeoff]

---

### 🟡 Non-blocking

**`file.py:89`** — **suggestion:** [subject]
[explanation]

---

### Verdict

> [Approve / Request changes / Needs human input — with reason]
```

Omit empty sections. Be direct about design problems — "this could be simpler" isn't useful; "this introduces a new Repository pattern when the existing service-level query pattern handles this case — adding a second pattern increases cognitive load for every future reader without clear benefit" is useful.
