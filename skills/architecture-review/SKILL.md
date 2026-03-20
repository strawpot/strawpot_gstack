---
name: architecture-review
description: "Structured engineering architecture review for plans, designs, and implementation proposals. Use when asked to review architecture, do an engineering review, evaluate a plan, lock in a design, assess technical approach, poke holes in a proposal, sanity check a design before building, check if something is over-engineered, or 'does this design make sense'. Also use proactively when a role is about to start implementation on a non-trivial plan, to catch architecture issues before code is written. Consumed by strategic-reviewer and design-system-architect roles. Covers scope challenge, system design, code quality, test coverage, performance, and failure modes. Do NOT use for code review of already-written code (use a code-reviewer role instead) or for UI/UX design review (use design-review-methodology instead)."
metadata:
  strawpot:
    dependencies:
      - engineering-principles
      - completeness-principle
---

# Architecture Review

A structured engineering review methodology for plans and designs. Walk through each section interactively, surfacing issues with opinionated recommendations and concrete tradeoffs. This skill is adapted from the `/plan-eng-review` methodology in [garrytan/gstack](https://github.com/garrytan/gstack).

## How to use this skill

You are reviewing a plan, design document, or implementation proposal, not finished code. For every issue or recommendation, explain the concrete tradeoffs, give an opinionated recommendation, and get input before assuming a direction.

For UI/UX review of plans, use the `design-review-methodology` skill instead. The two skills can run sequentially on the same plan: architecture-review covers engineering concerns, design-review-methodology covers design dimensions.

## Priority hierarchy

If running low on context or asked to compress: **Step 0 > Test diagram > Opinionated recommendations > Everything else**. Never skip Step 0 or the test diagram. Scope challenge prevents wasted work on the wrong thing, and the test diagram is the only reliable way to catch coverage gaps.

## Engineering preferences

Use these to guide your recommendations throughout the review:

- **DRY is important.** Flag repetition aggressively.
- **Well-tested code is non-negotiable.** Rather have too many tests than too few.
- **Engineered enough:** not under-engineered (fragile, hacky) and not over-engineered (premature abstraction, unnecessary complexity).
- **Handle more edge cases, not fewer.** Thoughtfulness over speed.
- **Explicit over clever.** Bias toward clarity.
- **Minimal diff.** Achieve the goal with the fewest new abstractions and files touched.

## Cognitive Patterns: How Great Eng Managers Think

These are not checklist items. They are the instincts experienced engineering leaders develop, the pattern recognition that separates "reviewed the plan" from "caught the landmine." Apply them throughout your review.

1. **State diagnosis.** Teams exist in four states: falling behind, treading water, repaying debt, innovating. Each demands a different intervention (Larson, *An Elegant Puzzle*).
2. **Blast radius instinct.** Every decision evaluated through "what's the worst case and how many systems/people does it affect?"
3. **Boring by default.** "Every company gets about three innovation tokens." Everything else should be proven technology (McKinley, *Choose Boring Technology*).
4. **Incremental over revolutionary.** Strangler fig, not big bang. Canary, not global rollout. Refactor, not rewrite (Fowler).
5. **Systems over heroes.** Design for tired humans at 3am, not your best engineer on their best day.
6. **Reversibility preference.** Feature flags, A/B tests, incremental rollouts. Make the cost of being wrong low.
7. **Failure is information.** Blameless postmortems, error budgets, chaos engineering. Incidents are learning opportunities, not blame events (Allspaw, *Google SRE*).
8. **Org structure IS architecture.** Conway's Law in practice. Design both intentionally (Skelton/Pais, *Team Topologies*).
9. **DX is product quality.** Slow CI, bad local dev, painful deploys lead to worse software and higher attrition. Developer experience is a leading indicator.
10. **Essential vs accidental complexity.** Before adding anything: "Is this solving a real problem or one we created?" (Brooks, *No Silver Bullet*).
11. **Two-week smell test.** If a competent engineer can't ship a small feature in two weeks, you have an onboarding problem disguised as architecture.
12. **Glue work awareness.** Recognize invisible coordination work. Value it, but don't let people get stuck doing only glue (Reilly, *The Staff Engineer's Path*).
13. **Make the change easy, then make the easy change.** Refactor first, implement second. Never make structural and behavioral changes simultaneously (Beck).
14. **Own your code in production.** No wall between dev and ops. Engineers write code and own it in production (Majors).
15. **Error budgets over uptime targets.** SLO of 99.9% means 0.1% downtime *budget to spend on shipping*. Reliability is resource allocation (*Google SRE*).

When evaluating architecture, think "boring by default." When reviewing tests, think "systems over heroes." When assessing complexity, ask Brooks's question. When a plan introduces new infrastructure, check whether it spends an innovation token wisely.

## Step 0: Scope Challenge

Before reviewing anything, answer these questions:

1. **What existing code already partially or fully solves each sub-problem?** Can you capture outputs from existing flows rather than building parallel ones?
2. **What is the minimum set of changes that achieves the stated goal?** Flag any work that could be deferred without blocking the core objective. Be ruthless about scope creep.
3. **Complexity check:** If the plan touches more than 8 files or introduces more than 2 new classes/services, treat that as a smell and challenge whether the same goal can be achieved with fewer moving parts.
4. **TODOS cross-reference:** Read `TODOS.md` (or equivalent task tracker) if it exists. Are any deferred items blocking this plan? Can any deferred items be bundled into this work without expanding scope? Does this plan create new work that should be captured?
5. **Completeness check:** Apply the completeness-principle skill: is this plan doing the complete version or a shortcut? If the shortcut saves minutes with AI but the complete version costs little more, recommend complete. Use the lake/ocean classification to distinguish boilable scope from unbounded scope.

If the complexity check triggers (8+ files or 2+ new classes/services), recommend scope reduction. Explain what's overbuilt, propose a minimal version that achieves the core goal, and ask whether to reduce or proceed as-is.

If the complexity check does not trigger, present your Step 0 findings and proceed to Section 1.

**Critical:** Once the user accepts or rejects a scope reduction recommendation, commit fully. Do not re-argue for smaller scope during later review sections. Do not silently reduce scope or skip planned components.

## Review Sections

After scope is agreed, work through each section interactively. Present issues one at a time with opinionated recommendations. Maximum 8 issues per section; beyond that, review fatigue sets in and the reviewer stops engaging meaningfully with each issue.

### 1. Architecture Review

Evaluate:

- Overall system design and component boundaries
- Dependency graph and coupling concerns
- Data flow patterns and potential bottlenecks
- Scaling characteristics and single points of failure
- Security architecture (auth, data access, API boundaries)
- Whether key flows deserve ASCII diagrams in the plan or in code comments
- For each new codepath or integration point, describe one realistic production failure scenario and whether the plan accounts for it

Present each issue individually. State options, your recommendation, and the reasoning. Only proceed to the next section after all issues are resolved.

### 2. Code Quality Review

Evaluate:

- Code organization and module structure
- DRY violations (be aggressive here)
- Error handling patterns and missing edge cases (call these out explicitly)
- Technical debt hotspots
- Areas that are over-engineered or under-engineered relative to the engineering preferences
- Existing ASCII diagrams in touched files: are they still accurate after this change?

Present each issue individually. Resolve all before proceeding.

### 3. Test Review

Make an ASCII diagram of all new UX, new data flow, new codepaths, and new branching logic or outcomes. Use a tree or flow format, for example:

```
New Login Flow
├── POST /auth/login
│   ├── valid credentials → JWT issued → 200
│   ├── invalid credentials → 401
│   └── rate limit exceeded → 429
└── POST /auth/refresh
    ├── valid token → new JWT → 200
    └── expired token → 401
```

For each item, note what is new about the features in this plan. Then for each new item in the diagram, verify there is a corresponding test.

Evaluate:

- Test coverage against the diagram: every new codepath needs a test
- Edge case coverage: are the hard cases tested, not just the happy path?
- Test quality: are tests testing behavior or implementation details?
- Test pyramid: right balance of unit, integration, and end-to-end tests?

Present each gap individually. Resolve all before proceeding.

### 4. Performance Review

Evaluate:

- N+1 queries and database access patterns
- Memory usage concerns
- Caching opportunities
- Pagination for unbounded result sets
- Slow or high-complexity code paths

Present each issue individually. Resolve all before proceeding.

## How to Present Issues

For each issue found in any section:

- **Number issues** (1, 2, 3...) and **letter options** (A, B, C...)
- **Describe the problem concretely**, with file and line references where applicable
- **Present 2-3 options**, including "do nothing" where that's reasonable
- **State your recommendation** and explain **why**, mapping to one of the engineering preferences or cognitive patterns
- **One sentence per option.** The user should be able to pick in under 5 seconds
- **Escape hatch:** If an issue has an obvious fix with no real alternatives, state what you'll do and move on. Only present options when there is a genuine decision with meaningful tradeoffs.

## Documentation and Diagrams

- Use ASCII art diagrams liberally: for data flow, state machines, dependency graphs, processing pipelines, and decision trees
- For complex designs, recommend embedding ASCII diagrams directly in code comments: Models (data relationships, state transitions), Controllers (request flow), Services (processing pipelines), Tests (non-obvious test structures)
- **Diagram maintenance is part of the change.** When modifying code that has ASCII diagrams in comments nearby, review whether those diagrams are still accurate. Update them as part of the same commit. Stale diagrams are worse than no diagrams because they actively mislead. Flag any stale diagrams you encounter during review even if they are outside the immediate scope of the change.

## Required Outputs

Every architecture review must produce all of the following:

### "NOT in scope" section
List work that was considered and explicitly deferred, with a one-line rationale for each item.

### "What already exists" section
List existing code/flows that already partially solve sub-problems in this plan, and whether the plan reuses them or unnecessarily rebuilds them.

### Failure modes
For each new codepath identified in the test review diagram, list one realistic way it could fail in production (timeout, nil reference, race condition, stale data, etc.) and whether:
1. A test covers that failure
2. Error handling exists for it
3. The user would see a clear error or a silent failure

If any failure mode has **no test AND no error handling AND would be silent**, flag it as a **critical gap**.

### TODOS updates
After all review sections are complete, present each potential TODO individually. For each:
- **What:** One-line description
- **Why:** The concrete problem it solves or value it unlocks
- **Pros/Cons:** What you gain vs. cost, complexity, or risks
- **Context:** Enough detail that someone picking this up in 3 months understands the motivation
- **Dependencies:** Any prerequisites or ordering constraints

Do not append vague bullet points. A TODO without context is worse than no TODO.

### Completion Summary

At the end of the review, display:

```
Step 0: Scope Challenge -- ___ (scope accepted / scope reduced)
Architecture Review: ___ issues found
Code Quality Review: ___ issues found
Test Review: diagram produced, ___ gaps identified
Performance Review: ___ issues found
NOT in scope: written
What already exists: written
TODOS updates: ___ items proposed
Failure modes: ___ critical gaps flagged
```

## Retrospective Learning

Check the git log for the relevant branch. If there are prior commits suggesting a previous review cycle (review-driven refactors, reverted changes), note what was changed and whether the current plan touches the same areas. Be more aggressive reviewing areas that were previously problematic.

## Escalation

It is always OK to stop and say "this is too hard for me" or "I'm not confident in this result." Bad work is worse than no work.

- If you have attempted a task 3 times without success, stop and escalate
- If you are uncertain about a security-sensitive change, stop and escalate
- If the scope of work exceeds what you can verify, stop and escalate
