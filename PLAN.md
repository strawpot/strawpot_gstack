# Gstack → StrawPot Migration Plan

## Executive Summary

This plan details the migration of 21 components from [garrytan/gstack](https://github.com/garrytan/gstack) — a sprint-workflow toolkit for Claude Code — into StrawPot-native roles and skills. Gstack provides a tightly integrated sprint cycle (Think → Plan → Build → Review → Test → Ship → Reflect) with headless browser infrastructure, safety guardrails, and multi-AI tooling.

**Why migrate?** Gstack's workflows encode high-value engineering methodology (YC-grade product thinking, Staff-level code review, completeness-driven QA) that StrawPot users would benefit from. By adapting these into StrawPot's role/skill system, we get:

- **Composability**: Skills can be mixed into any role, not locked to gstack's monolith
- **Orchestration**: Roles integrate with denden for delegation, scheduling, and coordination
- **Publishing**: Components become installable via StrawHub
- **Evaluation**: Each component can be independently tested and scored

**Scope**: 21 gstack components → ~12 StrawPot skills + ~8 StrawPot roles + 5 components skipped

---

## Migration Strategy

### What becomes a Role vs a Skill

| Gstack Pattern | StrawPot Type | Rationale |
|---|---|---|
| Persona-driven specialist (e.g., "You are a Staff Engineer") | **Role** | Roles embody identity, judgment, and workflow orchestration |
| Methodology/knowledge (e.g., completeness principle, review checklist) | **Skill** | Skills provide reusable knowledge that multiple roles consume |
| Infrastructure/tooling (e.g., browser commands, safety hooks) | **Skill** | Skills encapsulate tool usage patterns |
| Combined persona + tooling (e.g., /qa = QA Lead + browser) | **Role + Skill** | Split: role for persona/workflow, skill for tool knowledge |

### Key Adaptation Principles

1. **Strip gstack-specific scaffolding**: Remove telemetry preamble, upgrade checks, session tracking, ELI16 mode, contributor mode detection — these are gstack platform concerns, not workflow knowledge.

2. **Extract the Completeness Principle as a shared skill**: Gstack's "Boil the Lake" principle appears in every skill. Extract it once as a standalone skill that roles can depend on.

3. **Replace AskUserQuestion with denden**: Gstack uses a custom `AskUserQuestion` format. StrawPot roles use denden's `askUser` for user interaction.

4. **Preserve domain expertise**: The actual review checklists, QA methodology, design audit criteria, and shipping workflows are the high-value content — preserve these faithfully.

5. **Respect existing StrawPot overlap**: Where gstack functionality overlaps with existing StrawPot components, prefer enhancing the existing component over creating a duplicate.

---

## Component-by-Component Migration Plan

### Planning Phase

#### 1. /office-hours → Role: `product-advisor`

| Field | Value |
|---|---|
| **Gstack name** | `/office-hours` |
| **What it does** | YC-style product diagnostic. Six forcing questions expose demand reality. Produces a structured design doc. |
| **StrawPot type** | Role |
| **Proposed name** | `product-advisor` |
| **Dependencies** | Skills: `completeness-principle` |
| **Complexity** | Medium |
| **Notes** | The six forcing questions (who's the customer, what do they do today, what's broken, what do you build, how do you know it works, what's the go-to-market) are the core value. Strip gstack preamble, keep the diagnostic framework and design doc template. The 10-star product framework from plan-ceo-review could be referenced but not duplicated. |

#### 2. /plan-ceo-review → Role: `strategic-reviewer`

| Field | Value |
|---|---|
| **Gstack name** | `/plan-ceo-review` |
| **What it does** | CEO/founder-mode strategic review. Rethinks the problem, finds 10-star product opportunities. Four scope modes (startup, enterprise, infra, platform). |
| **StrawPot type** | Role |
| **Proposed name** | `strategic-reviewer` |
| **Dependencies** | Skills: `completeness-principle` |
| **Complexity** | Medium |
| **Notes** | Heavy persona content (YC partner voice, founder-mode thinking). The four scope archetypes and 10-star product framework are the key value. Adapts well to a standalone role that can be delegated to via denden. Remove gstack session management. |

#### 3. /plan-eng-review → Skill: `architecture-review` (enhance existing)

| Field | Value |
|---|---|
| **Gstack name** | `/plan-eng-review` |
| **What it does** | Engineering manager architecture review. Locks data flow, edge cases, test strategy. |
| **StrawPot type** | Skill |
| **Proposed name** | `architecture-review` |
| **Dependencies** | Skills: `engineering-principles` |
| **Complexity** | Low |
| **Notes** | This overlaps significantly with StrawPot's existing `engineering-principles` skill. The gstack version adds a structured review checklist (architecture decisions, data flow, edge cases, test plan). Best approach: create a new `architecture-review` skill that references `engineering-principles` and adds the review methodology. Can be consumed by `implementation-planner` role or a new reviewer role. |

#### 4. /plan-design-review → Skill: `design-review-methodology`

| Field | Value |
|---|---|
| **Gstack name** | `/plan-design-review` |
| **What it does** | Senior designer plan audit. Rates design dimensions 0-10. AI slop detection. |
| **StrawPot type** | Skill |
| **Proposed name** | `design-review-methodology` |
| **Dependencies** | None |
| **Complexity** | Low |
| **Notes** | The scoring rubric (layout, typography, color, spacing, hierarchy, consistency, responsiveness, accessibility, delight, AI-slop-free) is the core value. Extract as a methodology skill. The persona aspect is light enough that it doesn't warrant a full role — any role can apply this checklist. |

#### 5. /design-consultation → Role: `design-system-architect`

| Field | Value |
|---|---|
| **Gstack name** | `/design-consultation` |
| **What it does** | Design partner that builds complete design systems from scratch. Creates tokens, components, patterns. |
| **StrawPot type** | Role |
| **Proposed name** | `design-system-architect` |
| **Dependencies** | Skills: `design-review-methodology`, `completeness-principle`, `browse` |
| **Complexity** | High |
| **Notes** | This is a heavy persona role (Design Partner with strong opinions). Uses the browser for visual reference gathering. Produces design system artifacts (tokens, component specs, pattern library). The browser dependency makes this higher complexity. Overlaps with StrawPot's existing `frontend-design` skill — this role could consume that skill. |

### Review Phase

#### 6. /review → Skill: `pr-review-methodology` (enhance existing)

| Field | Value |
|---|---|
| **Gstack name** | `/review` |
| **What it does** | Staff Engineer pre-landing PR review. Find bugs, auto-fix obvious ones, flag gaps. Atomic commits for fixes. |
| **StrawPot type** | Skill (merge into existing) |
| **Proposed name** | `pr-review-methodology` → merge into existing `code-review` skill |
| **Dependencies** | Skills: `git-workflow`, `completeness-principle` |
| **Complexity** | Low |
| **Notes** | **Significant overlap with existing StrawPot `code-review` skill and `code-reviewer` role.** The gstack version adds: review logging (`gstack-review-log`), atomic commit fixes during review, and the completeness principle. Best approach: enhance the existing `code-review` skill with gstack's review checklist items and atomic fix patterns rather than creating a duplicate. The gstack review log concept doesn't apply (StrawPot uses denden for tracking). |

#### 7. /investigate → Role: `debugger`

| Field | Value |
|---|---|
| **Gstack name** | `/investigate` |
| **What it does** | Systematic root-cause debugging. Iron Law: no fixes without investigation. Hypothesis → test → narrow → root cause → fix. |
| **StrawPot type** | Role |
| **Proposed name** | `debugger` |
| **Dependencies** | Skills: `completeness-principle`, `engineering-principles` |
| **Complexity** | Medium |
| **Notes** | Strong persona (senior debugger who refuses to guess). The Iron Law methodology (never fix before understanding root cause) and structured hypothesis testing are the core value. No gstack-specific tooling dependencies. Maps cleanly to a StrawPot role. Could be consumed by `qa-engineer` role as a dependency. |

#### 8. /design-review → Role: `visual-qa-reviewer`

| Field | Value |
|---|---|
| **Gstack name** | `/design-review` |
| **What it does** | Designer who codes. Live site visual audit with browser, then atomic commit fixes. |
| **StrawPot type** | Role |
| **Proposed name** | `visual-qa-reviewer` |
| **Dependencies** | Skills: `browse`, `design-review-methodology`, `git-workflow`, `completeness-principle` |
| **Complexity** | High |
| **Notes** | Combines design expertise with browser-based visual testing and code fixes. The audit → screenshot → fix → verify loop is the core workflow. Heavy dependency on browser infrastructure. This is distinct from StrawPot's existing `qa-engineer` (which focuses on test code, not visual quality). |

### QA Phase

#### 9. /qa → Role: `browser-qa-engineer`

| Field | Value |
|---|---|
| **Gstack name** | `/qa` |
| **What it does** | QA lead. Opens real browser, finds bugs, fixes with atomic commits, re-verifies. Full test-fix-verify loop. |
| **StrawPot type** | Role |
| **Proposed name** | `browser-qa-engineer` |
| **Dependencies** | Skills: `browse`, `git-workflow`, `completeness-principle` |
| **Complexity** | High |
| **Notes** | **Overlaps with existing `qa-engineer` role** but is fundamentally different — gstack /qa uses a real browser to test running applications, while StrawPot's `qa-engineer` writes and runs test code. These are complementary, not duplicative. The gstack QA methodology (systematic page crawl, interaction testing, responsive checks, accessibility audit) is the core value. Needs `browse` skill for browser commands. |

#### 10. /qa-only → Skill: `browser-qa-reporting`

| Field | Value |
|---|---|
| **Gstack name** | `/qa-only` |
| **What it does** | Same QA methodology as /qa but report-only — no code changes. |
| **StrawPot type** | Skill (methodology variant) |
| **Proposed name** | `browser-qa-reporting` |
| **Dependencies** | Skills: `browse`, `completeness-principle` |
| **Complexity** | Low |
| **Notes** | This is essentially the /qa methodology minus the fix loop. In StrawPot terms, this is better expressed as a mode of the `browser-qa-engineer` role (report-only vs fix mode) rather than a separate component. **Recommendation: fold into `browser-qa-engineer` role as a "report-only" mode rather than creating a separate skill.** Document both modes in the role's workflow. |

### Ship Phase

#### 11. /ship → Skill: `release-workflow`

| Field | Value |
|---|---|
| **Gstack name** | `/ship` |
| **What it does** | Fully automated release: sync main, run tests, audit coverage, push, open PR. One command. |
| **StrawPot type** | Skill |
| **Proposed name** | `release-workflow` |
| **Dependencies** | Skills: `git-workflow`, `github-prs`, `completeness-principle` |
| **Complexity** | Medium |
| **Notes** | **Overlaps with existing `git-workflow` and `github-prs` skills**, but gstack /ship adds a structured multi-step release ceremony (pre-flight checks, test verification, coverage audit, PR creation with structured description, auto-invoke documentation update). The ceremony and checklist are the value — extract as a skill that the `implementer` or a new `release-engineer` role can consume. Strip gstack's `document-release` auto-invocation (handled by StrawPot orchestration). |

#### 12. /document-release → Skill: `release-documentation`

| Field | Value |
|---|---|
| **Gstack name** | `/document-release` |
| **What it does** | Post-ship documentation sync. Updates README, API docs, changelogs to match shipped code. |
| **StrawPot type** | Skill |
| **Proposed name** | `release-documentation` |
| **Dependencies** | Skills: `completeness-principle` |
| **Complexity** | Low |
| **Notes** | Methodology for systematically finding and updating all documentation after a release. The doc-hunting checklist (README, API docs, inline comments, changelog, migration guides) is the core value. Complements existing `docs-writer` role — this skill could be consumed by `docs-writer` for post-release documentation sweeps. |

### Reflection Phase

#### 13. /retro → Role: `retro-facilitator`

| Field | Value |
|---|---|
| **Gstack name** | `/retro` |
| **What it does** | Engineering manager retrospective. Analyzes git history, produces per-person breakdowns, identifies patterns, suggests improvements. |
| **StrawPot type** | Role |
| **Proposed name** | `retro-facilitator` |
| **Dependencies** | Skills: `completeness-principle` |
| **Complexity** | Medium |
| **Notes** | Persona-driven (engineering manager running a retro). Analyzes git log, PR history, and commit patterns to produce structured retrospective. The analysis methodology (velocity trends, quality signals, collaboration patterns, improvement actions) is the core value. No external tool dependencies beyond git. |

### Browser Infrastructure

#### 14. /browse → Skill: `browse`

| Field | Value |
|---|---|
| **Gstack name** | `/browse` |
| **What it does** | Headless Chromium browser interface. 50+ commands for navigation, interaction, inspection, screenshots. |
| **StrawPot type** | Skill |
| **Proposed name** | `browse` |
| **Dependencies** | None (foundational) |
| **Complexity** | **Very High** |
| **Notes** | This is the most complex migration item. The browse skill in gstack is backed by a compiled Bun binary (~58MB) that wraps Playwright. **The skill definition is just documentation** — the actual capability comes from the binary. Migration options: (A) Vendor the gstack browse binary and create a skill that documents its usage, (B) Build a new StrawPot-native browser tool using Playwright directly, (C) Use an MCP-based browser tool. **Recommendation: Option A for Phase 1** — vendor the binary, create a `browse` skill with command reference. Option B for long-term. The skill content (QA patterns, command reference, snapshot system) migrates cleanly. |

#### 15. /setup-browser-cookies → Skill: `browser-cookies`

| Field | Value |
|---|---|
| **Gstack name** | `/setup-browser-cookies` |
| **What it does** | Import session cookies from real browser for authenticated testing. |
| **StrawPot type** | Skill |
| **Proposed name** | `browser-cookies` |
| **Dependencies** | Skills: `browse` |
| **Complexity** | Low |
| **Notes** | Procedural knowledge for setting up authenticated browser sessions. The cookie picker UI and direct import commands are part of the browse binary. This skill just documents the workflow. Straightforward migration — document the setup flow in StrawPot skill format. |

### Power Tools

#### 16. /codex → **SKIP**

| Field | Value |
|---|---|
| **Gstack name** | `/codex` |
| **What it does** | Multi-AI second opinion via OpenAI Codex CLI. |
| **StrawPot type** | Skip |
| **Proposed name** | N/A |
| **Complexity** | N/A |
| **Notes** | StrawPot is agent-framework-agnostic but this skill is tightly coupled to OpenAI's Codex CLI binary. The concept of "second opinion" is interesting but the implementation is vendor-specific. **Skip for now.** If needed later, StrawPot could implement a generic "second-opinion" skill that delegates to any available LLM via denden. |

#### 17. /careful → Skill: `safety-guardrails`

| Field | Value |
|---|---|
| **Gstack name** | `/careful` |
| **What it does** | Pre-flight warning before destructive commands (rm -rf, DROP TABLE, force-push). |
| **StrawPot type** | Skill |
| **Proposed name** | `safety-guardrails` |
| **Dependencies** | None |
| **Complexity** | Medium |
| **Notes** | Gstack implements this as a PreToolUse hook — a Claude Code-specific mechanism. StrawPot skills don't have hook infrastructure. **Two options:** (A) Document the dangerous command patterns as a skill that roles reference (knowledge-based guardrails), (B) Implement as a StrawPot-level hook if the platform supports it. **Recommendation: Option A** — create a skill listing dangerous patterns and safe alternatives. Roles that depend on it will have the knowledge to avoid destructive commands. Overlaps with existing StrawPot `security-guidance` skill. |

#### 18-20. /freeze, /guard, /unfreeze → **SKIP**

| Field | Value |
|---|---|
| **Gstack name** | `/freeze`, `/guard`, `/unfreeze` |
| **What they do** | Directory-scoped edit locks via PreToolUse hooks. |
| **StrawPot type** | Skip |
| **Complexity** | N/A |
| **Notes** | These are Claude Code hook mechanisms for restricting file edits to specific directories. StrawPot doesn't have an equivalent hook system, and the concept is less relevant in a delegated-role architecture where each role operates on specific tasks. **Skip.** If scope restriction is needed, it's better implemented at the StrawPot platform level (e.g., worktree isolation). |

#### 21. /gstack-upgrade → **SKIP**

| Field | Value |
|---|---|
| **Gstack name** | `/gstack-upgrade` |
| **What it does** | Self-updater for gstack installation. |
| **StrawPot type** | Skip |
| **Complexity** | N/A |
| **Notes** | Gstack-specific infrastructure. StrawPot has its own update mechanisms via StrawHub (`strawhub-cli` skill). No value in migrating. |

---

## New Skills to Create

These skills don't exist in gstack as standalone components but need to be extracted to support the migrated roles.

### 1. `completeness-principle`

| Field | Value |
|---|---|
| **Source** | Gstack's "Boil the Lake" principle (embedded in every SKILL.md) |
| **Purpose** | Shared methodology for completeness-driven work. Classifies tasks as "lake" (bounded, boilable — do 100%) vs "ocean" (unbounded — flag scope). Includes effort estimation in both human-team and AI-agent time scales. |
| **Why extract** | Every gstack persona references this principle. Extracting it as a shared skill avoids duplication across 8+ roles and lets any StrawPot role opt into completeness-driven behavior. |
| **Dependencies** | None |
| **Complexity** | Low |

### 2. `browse`

| Field | Value |
|---|---|
| **Source** | Gstack browse binary + BROWSER.md + browse/SKILL.md |
| **Purpose** | Complete command reference and usage patterns for the headless browser. Covers navigation, interaction, inspection, screenshots, snapshots, cookie management, and multi-tab workflows. |
| **Why extract** | Multiple roles need browser access (browser-qa-engineer, visual-qa-reviewer, design-system-architect). A single skill provides the command reference. |
| **Dependencies** | Tool: `browse` binary (vendored or built) |
| **Complexity** | High (requires binary distribution) |

### 3. `atomic-commits`

| Field | Value |
|---|---|
| **Source** | Pattern used across gstack /review, /qa, /design-review |
| **Purpose** | Methodology for making small, focused commits during fix workflows. One logical change per commit, descriptive message, verify before committing. |
| **Why extract** | Multiple roles (debugger, browser-qa-engineer, visual-qa-reviewer) need this pattern. Currently embedded in `git-workflow` skill but gstack adds the "fix loop" pattern (find → fix → commit → verify → next). |
| **Dependencies** | Skills: `git-workflow` |
| **Complexity** | Low |
| **Notes** | May be better as an addition to the existing `git-workflow` skill rather than a standalone skill. Evaluate during implementation. |

---

## Dependency Graph

```
                    ┌─────────────────────┐
                    │ completeness-       │
                    │ principle            │
                    └──────┬──────────────┘
                           │ (consumed by most roles)
           ┌───────────────┼───────────────────────────────┐
           │               │               │               │
           ▼               ▼               ▼               ▼
    ┌──────────┐   ┌──────────────┐  ┌──────────┐  ┌──────────────┐
    │ product- │   │ strategic-   │  │ debugger │  │ retro-       │
    │ advisor  │   │ reviewer     │  │          │  │ facilitator  │
    └──────────┘   └──────────────┘  └──────────┘  └──────────────┘

                    ┌─────────────────────┐
                    │ browse (skill)      │
                    │ + browse binary     │
                    └──────┬──────────────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
    ┌──────────────┐ ┌────────────┐ ┌──────────────────┐
    │ browser-qa-  │ │ visual-qa- │ │ design-system-   │
    │ engineer     │ │ reviewer   │ │ architect         │
    └──────────────┘ └────────────┘ └──────────────────┘

    ┌──────────────────┐     ┌───────────────────┐
    │ engineering-     │     │ design-review-     │
    │ principles       │     │ methodology        │
    │ (existing)       │     │                    │
    └───────┬──────────┘     └────────┬───────────┘
            │                         │
            ▼                         ▼
    ┌──────────────────┐     ┌────────────────┐
    │ architecture-    │     │ visual-qa-     │
    │ review           │     │ reviewer       │
    └──────────────────┘     └────────────────┘

    ┌────────────────┐  ┌──────────────┐  ┌──────────────────┐
    │ git-workflow   │  │ github-prs   │  │ release-         │
    │ (existing)     │  │ (existing)   │  │ documentation    │
    └───────┬────────┘  └──────┬───────┘  └──────────────────┘
            │                  │                    │
            ▼                  ▼                    ▼
    ┌────────────────────────────────┐     ┌──────────────┐
    │ release-workflow               │     │ docs-writer  │
    │                                │     │ (existing)   │
    └────────────────────────────────┘     └──────────────┘
```

### Full Dependency Matrix

| Component | Type | Depends On (Skills) | Depends On (Roles) |
|---|---|---|---|
| `completeness-principle` | Skill | — | — |
| `browse` | Skill | — | — |
| `browser-cookies` | Skill | `browse` | — |
| `design-review-methodology` | Skill | — | — |
| `architecture-review` | Skill | `engineering-principles` | — |
| `release-workflow` | Skill | `git-workflow`, `github-prs`, `completeness-principle` | — |
| `release-documentation` | Skill | `completeness-principle` | — |
| `safety-guardrails` | Skill | — | — |
| `product-advisor` | Role | `completeness-principle` | — |
| `strategic-reviewer` | Role | `completeness-principle` | — |
| `debugger` | Role | `completeness-principle`, `engineering-principles` | — |
| `retro-facilitator` | Role | `completeness-principle` | — |
| `browser-qa-engineer` | Role | `browse`, `browser-cookies`, `git-workflow`, `completeness-principle` | — |
| `visual-qa-reviewer` | Role | `browse`, `design-review-methodology`, `git-workflow`, `completeness-principle` | — |
| `design-system-architect` | Role | `browse`, `design-review-methodology`, `completeness-principle`, `frontend-design` | — |

---

## Migration Phases

### Phase 0: Foundation Skills (No dependencies — everything else builds on these)

**Deliverables:**
1. `completeness-principle` skill — Extract from gstack preamble
2. `safety-guardrails` skill — Adapt from /careful

**Effort:** ~2 hours
**Prerequisite for:** All subsequent phases

---

### Phase 1: Standalone Roles (No browser dependency)

**Deliverables:**
1. `product-advisor` role — Adapt from /office-hours
2. `strategic-reviewer` role — Adapt from /plan-ceo-review
3. `debugger` role — Adapt from /investigate
4. `retro-facilitator` role — Adapt from /retro

**Effort:** ~4 hours
**Prerequisite for:** None (these are independently useful)

---

### Phase 2: Review & Ship Skills

**Deliverables:**
1. `architecture-review` skill — Adapt from /plan-eng-review
2. `design-review-methodology` skill — Adapt from /plan-design-review
3. `release-workflow` skill — Adapt from /ship
4. `release-documentation` skill — Adapt from /document-release
5. Enhance existing `code-review` skill with gstack /review patterns (PR to skills repo)

**Effort:** ~4 hours
**Prerequisite for:** Phase 3 (visual-qa-reviewer needs design-review-methodology)

---

### Phase 3: Browser Infrastructure

**Deliverables:**
1. `browse` skill — Command reference + binary vendoring strategy
2. `browser-cookies` skill — Adapt from /setup-browser-cookies
3. Decide and implement binary distribution (vendor vs build vs MCP)

**Effort:** ~8 hours (most time spent on binary distribution)
**Prerequisite for:** Phase 4

**Key decision:** How to distribute the browse binary.

| Option | Pros | Cons |
|---|---|---|
| **A. Vendor gstack binary** | Fast, proven, works today | ~58MB binary, macOS-only, tied to gstack releases |
| **B. Build StrawPot-native Playwright wrapper** | Full control, cross-platform | Significant engineering effort, duplicating work |
| **C. Use existing MCP browser tools** | No binary to maintain | Different command interface, may lack gstack's snapshot system |
| **D. Skill-only (no binary)** | Zero infrastructure | Roles can't actually use the browser — just methodology |

**Recommendation:** Start with **Option A** (vendor binary) for immediate usability, plan **Option C** (MCP) as the long-term path since StrawPot already supports MCP tools.

---

### Phase 4: Browser-Dependent Roles

**Deliverables:**
1. `browser-qa-engineer` role — Adapt from /qa + /qa-only
2. `visual-qa-reviewer` role — Adapt from /design-review
3. `design-system-architect` role — Adapt from /design-consultation

**Effort:** ~6 hours
**Prerequisite for:** None (final phase)

---

### Phase 5: Integration & Publishing

**Deliverables:**
1. Publish all skills to StrawHub
2. Publish all roles to StrawHub
3. Create eval tests for each component
4. Update strawpot_gstack README with installation instructions

**Effort:** ~4 hours

---

### Phase Summary

| Phase | Components | Effort | Cumulative |
|---|---|---|---|
| 0: Foundation | 2 skills | ~2h | 2h |
| 1: Standalone Roles | 4 roles | ~4h | 6h |
| 2: Review & Ship | 4 skills + 1 enhancement | ~4h | 10h |
| 3: Browser Infra | 2 skills + binary strategy | ~8h | 18h |
| 4: Browser Roles | 3 roles | ~6h | 24h |
| 5: Integration | Publishing + evals | ~4h | 28h |

**Total estimated effort: ~28 hours**

---

## What NOT to Migrate

| Component | Reason |
|---|---|
| `/gstack-upgrade` | Gstack-specific self-updater. StrawPot uses StrawHub for versioning. |
| `/codex` | Tightly coupled to OpenAI Codex CLI binary. Vendor-specific. |
| `/freeze` | Claude Code PreToolUse hook. No StrawPot equivalent. Worktree isolation serves the same purpose. |
| `/guard` | Combination of /careful + /freeze. /careful is migrated as safety-guardrails; /freeze is skipped. |
| `/unfreeze` | Counterpart to /freeze. Skipped with /freeze. |
| Telemetry system | Gstack's Supabase-backed telemetry (supabase/ directory). StrawPot has its own observability. |
| Preamble system | Auto-generated skill headers with session tracking, ELI16 mode, contributor detection. Platform-specific. |
| Template system (.tmpl) | Gstack's SKILL.md template generation. StrawPot skills are authored directly. |
| Conductor integration | Gstack's conductor.json workspace config. Not applicable to StrawPot. |
| Multi-agent CLI compat (.agents/) | Generated skill definitions for Codex/Gemini CLI. StrawPot handles multi-agent natively. |
| Eval infrastructure (test/) | Gstack's tiered test system. StrawPot has its own eval framework via skill-evaluator/role-evaluator. |

---

## Existing StrawPot Overlap

### Direct Overlaps (Enhance, Don't Duplicate)

| Gstack Component | Existing StrawPot Component | Resolution |
|---|---|---|
| `/review` (Staff Engineer PR review) | `code-reviewer` role + `code-review` skill | **Enhance** `code-review` skill with gstack's review checklist items (completeness audit, atomic fix patterns). Don't create a new reviewer role. |
| `/ship` (release workflow) | `git-workflow` skill + `github-prs` skill + `implementer` role | **Create** `release-workflow` skill as a ceremony layer on top of existing skills. The `implementer` role or a new release-engineer role can consume it. |
| `/investigate` (debugging) | No direct equivalent | **Create** `debugger` role. Fills a gap — StrawPot has no systematic debugging specialist. |
| `/qa` (browser QA) | `qa-engineer` role | **Create** `browser-qa-engineer` as a separate role. Existing `qa-engineer` writes test code; the new role tests running applications with a browser. They're complementary. |
| `/document-release` (docs sync) | `docs-writer` role | **Create** `release-documentation` skill consumed by existing `docs-writer`. Don't create a new role. |
| `/plan-eng-review` (arch review) | `engineering-principles` skill + `implementation-planner` role | **Create** `architecture-review` skill that extends `engineering-principles` with a structured review checklist. |
| `/careful` (safety guardrails) | `security-guidance` skill | **Create** `safety-guardrails` skill. Different focus — `security-guidance` is about secure coding, `safety-guardrails` is about preventing destructive CLI commands. |

### No Overlap (New Capabilities)

| Gstack Component | StrawPot Gap Filled |
|---|---|
| `/office-hours` | No product advisory/diagnostic capability |
| `/plan-ceo-review` | No strategic review capability |
| `/design-consultation` | No design system creation capability (frontend-design is styling, not system architecture) |
| `/design-review` | No visual QA capability |
| `/retro` | No retrospective/reflection capability |
| Browser infrastructure | No headless browser capability (MCP tools exist but aren't integrated as a skill) |

---

## Migration Checklist Template

For each component, follow this checklist during migration:

```markdown
- [ ] Read the original gstack SKILL.md thoroughly
- [ ] Identify what to keep (domain expertise, checklists, workflows)
- [ ] Identify what to strip (preamble, telemetry, gstack-specific commands)
- [ ] Write StrawPot YAML frontmatter (name, description, dependencies, tools, env)
- [ ] Write skill/role body following StrawPot conventions:
  - [ ] Progressive disclosure structure
  - [ ] Imperative form ("Do X", not "You should do X")
  - [ ] "What you are not" section (roles only)
  - [ ] Triggering description in the description field
- [ ] Verify dependencies exist (or are being created in same/earlier phase)
- [ ] Test with a StrawPot agent (manual smoke test)
- [ ] Create eval test (role-evaluator or skill-evaluator)
- [ ] Publish to StrawHub
```

---

## Orchestration & Inter-Role Delegation

### The `gstack-ceo` Role

`gstack-ceo` is the entry-point orchestrator for all gstack workflows. It maps incoming tasks to the correct **starting role** in the sprint cycle (Think → Plan → Build → Review → Test → Ship → Reflect) and delegates with full context. Unlike a hub-and-spoke model, gstack-ceo does **not** mediate every handoff — it routes to the starting point, and roles handle downstream handoffs autonomously through their own delegation targets.

**Location:** `roles/gstack-ceo/ROLE.md`

**Dependencies:**
- Skills: `completeness-principle`
- Roles: `"*"` (discovers all installed roles at runtime)

**Routing logic:** gstack-ceo reads every installed `ROLE.md`, matches the user's request to a sprint phase, and delegates to the starting role for that phase. The starting role carries the work forward through its natural delegation chain.

### Delegation Graph

Each gstack role declares which roles it can delegate to directly. Arrows show delegation direction (delegator → delegate). Existing StrawPot roles are marked with `(existing)`.

```
                            ┌─────────────┐
                            │  gstack-ceo │  (entry-point orchestrator)
                            └──────┬──────┘
                                   │ routes to starting role
          ┌────────────┬───────────┼───────────┬──────────────┬──────────────┐
          ▼            ▼           ▼           ▼              ▼              ▼
   ┌──────────────┐ ┌─────────┐ ┌───────────┐ ┌────────────┐ ┌────────────┐ ┌─────────┐
   │  product-    │ │debugger │ │  design-  │ │  browser-  │ │  visual-   │ │  retro- │
   │  advisor     │ │         │ │  system-  │ │  qa-       │ │  qa-       │ │  facili-│
   │              │ │         │ │  architect│ │  engineer  │ │  reviewer  │ │  tator  │
   └──────┬───────┘ └────┬────┘ └─────┬─────┘ └─────┬──────┘ └─────┬──────┘ └─────────┘
          │               │           │              │              │         (standalone)
          ▼               │           ▼              │              │
   ┌──────────────┐       │    ┌────────────┐        │              │
   │  strategic-  │       │    │  visual-   │◄───────┼──────────────┘
   │  reviewer    │       │    │  qa-       │        │
   └──────┬───────┘       │    │  reviewer  │        │
          │               │    └────────────┘        │
          ▼               ▼           │              │
   ┌──────────────────────────────────┼──────────────┘
   │       Existing StrawPot Roles    │
   │  ┌────────────────┐  ┌──────────┴───┐  ┌──────────────┐  ┌────────────┐
   │  │implementation- │  │ implementer  │  │ code-reviewer │  │ docs-writer│
   │  │planner         │  │              │  │              │  │            │
   │  └────────────────┘  └──────────────┘  └──────────────┘  └────────────┘
   │  ┌────────────────┐  ┌──────────────┐
   │  │ qa-engineer    │  │ ai-employee  │  (fallback)
   │  └────────────────┘  └──────────────┘
   └──────────────────────────────────────────────────────────────────────────
```

### Per-Role Delegation Targets

Each gstack role's `metadata.strawpot.dependencies.roles` field declares its delegation targets. These are the roles it can hand off to directly — no routing through gstack-ceo required.

#### `product-advisor`
| Delegates to | When | What it passes |
|---|---|---|
| `strategic-reviewer` | After producing a design doc | Design doc with problem definition, user profile, solution approaches |
| `implementation-planner` (existing) | For small/obvious features that don't need CEO review | Design doc + recommendation to skip strategic review |

#### `strategic-reviewer`
| Delegates to | When | What it passes |
|---|---|---|
| `implementation-planner` (existing) | After approving/revising the plan | Reviewed plan with scope decision, 10-star opportunities, and go/no-go |
| `design-system-architect` | When plan requires a new design system | Approved scope + design requirements extracted from plan |
| `product-advisor` | When strategic review reveals the problem isn't well-defined | Feedback on what needs rethinking, specific questions to answer |

#### `debugger`
| Delegates to | When | What it passes |
|---|---|---|
| `implementer` (existing) | After identifying root cause | Root cause analysis, affected files, proposed fix approach |
| `browser-qa-engineer` | When root cause needs verification in a running app | Hypothesis to verify, specific URLs/flows to test |
| `qa-engineer` (existing) | When root cause needs verification via test code | Test scenarios that reproduce the bug |

#### `browser-qa-engineer`
| Delegates to | When | What it passes |
|---|---|---|
| `implementer` (existing) | After finding bugs (in fix mode) | Bug report with reproduction steps, screenshots, severity |
| `debugger` | When a bug's root cause is unclear | Symptoms observed, URLs, screenshots, browser console output |
| `code-reviewer` (existing) | After making atomic fix commits | Branch/diff of fixes for review |
| `visual-qa-reviewer` | When visual inconsistencies are found during functional QA | Screenshots + specific visual issues to audit |

#### `visual-qa-reviewer`
| Delegates to | When | What it passes |
|---|---|---|
| `implementer` (existing) | After visual audit finds code-level fixes needed | Visual issue list with screenshots, CSS/component targets |
| `design-system-architect` | When issues are systemic (token inconsistency, pattern violations) | Audit findings that indicate design system gaps |
| `code-reviewer` (existing) | After making atomic fix commits | Branch/diff of visual fixes for review |

#### `design-system-architect`
| Delegates to | When | What it passes |
|---|---|---|
| `visual-qa-reviewer` | After creating/updating the design system | Design system artifacts (tokens, components, patterns) for validation |
| `implementer` (existing) | When design system needs to be implemented in code | Design specs with token values, component API, usage examples |
| `docs-writer` (existing) | When design system documentation needs creation | Design system reference material for documentation |

#### `retro-facilitator`
| Delegates to | When | What it passes |
|---|---|---|
| *(none)* | — | Standalone role. Reports retrospective findings back to delegator. |

### Workflow Sequences

These show the typical delegation chains through the distributed model. Each role delegates directly to the next — gstack-ceo only handles initial routing.

#### Feature Request Flow
```
User → gstack-ceo → product-advisor
                        ↓ (design doc)
                     strategic-reviewer
                        ↓ (approved plan)
                     implementation-planner (existing)
                        ↓ (sub-issues)
                     implementer (existing)
                        ↓ (code changes)
                     code-reviewer (existing)
                        ↓ (approved)
                     browser-qa-engineer
                        ↓ (tested, bugs fixed)
                     visual-qa-reviewer
                        ↓ (visual audit passed)
                     implementer (existing) → ship
                        ↓ (released)
                     docs-writer (existing)
```

#### Bug Fix Flow
```
User → gstack-ceo → debugger
                        ↓ (root cause + fix approach)
                     implementer (existing)
                        ↓ (fix committed)
                     code-reviewer (existing)
                        ↓ (approved)
                     qa-engineer (existing) or browser-qa-engineer
                        ↓ (verified)
                     implementer (existing) → ship
```

#### Design System Flow
```
User → gstack-ceo → product-advisor (or direct to design-system-architect)
                        ↓ (design requirements)
                     design-system-architect
                        ↓ (design system artifacts)
                     visual-qa-reviewer
                        ↓ (validated)
                     implementer (existing) → implement system in code
                        ↓ (implemented)
                     docs-writer (existing) → document the system
```

#### Visual Bug Flow
```
User → gstack-ceo → visual-qa-reviewer
                        ↓ (audit findings)
                     ├→ implementer (existing) → code fixes
                     └→ design-system-architect → systemic fixes
                        ↓ (fixes applied)
                     visual-qa-reviewer → re-verify
                        ↓ (passed)
                     code-reviewer (existing) → review fixes
```

#### Retrospective Flow
```
User → gstack-ceo → retro-facilitator
                        ↓ (retrospective report)
                     gstack-ceo → present to user
```

### Integration with Existing StrawPot Roles

Gstack roles don't replace existing StrawPot roles — they extend the team by adding sprint-cycle orchestration and specialized capabilities on top.

| Existing StrawPot Role | How Gstack Roles Use It |
|---|---|
| `implementer` | Primary build/ship executor. Receives work from `debugger`, `browser-qa-engineer`, `visual-qa-reviewer`, `design-system-architect`. Handles the Ship phase with `release-workflow` skill. |
| `implementation-planner` | Receives approved plans from `strategic-reviewer` or `product-advisor`. Locks architecture and creates sub-issues. |
| `code-reviewer` | Reviews fix commits from `browser-qa-engineer` and `visual-qa-reviewer`. Also engaged directly by gstack-ceo for Review phase. |
| `qa-engineer` | Complements `browser-qa-engineer` — writes/runs test code while browser-qa tests running applications. `debugger` delegates verification to either based on the bug type. |
| `docs-writer` | Handles documentation at end of Ship phase. `design-system-architect` delegates design system docs to it. |
| `ai-employee` | Fallback for tasks that don't fit any specialist role. |

### Avoiding Circular Delegation

The delegation graph is acyclic by design. Two potential cycles exist and are handled:

1. **`strategic-reviewer` → `product-advisor`**: This is an intentional back-channel for when strategic review reveals the problem definition is wrong. It's a revision loop, not infinite recursion — `product-advisor` produces a revised design doc and re-delegates to `strategic-reviewer`. The depth limit in denden (default `max_depth`) prevents infinite loops. In practice, one revision is sufficient.

2. **`browser-qa-engineer` ↔ `debugger`**: QA finds a bug → delegates to debugger → debugger finds root cause → delegates to implementer (not back to QA). The chain breaks because debugger routes fixes to implementer, not back to browser-qa-engineer. Re-verification is handled by browser-qa-engineer being re-invoked by the workflow, not by debugger delegating to it.

3. **`visual-qa-reviewer` ↔ `design-system-architect`**: Visual QA finds systemic issues → delegates to design-system-architect → architect fixes the system → delegates to visual-qa-reviewer for re-validation. The depth limit in denden prevents infinite ping-pong. In practice, one round of fix + re-validate is expected.

### Summary: Role Dependency Matrix (Updated with Delegation)

| Role | Skill Dependencies | Delegation Targets (Roles) |
|---|---|---|
| `gstack-ceo` | *(none — pure orchestrator)* | `"*"` (all roles) |
| `product-advisor` | `completeness-principle` | `strategic-reviewer`, `implementation-planner` |
| `strategic-reviewer` | `completeness-principle` | `implementation-planner`, `design-system-architect`, `product-advisor` |
| `debugger` | `completeness-principle`, `engineering-principles` | `implementer`, `browser-qa-engineer`, `qa-engineer` |
| `browser-qa-engineer` | `browse`, `browser-cookies`, `git-workflow`, `completeness-principle` | `implementer`, `debugger`, `code-reviewer`, `visual-qa-reviewer` |
| `visual-qa-reviewer` | `browse`, `design-review-methodology`, `git-workflow`, `completeness-principle` | `implementer`, `design-system-architect`, `code-reviewer` |
| `design-system-architect` | `browse`, `design-review-methodology`, `completeness-principle`, `frontend-design` | `visual-qa-reviewer`, `implementer`, `docs-writer` |
| `retro-facilitator` | `completeness-principle` | *(none)* |

---

## Open Questions

1. **Browse binary distribution**: Should we vendor the gstack binary, build our own, or adopt MCP browser tools? This is the highest-impact architectural decision.

2. **Completeness principle adoption**: Should `completeness-principle` be a default dependency for all StrawPot roles (like `engineering-principles`), or opt-in per role?

3. **Product advisor scope**: Is `product-advisor` too YC-specific for general StrawPot users? Should we generalize the six forcing questions?

4. **Retro data sources**: The `retro-facilitator` analyzes git history. Should it also integrate with StrawPot session data (via `strawpot-sessions` skill) for AI-agent-specific retrospectives?

5. **Existing skill PRs**: Several migrations enhance existing skills (`code-review`, `git-workflow`). Should these be PRs to the skills repo, or forked versions in strawpot_gstack?
