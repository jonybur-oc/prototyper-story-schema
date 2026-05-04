# Locus Story Schema

**An open specification for machine-readable user stories.**

Version: `1.1` — Stable  
License: MIT  
Schema URL: `https://locus.dev/schema/v1.json`  
Canonical writer: [Locus](https://locus.dev)  
Any tool may read.

---

## What is it?

The Prototyper Story Schema is a structured format for writing user stories that both humans and tools can read. A story contract is the source of truth for product intent — above code, below business goals.

Most user stories live in Jira tickets, Notion pages, or plain text files. They're written for people. This schema is written for people *and* tools — your AI coding assistant, your test generator, your CI pipeline.

---

## Quick example

```yaml
# yaml-language-server: $schema=https://locus.dev/schema/v1.json
schema_version: "1.1"
project:
  name: "Leave Management System"
stories:
  - story_id: US-01
    title: Employee can submit a leave request
    description: >
      Show a form with date range picker and reason field. On submit,
      create the request and show it in 'Pending' status.
    section: Leave Requests
    status: implemented
    acceptance_criteria:
      - Form shows date range picker and reason field
      - On submit, request appears in Pending status
      - Confirmation message shows the new request ID
    pr_refs:
      - "https://github.com/org/repo/pull/42"
```

---

## Specification

Full spec: [SCHEMA.md](./SCHEMA.md)  
JSON Schema: [`schema/v1.1/stories.schema.json`](./schema/v1.1/stories.schema.json)  
Canonical URL: `https://locus.dev/schema/v1.json`  
GitHub raw (fallback): `https://raw.githubusercontent.com/jonybur/locus-story-schema/main/schema/v1.1/stories.schema.json`

### Story object (summary)

| Field | Type | Required | Notes |
|---|---|---|---|
| `id` | UUID string | ✅ | Stable. Never changes, even if title changes. |
| `story_id` | string | ✅ | Human-readable ID, e.g. `US-01`. May change. |
| `title` | string | ✅ | Short, active-voice description. |
| `description` | string | ✅ | Observable behaviour. Implementation-agnostic. |
| `section` | string | ✅ | Feature area grouping, e.g. `Auth`, `Dashboard`. |
| `status` | enum | ✅ | `not-implemented` \| `partial` \| `implemented` \| `stale` |
| `priority` | 0–3 | ✅ | 0=urgent, 1=high, 2=medium, 3=low |
| `assignee` | string \| null | — | Username or email. |
| `reviewer` | string \| null | — | Username or email. |
| `created_at` | ISO 8601 | ✅ | |
| `updated_at` | ISO 8601 | ✅ | |
| `version` | integer | ✅ | Increments on every change. Starts at 1. |
| `pr_refs` | string[] | — | GitHub PR URLs implementing this story. |
| `jira_key` | string \| null | — | Linked Jira issue key, e.g. `PROJ-123`. |

---

## Who reads this format

| Tool | How |
|---|---|
| [Prototyper](https://prototyper.app) | Canonical writer and reader |
| Claude Code | MCP server (`@prototyper/mcp-server`) + agent template (`.claude/agents/story-implementer.md`) |
| Cursor | MCP server — stories as structured context for the agent fleet |
| Windsurf | MCP server — stories as the plan step before Devin handoff |
| GitHub Actions | `prototyper/audit` action — validates PR coverage against stories |
| CI pipelines | JSON export for automated gap detection |
| Documentation generators | Stories → user-facing feature docs |

---

## Using the schema

### With Prototyper (recommended)

Prototyper writes and manages stories in this format automatically. Sign up at [prototyper.app](https://prototyper.app), then export:

```bash
npm install -g prototyper
prototyper export --format json > stories.json
prototyper export --format yaml > stories.yaml
prototyper export --format claude-code > CLAUDE.md
```

#### Option C — CLAUDE.md export (no MCP server required)

If you don't want to run an MCP server, you can export your active stories to a `CLAUDE.md` file. Claude Code reads `CLAUDE.md` automatically at startup, so the agent gets story context without any server configuration.

```bash
prototyper export --format claude-code > CLAUDE.md
```

This generates a `CLAUDE.md` containing your active sprint's stories as structured context. The agent sees story titles, descriptions, and status before writing any code.

**When to use CLAUDE.md export vs. MCP:**

| | CLAUDE.md export | MCP server |
|---|---|---|
| Setup | Zero — just a file in the repo | `claude mcp add prototyper ...` |
| Story Guard (per-action scope check) | ❌ Not available | ✅ via PostToolUse hooks |
| Real-time story updates | ❌ Static snapshot | ✅ Live from Prototyper |
| Works without running server | ✅ Yes | ❌ Server must be running |
| Best for | Quick start, read-only context | Full Story Guard + audit |

**Hybrid pattern (recommended for teams moving to MCP):**
Use the CLAUDE.md export for planning context (zero friction) and add the MCP server when you're ready for Story Guard (per-action scope checking). The two are compatible — `CLAUDE.md` provides startup context; `get_active_story()` provides the live PostToolUse contract.

```bash
# Export once to seed CLAUDE.md planning context
prototyper export --format claude-code > CLAUDE.md

# Add MCP server when ready for Story Guard
claude mcp add prototyper \
  --env PROTOTYPER_API_KEY=your_api_key \
  -- npx -y @prototyper/mcp-server
```

Commit `CLAUDE.md` to your repo. Re-run the export command to refresh it when stories change.

---

### Without Prototyper

Write any valid JSON or YAML that matches the schema and passes validation:

```bash
npx @prototyper/validate stories.json
```

### JSON Schema (for editors and CI)

The canonical schema URL is:

```
https://locus.dev/schema/v1.json
```

GitHub raw fallback (use this until locus.dev is live):

```
https://raw.githubusercontent.com/jonybur/locus-story-schema/main/schema/v1.1/stories.schema.json
```

#### Editor integration

**JSON** — add the `$schema` field for autocomplete and inline validation:

```json
{
  "$schema": "https://locus.dev/schema/v1.json",
  "schema_version": "1.1",
  "project": { "name": "My Project" },
  "stories": []
}
```

**YAML** — add the `yaml-language-server` header comment:

```yaml
# yaml-language-server: $schema=https://locus.dev/schema/v1.json
schema_version: "1.1"
project:
  name: My Project
stories: []
```

This enables inline validation and autocomplete in VS Code (with the [YAML extension](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml)), Neovim (via yamlls), and any editor that supports JSON Schema.

**VS Code settings** — apply schema globally to all `stories.yaml` files:

```json
// .vscode/settings.json
{
  "yaml.schemas": {
    "https://locus.dev/schema/v1.json": "**/stories.yaml"
  }
}
```

#### CI validation

Validate locally with [ajv-cli](https://github.com/ajv-validator/ajv-cli):

```bash
npx ajv-cli validate -s schema/v1.1/stories.schema.json -d stories.json
```

Or use the GitHub Action:

```yaml
- uses: jonybur-oc/locus-audit-action@v1
  with:
    stories-path: stories.yaml
```

---

## API

The canonical source is the Prototyper API:

```
GET https://api.prototyper.app/v1/projects/:project_id/stories
Authorization: Bearer <token>
```

Returns the full root JSON object (schema_version + project + stories array).

---

## Versioning

Schema versions follow semver. Breaking changes bump the major version.

Readers should:
- Check `schema_version`
- Handle unknown fields gracefully (ignore, don't error)
- Surface an error if `schema_version` major is higher than supported

**Current: `1.1` — Stable.** Full changelog in [SCHEMA.md](./SCHEMA.md#changelog).

---

## Claude Code integration

### Option A — Claude Code Plugin (recommended)

Install the Prototyper plugin directly from Claude Code:

```bash
# Add the self-hosted plugin marketplace
/plugin marketplace add jonybur/locus-story-schema

# Install the plugin
/plugin install prototyper
```

This installs:
- **MCP server**: `@prototyper/mcp-server` — stories as MCP resources
- **Agent**: `story-implementer` — implements to acceptance criteria, marks done
- **Skills**: `write-story`, `sync-stories` — help writing and syncing stories
- **Command**: `/prototyper:sync` — pull open stories into your session

Set your environment variables:

```bash
export PROTOTYPER_API_KEY=your_api_key
export PROTOTYPER_PROJECT_ID=your_project_id
```

Then run `/prototyper:sync` to load your sprint's open stories.

### Option B — Manual setup

```bash
# 1. Install the MCP server
claude mcp add prototyper \
  --env PROTOTYPER_API_KEY=your_api_key \
  -- npx -y @prototyper/mcp-server

# 2. Copy the agent template
mkdir -p .claude/agents
curl -o .claude/agents/story-implementer.md \
  https://raw.githubusercontent.com/jonybur/locus-story-schema/main/.claude/agents/story-implementer.md
```

The agent will appear in Claude Code's `/` picker. When invoked, it reads your sprint backlog from Prototyper, implements to acceptance criteria, and marks stories as implemented when done. See [`.claude/agents/story-implementer.md`](./.claude/agents/story-implementer.md) for the full template.

---

## Contributing

We welcome PRs for:
- Bug fixes in the specification (ambiguous language, contradictions)
- New optional fields with clear use cases
- Validator improvements
- Integration examples

See [CONTRIBUTING.md](./CONTRIBUTING.md).

---

## License

MIT. Use it, read it, build on it.
