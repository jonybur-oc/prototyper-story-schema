# Prototyper Story Schema — Specification

**Version:** 1.0  
**Status:** Stable  
**Date:** 2026-04-24

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
  "schema_version": "1.0",       // Required. Semver string.
  "project": Project,             // Required.
  "stories": Story[]              // Required. May be empty.
}
```

---

## Project object

```typescript
{
  "id": "string",          // Stable UUID. Required.
  "name": "string",        // Human name. Required.
  "created_at": "string"   // ISO 8601 timestamp. Required.
}
```

---

## Story object

```typescript
{
  // ── Identity ──────────────────────────────────────────────────────────────
  "id": "string",          // Stable UUID. Never changes, even if title changes. Required.
  "story_id": "string",    // Human-readable ID, e.g. "US-01". May change. Required.

  // ── Content ───────────────────────────────────────────────────────────────
  "title": "string",       // Short, active-voice description. Required.
  "description": "string", // Observable behaviour. What a user can do or see.
                           //   Implementation-agnostic. Required.
  "section": "string",     // Feature area grouping, e.g. "Auth", "Dashboard". Required.

  // ── Status ────────────────────────────────────────────────────────────────
  "status": "not-implemented" | "partial" | "implemented" | "stale",  // Required.
  "priority": 0 | 1 | 2 | 3,  // 0=urgent, 1=high, 2=medium, 3=low. Required.

  // ── Collaboration ─────────────────────────────────────────────────────────
  "assignee": "string | null",   // Username or email. Optional.
  "reviewer": "string | null",   // Username or email. Optional.

  // ── Audit trail ───────────────────────────────────────────────────────────
  "created_at": "string",   // ISO 8601 timestamp. Required.
  "updated_at": "string",   // ISO 8601 timestamp. Required.
  "version": "integer",     // Increments on every change. Starts at 1. Required.

  // ── Links ─────────────────────────────────────────────────────────────────
  "pr_refs": ["string"],         // GitHub PR URLs implementing this story. Optional.
  "jira_key": "string | null"    // Linked Jira issue key, e.g. "PROJ-123". Optional.
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
  "schema_version": "1.0",
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
      "jira_key": "LEAVE-101"
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
      "jira_key": "LEAVE-102"
    }
  ]
}
```

---

## Full example (YAML)

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

**v1.0** — 2026-04-24  
Initial public release.
- Core story object: identity, content, status, collaboration, audit trail, links
- JSON and YAML canonical formats
- Status enum: `not-implemented`, `partial`, `implemented`, `stale`
- Priority: 0–3 integer
- Export format table
- Full examples in JSON and YAML
