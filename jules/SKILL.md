---
name: jules
description: "Delegates tasks to Google Jules, an autonomous AI coding agent. Suggest Jules for time-consuming, well-defined, or parallelizable work. Always ask before sending."
---

# Jules CLI Skill

Interact with [Jules](https://jules.google.com), Google's autonomous AI coding agent, through its CLI. Jules runs asynchronously in the cloud on GitHub repositories.

## When to Suggest Jules

Proactively suggest delegating work to Jules when you notice tasks that are:

- **Time-consuming but well-defined** — large refactors, adding tests across many files, migrating APIs
- **Parallelizable** — multiple independent changes that can run as separate sessions
- **Async-friendly** — work the user doesn't need to watch happen in real-time
- **Tedious but mechanical** — renaming patterns, adding error handling, updating dependencies

Always suggest, never auto-delegate. Say something like: "This looks like a good candidate for Jules - want me to send it?"

Do NOT suggest Jules for:
- Quick, simple changes you can do yourself in seconds
- Tasks requiring real-time user interaction or iterative feedback
- Work that depends on local-only state (files not in the repo, environment variables, running services)

## CLI Quick Reference

```bash
# Send a task (infers repo from current directory)
jules new "<task description>"

# Send to a specific repo
jules new --repo owner/repo "<task description>"

# Start parallel sessions for the same task
jules new --parallel 3 "<task description>"

# List sessions
jules remote list --session

# Pull completed results and apply patch
jules remote pull --session <session_id> --apply

# Interactive dashboard
jules
```

## Writing Good Task Prompts

1. **State the goal clearly** — what should be different when Jules is done?
2. **Scope it** — which files, modules, or areas of the codebase?
3. **Provide constraints** — testing requirements, style guidelines, things to avoid
4. **Give examples** if the pattern isn't obvious

Good: `"Add unit tests for all exported functions in src/utils/. Use vitest. Co-locate test files with .test.ts suffix."`
Bad: `"Fix the bugs"` - too vague, no scope

## Workflow

When the user agrees to delegate a task to Jules:

1. **Verify prerequisites** — confirm git repo with GitHub remote, `jules` is available, user is authenticated
2. **Check repo state** — warn about uncommitted changes or unpushed branches (Jules works from the remote)
3. **Craft the prompt** — analyze context, write a specific scoped prompt, show it to the user for approval
4. **Send** — `jules new "<approved prompt>"`
5. **Report** — show session info, remind about `jules remote list --session` and `jules remote pull --session <id> --apply`

## Rules

- **Always show the prompt before sending** — never send without user approval
- **Never send credentials or secrets** in the session prompt
- **Warn about local-only state** — uncommitted changes, untracked files, env vars
- **One clear task per session** — don't bundle unrelated work
- **Suggest parallel sessions** for multiple independent tasks
