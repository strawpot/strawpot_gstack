---
name: release-workflow
displayName: Release Workflow
description: "Structured release ceremony: merge base, run tests, review diff, bump version, update changelog, create bisectable commits, push, and create PR, all automated. Use when the user wants to 'ship', 'release', 'cut a release', 'ship it', 'let's ship', 'ready to go', 'finalize this', or says code is 'ready to ship'. Also trigger when asked to finalize a feature branch with full pre-flight checks, or when the user indicates they are done coding and want to push. Do NOT use for simple PR creation without the release ceremony (use github-prs). Do NOT use for basic git operations like branching or committing (use git-workflow)."
metadata:
  strawpot:
    dependencies:
      - completeness-principle
    tools:
      gh:
        description: GitHub CLI for PR detection, creation, and repo metadata
        install:
          macos: brew install gh
          linux: sudo apt install gh
          windows: winget install GitHub.cli
---

# Release Workflow

A fully automated, non-interactive ship ceremony. When the user invokes this skill, run straight through every step and output the PR URL at the end. Do not ask for confirmation at any step unless explicitly noted below.

## Stop and never-stop rules

**Only stop for:**
- Being on the base branch (abort; must ship from a feature branch)
- Merge conflicts that cannot be auto-resolved (show conflicts)
- Test failures (show failures)
- Pre-landing review finds issues that need user judgment
- MINOR or MAJOR version bump needed (ask the user)

**Never stop for:**
- Uncommitted changes (always include them)
- Version bump choice when PATCH (auto-pick)
- Changelog content (auto-generate from diff)
- Commit message approval (auto-commit)
- Multi-file changesets (auto-split into bisectable commits)

---

## Step 0: Pre-checks and detect base branch

**Check `gh` authentication.** Run `gh auth status`. If not authenticated, tell the user to run `gh auth login` and abort.

Determine which branch this PR targets. Use the result as "the base branch" in all subsequent steps.

1. Check if a PR already exists for this branch:
   ```bash
   gh pr view --json baseRefName -q .baseRefName
   ```
   If this succeeds, use the printed branch name.

2. If no PR exists (command fails), detect the repo's default branch:
   ```bash
   gh repo view --json defaultBranchRef -q .defaultBranchRef.name
   ```

3. If both commands fail, fall back to `main`.

Print the detected base branch name. In every subsequent command, substitute the detected branch name wherever these instructions say "the base branch" or `<base>`.

---

## Step 1: Pre-flight

1. **Check the current branch.** If on the base branch, **abort**: "You're on the base branch. Ship from a feature branch."

2. **Run `git status`** (never use `-uall`). Uncommitted changes are always included, no need to ask.

3. **Understand the diff.** Run:
   ```bash
   git diff <base>...HEAD --stat
   git log <base>..HEAD --oneline
   ```

4. **Review readiness check.** Look for evidence that the changes have been reviewed, such as an existing PR with review comments, a code review tool output, or a review artifact in the repo. If no review evidence exists, note it in the PR body but do not block shipping. This is informational, not a gate.

---

## Step 2: Merge the base branch (before tests)

Fetch and merge the base branch into the feature branch so tests run against the merged state:

```bash
git fetch origin <base> && git merge origin/<base> --no-edit
```

**If there are merge conflicts:** Try to auto-resolve if they are simple (VERSION files, changelog ordering, lockfile conflicts). If conflicts are complex or ambiguous, **STOP** and show them to the user.

**If already up to date:** Continue silently.

---

## Step 3: Run tests

Discover the project's test commands by checking, in order:
1. `CLAUDE.md` (look for a testing section with run commands)
2. Common project files: `package.json` (scripts.test), `Makefile` (test target), `Gemfile` (rake test / rspec), `pyproject.toml` (pytest), `Cargo.toml` (cargo test), `go.mod` (go test ./...)
3. Common test directories: `test/`, `tests/`, `spec/`, `__tests__/`

Run the full test suite. If multiple test suites exist (e.g., unit + integration), run them all.

**If any test fails:** Show the failures and **STOP**. Do not proceed.

**If all pass:** Continue, noting the counts briefly.

**If no test infrastructure exists:** Note "No test suite found" and continue. Do not block shipping for the absence of tests.

---

## Step 3.5: Pre-landing code review

Review the diff for structural issues that tests do not catch.

1. Run `git diff origin/<base>` to get the full diff.

2. Scan for common issues in two passes:

   **Pass 1, Low-risk (auto-fix with summary):**
   Apply these fixes automatically, but list every change so the user can review in the PR diff. If uncertain whether code is truly dead (e.g., dynamically called functions, reflective imports), move it to Pass 2 instead.
   - Dead code (unused imports, unreachable branches)
   - Stale comments that contradict the new code
   - Obvious N+1 query patterns
   - Hardcoded secrets or credentials (flag, do not auto-fix, move to Pass 2)
   - Debug/console statements left in production code

   **Pass 2, Judgment needed (ask user):**
   - Architectural concerns (wrong abstraction layer, tight coupling)
   - Performance issues that require design decisions
   - Security patterns that need confirmation

3. **Auto-fix all Pass 1 items.** Output one line per fix:
   `[AUTO-FIXED] [file:line] Problem → what you did`

4. **If Pass 2 items exist,** present them in one batch and ask the user:
   - List each with severity, problem, recommended fix
   - Options per item: A) Fix  B) Skip

5. **After all fixes:**
   - If any fixes were applied: commit them (`git commit -m "fix: pre-landing review fixes"`), then re-run tests (Step 3) before continuing.
   - If no fixes applied: continue to Step 4.

6. Output summary: `Pre-Landing Review: N issues, M auto-fixed, K asked (J fixed, L skipped)`
   If no issues: `Pre-Landing Review: No issues found.`

Save the review output for the PR body.

---

## Step 4: Version bump (if applicable)

Not every project uses versioning. Check for a version source:
- `VERSION` file
- `version` field in `package.json`, `pyproject.toml`, `Cargo.toml`, `mix.exs`
- `version.rb`, `version.py`, or similar

**If no version source found:** Skip this step entirely.

**If a version source exists:**

1. Read the current version.

2. Auto-decide the bump level based on semantic analysis of the diff:
   - First, scan for **breaking changes**: removed public APIs, changed function signatures, renamed exports, deleted fields. If found → **MAJOR** → **ask the user**.
   - Then scan for **new capabilities**: new public APIs, new exported functions, new CLI flags, new endpoints. If found → **MINOR** → **ask the user**.
   - Otherwise, fall back to line count as a tiebreaker: `git diff origin/<base>...HEAD --stat | tail -1`. Small diffs (< 50 lines) with no behavior changes are overwhelmingly patches, so auto-pick **PATCH**. Larger diffs without new APIs or breaking changes are also PATCH, so auto-pick.

3. Compute the new version. Bumping a digit resets all digits to its right to 0.

4. Write the new version to the version source.

---

## Step 5: Changelog

Check if `CHANGELOG.md` exists. If not, create one with a standard header.

Auto-generate the entry from all commits on the branch:
- Use `git log <base>..HEAD --oneline` to see every commit being shipped
- Use `git diff <base>...HEAD` to see the full diff

**Write in user-facing voice.** The changelog is for users, not contributors:
- Lead with what the user can now **do** that they could not before
- Use plain language, not implementation details: "You can now..." not "Refactored the..."
- Never mention internal tracking, dev infrastructure, or contributor-facing details
- Every entry should make someone think "oh nice, I want to try that"

Categorize changes into applicable sections:
- `### Added`: new capabilities
- `### Changed`: changes to existing behavior
- `### Fixed`: bug fixes
- `### Removed`: removed capabilities

Format: `## [X.Y.Z] - YYYY-MM-DD` (or without version if Step 4 was skipped).

Do NOT ask the user to describe changes. Infer everything from the diff and commit history.

---

## Step 5.5: TODOS update (if applicable)

Check if `TODOS.md`, `TODO.md`, or a similar task-tracking file exists.

**If it exists:**

1. Cross-reference each TODO item against the diff and commit history.
2. **Be conservative.** Only mark a TODO as completed if there is clear evidence in the diff.
3. Move completed items to a `## Completed` section (create it if it does not exist). Append: `**Completed:** vX.Y.Z (YYYY-MM-DD)` or just the date if no version.
4. Output: `TODOS: N items marked complete. M items remaining.`

**If it does not exist:** Skip silently.

---

## Step 6: Bisectable commits

**Goal:** Create small, logical commits that work well with `git bisect`. Every commit should represent one coherent change, not one file, but one logical unit.

1. Analyze the diff and group changes into logical commits.

2. **Commit ordering** (earlier commits first):
   - Infrastructure: migrations, config changes, route additions
   - Models and services (with their tests)
   - Controllers, views, UI components (with their tests)
   - VERSION + CHANGELOG + TODOS: always in the final commit

3. **Rules for splitting:**
   - A source file and its test file go in the same commit
   - Migrations are their own commit (or grouped with the model they support)
   - Config and route changes can group with the feature they enable
   - If the total diff is small (< 50 lines across < 4 files), a single commit is fine

4. **Each commit must be independently valid.** No broken imports, no references to code that does not exist yet. Order commits so dependencies come first.

5. Compose each commit message:
   - First line: `<type>: <summary>` (type = feat/fix/chore/refactor/docs/test)
   - Body: brief description of what this commit contains

---

## Step 6.5: Verification gate

Before pushing, verify if code changed during Steps 4–6:

1. If any application code changed after Step 3's test run (version bumps and changelog edits do not count), re-run the test suite. Paste fresh output.
2. If the project has a build step, run it.

**If tests fail:** STOP. Do not push. Fix the issue and return to Step 3.

---

## Step 7: Push and create PR

1. Push to the remote with upstream tracking:
   ```bash
   git push -u origin <branch-name>
   ```

2. Create a pull request:
   ```bash
   gh pr create --base <base> --title "<type>: <summary>" --body "$(cat <<'EOF'
   ## Summary
   <bullet points from changelog, or a concise summary of changes>

   ## Pre-Landing Review
   <findings from Step 3.5, or "No issues found.">

   ## Test Results
   <pass counts, or "No test suite found.">

   ## TODOS
   <completed items summary, or omit if no TODOS file>

   ## Test plan
   - [x] Tests pass on merged state (base branch merged first)
   - [x] Pre-landing review complete

   🤖 Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```

3. **Output the PR URL.** This is the primary deliverable of the skill.

---

## The goal

User invokes the skill, next thing they see is the PR URL.
