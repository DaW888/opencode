---
description: Review a GitHub PR and optionally post comments
subagent: false
---

# PR Review Command

Review a GitHub Pull Request: $ARGUMENTS

The argument should be a GitHub PR URL (e.g. `https://github.com/owner/repo/pull/123`).

## Tools Available

You have access to the `gh` CLI (GitHub CLI). Use it to fetch PR data and post comments.

## Your Task

1. **Fetch PR diff**: Run `gh pr diff <PR_URL>`
2. **Fetch PR info**: Run `gh pr view <PR_URL>` to understand context
3. **Analyze the diff** for issues across all categories below
4. **Generate a structured report** of all findings
5. **Ask the user** which issues to post as PR comments (see Posting Comments below)

## Check Categories

### Security Issues (CRITICAL)
- Hardcoded credentials, API keys, tokens
- SQL injection vulnerabilities
- XSS vulnerabilities
- Missing input validation
- Insecure dependencies
- Path traversal risks
- Authentication/authorization flaws

### Logic Correctness (HIGH)
- Off-by-one errors in loops, slices, or boundary checks
- Incorrect boolean expressions (wrong operator, missing negation, always-true/always-false conditions)
- Wrong comparison targets (comparing to the wrong variable, constant, or enum value)
- Unreachable code or dead branches that indicate a logic mistake
- Race conditions or incorrect ordering of operations
- Null/undefined dereferences where a value may not exist
- Incorrect early returns or missing early returns that skip necessary work
- Swapped arguments or parameters passed in the wrong order
- Flawed state transitions (missing state updates, wrong next state)
- Incorrect handling of edge cases (empty collections, zero values, negative numbers, boundary values)
- Mismatched types that compile/pass linting but produce wrong results at runtime (e.g. string vs number comparison)

### Code Quality (HIGH)
- Functions too long or too complex
- Missing error handling
- Debug statements left in (console.log, print, etc.)
- TODO/FIXME comments in new code
- Missing documentation for public APIs

### Best Practices (MEDIUM)
- Mutation patterns where immutable is preferred
- Unnecessary complexity
- Missing tests for new code
- Accessibility issues (a11y)
- Performance concerns

### Style (LOW)
- Inconsistent naming
- Missing type annotations
- Formatting issues

## Report Format

Present findings as well-structured paragraphs grouped by severity. Do NOT use tables. Each severity section is a heading, and each issue within it is a numbered paragraph with a descriptive title. Use this format (or better if is more suitable for suggestions):

### Critical

**1. Hardcoded API key in `auth.ts:42`**

The line `const key = "sk-abc123"` contains a hardcoded API key directly in the source code. This exposes sensitive credentials to anyone with access to the repository and can lead to unauthorized access to external services. Move the key to an environment variable and read it via `process.env.AUTH_KEY` or a secrets manager.

### High

**2. Always-true condition due to truthy empty array in `notify.ts:34`**

The check `if (recipients)` is always true when `recipients` is an empty array `[]`, because empty arrays are truthy in JavaScript. This means a notification is dispatched even when there are no recipients, resulting in a wasted API call and potentially an error downstream when the service receives an empty list. Replace with `if (recipients.length > 0)`.

**3. Missing error handling on fetch in `api.ts:18`**

The call `const res = fetch(url)` at line 18 has no error handling. If the network request fails or returns a non-OK status, the error will propagate unhandled and could crash the process or leave the application in an inconsistent state. Wrap the call in a try/catch block and handle both network errors and non-2xx responses explicitly.

### Medium

**4. Input array mutation in `utils.ts:95`**

The expression `items.sort()` mutates the original `items` array in place. Callers that pass an array to this function do not expect their data to be modified, which can lead to subtle bugs elsewhere. Use `[...items].sort()` or `items.slice().sort()` to sort a copy instead.

### Low

**5. Inconsistent naming in `types.ts:12`**

The function `getData()` at line 12 uses a generic name that does not match the naming convention used elsewhere in the codebase (e.g. `fetchUser`, `fetchOrders`). Rename it to `fetchData` (or a more descriptive name) for consistency and clarity.

Rules:
- Number issues sequentially across all severities (1, 2, 3, ...)
- Skip a severity section entirely if no issues were found for it
- Each issue paragraph must include: the title with issue number, a description of what is wrong and why it matters, and a concrete suggestion for how to fix it
- Reference the exact file, line number, and code expression in each issue
- At the end, show a summary: `Found X issues: Y critical, Z high, N medium, M low`

## Posting Comments

After presenting the full report, ask the user which issues they want posted as comments on the PR. Let them pick by number or say "all" / "none".

Once the user has selected which issues to post, delegate the posting to the `pr-commenter` agent. Pass it:
- The PR URL
- The **full diff output** you already fetched (do NOT make the agent re-fetch it)
- The list of selected issues, each containing:
  - `path`: relative file path
  - `line`: the line number in the new file
  - `line_content`: the **exact code text** at that line as it appears in the diff (e.g. `"if (linkedLoadout && addedMaterials) {"`)
  - `body`: the comment text

For `line_content`, copy the literal line from the diff output (without the `+`/`-`/` ` prefix). This lets the agent grep the diff to confirm the correct line before posting.

The `pr-commenter` agent handles all GitHub API interaction (fetching commit SHA, resolving diff positions, building the JSON payload, and submitting the review). You do NOT need to do any of that yourself.

### Comment style rules

Write comments that are structured, specific, and actionable. Each comment should:

1. **Name the problem** -- reference the specific variable, value, or expression that's wrong and explain *why* it's wrong.
2. **Explain the consequence** -- what happens at runtime because of this issue.
3. **Suggest a fix** -- include a concrete code snippet in a fenced code block when applicable.

Use inline code backticks for variable names, values, and short expressions to make them stand out. Keep the tone direct and collegial -- like a teammate who understands the code.

Do NOT use markdown headers, bullet lists, or severity labels/tags in the comments.

Example of a good comment:

> Bug: `addedMaterials` is an array, and an empty array `[]` is truthy -- this condition is always true when `linkedLoadout` is defined, even when no materials were actually added or removed.
>
> The early return inside `sendLoadoutMaterialsUpdatedNotification` prevents a notification from being created, but this still triggers an unnecessary async call to fetch coordinators.
>
> Should be:
> ```ts
> if (linkedLoadout && (addedMaterials.length > 0 || removedMaterials.length > 0)) {
> ```

---

**IMPORTANT**: Never approve code with security vulnerabilities!
