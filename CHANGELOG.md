# Changelog

All notable changes to the Prototyper Story Schema are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Schema versions follow [Semantic Versioning](https://semver.org/).

---

## [Unreleased]

Planned for v1.1 (no fixed release date — items ship when implemented in the JSON Schema, TypeScript types, and MCP server):

- `acceptance_criteria` — structured array of human-readable, testable conditions per story. Not BDD/Gherkin; not executable. Enables precise Story Guard scope checking and higher-fidelity `audit_stories()` matching.
- `notes` — freetext field for per-story rationale: design decisions, scope exclusions, ADR links, implementation constraints. Distinct from `description` (observable behaviour) and `acceptance_criteria` (testable contract).
- `deprecated` status value — for features intentionally removed from the product. Companion to the existing `stale` computed status.
- `prototyper migrate --from 1.0 --to 1.1` — CLI migration tool to add empty `acceptance_criteria: []` arrays to existing `stories.yaml` files.
- Story Guard community ports — Windsurf, Aider, GitHub Copilot (BYOK), Continue.dev. Claude Code ✅ and Cursor ✅ are already documented in [STORY_GUARD.md](./STORY_GUARD.md).

---

## [1.0.0] — 2026-04-24

Initial stable release of the Prototyper Story Schema.

### Schema (stories.schema.json)

- **Root object**: `schema_version` (string, required), `project` (object, required), `stories` (array, required).
- **Project object**: `id` (UUID, required), `name` (string, required), `created_at` (ISO 8601, required).
- **Story object**: 10 required fields + 4 optional fields. `additionalProperties: false` — no undeclared fields accepted.
  - Required: `id`, `story_id`, `title`, `description`, `section`, `status`, `priority`, `created_at`, `updated_at`, `version`
  - Optional: `assignee`, `reviewer`, `pr_refs`, `jira_key`
- **Status enum**: `not-implemented | partial | implemented | stale`
  - `stale`: computed by `audit_stories()` when source files change after an `implemented` verdict. Not hand-set.
- **Priority**: integer 0–3. `0`=urgent, `1`=high, `2`=medium, `3`=low.
- **Stable IDs**: `id` is a UUID that never changes, even if `title` or `story_id` changes. Used as the stable reference in commit messages, PR descriptions, CI logs, and audit reports. `story_id` is human-readable and may change.
- **Version**: integer, starts at 1, increments on every mutation. Enables drift detection by audit tooling.

### Added (2026-04-24 — initial commit, fc9481f)

- `SCHEMA.md` — full specification with TypeScript-style type signatures and field-level documentation.
- `examples/leave-management.json` — reference example: two stories (implemented + partial), realistic UUIDs and timestamps.
- `examples/leave-management.yaml` — YAML equivalent of the reference example.
- `CONTRIBUTING.md` — contribution guide: bug fixes to the specification, new optional fields (with use-case requirements), validator improvements, integration examples.
- `LICENSE` — MIT.
- `README.md` — quickstart, field table, integration table (Prototyper, Claude Code, Cursor, Windsurf, GitHub Actions), JSON Schema reference, API, versioning contract.

### Added (2026-04-25 — feat(schema): formal JSON Schema, 36f850e)

- `schema/v1.0/stories.schema.json` — machine-readable JSON Schema (Draft 2020-12). Validates the full root object including nested `project` and `stories` array. Used in editor autocomplete, CI workflows, and by the MCP server.
- `examples/leave-management.yaml` — YAML example updated to include all required fields.

### Added (2026-04-25 — feat(claude-code): story-implementer agent, 640a0bb)

- `.claude/agents/story-implementer.md` — Claude Code sub-agent template. Reads active sprint from Prototyper, implements to story `description`, marks story `implemented` on completion. Appears in Claude Code's `/` agent picker.
- `.claude/commands/check-story.md` — `/check-story` slash command: calls `get_active_story()` on demand and surfaces the story's description as a scope reminder.
- `CLAUDE.md` — project context for Claude Code: schema overview, tool list, Story Guard setup reminder. Read automatically by Claude Code at session start.

### Added (2026-04-26 — feat(plugin): Claude Code plugin structure, d3bc9f5)

- `.claude/settings.json` — Claude Code plugin config. Wires `get_active_story()` as a `PostToolUse` hook: after every file write, bash command, or MCP tool call, the hook calls `get_active_story()` and the agent checks its action scope against the active story's `description`.
- `.claude-plugin/` — self-hosted plugin manifest. Enables `/plugin marketplace add jonybur-oc/prototyper-story-schema` install path.
- `.mcp.json` — MCP server config. Points to `@prototyper/mcp-server`.
- `marketplace.json` — plugin marketplace entry for Claude Code.
- `skills/` — write-story and sync-stories skill templates.

### Added (2026-04-30 — docs: CLAUDE.md export guide, 36454cd)

- `README.md` updated: CLAUDE.md export documented as a first-class zero-setup path alongside the MCP server. Comparison table (CLAUDE.md export vs. MCP server). Hybrid pattern documented: CLAUDE.md for planning context + `get_active_story()` for per-action Story Guard. The two are compatible.

### Added (2026-04-30 — ci: GitHub Actions validation workflow, e606683)

- `.github/workflows/validate-stories.yml` — validates `stories.yaml` against the JSON Schema v1.0 on every push or PR touching `stories.yaml`. Requires no Prototyper account; fetches schema from GitHub raw URL. Uses `ajv-cli` + `ajv-formats` + `js-yaml`.

### Added (2026-04-30 — docs: STORY_GUARD.md, 54d74d6)

- `STORY_GUARD.md` — standalone community pattern document for Story Guard: per-action scope checking using `get_active_story()` as a PostToolUse hook. Complete implementation guides for:
  - **Claude Code** — `.claude/settings.json` PostToolUse hook config (JSON, ready to paste).
  - **Cursor** — `.cursor/rules` implementation: read `stories.yaml`, find active story, check action scope before proceeding.
  - **Option 2 (no Prototyper account)** — local `stories.yaml` + `check-story.md` slash command, no web app required.
  - Community contribution matrix: Windsurf, Aider, GitHub Copilot (BYOK), Continue.dev — open for community PRs.

### Added (2026-05-01 — docs(story-guard): TL;DR entry point, 8addde0)

- `STORY_GUARD.md` — 3-line TL;DR block added at the top: one-sentence value prop and anchor nav links to Claude Code, Cursor, and local-file sections. Reduces time-to-find for cold GitHub visitors.

---

## Versioning contract

- **Patch** (e.g. 1.0.1): bug fixes, specification clarifications, documentation improvements. No schema changes.
- **Minor** (e.g. 1.1.0): new optional fields, new status values, new MCP tools. Backward compatible. Readers of v1.0 files can read v1.1 files without changes if they handle unknown fields gracefully.
- **Major** (e.g. 2.0.0): breaking changes — renamed fields, removed fields, changed field types. Migration tooling ships before or with any major version.

Current: **1.0.0 — Stable.**

---

## Design decisions (recorded here for stability)

**Why are required fields not restricted to the three most obvious ones (id, title, description)?**  
`section`, `priority`, `created_at`, `updated_at`, and `version` are required because audit tooling and agent context depend on them being present and typed. Making them optional would produce unreliable agent behaviour when they're missing.

**Why is `stale` a computed status, not hand-set?**  
`stale` is emitted by `audit_stories()` when `updated_at` predates recent changes to related source files. A human setting `stale` manually provides no signal that tooling can act on; a computed `stale` signals that re-review is warranted.

**Why is `acceptance_criteria` not in v1.0?**  
Shipping a stable v1.0 with a clean minimal field set is more important than completeness. The `description` field carries behavioural intent in v1.0 and is sufficient for Story Guard's scope-checking function. When `acceptance_criteria` lands in v1.1, Story Guard gains a more precise contract. The v1.0 → v1.1 migration is a mechanical addition of empty arrays — not a breaking change.

**Why YAML and not JSON as the canonical format?**  
Both are valid. YAML is the recommended authoring format because multiline strings (`>` block scalar) are significantly more ergonomic for `description` fields than JSON escaped strings. The schema is validated against a JSON Schema definition — machine consumption is sound; human authoring stays readable.

---

[Unreleased]: https://github.com/jonybur-oc/prototyper-story-schema/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/jonybur-oc/prototyper-story-schema/releases/tag/v1.0.0
