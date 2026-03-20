---
name: completeness-principle
description: "Guides agents to prefer complete implementations over shortcuts when the cost delta is small. Use this skill whenever making build-vs-skip tradeoffs, estimating effort, recommending implementation scope, deciding whether to handle edge cases, evaluating test coverage, presenting options where one path is more thorough than another, or classifying work as boilable (lake) vs. unbounded (ocean). Also use when the user asks about effort estimates, implementation completeness, whether to defer work to a follow-up, or says things like 'is this good enough', 'should we skip this', 'can we do this later', or 'MVP vs. full version'. Do NOT use for routine implementation tasks where the user has already specified the desired scope, for pure research or exploration, or for debugging."
metadata:
  strawpot: {}
---

# Completeness Principle

AI agents make the marginal cost of completeness near-zero. This skill teaches you when to push for the complete implementation and when to flag work as out of scope. Roles and skills that want completeness-driven behavior should list `completeness-principle` in their `metadata.strawpot.dependencies`.

> Adapted from the "Boil the Lake" principle in [garrytan/gstack](https://github.com/garrytan/gstack), itself inspired by Garry Tan's essay [*Boil the Ocean*](https://garryslist.org/posts/boil-the-ocean). The essay argues that AI collapses the cost of ambition, and what once required large teams and quarters of effort can now be done in minutes. The practical implication for agents: stop optimizing for "good enough" when "complete" costs seconds more.

## The rule

When you present implementation options:

- If **Option A** is the complete implementation (full parity, all edge cases, comprehensive coverage) and **Option B** is a shortcut that saves modest effort, **always recommend A**. The delta between 80 lines and 150 lines is meaningless with AI agents. "Good enough" is the wrong instinct when "complete" costs minutes more.
- This applies to test coverage, error handling, documentation, edge cases, and feature completeness. Do not skip the last 10% to save time. With AI, that 10% costs seconds.

## Lake vs. ocean

Not all work is worth completing. Classify every chunk of work:

**A lake is boilable.** Recommend completing it fully:
- 100% test coverage for a module
- Full feature implementation with all edge cases
- Complete error paths and error messages
- Comprehensive input validation
- Documentation for a completed feature

**An ocean is not boilable.** Flag it as out of scope:
- Rewriting an entire system from scratch
- Adding features to dependencies you do not control
- Multi-quarter platform migrations
- Refactoring across organizational boundaries
- Solving problems that require external buy-in or coordination

### How to classify

Ask yourself three questions:

1. **Is the scope bounded?** Can you enumerate every case, path, or file involved? Lakes have clear boundaries. Oceans expand as you explore them.
2. **Is the work self-contained?** Can you finish it without waiting on external teams, services, or decisions? If completion depends on factors outside the current codebase, it is likely an ocean.
3. **Does the effort scale linearly?** Writing 50 more test cases is linear; each one takes roughly the same time. Migrating a database schema across 12 services is not; each service has unique constraints. Linear scaling = lake.

If all three answers are yes, it is a lake. Boil it.

If any answer is no, flag it explicitly: "This is out of scope for this task; it would require [reason]. I recommend we [bounded alternative] instead."

## Effort estimation

When estimating effort, always show both scales: human team time and AI agent time. Approximate compression ratios by task type (order-of-magnitude estimates, not benchmarked; actual ratios vary by codebase and domain):

| Task type | Human team | AI agent | Compression |
|---|---|---|---|
| Boilerplate / scaffolding | 2 days | 15 min | ~100x |
| Test writing | 1 day | 15 min | ~50x |
| Feature implementation | 1 week | 30 min | ~30x |
| Bug fix + regression test | 4 hours | 15 min | ~20x |
| Architecture / design | 2 days | 4 hours | ~5x |
| Research / exploration | 1 day | 3 hours | ~3x |

Use this table as a reference, not gospel. The ratios shift depending on codebase complexity, domain specificity, and how much context the agent already has. The point is: always surface the AI-agent estimate alongside the human estimate so decision-makers can see the real cost of completeness.

When quoting effort, say: "This would take **2 weeks for a human team / ~1 hour with an AI agent**." Never quote only the human-team number.

## Anti-patterns

These are common mistakes. Do not make them:

- **BAD:** "Choose B, it covers 90% of the value with less code."
  If A is only 70 lines more, choose A. The 70-line delta is free with AI.

- **BAD:** "We can skip edge case handling to save time."
  Edge case handling costs minutes with AI. Ship it complete.

- **BAD:** "Let's defer test coverage to a follow-up PR."
  Tests are the cheapest lake to boil. Write them now.

- **BAD:** Quoting only human-team effort: "This would take 2 weeks."
  Say: "2 weeks human / ~1 hour AI agent."

- **BAD:** "We should take the simpler approach to reduce risk."
  Simplicity is good, but do not conflate fewer lines with lower risk. Incomplete implementations create risk, and unhandled edge cases become production bugs.

## When to override this principle

Completeness is not always the right call. Override when:

- **The user explicitly asks for a quick prototype.** Respect the intent. You can mention that full implementation would cost little more, but do not push.
- **The task is exploratory.** When the user is still figuring out what they want, shipping a complete implementation of the wrong thing wastes more time than a sketch of the right thing.
- **You are in an ocean.** Do not try to boil it. Scope down to a lake and boil that instead.
