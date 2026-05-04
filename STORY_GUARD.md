# The Story Guard Pattern

**A pre-execution scope check for AI coding agents.**

Before your agent writes code, it reads the story and verifies the action is within acceptance criteria.

> **TL;DR:** Add a Claude Code PostToolUse hook. After every agent action, your agent reads the active story and checks whether what it just did was within the acceptance criteria. Before: agent goes off-piste silently. After: agent catches itself before the next step. 15-minute setup — works with or without the Prototyper web app.
>
> [Skip to Claude Code setup →](#option-1-with-the-prototyper-mcp-server-full-stack) · [Skip to Cursor setup →](#option-3-cursor-rules-without-claude-code-hooks) · [Skip to local-file setup (no account) →](#option-2-with-a-local-prototyper-storiesyaml-file-no-web-app-required)

---

## The problem

In April 2026, a Cursor AI coding agent deleted a production database and all its backups in 9 seconds. It had no story contract to check its own actions against. The agent's intent was locally coherent — fix a credential problem — but there was no acceptance criterion it could check its actions against. The Railway API call was out of scope by any reasonable definition of any user story. Nobody had told the agent what "out of scope" looked like.

Story Guard is the structural answer: put the intent contract upstream, before execution, where the agent can reason about its own scope.

---

## What it is

Story Guard is a PostToolUse hook for AI coding agents that fires after every agent action — including file writes, bash commands, and API calls — and surfaces the active story's acceptance criteria. The agent evaluates whether its action was in scope before proceeding to the next step.

It does not hard-stop the agent. It creates a reasoning checkpoint the agent can see.

The core principle: **an agent that knows what it's supposed to build will notice when it's doing something else.**

---

## How it works

```
User says: "implement the leave request form"
     │
     ▼
Agent writes a file
     │
     ▼
[Story Guard fires]
     │
     ▼
Agent sees: active story + acceptance criteria
Agent reasons: "was that file write within the story scope?"
     │
     ├── YES → proceed
     └── NO  → flag it, ask user before continuing
```

Story Guard fires on **all tool types** — not just file writes:
- **File writes** (`Write`, `Edit`, `MultiEdit`) — was this file change in scope?
- **Bash commands** (`Bash`) — did this shell command (including `curl`, AWS CLI, Railway CLI) match an acceptance criterion?
- **MCP tool calls** — was this tool invocation authorised by the active story?

This matters because the most dangerous out-of-scope actions are often bash commands (API calls, deployments, deletions) — not file edits.

---

## Implementation

### Option 1: With the Prototyper MCP server (full stack)

If your team uses [Prototyper](https://prototyper.app) for story management, Story Guard is built in via the `get_active_story` tool.

Install the MCP server:
```bash
# Claude Code
claude mcp add prototyper -- npx -y @prototyper/mcp-server

# Or add to .mcp.json for Cursor / Windsurf
```

Create `.claude/settings.json` in your project root:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "mcp_tool",
            "tool": "prototyper/get_active_story",
            "description": "Story Guard: check action scope against active story acceptance criteria"
          }
        ]
      }
    ]
  }
}
```

The `get_active_story` tool returns the story currently in `partial` status (or the first `not-implemented` story if none are in progress), including its full acceptance criteria. The agent sees this after every action.

### Option 2: With a local `prototyper-stories.yaml` file (no web app required)

You don't need the Prototyper web app to use Story Guard. If you have a `prototyper-stories.yaml` file in your repo (using the [open schema](https://schema.prototyper.dev)), you can implement Story Guard with a Claude Code command:

**Step 1:** Create a `.claude/commands/check-story.md` file:

```markdown
---
description: Story Guard — check whether the last action was within the active story's scope
---

Read the prototyper-stories.yaml file in the project root.
Find the story with status "partial" (in-progress). If none, use the first story with status "not-implemented".

This is the active story.

Review the action I just took. Compare it against the story's acceptance_criteria.

For each criterion, determine:
- Is this action implementing or supporting one of these criteria? (in scope ✅)
- Is this action unrelated to all criteria? (out of scope ⚠️)
- Is this action potentially harmful if it's out of scope? (flag for user review 🛑)

If the action is out of scope, explain which criterion it doesn't satisfy and ask whether to continue.
If the action is in scope, confirm and proceed.
```

**Step 2:** Reference this command in your Claude Code session:

```
"Before you proceed with each action, run the check-story command."
```

Or set it as a PostToolUse hook if you're running Claude Code with hooks support.

**Step 3:** Add the active story to your session context at the start:

```
"The active story is US-03. Read prototyper-stories.yaml and find it."
```

### Option 3: Cursor rules (without Claude Code hooks)

Add to your `.cursor/rules` or project-level Cursor rules:

```
Before writing any code, making any bash command, or calling any external API:
1. Read prototyper-stories.yaml in the project root
2. Find the story with status "partial" (or first "not-implemented" if none)
3. Check whether your intended action falls within that story's acceptance_criteria
4. If it does not, stop and ask the user before proceeding

Never take an action that cannot be mapped to an acceptance criterion in the active story.
```

---

## What the agent sees (example)

After a file write, the agent receives something like:

```yaml
story_id: US-04
title: User can filter leave requests by date range
status: partial
acceptance_criteria:
  - Filtering by date range updates the results table immediately
  - Invalid date ranges show an error message (not a 500)
  - Selected filters persist on page reload (localStorage)
  - Filter UI is accessible via keyboard navigation
story_guard_note: |
  You just took an action. Does it correspond to one of the acceptance criteria above?
  If not, explain why it was necessary as a sub-step, or flag it before continuing.
```

The agent evaluates: "I just wrote `LeaveFilterComponent.tsx`. That implements criterion 1 (filtering updates the table) and criterion 4 (keyboard nav). In scope. ✅"

Or: "I just modified `database.ts` and dropped the `leave_audit_log` table. That doesn't appear in any acceptance criterion for US-04. ⚠️ Flagging before proceeding."

---

## The `prototyper-stories.yaml` format

Story Guard works with the [Prototyper open schema](https://schema.prototyper.dev). A minimal story with the fields Story Guard needs:

```yaml
schema_version: "1.0"
project:
  name: "My Project"
stories:
  - story_id: US-04
    title: "User can filter leave requests by date range"
    status: partial          # in-progress — this is the active story
    description: |
      As a manager, I want to filter the leave request table by date range
      so that I can review requests for specific periods without scrolling.
    acceptance_criteria:
      - Filtering by date range updates the results table immediately
      - Invalid date ranges show an error message (not a 500)
      - Selected filters persist on page reload (localStorage)
      - Filter UI is accessible via keyboard navigation
```

The minimum fields for Story Guard:
- `story_id` — unique identifier
- `status` — `not-implemented` | `partial` | `implemented`
- `acceptance_criteria` — list of specific, testable criteria

Optional but useful:
- `title` — human-readable description for the agent's reasoning
- `description` — fuller context for ambiguous cases

Full schema reference: [schema.prototyper.dev](https://schema.prototyper.dev)

---

## Why this matters for the "AI built the wrong thing" problem

The most common AI coding failure isn't the agent making a technical mistake. It's the agent doing technically correct work that wasn't what the PM meant.

This happens because:
1. The PM's intent is in their head, a Notion doc, or a Slack message — not in the agent's context
2. The agent optimises for local coherence (each step makes sense given the last) rather than global alignment (does this series of steps implement what the PM meant?)
3. There's no checkpoint between "agent acts" and "agent acts again"

Story Guard addresses all three:
1. Intent is in the story file — in the agent's context before the first action
2. Each action is evaluated against the intent — not just the previous action
3. Out-of-scope actions are flagged before the next step

It doesn't prevent every wrong decision. It catches the ones that can still be corrected.

---

## Relationship to the open schema

Story Guard is a practice, not a product. It works with:
- The [Prototyper web app](https://prototyper.app) — stories managed in the app, surfaced via MCP
- A raw `prototyper-stories.yaml` file — stories managed manually in the repo, read directly by the agent
- Any tool that outputs the [Prototyper schema format](https://schema.prototyper.dev) — including custom scripts, Linear exports, or Jira adapters

The schema is MIT-licensed and designed to be readable by any AI coding agent without any integration work. If a `prototyper-stories.yaml` exists in the repo root, Claude Code, Cursor, Windsurf, and Aider can all read it. Story Guard is just the discipline of making the agent use it.

---

## Contributing

If you've implemented Story Guard with a different agent (Aider, Devin, GitHub Copilot, Continue.dev), we'd like to know how it worked. Open an issue or PR in this repo with your implementation pattern. The goal is a reference implementation for every major AI coding agent.

Current reference implementations:
- [x] Claude Code (PostToolUse hooks via `.claude/settings.json`)
- [x] Cursor (rules via `.cursor/rules`)
- [ ] Windsurf — [contribute](https://github.com/jonybur/prototyper-story-schema/issues)
- [ ] Aider — [contribute](https://github.com/jonybur/prototyper-story-schema/issues)
- [ ] GitHub Copilot — [contribute](https://github.com/jonybur/prototyper-story-schema/issues)
- [ ] Continue.dev — [contribute](https://github.com/jonybur/prototyper-story-schema/issues)

---

## Discussion

The Story Guard pattern is new. The implementation details are rough in places. If you've hit a case it doesn't handle, or have a better implementation for your agent setup, open an issue — or post in the `#story-guard` channel if one exists.

The principle is sound. The implementation will get better with use.

---

*Part of the [Prototyper open schema](https://github.com/jonybur/prototyper-story-schema). MIT licensed.*
