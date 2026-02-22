---
name: forge-report
description: "Auto-collect bugs and improvements from the current session and file a structured issue on forge-skills. Triggers on: forge report, session report, file session issues, report skill issues."
argument-hint: "[target-repo]"
context: current
allowed-tools: Read, Grep, Glob, Bash
---

# Forge Session Reporter

You scan the current Claude Code session for errors, workarounds, and deviations that occurred while using forge-skills. You categorize findings, deduplicate against existing issues, draft a structured report, and file it after user confirmation.

Default target repo: `dmitrymomot/forge-skills`. If `$ARGUMENTS` contains a repo in `owner/repo` format, use that instead.

---

## Phase 1: Identify Which Skills Were Used (silent — no output to user)

Scan the conversation history to determine which forge-skills were invoked during this session. Look for:

- `/forge-init` invocations — project scaffolding
- `/forge-build` invocations — feature building
- `/forge-bug` invocations — bug reporting
- `/templui` invocations — UI component generation
- Any other skill invocations (check for `/` command patterns)

Record which skills were used — each finding will be tagged with its source skill.

---

## Phase 2: Scan Session for Findings (silent — no output to user)

Review the full conversation history and collect every instance of:

### Bugs (broken behavior)
- **Compilation errors** after skill-generated code — `go build` failures, type mismatches, missing imports
- **Runtime panics or errors** caused by generated code
- **Incorrect generated code** — wrong API calls, wrong types, wrong signatures
- **Template bugs** — wrong file paths, missing files, malformed output
- **Config mismatches** — env var names that don't match between generated files (e.g., `DATABASE_URL` vs `DB_URL`)

### Improvements (suboptimal but not broken)
- **Manual fixes the user had to make** — code the user edited after generation
- **Missing features** — things the user expected the skill to handle but didn't
- **Suboptimal defaults** — values that needed tweaking for common use cases
- **Lint/vet warnings** on generated code
- **Missing error handling** in generated code
- **Documentation gaps** — unclear or missing instructions in skill output

For each finding, record:
- **Category:** bug or improvement
- **Source skill:** which `/forge-*` command produced it
- **What happened:** brief description
- **Files affected:** which skill data files or templates are responsible (paths within `skills/` directory)
- **Suggested fix:** what should change in the skill to prevent this

---

## Phase 3: Deduplicate Against Existing Issues

Run:

```bash
gh issue list --repo <target-repo> --state open --limit 50
```

For each finding from Phase 2, check if an existing open issue already covers it. If so:
- Note the existing issue number next to the finding
- Mark it as "already reported" — it will be listed separately in the draft, not included in the new issue body

---

## Phase 4: Draft the Issue

If there are no new (non-duplicate) findings, tell the user:
> No new issues found in this session. Everything either worked correctly or is already tracked in existing issues.

Otherwise, compose the issue:

```
Title: Issues found during `<skill-name>` session [YYYY-MM-DD]
```

If multiple skills were used, use the primary one (most findings) or say "forge-skills session".

**Body format:**

```markdown
## Issues found during `<skill-name>` session

<One-sentence summary: N bugs and M improvements found.>

### Bugs

#### 1. Short description
- **Source skill:** `/forge-init`
- **What went wrong:** ...
- **Files affected:** `skills/forge-init/data/templates/justfile.md`
- **Suggested fix:** ...

#### 2. ...

### Improvements

#### 1. Short description
- **Source skill:** `/forge-init`
- **What's missing:** ...
- **Files affected:** `skills/forge-init/data/subsystems/db.md`
- **Suggested fix:** ...

### Already Tracked

- #14 — DATABASE_URL mismatch (covers finding X)
- #22 — betteralign exit code (covers finding Y)
```

**Drafting rules:**
- Number findings within each category
- Keep descriptions concise — one or two sentences each
- File paths should be relative to the repo root (e.g., `skills/forge-init/data/...`)
- "Suggested fix" should describe what to change in the skill, not a code patch
- The "Already Tracked" section is informational — it helps the user see what's covered
- If all findings are duplicates, still show the draft but note that no new issue needs to be filed
- Add the `enhancement` label if all findings are improvements, `bug` if any are bugs

---

## Phase 5: Present for Confirmation

Show the user the complete draft (title + body) and ask:

> Here's the session report I'll file on `<target-repo>`. Review it and let me know if you want to:
> - Remove any findings
> - Edit descriptions or suggested fixes
> - Add context I missed
> - Confirm to file it

**Wait for explicit confirmation before proceeding.** Do not file until the user says yes.

---

## Phase 6: File the Issue

After user confirms, create the issue:

```bash
gh issue create --repo <target-repo> --title "the title" --label "bug" --body "$(cat <<'EOF'
issue body here
EOF
)"
```

Label logic:
- If any finding is a bug → use `--label "bug"`
- If all findings are improvements → use `--label "enhancement"`

After filing, show the user the issue URL returned by `gh`.

If `gh` fails (auth issues, network problems), show the error and present the full draft so the user can file manually.
