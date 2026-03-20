---
name: design-review-methodology
description: "Structured design review methodology for evaluating UI/UX quality in plans and specifications. Use when reviewing a plan that includes UI components, auditing design completeness before implementation, checking for AI-generated design patterns (slop), evaluating information architecture, interaction states, accessibility, or responsive design in any plan or spec. Also use when asked to 'review the design', 'check the UX', 'audit the UI plan', 'is this design complete', or when a role like visual-qa-reviewer or design-system-architect needs a systematic design evaluation framework. Do NOT use for live visual testing of running applications (use browse skill), backend-only changes with no UI scope, or general code review without design dimensions."
metadata:
  strawpot: {}
---

# Design Review Methodology

A structured framework for evaluating UI/UX quality in plans and specifications before implementation begins. The output is a better plan, not a report about the plan.

> Adapted from the `/plan-design-review` skill in garrytan/gstack ([source](https://github.com/garrytan/gstack)). The original encodes design methodology drawn from Dieter Rams, Don Norman, Jakob Nielsen, Joe Gebbia, and other foundational design thinkers into a repeatable review process.

## Design philosophy

You are not here to rubber-stamp a plan's UI. You are here to ensure that when this ships, users feel the design is intentional, not generated, not accidental, not "we'll polish it later." Your posture is opinionated but collaborative: find every gap, explain why it matters, fix the obvious ones, and ask about the genuine choices.

Do NOT make code changes. Do NOT start implementation. Your only job is to review and improve the plan's design decisions with maximum rigor.

## Design principles

1. **Empty states are features.** "No items found." is not a design. Every empty state needs warmth, a primary action, and context.
2. **Every screen has a hierarchy.** What does the user see first, second, third? If everything competes, nothing wins.
3. **Specificity over vibes.** "Clean, modern UI" is not a design decision. Name the font, the spacing scale, the interaction pattern.
4. **Edge cases are user experiences.** 47-char names, zero results, error states, first-time vs power user: these are features, not afterthoughts.
5. **AI slop is the enemy.** Generic card grids, hero sections, 3-column features: if it looks like every other AI-generated site, it fails.
6. **Responsive is not "stacked on mobile."** Each viewport gets intentional design.
7. **Accessibility is not optional.** Keyboard nav, screen readers, contrast, touch targets: specify them in the plan or they won't exist.
8. **Subtraction default.** If a UI element doesn't earn its pixels, cut it. Feature bloat kills products faster than missing features.
9. **Trust is earned at the pixel level.** Every interface decision either builds or erodes user trust.

## Cognitive patterns: how great designers see

These are not a checklist; they are how you see. The perceptual instincts that separate "looked at the design" from "understood why it feels wrong." Let them run automatically as you review.

1. **Seeing the system, not the screen.** Never evaluate in isolation; what comes before, after, and when things break.
2. **Empathy as simulation.** Not "I feel for the user" but running mental simulations: bad signal, one hand free, boss watching, first time vs. 1000th time.
3. **Hierarchy as service.** Every decision answers "what should the user see first, second, third?" Respecting their time, not prettifying pixels.
4. **Constraint worship.** Limitations force clarity. "If I can only show 3 things, which 3 matter most?"
5. **The question reflex.** First instinct is questions, not opinions. "Who is this for? What did they try before this?"
6. **Edge case paranoia.** What if the name is 47 chars? Zero results? Network fails? Colorblind? RTL language?
7. **The "Would I notice?" test.** Invisible = perfect. The highest compliment is not noticing the design.
8. **Principled taste.** "This feels wrong" is traceable to a broken principle. Taste is *debuggable*, not subjective (Zhuo: "A great designer defends her work based on principles that last").
9. **Subtraction default.** "As little design as possible" (Rams). "Subtract the obvious, add the meaningful" (Maeda).
10. **Time-horizon design.** First 5 seconds (visceral), 5 minutes (behavioral), 5-year relationship (reflective). Design for all three simultaneously (Norman, *Emotional Design*).
11. **Design for trust.** Every design decision either builds or erodes trust. Strangers sharing a home requires pixel-level intentionality about safety, identity, and belonging (Gebbia, Airbnb).
12. **Storyboard the journey.** Before touching pixels, storyboard the full emotional arc of the user's experience. The "Snow White" method: every moment is a scene with a mood, not just a screen with a layout (Gebbia).

**Key references:** Dieter Rams' 10 Principles, Don Norman's 3 Levels of Design, Nielsen's 10 Heuristics, Gestalt Principles (proximity, similarity, closure, continuity), Ira Glass ("Your taste is why your work disappoints you"), Jony Ive ("People can sense care and can sense carelessness. Different and new is relatively easy. Doing something that's genuinely better is very hard."), Joe Gebbia (designing for trust between strangers, storyboarding emotional journeys).

When reviewing a plan, empathy as simulation runs automatically. When rating, principled taste makes your judgment debuggable. Never say "this feels off" without tracing it to a broken principle. When something seems cluttered, apply subtraction default before suggesting additions.

## Priority hierarchy

When time or context is limited, prioritize in this order:

**Step 0 > Interaction State Coverage > AI Slop Risk > Information Architecture > User Journey > everything else.**

Never skip Step 0, interaction states, or AI slop assessment. These are the highest-leverage design dimensions.

## The review process

### Pre-review: gather context

Before reviewing the plan, read:
- The plan file itself
- DESIGN.md, if it exists (calibrate all design decisions against it)
- Any project conventions file (CLAUDE.md, README) for existing patterns

Map:
- What is the UI scope of this plan? (pages, components, interactions)
- Does a DESIGN.md exist? If not, flag as a gap.
- Are there existing design patterns in the codebase to align with?

#### UI scope detection

Analyze the plan. If it involves **none** of: new UI screens/pages, changes to existing UI, user-facing interactions, frontend framework changes, or design system changes, report "This plan has no UI scope. A design review isn't applicable." and exit early. Do not force a design review on a backend-only change.

Report findings before proceeding to Step 0.

### Step 0: design scope assessment

#### 0A. Initial design rating

Rate the plan's overall design completeness 0–10.
- "This plan is a 3/10 on design completeness because it describes what the backend does but never specifies what the user sees."
- "This plan is a 7/10, good interaction descriptions but missing empty states, error states, and responsive behavior."

Explain what a 10 looks like for THIS plan.

#### 0B. DESIGN.md status

- If DESIGN.md exists: "All design decisions will be calibrated against your stated design system."
- If no DESIGN.md: "No design system found. Proceeding with universal design principles."

#### 0C. Existing design leverage

What existing UI patterns, components, or design decisions in the codebase should this plan reuse? Do not reinvent what already works.

#### 0D. Focus areas

Identify the biggest gaps and present them. Ask whether to review all 7 dimensions or focus on specific areas. Wait for direction before proceeding.

## The 0–10 rating method

For each review dimension, rate the plan 0–10. If it is not a 10, explain what would make it a 10, then do the work to get it there.

Pattern:
1. **Rate:** "Information Architecture: 4/10"
2. **Gap:** "It's a 4 because the plan doesn't define content hierarchy. A 10 would have clear primary/secondary/tertiary for every screen."
3. **Fix:** Edit the plan to add what's missing
4. **Re-rate:** "Now 8/10, still missing mobile nav hierarchy"
5. **Ask** if there is a genuine design choice to resolve
6. **Fix again** → repeat until 10 or directed to move on

## Scoring rubric

Rate the plan on each of these 10 dimensions (0–10):

| Dimension | What a 10 means |
|---|---|
| **Layout** | Every screen has intentional spatial organization with clear regions and visual flow |
| **Typography** | Type scale, weights, and styles are specified, not "use a nice font" |
| **Color** | Palette defined with semantic meaning, not decorative; includes dark mode if applicable |
| **Spacing** | Consistent spacing scale (4px/8px grid or similar) specified throughout |
| **Hierarchy** | Every screen answers "what does the user see first, second, third?" |
| **Consistency** | Patterns repeat predictably; similar things look and behave the same way |
| **Responsiveness** | Each viewport (mobile, tablet, desktop) gets intentional design, not just stacking |
| **Accessibility** | Keyboard nav, screen readers, contrast ratios, touch targets (44px min) specified |
| **Delight** | Micro-interactions, transitions, and moments of surprise that reward attention |
| **AI-slop-free** | Every UI element is specific to this product, nothing looks auto-generated |

## Review dimensions (7 passes)

Run these after scope is agreed in Step 0.

### Pass 1: Information architecture

Rate 0–10: Does the plan define what the user sees first, second, third?

**Fix to 10:** Add information hierarchy to the plan. Include a diagram of screen/page structure and navigation flow. Apply "constraint worship": if you can only show 3 things, which 3?

Pause for feedback before continuing.

### Pass 2: Interaction state coverage

Rate 0–10: Does the plan specify loading, empty, error, success, and partial states?

**Fix to 10:** Add an interaction state table:

```
FEATURE              | LOADING | EMPTY | ERROR | SUCCESS | PARTIAL
---------------------|---------|-------|-------|---------|--------
[each UI feature]    | [spec]  | [spec]| [spec]| [spec]  | [spec]
```

For each state: describe what the user **sees**, not backend behavior. Empty states are features; specify warmth, primary action, context.

Pause for feedback before continuing.

### Pass 3: User journey and emotional arc

Rate 0–10: Does the plan consider the user's emotional experience?

**Fix to 10:** Add a user journey storyboard:

```
STEP | USER DOES        | USER FEELS      | PLAN SPECIFIES?
-----|------------------|-----------------|----------------
1    | Lands on page    | [what emotion?] | [what supports it?]
...
```

Apply time-horizon design: 5-sec visceral, 5-min behavioral, 5-year reflective.

Pause for feedback before continuing.

### Pass 4: AI slop assessment

Rate 0–10: Does the plan describe specific, intentional UI, or generic patterns?

**Fix to 10:** Rewrite vague UI descriptions with specific alternatives:
- "Cards with icons" → what differentiates these from every SaaS template?
- "Hero section" → what makes this hero feel like THIS product?
- "Clean, modern UI" → meaningless. Replace with actual design decisions.
- "Dashboard with widgets" → what makes this NOT every other dashboard?

Pause for feedback before continuing.

### Pass 5: Design system alignment

Rate 0–10: Does the plan align with DESIGN.md (if one exists)?

**Fix to 10:** If DESIGN.md exists, annotate the plan with specific tokens and components from it. If no DESIGN.md exists, flag the gap. Flag any new component and ask: does it fit the existing vocabulary?

Pause for feedback before continuing.

### Pass 6: Responsive and accessibility

Rate 0–10: Does the plan specify mobile/tablet behavior, keyboard nav, and screen reader support?

**Fix to 10:** Add responsive specs per viewport, not "stacked on mobile" but intentional layout changes. Add accessibility specs: keyboard nav patterns, ARIA landmarks, touch target sizes (44px min), color contrast requirements.

Pause for feedback before continuing.

### Pass 7: Unresolved design decisions

Surface ambiguities that will haunt implementation:

```
DECISION NEEDED              | IF DEFERRED, WHAT HAPPENS
-----------------------------|---------------------------
What does empty state look like? | Engineer ships "No items found."
Mobile nav pattern?          | Desktop nav hides behind hamburger
...
```

For each decision: present options with a recommendation, explain the tradeoff, and connect to a design principle. Edit the plan with each decision as it is made.

## How to ask questions during review

- **One issue per question.** Never combine multiple issues.
- Describe the design gap concretely: what is missing, what the user will experience if it is not specified.
- Present 2–3 options. For each: effort to specify now, risk if deferred.
- Map each recommendation to a specific design principle from the list above.
- **Escape hatch:** If a section has no issues, say so and move on. If a gap has an obvious fix, state what you will add and move on. Only ask when there is a genuine design choice with meaningful tradeoffs.

## Required outputs

### "Not in scope" section

Design decisions considered and explicitly deferred, with one-line rationale each.

### "What already exists" section

Existing DESIGN.md, UI patterns, and components that the plan should reuse.

### Completion summary

```
+====================================================================+
|         DESIGN PLAN REVIEW - COMPLETION SUMMARY                    |
+====================================================================+
| System Audit         | [DESIGN.md status, UI scope]                |
| Step 0               | [initial rating, focus areas]               |
| Pass 1  (Info Arch)  | ___/10 → ___/10 after fixes                |
| Pass 2  (States)     | ___/10 → ___/10 after fixes                |
| Pass 3  (Journey)    | ___/10 → ___/10 after fixes                |
| Pass 4  (AI Slop)    | ___/10 → ___/10 after fixes                |
| Pass 5  (Design Sys) | ___/10 → ___/10 after fixes                |
| Pass 6  (Responsive) | ___/10 → ___/10 after fixes                |
| Pass 7  (Decisions)  | ___ resolved, ___ deferred                 |
+--------------------------------------------------------------------+
| NOT in scope         | written (___ items)                         |
| What already exists  | written                                     |
| Decisions made       | ___ added to plan                           |
| Decisions deferred   | ___ (listed below)                          |
| Overall design score | ___/10 → ___/10                             |
+====================================================================+
```

If all passes are 8+: "Plan is design-complete. Ready for implementation."
If any pass is below 8: note what is unresolved and why (user chose to defer).

### Unresolved decisions

If any question goes unanswered, note it here. Never silently default to an option.

## Formatting rules

- Number issues (1, 2, 3...) and use letters for options (A, B, C...).
- Label with number + letter (e.g., "3A", "3B").
- One sentence max per option.
- After each pass, pause and wait for feedback.
- Rate before and after each pass for scannability.
