---
name: agent-session-migration
description: "Use this skill when moving or re-attaching OpenCode agent sessions to a different project/workspace, especially when a session was accidentally started from $HOME or the wrong directory."
---

# Agent Session Migration

## Purpose

Use this skill when an OpenCode session exists, but it is attached to the wrong project, directory, or workspace context.

Common cases:

- A session was started from `/home/rswift` and should belong to a repo under `~/dev/...`.
- A long-running session needs to be resumed from a more specific project context.
- An exported OpenCode session should be imported into the current project.
- The session list shows the right title, but resume/project grouping is wrong.

This skill is primarily for OpenCode. Do not assume other agents use the same SQLite schema or import semantics.

## Safety Rules

1. Never rewrite OpenCode session storage without first creating a database backup.
2. Prefer `opencode export` and `opencode import` over direct SQLite edits when possible.
3. Only update the target session row, never bulk-update sessions by title.
4. Do not delete sessions unless the user explicitly asks.
5. Do not modify message, part, event, permission, or tool-output rows unless there is a specific migration need.
6. If the target session is currently open in another OpenCode process, warn that the running process may keep stale in-memory context until reopened.

## Key Paths

OpenCode's local data is usually here:

```bash
~/.local/share/opencode/opencode.db
```

Local OpenCode config and skills are usually here:

```bash
~/.config/opencode/
~/.config/opencode/skills/
```

Use the actual filesystem if these paths differ. Do not hardcode them in scripts intended for other machines.

## Workflow

### 1. Identify the session

List sessions:

```bash
opencode session list
```

Pick the session ID from the title and timestamp. Prefer exact session IDs over title matching.

Export the session to a temporary JSON file:

```bash
opencode export <session_id> > /tmp/opencode/session-export.json
```

Inspect metadata:

```bash
jq '.info' /tmp/opencode/session-export.json
```

Important fields:

- `id`
- `projectID`
- `directory`
- `path`
- `title`

### 2. Identify the target project

Run the import from the directory that should own the session. OpenCode's import command uses the current project from `process.cwd()`.

If the project already exists in the OpenCode database, confirm it:

```bash
sqlite3 ~/.local/share/opencode/opencode.db \
  "select id, worktree, name from project where worktree like '%<repo-name>%' order by time_updated desc;"
```

If the project does not exist yet, opening/running OpenCode from that directory may be needed to initialize the project record.

### 3. Back up the database

Before import or manual SQLite updates:

```bash
sqlite3 ~/.local/share/opencode/opencode.db \
  ".backup '$HOME/.local/share/opencode/opencode.db.before-session-migration-$(date +%Y%m%d-%H%M%S)'"
```

This is non-negotiable. The database contains all local OpenCode session state.

### 4. Re-attach via import

From the target project directory:

```bash
opencode import /tmp/opencode/session-export.json
```

OpenCode import behavior, as observed in OpenCode `1.14.33`:

- It reads the exported session JSON.
- It replaces `projectID` with the current `Instance.project.id`.
- It inserts the session row or updates an existing row on session ID conflict.
- On conflict, it updates `project_id` only.
- It does not necessarily update existing `directory` or `path` fields.
- It inserts missing messages/parts with conflict-do-nothing behavior.

Because of this, import is usually enough to move the session into the right project grouping, but it may leave stale directory metadata.

### 5. Verify session attachment

Check the session row:

```bash
sqlite3 -header -column ~/.local/share/opencode/opencode.db \
  "select s.id, s.project_id, p.worktree, s.directory, s.path, s.title
   from session s
   left join project p on p.id=s.project_id
   where s.id='<session_id>';"
```

Expected:

- `project_id` points to the target project.
- `worktree` is the target repo/directory.
- `directory` and `path` ideally match the target directory.

### 6. Fix stale directory metadata if needed

If import updated `project_id` but left `directory` and `path` pointing to the old location, update only that session row:

```bash
sqlite3 ~/.local/share/opencode/opencode.db \
  "update session
   set directory='<absolute-target-directory>',
       path='<target-path-without-leading-slash>',
       time_updated=strftime('%s','now') * 1000
   where id='<session_id>';"
```

Example:

```bash
sqlite3 ~/.local/share/opencode/opencode.db \
  "update session
   set directory='/home/rswift/dev/personal/claws/hermes',
       path='home/rswift/dev/personal/claws/hermes',
       time_updated=strftime('%s','now') * 1000
   where id='ses_...';"
```

Then verify again with the query from step 5.

## Schema Notes

Useful tables:

- `project`: project/worktree records.
- `session`: session metadata, including `project_id`, `directory`, `path`, and `workspace_id`.
- `session_message`: message records attached to `session_id`.
- `part`: message parts attached to `session_id` and `message_id`.
- `workspace`: optional workspace metadata. Many sessions have an empty `workspace_id`.

Useful schema inspection:

```bash
sqlite3 ~/.local/share/opencode/opencode.db ".tables"
sqlite3 ~/.local/share/opencode/opencode.db "pragma table_info(session);"
sqlite3 ~/.local/share/opencode/opencode.db "pragma table_info(project);"
sqlite3 ~/.local/share/opencode/opencode.db "pragma table_info(workspace);"
```

## Recovery

If a migration goes wrong, stop OpenCode processes first if possible, then restore from the backup:

```bash
cp ~/.local/share/opencode/opencode.db.before-session-migration-YYYYMMDD-HHMMSS \
  ~/.local/share/opencode/opencode.db
```

If WAL/SHM files exist and OpenCode is stopped, remove the stale sidecars only after restoring the main DB:

```bash
rm -f ~/.local/share/opencode/opencode.db-wal ~/.local/share/opencode/opencode.db-shm
```

Only do this for local recovery when OpenCode is not running.

## Reporting Back

When done, report:

- migrated session ID and title
- target project/worktree
- whether `directory` and `path` were updated manually
- backup path
- any caveat about needing to reopen/resume the session
