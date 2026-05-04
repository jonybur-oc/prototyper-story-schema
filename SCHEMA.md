# Locus Story Schema — Specification

**Version:** 1.2.1  
**Status:** Stable  
**Date:** 2026-05-04

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
  "status": "not-implemented" | "in-progress" | "partial" | "implemented" | "stale" | "deprecated",  // Required.
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
  "compliance": Compliance | null,  // Formal approval block. Include for regulated-sector stories.

  // ── v1.2 additions ────────────────────────────────────────────────────────
  "tags": ["string"] | null,         // Free-form labels for cross-cutting concerns. Optional.
  "persona": "string" | null,        // User role this story is written for. Optional.
  "risk": Risk | null,               // Risk assessment block. Optional.
  "non_functional": NonFunctional | null  // Story-level NFRs (perf, a11y, retention). Optional.
}
```

---

## Compliance object

The `compliance` block is an optional sub-object on a Story. Include it when a human must formally sign off before implementation — e.g. under PSD2/3, HIPAA, ISO 27001, SOC 2, or an internal change-advisory board (CAB) process.

```typescript
{
  "approved_by": "string",        // Required. Email or username of the approver.
  "approved_at": "string",        // Required. ISO 8601 UTC timestamp of approval.
  "review_ref": "string | null",  // Optional. URL to the review issue, PR, or document.
  "audit_note": "string | null"   // Optional. Free-text reason, e.g. regulatory article reference.
}
```

---

### Compliance field reference

| Field | Type | Required | Description |
|---|---|---|---|
| `approved_by` | string | ✅ | Email or username of the approver (CTO, compliance officer, auditor). Max 255 chars. |
| `approved_at` | ISO 8601 string | ✅ | Timestamp of when approval was granted. UTC. |
| `review_ref` | string (URI) \| null | ✗ | URL to the review issue, PR, or document where approval is recorded. Recommended for audit trails. |
| `audit_note` | string \| null | ✗ | Free-text explanation of why this story requires compliance sign-off. Examples: `"PSD2 Article 95 — strong authentication requirement"`, `"ISO 27001 A.9.2 — access control change"`. |

### Compliance example (YAML)

```yaml
story_id: PAY-07
title: User can initiate a high-value payment
description: >
  Allow users to initiate payments above €1,000. Require re-authentication
  via biometric or 2FA before confirming the transfer.
status: not-implemented
compliance:
  approved_by: chief-compliance-officer@example.com
  approved_at: "2026-05-01T10:30:00Z"
  review_ref: "https://jira.example.com/browse/COMP-219"
  audit_note: "PSD2 Article 97 — strong customer authentication required for transactions > €1,000"
```

**Rules:**
- `approved_by` and `approved_at` are required when the `compliance` block is present.
- `audit_note` SHOULD reference the specific regulatory article or control — this makes audit trails actionable.
- The `stale` status MUST NOT be set on a story that has a `compliance` block without re-approval. Tooling should surface a warning when a `stale` story carries a compliance block.
- Do not set the `compliance` block as a placeholder. It signals human sign-off has occurred; use `audit_note` to explain what is pending if sign-off is not yet complete.

---

## tags (v1.2)

Free-form labels for cross-cutting concerns. Lowercase alphanumeric; hyphens allowed. Max 20 tags per story, max 64 chars each. Unique within a story.

Use `tags` for concerns that don't fit neatly into `section` or `priority`: delivery milestones (`mvp`, `wave-2`), platform variants (`mobile`, `desktop`, `api`), quality dimensions (`a11y`, `perf`, `security`), or ad-hoc groupings (`external-api`, `beta-only`).

**Tooling usage:** `locus list --tag mvp` filters to tagged stories. The GitHub Action can be configured to fail if `mvp`-tagged stories are not implemented at release.

```yaml
story_id: DASH-04
title: Dashboard renders on mobile
tags: [mobile, mvp, a11y]
```

**Rules:**
- Tags must match `^[a-z0-9][a-z0-9-]*[a-z0-9]$` (or single char `[a-z0-9]`).
- No spaces, no uppercase, no underscores.
- Duplicate tags within a story are invalid (schema enforces `uniqueItems`).
- Tags are not a substitute for `section` — use `section` for the primary feature area and tags for secondary concerns.

---

## persona (v1.2)

The user role this story is written for. Single value — who benefits from the story. Should match the subject of the story title.

```yaml
story_id: AUTH-01
title: Employee can reset their password
persona: employee
```

**Rules:**
- One persona per story. If a story serves two personas, split it.
- Use the same persona strings consistently across all stories (e.g. always `admin`, not sometimes `administrator`).
- `persona` is who benefits, not who implements (`assignee` is the implementer).
- System-level stories with no human actor may omit `persona` (null/absent).

**Recommended approach:** Define your project's persona vocabulary in a comment at the top of `stories.yaml` or in a companion `personas.md`. Tooling can then warn when an unknown persona string appears.

---

## risk (v1.2)

Risk assessment for a story. Include when the story touches security, data, regulated behaviour, or when implementation complexity is non-trivial.

```typescript
{
  "level": "critical" | "high" | "medium" | "low",  // Required.
  "category": "security" | "data-loss" | "privacy" | "performance" | "reliability" | "compliance" | "third-party" | "scope-creep" | "other" | null,
  "description": "string | null",  // What could go wrong.
  "mitigation": "string | null",   // What is being done about it.
  "review_ref": "string (URI) | null"  // Link to threat model, ADR, or review doc.
}
```

### Risk field reference

| Field | Type | Required | Description |
|---|---|---|---|
| `level` | enum | ✅ | Severity: `critical` (blocks ship), `high` (explicit mitigation required), `medium` (monitor), `low` (noted). |
| `category` | enum \| null | ✗ | Risk type. Use canonical values where applicable. |
| `description` | string \| null | ✗ | What could go wrong. Specific. Required for `high`/`critical`. |
| `mitigation` | string \| null | ✗ | Action taken or planned to reduce the risk. Required for `high`/`critical`. |
| `review_ref` | URI \| null | ✗ | Link to supporting documentation (threat model, ADR, security review). |

### Risk example (YAML)

```yaml
story_id: PAY-11
title: Payment provider webhook validates event signatures
status: not-implemented
risk:
  level: high
  category: security
  description: Unsigned or replayed webhooks could trigger fraudulent payment state transitions.
  mitigation: Verify Stripe-Signature header on every webhook. Reject replays older than 5 minutes. Log and alert on failures.
  review_ref: https://stripe.com/docs/webhooks/signatures
```

**Rules:**
- `level` is the only required field.
- `description` and `mitigation` SHOULD be present for `high` and `critical` stories. Tooling should warn when they are absent.
- `risk.level` is independent of `priority`. A low-priority story can carry a high risk (e.g. a rarely-used admin action with a serious data-loss risk).
- Do not use `risk` as a substitute for the `compliance` block. `compliance` is for formal regulatory sign-off; `risk` is for engineering risk assessment.

---

## non_functional (v1.2)

Story-level non-functional requirements. Constraints on *how* the story must be implemented — latency, availability, accessibility, data retention, security. Only include constraints specific to this story; project-wide defaults live in project documentation.

```typescript
{
  "performance": PerformanceNFR | null,
  "availability": "string | null",      // SLO string, e.g. '99.9%'
  "accessibility": "WCAG-2.1-AA" | ... | null,
  "data_retention": "string | null",    // ISO 8601 duration or policy string
  "security": "string | null",          // Freeform security constraint
  "notes": "string | null"             // Anything else
}
```

### PerformanceNFR sub-object

```typescript
{
  "p50_ms": integer,          // 50th percentile latency target (ms)
  "p95_ms": integer,          // 95th percentile latency target (ms)
  "p99_ms": integer,          // 99th percentile latency target (ms)
  "max_ms": integer,          // Hard maximum latency — requests above this are failures
  "throughput_rps": number,   // Required sustained throughput (requests/sec)
  "budget_note": "string | null"  // Freeform budget note (e.g. bundle size)
}
```

All PerformanceNFR fields are optional — include only what is relevant.

### non_functional example (YAML)

```yaml
story_id: SEARCH-01
title: User can search by keyword and receive results
status: not-implemented
non_functional:
  performance:
    p95_ms: 300
    p99_ms: 800
    budget_note: Search index must not exceed 50MB in-memory
  accessibility: WCAG-2.1-AA
  data_retention: P90D
  security: Query input sanitised against injection before index lookup.
```

### Accessibility levels

Use the canonical `WCAG-X.Y-{A|AA|AAA}` enum values:
- `WCAG-2.0-A`, `WCAG-2.0-AA`, `WCAG-2.0-AAA`
- `WCAG-2.1-A`, `WCAG-2.1-AA`, `WCAG-2.1-AAA`
- `WCAG-2.2-A`, `WCAG-2.2-AA`, `WCAG-2.2-AAA`

**Rules:**
- Include `non_functional` only when a story has constraints beyond the project default.
- `non_functional.performance` latency targets should be set with measurement methodology in mind — document how they are validated (load test tool, p50/p95 in Datadog, etc.).
- `data_retention` uses ISO 8601 duration strings (`P7Y` = 7 years, `P90D` = 90 days) or free-text policy strings when duration alone is insufficient.

---

## Status values

| Value | Meaning |
|---|---|
| `not-implemented` | Story exists; no implementation detected |
| `in-progress` | Work has started; human-set to signal active development |
| `partial` | Implementation detected but incomplete (audit-set) |
| `implemented` | Implementation detected and verified |
| `stale` | Previously implemented; source files changed since last audit |
| `deprecated` | Feature was intentionally removed; story retained for history |

Status is set by Prototyper's audit process or manually. Readers should not infer status from other fields.

### Status semantics

Each value has a precise meaning. Use the decision guide below when setting status manually:

**`not-implemented`** — The story has been written and accepted, but no implementation exists. This is the starting state for all new stories. Use this when:
- A feature is planned but no code has been written.
- A story was rolled back completely and code was deleted.

**`in-progress`** — Work has started on this story. This status is human-set — it signals active development intent. Use this when:
- An engineer has begun implementation but acceptance criteria are not yet satisfied.
- A story is actively being worked in the current sprint.
- You want Story Guard to know work is underway without claiming completion.

`in-progress` is distinct from `partial`: `partial` is set by the audit tool when it detects evidence of incomplete implementation; `in-progress` is set by a human to express intent. A story may transition from `in-progress` → `partial` when the audit first detects partial implementation, or → `implemented` when all criteria are met.

Do not leave stories in `in-progress` across sprints. If work stalls, move back to `not-implemented`.

**`partial`** — Some acceptance criteria are satisfied but not all. The feature exists but is incomplete. Use this when:
- The UI renders but a key interaction is missing.
- The happy path works but edge cases (error states, empty states, loading states) do not.
- An acceptance criterion is behind a feature flag or not yet merged.

Do not use `partial` as a permanent state. If a story has been `partial` for more than one sprint without movement, either close it (mark `not-implemented` and rewrite), complete it, or split it into a done story + a new story for the remainder.

**`implemented`** — All acceptance criteria are satisfied and verified. The feature is shipped. Use this when:
- Every acceptance criterion in the `acceptance_criteria` list has a corresponding passing test or manual sign-off.
- If no `acceptance_criteria` are defined: the description is fully observable in the product.

Once `implemented`, a story should only leave this status if the acceptance criteria change (→ `partial`) or source files covering the story are significantly modified without a corresponding update (→ `stale`).

**`stale`** — The story was previously `implemented`, but a subsequent code change may have broken or changed the behaviour. The story needs re-audit. Use this when:
- The Prototyper audit detects that files referenced by `test_refs` have changed.
- A PR is merged that touches the story's feature area without updating the story.
- A refactor changes observable behaviour that the story described.

`stale` is not a permanent resting state. Every `stale` story is a liability: it means the spec and the code disagree. Tooling SHOULD surface `stale` stories in PR reviews. Authors SHOULD resolve `stale` within the same sprint — either by re-verifying (→ `implemented`) or updating the story to match the new behaviour.

`stale` is distinct from `deprecated`: `stale` means _"we don't know if it still works"_; `deprecated` means _"it was intentionally removed."_

**`deprecated`** — The feature was intentionally removed from the product. The story is retained for historical record and audit continuity. Use this when:
- A feature was shipped (`implemented`) and then deliberately removed in a later release.
- A product decision was made to discontinue the feature.

Do not use `deprecated` for:
- Features that were never shipped (use `not-implemented` or delete the story).
- Features that are temporarily disabled (use `partial` or a `notes` field).
- Features that are broken but still in the product (use `stale`).

Deprecated stories MUST retain their `story_id` permanently. Other stories may reference a deprecated story in `depends_on` — those references remain valid.

### Status transitions

```
not-implemented → in-progress  (work started; human-set)
not-implemented → partial      (some criteria met; audit-set)
not-implemented → implemented  (all criteria met in one pass)
in-progress     → partial      (audit detects incomplete implementation)
in-progress     → implemented  (all criteria met)
in-progress     → not-implemented  (work abandoned or deferred)
partial         → implemented  (remaining criteria met)
partial         → not-implemented  (rollback; criteria no longer met)
implemented     → stale        (source changed; re-audit needed)
implemented     → deprecated   (feature intentionally removed)
stale           → implemented  (re-audit confirms criteria still met)
stale           → partial      (re-audit: only some criteria still met)
stale           → deprecated   (feature was removed, not just changed)
deprecated      → (terminal; no outgoing transitions)
```

Tooling SHOULD warn when:
- A story has been `partial` for > 30 days.
- A story is `stale` and is referenced in a currently open PR.
- A story transitions from `implemented` → `deprecated` without a `notes` entry explaining why.

---

## The `notes` field

The `notes` field is for author commentary — things that are true about the story but are not the observable behaviour itself. It is distinct from `description`, which describes what a user can do or see.

### What belongs in `notes`

- **Rationale** — why this story exists, what was considered and rejected, what the business driver is.
- **Scope exclusions** — what is explicitly out of scope for this story ("Out of scope: mobile view — see US-12").
- **ADR or Slack links** — references to decisions recorded elsewhere.
- **Implementation constraints** — technical constraints that limit how the story can be implemented ("Must not block the main thread; use a Web Worker").
- **Deferred items** — work known to be missing that will be addressed in a follow-up story.

### What does NOT belong in `notes`

- Observable behaviour. If a user can see or do it, it belongs in `description` or `acceptance_criteria`.
- Acceptance criteria. `notes` is not a secondary AC list. If it is testable, it goes in `acceptance_criteria`.
- Implementation details that don't constrain the story. How you implement it is not a story concern.

### Why `notes` matters for Story Guard

Story Guard's scope-checking function reads the `description` and `acceptance_criteria` to determine whether an agent action is in scope. Mixing implementation rationale into `description` degrades this check — the LLM sees implementation details as part of the contract and may allow or refuse actions based on them incorrectly. `notes` is excluded from scope-checking by design.

### Example

```yaml
story_id: PERF-03
title: Dashboard loads within 2 seconds on a 4G connection
description: >
  The main dashboard view is fully interactive within 2 seconds of navigation
  on a 4G connection (20 Mbps down, 10 ms RTT). A loading skeleton is shown
  during data fetch; the page does not flash an empty state.
status: not-implemented
acceptance_criteria:
  - "Lighthouse performance score ≥ 90 on mobile preset"
  - "Skeleton renders immediately; data appears within 2s on simulated 4G"
  - "No layout shift after initial render (CLS < 0.1)"
notes: >
  Background: dashboard was taking 6–8s on the old architecture (Slack thread:
  #perf-2026-04). The root cause was a waterfall of three sequential API calls;
  the fix is to fan them out in parallel. We are not targeting 3G or offline
  performance in this story — see PERF-07 for the offline caching work.
  ADR-14 records the decision to use React Query with stale-while-revalidate
  instead of Redux for this feature.
```

Note that the `description` and `acceptance_criteria` contain only observable, testable behaviour. The `notes` field carries the rationale and the implementation constraint (parallel API calls, not targeting 3G) — information that is useful to the author and reviewer but that should not influence Story Guard's scope check.

---

## The `story_id` field

The `story_id` is the primary human-readable identifier for a story. It appears in `depends_on` references, PR comments, CI output, and test annotations. It must be stable once set.

### Format conventions

A `story_id` follows the pattern `<PREFIX>-<NUMBER>`, where:
- **PREFIX** is 2–8 uppercase ASCII letters identifying the area (e.g. `US`, `AUTH`, `PAY`, `DASH`, `API`, `ADMIN`).
- **NUMBER** is a zero-padded decimal integer starting at 01 (e.g. `01`, `02`, `10`, `100`).

Examples of valid story IDs: `US-01`, `AUTH-07`, `PAY-12`, `DASH-03`, `API-100`.

Prefix guidelines:

| Prefix | Typical use |
|---|---|
| `US` | General user stories (no specific area) |
| `AUTH` | Authentication and authorisation |
| `PAY` | Payments and billing |
| `DASH` | Dashboard and analytics |
| `API` | Developer-facing API |
| `ADMIN` | Admin panel features |
| `PERF` | Performance stories |
| `INFRA` | Infrastructure and DevOps |

If none of the above fit, use any short, memorable uppercase prefix. The prefix is informational — it does not affect validation.

### Uniqueness

A `story_id` MUST be unique within a `stories.yaml` file. Duplicate `story_id` values are a validation error.

Across multiple `stories.yaml` files in the same repository, `story_id` uniqueness is RECOMMENDED but not required — tools that merge stories across files SHOULD warn on collision.

### Stability

`story_id` values MUST NOT change once assigned. Stories are referenced by `story_id` in:
- `depends_on` arrays of other stories
- PR titles, descriptions, and commit messages
- Test file annotations (`test_refs`)
- External issue trackers synced via `jira_key` or Linear equivalents

If you need to reorganise stories, renumber sections, or reflect a refactor:
1. Keep the old `story_id` on the existing story.
2. Create a new story with a new `story_id` for the new work.
3. Mark the old story `deprecated` with a `notes` entry explaining the successor.

### Numbering

Numbers are scoped to their prefix. `US-01` and `AUTH-01` are distinct stories. Numbers are recommended to be sequential within a prefix, but gaps are acceptable — do not renumber existing stories to fill a gap. A story that was `US-04` remains `US-04` even if you delete `US-03`.

### Tooling rules

Validators MUST:
- Reject duplicate `story_id` values within a file.
- Reject `depends_on` references that point to a `story_id` not present in the same file (unless the tool supports cross-file resolution).

Validators SHOULD warn when:
- A `story_id` does not match the `<PREFIX>-<NUMBER>` pattern.
- A `depends_on` reference points to a `deprecated` story.

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

**v1.2.0** — 2026-05-04  
Minor version. Backwards-compatible additions. All new fields are optional; files valid against v1.1 are valid against v1.2.
- **`tags`** — Free-form labels for cross-cutting concerns. Array of lowercase alphanumeric strings (hyphens allowed, max 20 per story, max 64 chars each, unique). Use for delivery milestones (`mvp`, `wave-2`), platform variants (`mobile`, `api`), quality dimensions (`a11y`, `perf`, `security`). Tooling: `locus list --tag <tag>` filter; GitHub Action can gate on tag coverage.
- **`persona`** — User role the story is written for. Single string, max 128 chars. Distinct from `assignee`. Split stories serving multiple personas. Omit for system-level stories.
- **`risk`** — Risk assessment sub-object. Fields: `level` (required: `critical`/`high`/`medium`/`low`), `category` (enum), `description`, `mitigation`, `review_ref`. `description` and `mitigation` are required by convention for `high`/`critical` stories — tooling should warn when absent.
- **`non_functional`** — Story-level non-functional requirements. Sub-objects: `performance` (p50/p95/p99/max ms, throughput, budget note), `availability` (SLO string), `accessibility` (WCAG level enum), `data_retention` (ISO 8601 duration or policy string), `security` (freeform), `notes` (freeform). Include only story-specific constraints; project-wide defaults belong in project documentation.

**v1.1.5** — 2026-05-03  
`story_id` field documented with authoring conventions, uniqueness rules, and stability rules. Added: format pattern (`<PREFIX>-<NUMBER>`), prefix guideline table, uniqueness constraints (MUST be unique within file, RECOMMENDED across files), stability rules (MUST NOT change once assigned), numbering guidance (sequential, gaps permitted, never renumber), and three tooling rules (validators MUST reject duplicates and invalid cross-file depends_on; SHOULD warn on non-conforming IDs and deprecated depends_on). No schema changes — documentation patch only.

**v1.1.4** — 2026-05-03  
`deprecated` status value added to TypeScript type signature and JSON Schema enum. The value was documented in the Status values table and semantics section (v1.1.2) but was missing from the Story object type definition on line 64 and from `schema/v1.1/stories.schema.json`. A validator running against the JSON Schema would incorrectly reject stories with `status: deprecated`. This patch aligns the machine-readable schema with the documented semantics. No behaviour change — `deprecated` was always intended to be valid.

**v1.1.3** — 2026-05-03  
`notes` field documented. Added authoring guidance: what belongs in `notes` vs `description` vs `acceptance_criteria`, why `notes` is excluded from Story Guard scope-checking, and a complete worked example showing the separation between observable behaviour (in `description`/`acceptance_criteria`) and rationale/constraints (in `notes`). No schema changes — documentation patch only.

**v1.1.2** — 2026-05-03  
Status semantics documented. `deprecated` status value added. Status transition diagram added. Three tooling-SHOULD rules for `partial` > 30 days, `stale` in open PRs, and `implemented` → `deprecated` without `notes`. No schema changes — documentation patch only.

**v1.1.1** — 2026-05-03  
Compliance object documented. Added field table, TypeScript signature, YAML example, and four authoring rules (stale + compliance interaction, placeholder prohibition, audit_note SHOULD reference regulatory article). No schema changes — documentation patch only.

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
