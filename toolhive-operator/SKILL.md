---
name: toolhive-operator
description: "Use this skill when configuring ToolHive itself: installing MCP servers, registering clients, managing groups, adding skills, handling auth/secrets, and debugging ToolHive workflows across local AI clients."
---

# ToolHive Operator

## Purpose

Use this skill when the task is about operating ToolHive rather than just using an MCP server through a client.

Examples:

- installing an MCP server with `thv run`
- organizing workloads into groups
- registering clients with `thv client register`
- installing skills with `thv skill install`
- wiring credentials or secrets for MCP servers
- debugging why a client cannot see an MCP or skill
- deciding how to structure ToolHive workloads for multiple accounts or environments

## Core Rules

1. Prefer ToolHive as the source of truth for MCP client integration.
2. Prefer grouping related workloads (for example `google-cloud`) instead of putting everything into `default`.
3. Treat client skill directories as deployment targets, not authoring sources.
4. Use a hosted Git repo, registry, or remote OCI reference for ToolHive skill installs. Do not assume local build artifacts can be installed directly through `thv skill install`.
5. When available, use the ToolHive docs MCP tools `query_docs` and `get_chunk` before guessing at current behavior.

## Documentation First

If the ToolHive docs MCP is available, use it as the preferred source of truth:

- `query_docs`
- `get_chunk`

Use it for:

- command behavior
- skill install/build limitations
- client compatibility
- group behavior
- auth and secrets workflows

If the docs MCP is not available, fall back to:

- `thv <command> --help`
- official ToolHive docs

## MCP Workflow

### Installing servers

Use `thv run` for:

- registry entries
- container images
- remote MCP URLs

After install, always verify with:

```bash
thv list
thv status <name>
```

If applicable, also inspect logs:

```bash
thv logs <name>
```

### Groups

Use groups for clear separation of concerns.

Examples:

- `default` for general-purpose MCPs
- `google-cloud` for GCP servers
- separate workloads per account/context when auth differs

Good pattern:

- `cloud-run-personal`
- `cloud-run-work`

Avoid using one workload for multiple credential contexts when env vars or auth defaults differ.

### Client registration

Register clients explicitly:

```bash
thv client register <client>
thv client register <client> --group <group>
```

Remember that a client can be registered to multiple groups.

Use these to verify:

```bash
thv client list-registered
thv client status
```

## Skill Workflow

### Installing skills

Use `thv skill install` from:

- registry name
- Git URL
- remote OCI reference

Examples:

```bash
thv skill install "git://github.com/org/repo@main#path/to/skill" --clients all
thv skill install my-skill --clients claude-code,opencode
```

### Authoring custom skills

Do not author custom skills inside client directories like:

- `~/.config/opencode/skills`
- `~/.claude/skills`

Instead, keep them in a neutral source repo, for example:

- `~/dev/skills`

Then install them into ToolHive from a hosted Git repository.

### Important limitation

ToolHive can build local skills into the local OCI store with:

```bash
thv skill build ./my-skill
```

But do not assume the built local OCI artifact can be installed directly with `thv skill install`. In practice, use a hosted Git repo or published registry reference for reliable installation.

### Verification

Always check:

```bash
thv skill list
thv skill info <skill-name>
```

If a client does not see a skill, check the client’s installed skill directory on disk.

## Auth and Secrets

### General

Prefer dedicated credentials per integration rather than reusing your main CLI identity when possible.

Examples:

- dedicated GitHub token for GitHub MCP
- separate Google Cloud workloads per account/context

### ToolHive secrets

Use ToolHive secrets for reusable credentials:

```bash
thv secret setup
thv secret set <name>
thv secret list
```

Then inject them when supported.

### Google Cloud guidance

For Google Cloud MCP servers:

- separate personal and work contexts into separate workloads
- prefer safer auth boundaries for high-privilege environments
- use explicit default project and region per workload

## Troubleshooting Checklist

When something does not show up in a client:

1. Check workload/skill status in ToolHive.
2. Check client registration and group membership.
3. Check the generated client config on disk.
4. Restart the client.
5. Inspect MCP tools via ToolHive debugging commands when needed.

Useful commands:

```bash
thv list
thv status <name>
thv client list-registered
thv client status
thv skill list
thv mcp list tools --server <url> --transport streamable-http
```

## Recommended Defaults

- Use hosted Git repos as the source for custom ToolHive skills.
- Use groups to separate domains like `google-cloud`.
- Keep `default` for broadly useful shared MCPs.
- Verify every change end-to-end in both ToolHive and at least one client.
