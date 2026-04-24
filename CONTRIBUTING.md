# Contributing to the Prototyper Story Schema

Thanks for your interest. The goal is a stable, useful specification that makes story-first development practical for real teams.

---

## What we welcome

- **Bug fixes** — ambiguous language, contradictions, unclear field behaviour
- **New optional fields** — must have a clear use case and real-world motivation
- **Validator improvements** — better error messages, edge case coverage
- **Integration examples** — how to read the schema in your tool/pipeline

## What we don't accept

- Mandatory new fields (schema must stay easy to emit from a minimal tool)
- Fields that encode implementation details (the schema is implementation-agnostic by design)
- Breaking changes to existing field semantics

---

## Process

1. **Open an issue first** — describe the problem and proposed change. For small fixes (typos, wording), a PR is fine directly.
2. **Fork and branch** — branch from `main`.
3. **Edit `SCHEMA.md`** — the spec is the source of truth. Update examples if the change affects them.
4. **Submit a PR** — include: what changed, why, and a real-world use case if it's a new field.
5. **All CI checks must pass** — PR is blocked until checks are green.

---

## Versioning policy

We follow semver. All PRs that add or change fields must include a `CHANGELOG.md` entry and a `schema_version` bump in the appropriate position:

- New optional field → minor bump (e.g. `1.0` → `1.1`)
- Breaking change → major bump (e.g. `1.x` → `2.0`) — these are rare and require strong justification

---

## Running the validator locally

```bash
npm install
npm test
```

Tests validate all example files in `examples/` against the JSON Schema.

---

## Code of conduct

Be direct and specific. This is a technical specification; discuss changes in terms of their effect on readers and writers, not preferences.
