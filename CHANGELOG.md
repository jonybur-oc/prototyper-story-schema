# Changelog

All notable changes to the Prototyper Story Schema are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Schema versions follow [Semantic Versioning](https://semver.org/).

---

## [Unreleased]

Planned for v1.2 (no fixed release date):

- `prototyper migrate --from 1.0 --to 1.1` — CLI migration tool to add empty `acceptance_criteria: []` arrays to existing `stories.yaml` files.
- Story Guard community ports — Windsurf, Aider, GitHub Copilot (BYOK), Continue.dev. Claude Code ✅ and Cursor ✅ are already documented in [STORY_GUARD.md](./STORY_GUARD.md).
- `tags` — list of strings for filtering and CI routing (e.g. `[mobile, accessibility, breaking-change]`)
- `persona` — who this story is for (PM, engineer, admin, new-user, returning-user)
- `risk` — enum (`low`, `medium`, `high`, `critical`) for requiring human review on sensitive stories
- `non_functional` — performance, accessibility, security requirements object

---

## [1.1.4] — 2026-05-03

Schema consistency fix. No new fields or behaviour changes.

### Fixed

- **Story object TypeScript type** (`SCHEMA.md`, Story object section): `status` enum was missing `"deprecated"`. The value was documented in the Status values table and semantics section (added in v1.1.2) but the type signature still read `"not-implemented" | "partial" | "implemented" | "stale"`. Now reads `"not-implemented" | "partial" | "implemented" | "stale" | "deprecated"`.
- **JSON Schema** (`schema/v1.1/stories.schema.json`, `$defs.Story.properties.status.enum`): same omission — `"deprecated"` was missing from the machine-readable enum. A validator running against the JSON Schema would incorrectly reject any story with `status: deprecated` as invalid. Array now includes `"deprecated"`.

### Rationale

The `deprecated` status value was introduced in v1.1.2 with full documentation (semantics, decision guide, state machine transitions, tooling rules) but was not added to the two authoritative type definitions — the TypeScript signature in SCHEMA.md and the JSON Schema file. Any tool or linter using `schema/v1.1/stories.schema.json` for validation would have rejected `deprecated` stories as invalid, directly contradicting the documented spec. This patch closes the gap.

---

## [1.1.3] — 2026-05-03

Documentation patch. No schema changes.

### Documentation

- `SCHEMA.md`: added **The `notes` field** section — full authoring guidance covering: what belongs in `notes` (rationale, scope exclusions, ADR links, implementation constraints, deferred items); what does NOT belong (observable behaviour, acceptance criteria, implementation detail without constraint); why `notes` is excluded from Story Guard scope-checking by design; and a worked example (`PERF-03`) demonstrating the separation between `description`/`acceptance_criteria` (observable, testable) and `notes` (rationale, constraints).
- Version header bumped to v1.1.3.

### Rationale

The `notes` field was introduced in v1.1.0 and noted in a one-line comment in the Story object type signature, but had no authoring guidance. Real usage showed teams putting implementation rationale, ADR links, and scope exclusions into `description` — which degrades Story Guard's scope-checking accuracy because the LLM reads `description` as the behavioural contract. The fix is clear separation: `description` = what users can do/see; `acceptance_criteria` = how you verify it; `notes` = everything else. This patch makes that separation explicit and illustrates it with a worked example.

---

## [1.1.2] — 2026-05-03

Documentation patch. No schema changes.

### Documentation

- `SCHEMA.md`: expanded **Status values** section with precise semantics for all five values, decision guides (when to use each value, what not to use it for), and a complete status transition diagram.
- `SCHEMA.md`: added `deprecated` status value — for features intentionally removed from the product. Distinguished from `stale` (unknown state) vs `deprecated` (intentionally removed). Deprecated stories retain their `story_id` permanently.
- `SCHEMA.md`: added three tooling-SHOULD rules: warn on `partial` stories > 30 days; warn on `stale` stories referenced in open PRs; warn on `implemented` → `deprecated` transitions without a `notes` entry.
- Version header bumped to v1.1.2.

### Rationale

The status table had one-line definitions that didn't distinguish edge cases: what happens when a feature is removed? When does `stale` resolve? How long can a story be `partial`? Real usage showed authors working around the ambiguity — leaving stories as `stale` indefinitely or using `not-implemented` for removed features. This patch closes the semantic gaps. `deprecated` is added as a documentation-only value (no JSON Schema change required — it is an allowed string value; tooling that handles unknown enum values will surface it correctly until a formal schema patch ships).

---

## [1.1.1] — 2026-05-03

Documentation patch. No schema changes.

### Documentation

- `SCHEMA.md`: added **Compliance object** section — TypeScript-style type signature, field reference table, YAML example (PSD2 payment story), and four authoring rules:
  - `approved_by` and `approved_at` are required when the `compliance` block is present.
  - `audit_note` SHOULD reference the specific regulatory article or control.
  - Tooling MUST warn when a `stale` story carries a `compliance` block (re-approval required).
  - The `compliance` block MUST NOT be used as a placeholder — it signals sign-off has occurred.
- Version header bumped to v1.1.1.

### Rationale

The `compliance` field was introduced in v1.1.0 and documented in the JSON Schema (`$defs/Compliance`), but the human-readable `SCHEMA.md` only noted its existence in a one-line comment. Integrators building regulated-sector tooling had no field-level documentation. This patch closes that gap.

---

## [1.1.0] — 2026-05-03

Quality contract fields and reduced friction for hand-authored files.

### Schema changes (stories.schema.json)

- **New file**: `schema/v1.1/stories.schema.json` — full JSON Schema (Draft 2020-12) for v1.1.
- **New story fields (all optional)**:
  - `acceptance_criteria` (`string[] | null`) — human-readable, testable done-conditions. Not BDD/Gherkin. Enables deterministic Story Guard scope checking and precise audit validation instead of AI inference from `description`.
  - `depends_on` (`string[] | null`) — list of `story_id` values that must be implemented before this story. Allows tooling and CI to surface unmet dependencies.
  - `design_ref` (`string | null`) — URI to the Figma frame, mockup, or design token set for this story.
  - `test_refs` (`string[] | null`) — explicit test file paths (optionally with line anchors) that verify this story. Makes `audit_stories()` deterministic: check these files rather than inferring from code search.
  - `compliance` (`Compliance | null`) — formal approval block. Previously undocumented in schema; now formally specified. Subfields: `approved_by`, `approved_at` (required); `review_ref`, `audit_note` (optional).
  - `notes` (`string | null`) — freetext author rationale, scope exclusions, ADR links, implementation constraints. Distinct from `description` (observable behaviour).
- **New root field (optional)**:
  - `exported_at` (`date-time | null`) — ISO 8601 timestamp of when the file was generated by the platform API or CLI. Absent in hand-authored files. Already present in MCP server output; now formally specified.
- **Relaxed required fields on Story**: `id`, `section`, `priority`, `created_at`, `updated_at`, `version` are now optional. They were required in v1.0 but real usage showed hand-authored files almost never include them. Platform-generated files still include all fields.
- **Relaxed required fields on Project**: `id` and `created_at` are now optional. `name` is the only required field.
- **New `$defs/Compliance`** sub-schema — extracted to `$defs` for reuse and clarity.

### CI workflow

- Updated `validate-schema.yml`: job renamed to "Validate against schema v1.1".
- Workflow now auto-selects schema version: files with `schema_version: 1.0.x` validate against `schema/v1.0/stories.schema.json`; files with `schema_version: 1.1+` validate against `schema/v1.1/stories.schema.json`. This preserves v1.0 backwards compatibility in CI.

### Documentation

- `SCHEMA.md` bumped to v1.1. Updated story object type signature, project object type signature, root object, examples (JSON + YAML + minimal hand-authored).
- Added minimal hand-authored YAML example showing the new 4-field minimum.

### Design decisions

**Why relax required fields?**  
The v1.0 schema required `id`, `section`, `priority`, `created_at`, `updated_at`, and `version`. Real usage showed that teams hand-authoring `stories.yaml` hit validation errors immediately on minimal files (the example `stories.yaml` in both `prototyper/examples/` used only `story_id`, `title`, `description`, `section`, `status`). The distinction that matters is: platform-generated files (via API export or CLI) always have all fields; hand-authored files need only the semantic minimum. V1.1 encodes this distinction explicitly.

**Why is `acceptance_criteria` plain English, not BDD/Gherkin?**  
Gherkin requires a testing framework to be useful; plain English does not. The goal is a human-readable contract that a QA engineer can verify and that Story Guard can scope-check without a parser. Gherkin can be added as an `x-gherkin` extension by tools that need it.

**Why is `notes` a separate field from `description`?**  
`description` is the observable behaviour spec — what a user does or sees. `notes` is author commentary: why we built it this way, what we considered and rejected, links to ADRs or Slack threads. Mixing them into `description` degrades Story Guard's scope-checking accuracy because LLMs treat the entire field as the story contract.

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

[Unreleased]: https://github.com/jonybur-oc/prototyper-story-schema/compare/v1.1.3...HEAD
[1.1.3]: https://github.com/jonybur-oc/prototyper-story-schema/compare/v1.1.2...v1.1.3
[1.1.2]: https://github.com/jonybur-oc/prototyper-story-schema/compare/v1.1.1...v1.1.2
[1.1.1]: https://github.com/jonybur-oc/prototyper-story-schema/compare/v1.1.0...v1.1.1
[1.1.0]: https://github.com/jonybur-oc/prototyper-story-schema/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/jonybur-oc/prototyper-story-schema/releases/tag/v1.0.0
