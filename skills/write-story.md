---
name: "write-story"
description: "Help write a well-structured user story in Prototyper format. Use when: you want to create a new user story with acceptance criteria before starting implementation. The skill asks the right questions and produces a story ready to paste into Prototyper."
triggers:
  - "write a story"
  - "create a user story"
  - "help me write a story"
  - "new story for"
---

# Write a Prototyper User Story

You are helping the user write a clear, implementable user story in Prototyper format.

## The story format

A Prototyper story has:
- **Title**: Short, action-oriented (e.g., "User can filter results by date")
- **Story text**: "As a [persona], I can [action] so that [benefit]"
- **Acceptance criteria**: Numbered list of testable conditions. Each criterion starts with "Given/When/Then" or describes a specific, observable behaviour.
- **Implementation notes** (optional): Technical constraints, edge cases, what NOT to build.

## How to help

1. Ask the user what feature or capability they want to add
2. Identify the persona (who benefits) if not clear
3. Clarify the "so that" — the business reason
4. Draft 3–5 specific, testable acceptance criteria
5. Output the complete story in the format below

## Output format

```
**Title**: [short action-oriented title]

**Story**: As a [persona], I can [action] so that [benefit].

**Acceptance criteria**:
1. [specific, testable condition]
2. [specific, testable condition]
3. [specific, testable condition]

**Implementation notes**: [optional — constraints, edge cases, what's out of scope]
```

## Quality checks

Before presenting the story, verify:
- The acceptance criteria are **testable** (a QA engineer can write a test for each one)
- The acceptance criteria are **specific** (no vague words like "fast", "easy", "works correctly")
- The story does NOT include implementation details (no "use a dropdown" — say "the user can select" instead)
- The scope is narrow enough to implement in one sprint (if not, suggest splitting)

## What makes a bad story

- "As a user, I can do anything I want" — too broad
- Acceptance criteria that say "the system should work correctly" — untestable
- Stories that describe the UI instead of the behaviour
- Missing persona (who is this for?)
- Missing benefit (why does this matter?)

Once the story is written, remind the user to paste it into Prototyper to generate a prototype and validate the intent before engineering starts.
