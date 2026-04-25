# Prototyper Story Schema

**An open specification for machine-readable user stories.**

Version: `1.0` — Stable  
License: MIT  
Canonical writer: [Prototyper](https://prototyper.app)  
Any tool may read.

---

## What is it?

The Prototyper Story Schema is a structured format for writing user stories that both humans and tools can read. A story contract is the source of truth for product intent — above code, below business goals.

Most user stories live in Jira tickets, Notion pages, or plain text files. They're written for people. This schema is written for people *and* tools — your AI coding assistant, your test generator, your CI pipeline.

---

## Quick example

```yaml
schema_version: "1.0"
project:
  id: "550e8400-e29b-41d4-a716-446655440000"
  name: "Leave Management System"
  created_at: "2026-04-01T09:00:00Z"
stories:
  - id: "7f3d9a2b-1c4e-4f8a-b5d6-9e0f1a2b3c4d"
    story_id: "US-01"
    title: "Employee can submit a leave request"
    description: "Show a form with date range picker and reason field. On submit, create the request and show it in 'Pending' status. Show a confirmation message with the request ID."
    section: "Leave Requests"
    status: "implemented"
    priority: 1
    assignee: "jony@example.com"
    created_at: "2026-04-01T09:00:00Z"
    updated_at: "2026-04-15T14:30:00Z"
    version: 3
    pr_refs:
      - "https://github.com/org/repo/pull/42"
```

---

## Specification

Full spec: [SCHEMA.md](./SCHEMA.md)  
JSON Schema: [`schema/v1.0/stories.schema.json`](./schema/v1.0/stories.schema.json)

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

### Without Prototyper

Write any valid JSON or YAML that matches the schema and passes validation:

```bash
npx @prototyper/validate stories.json
```

### JSON Schema (for editors and CI)

The machine-readable JSON Schema is in this repo:

```
https://raw.githubusercontent.com/jonybur-oc/prototyper-story-schema/main/schema/v1.0/stories.schema.json
```

Add the `$schema` field to your `stories.json` for editor autocomplete and validation:

```json
{
  "$schema": "https://raw.githubusercontent.com/jonybur-oc/prototyper-story-schema/main/schema/v1.0/stories.schema.json",
  "schema_version": "1.0",
  "project": { ... },
  "stories": [ ... ]
}
```

Add to your `.vscode/settings.json` for YAML validation:

```json
{
  "yaml.schemas": {
    "https://raw.githubusercontent.com/jonybur-oc/prototyper-story-schema/main/schema/v1.0/stories.schema.json": "stories.yaml"
  }
}
```

Validate locally with [ajv-cli](https://github.com/ajv-validator/ajv-cli):

```bash
npx ajv-cli validate -s schema/v1.0/stories.schema.json -d examples/leave-management.json
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

**Current: `1.0` — Stable.**

---

## Claude Code integration

Prototyper ships a Claude Code subagent template that connects the MCP server to an implementation workflow. Drop it into your project:

```bash
# 1. Install the MCP server
claude mcp add prototyper \
  --env PROTOTYPER_API_KEY=your_api_key \
  --env PROTOTYPER_SUPABASE_URL=your_supabase_url \
  -- npx -y @prototyper/mcp-server

# 2. Copy the agent template
curl -o .claude/agents/story-implementer.md \
  https://raw.githubusercontent.com/jonybur-oc/prototyper-story-schema/main/.claude/agents/story-implementer.md
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
