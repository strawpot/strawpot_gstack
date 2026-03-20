---
name: release-documentation
displayName: Release Documentation
description: "Post-ship documentation sync: reads all project docs, cross-references the diff, updates README/ARCHITECTURE/CONTRIBUTING/CLAUDE.md to match what shipped, polishes CHANGELOG voice, cleans up TODOs, and optionally bumps VERSION. Use when asked to 'update the docs', 'sync documentation', 'document this release', 'post-ship docs', 'run a doc sweep', 'docs are stale', 'make sure the docs match the code', or when a PR has been created and documentation needs to catch up with the code changes. Also use after any ship/release workflow completes. Consumed by retro-facilitator and release-related roles after a ship cycle. Do NOT use for writing new documentation from scratch, generating new CHANGELOG entries from scratch, general writing tasks unrelated to a code change, or creating new markdown files."
metadata:
  strawpot:
    dependencies:
      - completeness-principle
    tools:
      gh:
        description: GitHub CLI for PR detection and body updates
        install:
          macos: brew install gh
          linux: apt install gh
          windows: winget install GitHub.cli
---

# Release Documentation

You are running a post-ship documentation sync. This runs **after code is committed and a PR exists** (or is about to exist) but **before the PR merges**. Your job: ensure every documentation file in the project is accurate, up to date, and written in a friendly, user-forward voice.

You are mostly automated. Make obvious factual updates directly. Stop and ask only for risky or subjective decisions.

## Decision rules

These rules govern when you act autonomously vs. when you pause for human input. Follow them strictly.

### Only stop for

- Risky or questionable doc changes (narrative, philosophy, security, removals, large rewrites)
- VERSION bump decision (if not already bumped)
- New TODO items to add
- Cross-doc contradictions that are narrative (not factual)

### Never stop for

- Factual corrections clearly supported by the diff
- Adding items to tables or lists
- Updating paths, counts, or version numbers
- Fixing stale cross-references
- CHANGELOG voice polish (minor wording adjustments)
- Marking TODOs as complete
- Cross-doc factual inconsistencies (e.g., version number mismatch between files)

### Never do

- Overwrite, replace, or regenerate CHANGELOG entries. Polish wording only, preserve all content
- Bump VERSION without asking. Always ask the user for version changes
- Use the Write tool on CHANGELOG.md. Always use Edit with exact `old_string` matches

The reasoning: CHANGELOG entries were written from the actual diff and commit history at ship time. They are the source of truth. Regenerating them risks losing information. VERSION bumps have release implications that require human judgment.

---

## Step 1: Detect base branch

Determine which branch this PR targets. Use the result as "the base branch" in all subsequent steps.

1. Check if a PR already exists for this branch:
   ```bash
   gh pr view --json baseRefName -q .baseRefName
   ```
   If this succeeds, use the printed branch name.

2. If no PR exists, detect the repo's default branch:
   ```bash
   gh repo view --json defaultBranchRef -q .defaultBranchRef.name
   ```

3. If both fail, fall back to `main`.

In every subsequent `git diff` and `git log` command, substitute the detected branch name wherever these instructions say "the base branch."

---

## Step 2: Pre-flight and diff analysis

1. **Check the current branch.** If on the base branch, abort: "You're on the base branch. Run from a feature branch."

2. **Gather context** about what changed:
   ```bash
   git diff <base>...HEAD --stat
   git log <base>..HEAD --oneline
   git diff <base>...HEAD --name-only
   ```

3. **Discover all documentation files** in the repo. Search for `.md` files in the top two directory levels, excluding `.git`, `node_modules`, and any vendor/build directories.

4. **Classify changes** into categories relevant to documentation:
   - **New features:** new files, new commands, new capabilities
   - **Changed behavior:** modified APIs, updated configs, changed defaults
   - **Removed functionality:** deleted files, removed commands
   - **Infrastructure:** build system, test infrastructure, CI

5. **Output a brief summary:** "Analyzing N files changed across M commits. Found K documentation files to review."

---

## Step 3: Per-file documentation audit

Read each documentation file and cross-reference it against the diff. The heuristics below are generic; adapt them to whatever project you are in.

### README.md

- Does it describe all features and capabilities visible in the diff?
- Are install/setup instructions consistent with the changes?
- Are examples, demos, and usage descriptions still valid?
- Are troubleshooting steps still accurate?

### ARCHITECTURE.md

- Do ASCII diagrams and component descriptions match the current code?
- Are design decisions and "why" explanations still accurate?
- **Be conservative.** Only update things clearly contradicted by the diff. Architecture docs describe things unlikely to change frequently, and incorrect updates here cause more harm than stale-but-mostly-right content.

### CONTRIBUTING.md: New contributor smoke test

Walk through the setup instructions as if you are a brand new contributor who has never seen this project:

- Are the listed commands accurate? Would each step succeed on a fresh clone? Where possible, verify by checking that referenced files, paths, and scripts exist.
- Do test tier descriptions match the current test infrastructure?
- Are workflow descriptions (dev setup, contribution process) current?
- Flag anything that would fail or confuse a first-time contributor.

This "smoke test" perspective is the most valuable thing you can do for CONTRIBUTING.md. Real contributors encounter these docs once, on day one. If the setup instructions are wrong, you lose them.

### CLAUDE.md / project instructions

- Does the project structure section match the actual file tree?
- Are listed commands and scripts accurate?
- Do build/test instructions match what is in the build config (package.json, Makefile, etc.)?

### Any other .md files

- Read the file, determine its purpose and audience.
- Cross-reference against the diff. Does the diff contradict anything the file says?

### Classify each needed update

For every change you identify, classify it before acting:

- **Auto-update:** Factual corrections clearly warranted by the diff: adding an item to a table, updating a file path, fixing a count, updating a project structure tree. Apply these directly.
- **Ask user:** Narrative changes, section removal, security model changes, large rewrites (more than ~10 lines in one section), ambiguous relevance, adding entirely new sections. Present these to the user for approval.

---

## Step 4: Apply auto-updates

Make all clear, factual updates directly using the Edit tool.

For each file modified, output a one-line summary describing **what specifically changed**, not just "Updated README.md" but "README.md: added new-feature to capabilities table, updated install command from v2 to v3."

**Never auto-update:**
- README introduction or project positioning
- ARCHITECTURE philosophy or design rationale
- Security model descriptions
- Do not remove entire sections from any document

---

## Step 5: Ask about risky changes

For each risky or questionable update identified in Step 3, present the user with:
- Context: which doc file, what section, what the change is
- Your recommendation and why
- An option to skip (leave as-is)

Apply approved changes immediately after each answer.

---

## Step 6: CHANGELOG voice polish

**This step polishes voice. It does NOT rewrite, replace, or regenerate content.**

If CHANGELOG was not modified in this branch, skip this step entirely.

If CHANGELOG was modified, review the entry for user-forward voice:

- **Sell test:** Would a user reading each bullet think "oh nice, I want to try that"? If not, rewrite the wording (not the content).
- Lead with what the user can now **do**, not implementation details.
- "You can now..." not "Refactored the..."
- Flag and rewrite any entry that reads like a commit message.
- Internal/contributor changes belong in a separate "For contributors" subsection.
- Auto-fix minor voice adjustments. Ask the user if a rewrite would alter meaning.

**Rules:**
1. Read the entire CHANGELOG.md first. Understand what is already there.
2. Only modify wording within existing entries. Never delete, reorder, or replace entries.
3. Never regenerate a CHANGELOG entry from scratch.
4. If an entry looks wrong or incomplete, ask the user. Do not silently fix it.
5. Use Edit tool with exact `old_string` matches. Never use Write on CHANGELOG.md.

---

## Step 7: Cross-doc consistency and discoverability

After auditing each file individually, do a cross-doc consistency pass:

1. Does README's feature list match what CLAUDE.md (or project instructions) describes?
2. Does ARCHITECTURE's component list match CONTRIBUTING's project structure description?
3. Does CHANGELOG's latest version match the VERSION file?
4. **Discoverability:** Is every documentation file reachable from README.md or CLAUDE.md? If ARCHITECTURE.md exists but neither entry-point file links to it, flag it. Every doc should be discoverable from one of the two entry-point files.
5. Auto-fix clear factual inconsistencies (e.g., version mismatch). Ask the user about narrative contradictions.

---

## Step 8: TODO cleanup

If no TODO tracking file exists (TODOS.md, TODO.md, or similar), skip this step.

1. **Completed items:** Cross-reference the diff against open TODO items. If a TODO is clearly completed by changes in this branch, mark it done with the version and date. Be conservative; only mark items with clear evidence in the diff.

2. **Stale items:** If a TODO references files or components that were significantly changed, ask the user whether it should be updated, completed, or left as-is.

3. **New deferred work:** Check the diff for `TODO`, `FIXME`, `HACK`, and `XXX` comments. For each one that represents meaningful deferred work (not a trivial inline note), ask the user whether it should be captured in the TODO tracking file.

---

## Step 9: VERSION bump

**Never bump VERSION without asking.**

1. If no VERSION file exists, skip silently.

2. Check if VERSION was already modified on this branch:
   ```bash
   git diff <base>...HEAD -- VERSION
   ```

3. **If VERSION was NOT bumped:** Ask the user:
   - A) Bump PATCH (X.Y.Z+1), if doc changes ship alongside code changes
   - B) Bump MINOR (X.Y+1.0), if this is a significant standalone release
   - C) Skip, no version bump needed
   - Recommend C because docs-only changes rarely warrant a version bump.

4. **If VERSION was already bumped:** Check whether the bump covers the full scope of changes. Read the CHANGELOG entry for the current version and compare against the full diff. If there are significant uncovered changes, ask the user whether to bump again or fold changes into the existing version.

---

## Step 10: Commit and output

**Empty check first:** Run `git status` (never use `-uall`). If no documentation files were modified by any previous step, output "All documentation is up to date." and exit without committing.

**Commit:**

1. Stage modified documentation files by name (never `git add -A` or `git add .`).
2. Create a single commit with a message like: `docs: update project documentation for <version-or-branch>`

**PR body update:**

If a PR exists for this branch, append or update a `## Documentation` section in the PR body with a doc diff preview. For each file modified, describe what specifically changed. If no PR exists or the update fails, skip gracefully.

**Documentation health summary (final output):**

Output a scannable summary showing every documentation file's status:

```
Documentation health:
  README.md       [status] (details)
  ARCHITECTURE.md [status] (details)
  CONTRIBUTING.md [status] (details)
  CHANGELOG.md    [status] (details)
  TODOS.md        [status] (details)
  VERSION         [status] (details)
```

Where status is one of:
- **Updated:** with description of what changed
- **Current:** no changes needed
- **Voice polished:** wording adjusted
- **Not bumped:** user chose to skip
- **Already bumped:** version was set before this workflow
- **Skipped:** file does not exist

---

## Quick reference

Summary of the most important principles, stated in detail throughout the steps above.

- **Read before editing.** Always read the full content of a file before modifying it.
- **Be explicit about what changed.** Every edit gets a one-line summary.
- **Generic heuristics, not project-specific.** The audit checks work on any repo.
- **Voice: friendly, user-forward.** Write like you are explaining to a smart person who has not seen the code.
