# Locus Story Schema — Specification

**Version:** 1.1  
**Status:** Stable  
**Date:** 2026-05-03

---

## Overview

A story contract is a machine-readable description of what a piece of software should do. It is the source of truth for product intent — above code, below business goals.

Stories are the unit of truth. Designs, code, tickets, and documentation are derived from stories, not the other way around.

---

## Format

Stories are stored as JSON (canonical) or YAML (human-friendly alias). Both are valid. The schema is versioned; readers should check the `schema_version` field and handle unknown versions gracefully.

---

## Root object

```typescript
{
  "schema_version": "1.1",       // Required. Semver string.
  "project": Project,             // Required.
  "stories": Story[],             // Required. May be empty.
  "exported_at": "string"         // Optional. ISO 8601 timestamp of export. API/CLI-generated only.
}
```

---

## Project object

```typescript
{
  "id": "string",          // Stable UUID. Optional in hand-authored files.
  "name": "string",        // Human name. Required.
  "created_at": "string"   // ISO 8601 timestamp. Optional in hand-authored files.
}
```

---

## Story object

```typescript
{
  // ── Identity ──────────────────────────────────────────────────────────────
  "id": "string",          // Stable UUID. Never changes, even if title changes. Optional in hand-authored files.
  "story_id": "string",    // Human-readable ID, e.g. "US-01". May change. Required.

  // ── Content ───────────────────────────────────────────────────────────────
  "title": "string",       // Short, active-voice description. Required.
  "description": "string", // Observable behaviour. What a user can do or see.
                           //   Implementation-agnostic. Required.
  "section": "string",     // Feature area grouping, e.g. "Auth", "Dashboard". Optional.
  "notes": "string | null",  // Rationale, scope exclusions, ADR links. Distinct from description. Optional.

  // ── Status ────────────────────────────────────────────────────────────────
  "status": "not-implemented" | "partial" | "implemented" | "stale",  // Required.
  "priority": 0 | 1 | 2 | 3,  // 0=urgent, 1=high, 2=medium, 3=low. Optional.

  // ── Collaboration ─────────────────────────────────────────────────────────
  "assignee": "string | null",   // Username or email. Optional.
  "reviewer": "string | null",   // Username or email. Optional.

  // ── Audit trail ───────────────────────────────────────────────────────────
  "created_at": "string",   // ISO 8601 timestamp. Optional.
  "updated_at": "string",   // ISO 8601 timestamp. Optional.
  "version": "integer",     // Increments on every change. Starts at 1. Optional.

  // ── Links ─────────────────────────────────────────────────────────────────
  "pr_refs": ["string"],         // GitHub PR URLs implementing this story. Optional. Strings only — see A.3.
  "jira_key": "string | null",   // Linked Jira issue key, e.g. "PROJ-123". Optional.
  "design_ref": "string | null", // Figma frame or design artefact URL. Optional.

  // ── Quality contract (v1.1) ───────────────────────────────────────────────
  "acceptance_criteria": ["string"] | null,  // Testable done-conditions. Plain English. Optional.
  "test_refs": ["string"] | null,            // Test file paths verifying this story. Optional.
  "depends_on": ["string"] | null,           // story_id values that must be done first. Optional.

  // ── Compliance (optional) ─────────────────────────────────────────────────
  "compliance": Compliance | null  // Formal approval block. Include for regulated-sector stories.
}
```

---

## Status values

| Value | Meaning |
|---|---|
| `not-implemented` | Story exists; no implementation detected |
| `partial` | Implementation detected but incomplete |
| `implemented` | Implementation detected and verified |
| `stale` | Previously implemented; source files changed since last audit |

Status is set by Prototyper's audit process or manually. Readers should not infer status from other fields.

---

## Rules for story descriptions

A good description is:

1. **Observable** — describes what a user can see or do, not how it's implemented
2. **Specific** — "Show an error if the email is already in use" rather than "handle errors"
3. **Implementation-agnostic** — doesn't reference components, APIs, or technical choices
4. **Testable** — a QA engineer can verify it without asking the author

**Bad:** "Use the AuthService to validate the JWT and redirect"  
**Good:** "If the session is expired, redirect to the login page and show a 'Session expired' message"

---

## Full example (JSON)

```json
{
  "schema_version": "1.1",
  "project": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Leave Management System",
    "created_at": "2026-04-01T09:00:00Z"
  },
  "stories": [
    {
      "id": "7f3d9a2b-1c4e-4f8a-b5d6-9e0f1a2b3c4d",
      "story_id": "US-01",
      "title": "Employee can submit a leave request",
      "description": "Show a form with date range picker and reason field. On submit, create the request and show it in 'Pending' status. Show a confirmation message with the request ID.",
      "section": "Leave Requests",
      "status": "implemented",
      "priority": 1,
      "assignee": "jony@example.com",
      "reviewer": null,
      "created_at": "2026-04-01T09:00:00Z",
      "updated_at": "2026-04-15T14:30:00Z",
      "version": 3,
      "pr_refs": ["https://github.com/org/repo/pull/42"],
      "jira_key": "LEAVE-101",
      "acceptance_criteria": [
        "Form shows date range picker and reason field",
        "On submit, request appears in 'Pending' status in the leave list",
        "Confirmation message shows the new request ID"
      ],
      "depends_on": null,
      "design_ref": "https://figma.com/file/abc/LeaveRequest?node-id=1",
      "test_refs": ["cypress/e2e/leave-request.spec.ts#L12"]
    },
    {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "story_id": "US-02",
      "title": "Manager can approve or reject a leave request",
      "description": "Show a list of pending requests from the manager's team. Each row shows employee name, dates, reason, and Approve/Reject buttons. On approval, notify the employee by email. On rejection, require a short reason.",
      "section": "Leave Requests",
      "status": "partial",
      "priority": 1,
      "assignee": "sara@example.com",
      "reviewer": "jony@example.com",
      "created_at": "2026-04-01T09:00:00Z",
      "updated_at": "2026-04-20T11:00:00Z",
      "version": 2,
      "pr_refs": [],
      "jira_key": "LEAVE-102",
      "acceptance_criteria": [
        "Only requests from the manager's direct reports appear in the list",
        "Approving sends an email notification to the employee",
        "Rejecting requires a reason field (minimum 10 characters)"
      ],
      "depends_on": ["US-01"]
    }
  ]
}
```

---

## Minimal example (hand-authored YAML)

For hand-authored files, only `story_id`, `title`, `description`, and `status` are required:

```yaml
schema_version: "1.1"
project:
  name: "Leave Management System"
stories:
  - story_id: US-01
    title: Employee can submit a leave request
    description: >
      Show a form with date range picker and reason field. On submit,
      create the request and show it in 'Pending' status.
    status: not-implemented
    acceptance_criteria:
      - Form shows date range picker and reason field
      - Request appears in Pending status after submit
```

---

## Full example (YAML — platform-exported)

```yaml
schema_version: "1.1"
project:
  id: "550e8400-e29b-41d4-a716-446655440000"
  name: "Leave Management System"
  created_at: "2026-04-01T09:00:00Z"
exported_at: "2026-05-03T19:00:00Z"
stories:
  - id: "7f3d9a2b-1c4e-4f8a-b5d6-9e0f1a2b3c4d"
    story_id: "US-01"
    title: "Employee can submit a leave request"
    description: >
      Show a form with date range picker and reason field. On submit,
      create the request and show it in 'Pending' status. Show a
      confirmation message with the request ID.
    section: "Leave Requests"
    status: "implemented"
    priority: 1
    assignee: "jony@example.com"
    reviewer: null
    created_at: "2026-04-01T09:00:00Z"
    updated_at: "2026-04-15T14:30:00Z"
    version: 3
    pr_refs:
      - "https://github.com/org/repo/pull/42"
    jira_key: "LEAVE-101"
    acceptance_criteria:
      - Form shows date range picker and reason field
      - On submit, request appears in Pending status
      - Confirmation message shows the new request ID
    design_ref: "https://figma.com/file/abc/LeaveRequest?node-id=1"
    test_refs:
      - cypress/e2e/leave-request.spec.ts#L12
```

---

## Export formats

| Format | Command | Use case |
|---|---|---|
| JSON | `prototyper export --format json` | Machine consumption, CI, integrations |
| YAML | `prototyper export --format yaml` | Human review, git diff |
| Markdown | `prototyper export --format markdown` | Documentation, Confluence, Notion |
| CLAUDE.md | `prototyper export --format claude-code` | Claude Code context injection |

---

## Reading the schema (for integrators)

Any tool may read a Prototyper story contract. The canonical source is the Prototyper API:

```
GET https://api.prototyper.app/v1/projects/:project_id/stories
Authorization: Bearer <token>
```

Response: the root JSON object.

Tools that read stories:
- **Claude Code** — `CLAUDE.md` export turns stories into implementation instructions
- **GitHub Actions** — `prototyper/audit` action validates PR coverage against stories
- **Jira sync** — stories ↔ Jira issues, status synced bidirectionally
- **Documentation generators** — stories → user-facing feature documentation

---

## Versioning

Schema versions follow semver. Rules:

- **Patch** (1.0.x): Clarification-only changes to this document. No format changes.
- **Minor** (1.x.0): New optional fields added. Backwards-compatible. Readers that ignore unknown fields are unaffected.
- **Major** (x.0.0): Breaking changes. `schema_version` major bumped. Readers must handle the new version explicitly.

Reader contract:
- Check `schema_version`
- Handle unknown fields gracefully — ignore them, don't error
- Surface an error if `schema_version` major is higher than your supported version

---

## Changelog

**v1.1** — 2026-05-03  
Quality contract fields + relaxed required-fields for hand-authored files.
- `acceptance_criteria` — testable done-conditions per story (plain English, not Gherkin)
- `depends_on` — list of `story_id` values that must be implemented first
- `design_ref` — link to Figma frame or design artefact
- `test_refs` — explicit test file paths for deterministic audit validation
- `compliance` — formal approval block for regulated-sector stories
- `notes` — freetext rationale, scope exclusions, ADR links
- `exported_at` root field — API/CLI-generated export timestamp
- Relaxed required fields: `id`, `section`, `priority`, `created_at`, `updated_at`, `version` are now optional (hand-authored files rarely have them; platform-generated files always do)
- Project `id` and `created_at` are now optional
- CI workflow auto-selects schema version based on `schema_version` field — v1.0 files use v1.0 schema, v1.1+ files use v1.1 schema

**v1.0** — 2026-04-24  
Initial public release.
- Core story object: identity, content, status, collaboration, audit trail, links
- JSON and YAML canonical formats
- Status enum: `not-implemented`, `partial`, `implemented`, `stale`
- Priority: 0–3 integer
- Export format table
- Full examples in JSON and YAML
