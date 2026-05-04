# GitHub Actions — stories.yaml validation

This directory contains CI workflows for the schema repository itself.

---

## Adding validation to your own repo

Copy this snippet into `.github/workflows/validate-stories.yml` in your project:

```yaml
name: Validate stories.yaml

on:
  push:
    paths:
      - 'stories.yaml'
  pull_request:
    paths:
      - 'stories.yaml'

jobs:
  validate:
    name: Validate stories.yaml against Prototyper schema v1.0
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install validator
        run: npm install -g ajv-cli ajv-formats js-yaml

      - name: Convert YAML to JSON
        run: |
          node -e "
            const yaml = require('js-yaml');
            const fs = require('fs');
            const doc = yaml.load(fs.readFileSync('stories.yaml', 'utf8'));
            fs.writeFileSync('/tmp/stories.json', JSON.stringify(doc, null, 2));
          "

      - name: Validate against schema
        run: |
          curl -s https://raw.githubusercontent.com/jonybur/locus-story-schema/main/schema/v1.0/stories.schema.json \
            > /tmp/stories.schema.json
          ajv validate \
            -s /tmp/stories.schema.json \
            -d /tmp/stories.json \
            --strict=false \
            --spec=draft2020
```

This validates your `stories.yaml` against the Prototyper schema v1.0 on every push or pull request that touches the file.

**No Prototyper account required.** The schema is fetched directly from GitHub.

---

## What this checks

- All required fields are present (`id`, `story_id`, `title`, `description`, `section`, `status`, `priority`, `created_at`, `updated_at`, `version`)
- No unrecognised fields (the schema uses `additionalProperties: false`)
- `status` is one of `not-implemented`, `partial`, `implemented`, or `stale`
- `priority` is an integer (0–3)
- `id` and project `id` are UUID format
- Timestamps are ISO 8601 strings

---

Schema source: `schema/v1.0/stories.schema.json`  
License: MIT
