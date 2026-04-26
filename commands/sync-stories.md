---
name: "prototyper:sync"
description: "Pull open stories from your Prototyper project into the current Claude Code session. Loads acceptance criteria for the current sprint so you can start implementing immediately."
---

# /prototyper:sync

Syncs open user stories from Prototyper into your current Claude Code session.

## What it does

1. Connects to your Prototyper project via the MCP server
2. Lists all stories with status `not-implemented`
3. Presents a numbered list for you to choose which to work on
4. Loads the full story (including acceptance criteria) into context

## Usage

```
/prototyper:sync
```

Or with a specific story:

```
/prototyper:sync US-04
```

## Prerequisites

- `PROTOTYPER_API_KEY` must be set (from your Prototyper account settings)
- `PROTOTYPER_PROJECT_ID` must be set (from the Prototyper app URL)
- The Prototyper MCP server must be connected (should happen automatically via `.mcp.json`)

## After syncing

Claude Code will have the full story context and can implement to the acceptance criteria without you having to re-paste from Jira or Notion. Once implementation is complete, Claude Code will call `mark_story_status` to close the loop in Prototyper.
