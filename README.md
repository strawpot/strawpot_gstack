# strawpot_gstack

A collection of StrawPot roles and skills adapted from [garrytan/gstack](https://github.com/garrytan/gstack) — a sprint-cycle framework for AI-powered software development.

Implements the **Think → Plan → Build → Review → Test → Ship → Reflect** cycle using specialized AI agents orchestrated through StrawPot.

## Entry Point

The `gstack-ceo` role is the orchestrator entry point. It maps the sprint cycle phases to role delegation sequences, coordinating all other roles and skills to drive a project from idea to shipped code.

## Roles

| Role | Description |
|------|-------------|
| `gstack-ceo` | Orchestrator — drives the full sprint cycle and delegates to specialized roles |
| `product-advisor` | Provides product strategy and prioritization guidance |
| `design-system-architect` | Defines and maintains design system standards |
| `debugger` | Diagnoses and resolves software defects |
| `strategic-reviewer` | Reviews code and architecture decisions for strategic alignment |
| `retro-facilitator` | Facilitates sprint retrospectives and captures learnings |
| `browser-qa-engineer` | Runs browser-based QA testing workflows |
| `visual-qa-reviewer` | Reviews UI for visual correctness and design fidelity |

## Skills

| Skill | Description |
|-------|-------------|
| `completeness-principle` | Ensures thorough coverage across all deliverables |
| `safety-guardrails` | Enforces safety checks and risk mitigation patterns |
| `architecture-review` | Structured methodology for evaluating architecture decisions |
| `design-review-methodology` | Framework for systematic design reviews |
| `release-workflow` | Manages release processes and deployment pipelines |
| `release-documentation` | Generates and maintains release notes and changelogs |
| `browse` | Browser automation for testing and verification |
| `browser-cookies` | Cookie and session management for browser-based testing |

## Installation

These roles and skills are installed via [strawhub](https://github.com/strawpot/strawhub):

```bash
strawhub install strawpot/strawpot_gstack
```

## Credits

Adapted from [garrytan/gstack](https://github.com/garrytan/gstack) by Garry Tan — an opinionated AI-agent sprint-cycle framework.

## License

MIT — see [LICENSE](LICENSE) for details.
