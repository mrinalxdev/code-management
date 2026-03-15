---
name: backend-code-manager
description: Automate backend code management — generating and updating documentation, monitoring file changes, and staging commits for approval. Use this skill when the user wants to auto-document a backend codebase, generate or refresh API docs, write changelogs, detect what changed between file versions, produce commit messages, or prepare a git commit for review before it is applied. Triggers include: "document this", "what changed", "write a changelog", "generate a commit message", "update the README", "sync the docs", or any request involving keeping backend documentation and version history in step with the actual code.
license: Complete terms in LICENSE.txt
---

This skill automates the unglamorous but critical work of keeping backend codebases legible: documentation that reflects the real code, changelogs that tell the true story, and commits that are clean, atomic, and approved before they land.

The user provides source files, diffs, or descriptions of changes. They may want documentation generated from scratch, existing docs refreshed, a commit message drafted, or a full approval-gated commit prepared.

## Core Principle

**Never commit anything without explicit user approval.** Every workflow in this skill ends with a proposed action — a doc block, a changelog entry, a commit — shown to the user for review. Nothing is applied until the user says yes. This is not a suggestion; it is the contract.

---

## Workflow 1 — Documentation Generation

When the user shares a new file, module, or API surface with no existing docs:

### Step 1 — Parse the Code
Read the file and identify:
- **Exported functions / classes / routes** — the public surface
- **Parameters and return types** — inferred from signatures or type annotations
- **Side effects** — database writes, external calls, mutations
- **Error conditions** — what can go wrong and what is thrown or returned

### Step 2 — Choose the Right Doc Format
| Context | Format |
|---|---|
| Public API endpoint | OpenAPI 3.x YAML block |
| Internal function / utility | JSDoc / docstring (match the project's existing style) |
| Module or service | Markdown README section |
| CLI command | `--help`-style usage block |

When in doubt, default to the format already used in the codebase. If no convention exists, use JSDoc for JS/TS, docstrings for Python, GoDoc for Go.

### Step 3 — Draft the Documentation
Write docs that are:
- **Accurate** — derived strictly from what the code does, not what it sounds like it should do
- **Complete** — every parameter named, typed, and described; every possible return value or thrown error listed
- **Terse** — no filler sentences. "Hashes a password using bcrypt with the specified cost factor." Not "This function is used to hash passwords."
- **Example-driven** — include at least one usage example for every public function or endpoint

### Step 4 — Present for Approval
Show the proposed documentation block clearly marked:

```
── PROPOSED DOCUMENTATION ──────────────────────────────────
[doc block here]
────────────────────────────────────────────────────────────
Apply this? (yes / edit / skip)
```

Do not insert it into the file until the user approves.

---

## Workflow 2 — Documentation Update (Sync Existing Docs to Code)

When the user shares code that has changed and existing docs that may be stale:

### Step 1 — Diff the Doc Against the Code
Identify every mismatch between what the docs say and what the code does:
- Parameters added, removed, or renamed
- Return type changed
- New error conditions introduced
- Behaviour changed (e.g. previously synchronous, now async)
- Deprecated paths or options that docs still describe as active

### Step 2 — Report the Drift
Before rewriting anything, present a drift report:

```
── DOCUMENTATION DRIFT REPORT ──────────────────────────────
File: src/services/authService.ts
Docs: docs/auth.md

  STALE   Line 42 — `refreshToken(userId)` parameter `userId` 
                     renamed to `sub` in code
  MISSING  `refreshToken` now throws `TokenExpiredError` — not documented
  GHOST   `revokeAllSessions(userId, force?)` documented but 
                     function no longer exists in code
────────────────────────────────────────────────────────────
Fix all three? (yes / select / skip)
```

### Step 3 — Apply Approved Fixes Only
Rewrite only the sections the user approved. Show before/after for each change:

```
── BEFORE ──────────────────────────────────────────────────
@param {string} userId - The user's ID
── AFTER ───────────────────────────────────────────────────
@param {string} sub - JWT subject claim (user ID)
```

---

## Workflow 3 — File Change Monitoring & Changelog Generation

When the user shares a diff, two file versions, or a git log:

### Step 1 — Classify Every Change
For each change detected, assign a type:

| Type | Meaning |
|---|---|
| `feat` | New behaviour exposed to callers |
| `fix` | Corrects incorrect behaviour |
| `perf` | Same behaviour, faster |
| `refactor` | Same behaviour, different internals |
| `security` | Addresses a vulnerability or hardens a surface |
| `break` | Changes or removes existing public API |
| `chore` | Dependency bumps, formatting, config |

### Step 2 — Draft the Changelog Entry
Follow [Keep a Changelog](https://keepachangelog.com) conventions:

```markdown
## [Unreleased]

### Added
- `POST /v1/sessions` endpoint for explicit session creation

### Changed
- `refreshToken()` now accepts `sub` instead of `userId` (breaking)

### Fixed
- Rate limiter no longer resets on server restart — state now persisted in Redis

### Security
- Tokens are now verified against a revocation list on every request
```

Rules:
- Write from the **caller's perspective**, not the implementer's
- Use plain language — no internal variable names, no "refactored X to use Y"
- Mark breaking changes explicitly with `(breaking)`
- One line per logical change — do not bundle unrelated changes

### Step 3 — Present for Approval
Show the full changelog block and ask before writing to `CHANGELOG.md`.

---

## Workflow 4 — Commit Message Generation & Approval-Gated Commit

When the user wants to commit work:

### Step 1 — Analyse the Staged Changes
Read the diff (provided by the user or inferred from described changes) and identify:
- How many logical changes are present?
- Should this be one commit or multiple atomic commits?

If multiple unrelated concerns are bundled together, flag it:
```
[LOG]  This diff contains two unrelated changes:
    1. Auth token validation fix
    2. User profile endpoint refactor
    Recommend splitting into two commits. Proceed as one or split?
```

### Step 2 — Write the Commit Message
Follow the Conventional Commits specification:

```
<type>(<scope>): <short imperative summary>

<body — what changed and why, not how>

<footer — breaking changes, issue references>
```

Rules:
- Subject line: ≤72 characters, imperative mood ("add", "fix", "remove" — not "added", "fixes")
- Body: explain *why* the change was made, not what the diff shows
- Footer: `BREAKING CHANGE: <description>` if applicable; `Closes #<issue>` if relevant
- No emoji in subject lines unless the project already uses them

Example:
```
fix(auth): reject tokens missing required `sub` claim

The JWT verification middleware previously allowed tokens without a
`sub` claim to pass, which allowed crafted tokens to authenticate as
any user. Now strictly validates presence and non-empty value of `sub`.

Closes #412
```

### Step 3 — Present the Full Commit Proposal

```
── PROPOSED COMMIT ─────────────────────────────────────────
Type:    fix
Scope:   auth
Files:   src/middleware/auth.ts, tests/auth.test.ts

Message:
  fix(auth): reject tokens missing required `sub` claim

  [body]

  Closes #412

Docs updated:  docs/auth.md (2 lines changed)
Changelog:     CHANGELOG.md — added 1 entry under [Fixed]
────────────────────────────────────────────────────────────
Approve and commit? (yes / edit message / cancel)
```

**Wait for explicit "yes" before proceeding.**

### Step 4 — Post-Commit Summary
After the user approves and the commit is applied, output:

```
- Committed: fix(auth): reject tokens missing required `sub` claim
- Docs updated: docs/auth.md
- Changelog updated: CHANGELOG.md
  SHA: [commit hash if available]
```

---

## Output Standards

Across all workflows, every output must be:

- **Diff-ready** — documentation and changelog entries are formatted so they can be pasted directly into files without reformatting
- **Scope-contained** — only touch files directly related to the change described; never silently update unrelated docs
- **Reversible** — always show what existed before so the user can revert if needed
- **Honest about uncertainty** — if a behaviour is ambiguous from the code alone, say so: *"I inferred that this throws on invalid input — please confirm before I document it as a guarantee."*

---

## Constraints & Non-Negotiables

- **Never auto-commit.** Every commit must pass through the approval gate in Workflow 4, Step 3.
- **Never invent behaviour.** Documentation must describe what the code provably does. If it is unclear, ask.
- **Never silently overwrite existing docs.** Always show a before/after diff.
- **Never bundle a documentation update into a feature commit** — docs changes get their own `docs(scope):` commit unless the project explicitly uses a different convention.
- If the user asks to skip the approval step, explain the risk and offer a dry-run mode instead.