# LLM Toolkit

A collection of reusable Claude Code skills and prompts for common development workflows.

## Motivation

This library reflects explorations with high and low levels of delegation. 

One thing is certain: first pass LLM output is almost always unsatisfactory. Iteration helps, but it's not a magic wand or a replacement for critical thinking.

## Skills

### PR review (at a glance)

| | [`review-pr`](skills/review-pr/SKILL.md) | [`review-pr-deep`](skills/review-pr-deep/SKILL.md) | [`review-pr-escalate`](skills/review-pr-escalate/SKILL.md) |
|---|---|---|---|
| **Passes** | 1 (orchestrator does everything) | 1 round, **3 parallel specialists** | **5 sequential passes**, up to 3 iterations each |
| **Agents** | None (one conversation) | General + security + integration (by PR type) | Fresh agent per pass/iteration |
| **Lens** | Design + full code checklist in one go | **Domain split** (quality vs security vs wiring) | **Zoom out over time** (code → gaps → design → architecture → convergence) |
| **Fixes code?** | No | No (optional [`converge.md`](skills/review-pr-deep/converge.md) loop) | **Yes** (auto-fixes `issue`/`todo` between passes) |
| **Prior reviews** | No | **Yes** (fetches existing comments/reviews) | No |
| **Best for** | Quick, efficient review | Non-trivial PRs needing specialist coverage | Critical PRs you want **reviewed until converged** |

### [`review-pr`](skills/review-pr/SKILL.md)
Single-pass PR review with ruthless critical rigor. Provides structured, clearly labeled findings for design, correctness, tests, clarity, and style.

**Use when:** You need a quick, efficient review or one pass is sufficient.

```bash
/review-pr <PR-number>
/review-pr <branch-name>
```

### [`review-pr-deep`](skills/review-pr-deep/SKILL.md)
Multi-agent PR review with specialized passes: general code review, integration testing, and security analysis. Agents converge on findings.

**Use when:** You need a thorough, comprehensive review of high-stakes or complex changes.

```bash
/review-pr-deep <PR-number>
```

### [`review-pr-escalate`](skills/review-pr-escalate/SKILL.md)
Escalated multi-pass review covering code quality, design gaps, architecture alignment, and final convergence. For the most critical reviews.

**Use when:** You need exhaustive analysis across five specialized passes.

```bash
/review-pr-escalate <PR-number>
```

### [`python-expert`](skills/python-expert/SKILL.md)
Expert guidance on Python best practices, style guides, type hints, and code quality. Knowledge base includes PEP 8, Google Style Guide, PEP 257 (docstrings), and PEP 484 (type hints).

**Use when:** You need advice on Python coding standards, naming conventions, or documentation.

```bash
/python-expert How should I name my class variables?
/python-expert What's the correct format for multi-line docstrings?
/python-expert Should I use List[int] or list[int]?
```

### [`babysit-ci`](skills/babysit-ci/SKILL.md)
Monitors a PR for CI failures and bot review comments. Diagnoses issues, applies fixes, and pushes. Repeats until all checks pass.

**Use when:** You want automated CI fixing until a PR is green.

```bash
/babysit-ci
/babysit-ci <PR-number>
/babysit-ci <branch-name>
```

Works well with `/loop` for hands-off monitoring:
```bash
/loop 1m /babysit-ci 359
```

## Installation

These skills integrate directly with Claude Code. To use them:

1. **Ensure Claude Code is installed.** Available as CLI, desktop app, web app, or IDE extensions.
2. **Authenticate with git and gh.** Required for all PR and CI-related skills:
   ```bash
   git config user.name "Your Name"
   git config user.email "you@example.com"
   gh auth login
   ```
3. **Navigate to a git repository** — Skills operate on the current repo's code and PRs
4. **Invoke a skill** using `/skill-name` in Claude Code (e.g., `/review-pr 123`).

## Skill Framework

Each skill is self-contained with:
- **SKILL.md** — Detailed step-by-step instructions and methodology
- **knowledge/** — Reference materials (Python expert knowledge base)
- **agents/** — Specialized agent definitions (review-pr-deep, review-pr-escalate)

## Principles

### Code Review
- **Be firm on principles, flexible on implementation.** Require that a problem be solved while giving the author latitude in how to solve it.
- **Be ruthlessly direct.** State what is wrong and why. Don't hedge or soften.
- **Label every finding** using [Conventional Comments](https://conventionalcomments.org/) format.
- **Provide concrete alternatives** when proposing changes.

### Python Expertise
- **Code is read more often than written.** Prioritize readability.
- **Consistency matters.** Consistency within a module matters more than within a project, which matters more than with style guides.
- **Explicit is better than implicit.** Write clear code over clever code.

### CI Automation
- **Minimal fixes only.** Fix exactly what CI is complaining about, don't refactor
- **Understand before fixing.** Read the error and code, then act.
- **Report what you did.** Explain the diagnosis and fix for user visibility.

## Examples

### Review a pull request
```bash
# Quick review
/review-pr 123

# Comprehensive multi-agent review
/review-pr-deep 123

# Exhaustive five-pass review
/review-pr-escalate 123
```

### Get Python guidance
```bash
/python-expert How do I write a proper docstring for a function with multiple return values?
/python-expert What are the differences between Google Style and PEP 8 naming conventions?
```

### Monitor and fix CI
```bash
# One-shot fix
/babysit-ci 359

# Hands-off loop until green
/loop 1m /babysit-ci 359
```

## Prerequisites

- **Git CLI** and **gh CLI** (GitHub CLI) installed and authenticated
- **Current working directory** in a git repository (for most skills)
- **Push access** to branches (for babysit-ci)

## License & Attribution

Created by Anna Baker (@annabkr)
