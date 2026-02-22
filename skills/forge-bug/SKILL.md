---
name: forge-bug
description: "File bug reports against the Forge framework repo. Use when hitting a Forge bug during development. Triggers on: forge bug, report forge bug, file forge issue."
argument-hint: <bug description>
context: current
allowed-tools: Read, Grep, Glob, Bash
---

# Forge Bug Reporter

You file structured bug reports on `dmitrymomot/forge` based on the current session context. The user hit a bug while working with Forge — your job is to collect context, draft a clean issue, and file it after confirmation.

The bug description from `$ARGUMENTS` (if provided) is the starting point.

---

## Phase 1: Gather Environment (silent — no output to user)

Run these commands to collect environment details:

1. **Go version** — `go version`
2. **Forge version** — read `go.mod` in the current project, find the `github.com/dmitrymomot/forge` dependency line and extract the version
3. **OS/arch** — `go env GOOS GOARCH`
4. **Module path** — from `go.mod` (to understand which project hit the bug)

If there's no `go.mod` in the current directory, note that — environment info will be partial and that's fine.

---

## Phase 2: Analyze Session Context

Review the conversation history for:

- **Error messages** — compilation errors, runtime panics, unexpected output
- **Stack traces** — any Go stack traces or panic output
- **Workarounds** — code the user had to change to work around a Forge issue
- **Failed commands** — `go build`, `go run`, or test failures related to Forge APIs
- **API mismatches** — where Forge's actual behavior differed from expected/documented behavior

Combine this with `$ARGUMENTS` to build a complete picture of the bug.

If the session doesn't contain enough context to write a useful bug report, ask the user to describe:
- What they were trying to do
- What happened instead
- Any error messages they saw

---

## Phase 3: Draft the Issue

Compose a GitHub issue with this exact structure:

```
Title: [concise description of the bug — under 80 chars]

Body:

## Description

[Clear explanation of the bug. What's broken and in what context.]

## Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Step 3]

## Expected Behavior

[What should happen]

## Actual Behavior

[What actually happens. Include error messages, stack traces, or unexpected output verbatim.]

## Environment

- **Go:** [version]
- **Forge:** [version from go.mod]
- **OS/Arch:** [GOOS/GOARCH]
```

**Drafting rules:**
- Keep the title short and specific — describe the symptom, not the fix
- Steps to Reproduce should be minimal — fewest steps to trigger the bug
- Include actual error messages/stack traces in the "Actual Behavior" section using code blocks
- Don't speculate about the fix in the issue body — just report what's broken
- If `$ARGUMENTS` already contains a clear description, use it as the basis; don't make the user repeat themselves

---

## Phase 4: Present for Confirmation

Show the user the complete draft (title + body) and ask:

> Here's the bug report I'll file on `dmitrymomot/forge`. Review it and let me know if you want to change anything, or confirm to file it.

**Wait for explicit confirmation before proceeding.** The user may want to:
- Edit the title or description
- Add more context
- Remove sensitive information
- Cancel entirely

---

## Phase 5: File the Issue

After user confirms, create the issue:

```bash
gh issue create --repo dmitrymomot/forge --title "<title>" --body "<body>"
```

Use a heredoc for the body to preserve formatting:

```bash
gh issue create --repo dmitrymomot/forge --title "the title" --body "$(cat <<'EOF'
issue body here
EOF
)"
```

After filing, show the user the issue URL returned by `gh`.

If `gh` fails (auth issues, network problems), show the error and suggest the user file manually with the drafted content.
