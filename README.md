# code management for claude code

Automating backend documentation, changelog generation, and approval-gated commits using Claude Code.

---

## What This Skill Does

Once installed, this skill gives Claude Code four automated workflows you can trigger by natural language:

| Workflow | What it does |
|---|---|
| **Document this** | Reads a file and generates JSDoc, docstrings, OpenAPI blocks, or README sections |
| **Sync the docs** | Compares existing docs against changed code and reports every mismatch |
| **Write a changelog** | Classifies a diff and drafts a Keep-a-Changelog entry |
| **Commit this** | Drafts a Conventional Commits message and stages a full commit for your approval |

Nothing is written to disk or committed until you explicitly say yes.

---

## Installation

### Step 1 — Copy the skill into your project

Claude Code looks for skills in a `.claude/skills/` directory at your project root (or globally in `~/.claude/skills/`).

```bash
# Per-project (recommended)
mkdir -p .claude/skills/backend-code-manager
cp SKILL.md .claude/skills/backend-code-manager/SKILL.md

# Or globally (available in every project)
mkdir -p ~/.claude/skills/backend-code-manager
cp SKILL.md ~/.claude/skills/backend-code-manager/SKILL.md
```

### Step 2 — Verify Claude Code sees the skill

Start a Claude Code session in your project and run:

```
/skills
```

You should see `backend-code-manager` listed. If not, check the path — the file must be at exactly `SKILL.md` inside the skill folder.

### Step 3 — No other setup required

This skill has no dependencies. It works entirely from files you share in the conversation — no additional config, API keys, or packages needed.

---

## Usage

### Workflow 1 — Generate documentation for a new file

Share the file you want documented and ask Claude to document it.

**Example prompts:**
```
Document src/services/paymentService.ts
```
```
Generate JSDoc for all exported functions in this file: [paste code]
```
```
Write an OpenAPI block for this route handler
```

**What happens:**
1. Claude reads the file and identifies the public surface, parameters, return types, and error conditions
2. It picks the right doc format for the context (JSDoc for TS utilities, OpenAPI for routes, etc.)
3. It shows you the proposed documentation block in a clearly labelled preview
4. You type `yes`, `edit`, or `skip` — nothing is inserted until you approve

---

### Workflow 2 — Sync stale docs to updated code

Share both the updated source file and the existing documentation file.

**Example prompts:**
```
My auth service changed — sync the docs to match the new code
```
```
Check if docs/api/users.md is still accurate for src/routes/users.ts
```
```
Update the README section for the cache module — here's what changed: [paste diff]
```

**What happens:**
1. Claude diffs the docs against the code and finds every mismatch
2. It prints a **Drift Report** categorising issues as `STALE`, `MISSING`, or `GHOST`
3. You choose which issues to fix (all, selected, or none)
4. Claude shows a before/after for each approved change before writing anything

---

### Workflow 3 — Generate a changelog entry from a diff

Paste a `git diff`, describe what changed, or share two versions of a file.

**Example prompts:**
```
Write a changelog entry for these changes: [paste git diff]
```
```
What changed between these two versions of authMiddleware.ts?
```
```
Generate a CHANGELOG.md entry — I added rate limiting to the login endpoint
```

**What happens:**
1. Claude classifies every change by type (`feat`, `fix`, `security`, `break`, `perf`, etc.)
2. It drafts a Keep-a-Changelog block written from the caller's perspective
3. It shows you the entry and asks before appending to `CHANGELOG.md`

---

### Workflow 4 — Draft and approve a commit

Tell Claude you're ready to commit, or ask it to commit specific changes.

**Example prompts:**
```
Write a commit message for my staged changes
```
```
Commit this — I fixed the token validation bug in auth middleware
```
```
Generate a commit for everything in src/services/ that changed today
```

**What happens:**
1. Claude analyses the diff and checks whether changes should be one commit or several atomic ones — it will flag mixed concerns and ask you to decide
2. It writes a Conventional Commits message with a proper subject line, body (why, not what), and footer
3. It shows a full **Commit Proposal** — type, scope, files, message, doc updates, changelog entry — all in one block
4. You approve with `yes`, request edits, or cancel
5. After your approval, Claude applies the commit and prints the SHA

---

## Approval Gate — How It Works

Every workflow ends with a proposal block before any file is touched:

```
── PROPOSED COMMIT ─────────────────────────────────────────
Type:    fix
Scope:   auth
Files:   src/middleware/auth.ts, tests/auth.test.ts

Message:
  fix(auth): reject tokens missing required `sub` claim

  The JWT verification middleware previously allowed tokens without
  a `sub` claim to pass. Now strictly validates presence and value.

  Closes #412

Docs updated:  docs/auth.md (2 lines changed)
Changelog:     CHANGELOG.md — added 1 entry under [Fixed]
────────────────────────────────────────────────────────────
Approve and commit? (yes / edit message / cancel)
```

**The skill will never auto-commit.** If you ask it to skip the approval step, it will explain the risk and offer a dry-run instead.

---

## Tips

**Trigger it by describing the task, not the workflow name.** You don't need to say "run Workflow 3". Just say "write a changelog for this diff" or "what changed in this file?" and the skill activates the right workflow automatically.

**Paste code directly into the conversation.** Claude Code can also read files from your project directory — both work fine.

**For large PRs, ask for split commits first.** If your diff covers multiple unrelated concerns, start with: `"Should this be one commit or several?"` Claude will analyse and recommend before you commit anything.

**Uncertainty is always surfaced.** If Claude can't determine a behaviour from the code alone, it will say so and ask before documenting it as a guarantee. Don't override this — it's protecting your docs from being wrong.

**Docs-only changes get their own commit.** By default, documentation updates are committed separately as `docs(scope): ...` commits. If your project uses a different convention, just tell Claude at the start of the session.

---

## Example Session

```
You:    document src/services/emailService.ts

Claude: [reads file]
        ── PROPOSED DOCUMENTATION ──────────────────────────────
        /**
         * Sends a transactional email via the configured SMTP provider.
         * @param {string} to - Recipient email address
         * @param {string} subject - Email subject line
         * @param {string} html - HTML body content
         * @returns {Promise<{ messageId: string }>}
         * @throws {SMTPConnectionError} If the SMTP server is unreachable
         * @throws {InvalidRecipientError} If `to` fails RFC 5321 validation
         *
         * @example
         * const { messageId } = await sendEmail(
         *   'user@example.com',
         *   'Welcome!',
         *   '<h1>Thanks for signing up</h1>'
         * );
         */
        ────────────────────────────────────────────────────────
        Apply this? (yes / edit / skip)

You:    yes

Claude: ✓ Documentation added to src/services/emailService.ts
```

---

## File Structure After Installation

```
your-project/
├── .claude/
│   └── skills/
│       └── backend-code-manager/
│           └── SKILL.md          ← the skill
├── src/
├── docs/
├── CHANGELOG.md
└── ...
```

---

## Troubleshooting

**The skill isn't triggering.**
Make sure `SKILL.md` is inside a named folder (`backend-code-manager/SKILL.md`), not loose in the skills directory. Run `/skills` to confirm Claude sees it.

**Claude is editing files without asking.**
This shouldn't happen — it's a constraint in the skill. If it does, start a new session and paste the relevant section of the SKILL.md into the conversation to re-enforce the approval gate.

**The commit format doesn't match our convention.**
Tell Claude your convention at the start of the session: *"We use `[TYPE] scope: message` format instead of Conventional Commits."* It will follow project-specific conventions for the rest of the session.

**Claude says a behaviour is ambiguous.**
That's intentional. Answer the question so the documentation is accurate. Do not ask Claude to assume — wrong docs are worse than incomplete docs.