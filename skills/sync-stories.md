---
name: "sync-stories"
description: "Pull open user stories from Prototyper into the current session context. Use at the start of a coding session to load the sprint's planned stories so Claude Code knows what to implement."
triggers:
  - "sync stories"
  - "load my stories"
  - "what stories are open"
  - "pull stories from prototyper"
  - "what should I build"
---

# Sync Stories from Prototyper

Pull the current sprint's open stories into context so Claude Code knows what to implement.

## Steps

1. Call `list_projects()` to get available projects (or use `PROTOTYPER_PROJECT_ID` if set)
2. Call `list_stories(project_id, status="not-implemented")` to get open stories
3. Present a numbered summary to the user: story ID, title, priority
4. Ask the user which story to start with
5. Call `get_story(story_id)` to load the full story with acceptance criteria
6. Confirm: "I'll implement [story title]. The acceptance criteria are: [list]. Shall I proceed?"

## Output format (summary view)

```
Open stories in [project name]:

1. US-01 · [title] · P1
2. US-02 · [title] · P2
3. US-03 · [title] · P1

Which story would you like to implement?
```

## After story selection

Load the full story and present:

```
**Story**: [title]
**As a** [persona], **I can** [action] **so that** [benefit].

**Acceptance criteria**:
1. [criterion]
2. [criterion]
...

**Implementation notes**: [if any]

Ready to implement. Shall I proceed?
```

## Notes

- Only load `not-implemented` stories by default — do not present already-completed work
- If no stories are open, say so and suggest writing a new story in Prototyper first
- If `PROTOTYPER_PROJECT_ID` is set, use it directly and skip the project selection step
