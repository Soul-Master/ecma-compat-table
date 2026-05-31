# Updating `data/edition-mapping.json` With An LLM

This guide is for maintaining `data/edition-mapping.json` with an LLM, especially OpenAI GPT models. The file is source data for this app, so every generated change should be grounded in official sources and checked against `@mdn/browser-compat-data`.

## Authoritative Sources

Use these sources as the basis for updates:

- TC39 finished proposals: <https://github.com/tc39/proposals/blob/HEAD/finished-proposals.md>
- MDN Browser Compat Data package: `@mdn/browser-compat-data`
- MDN Browser Compat Data JSON URL used by the app:

```text
https://unpkg.com/@mdn/browser-compat-data/data.json
```

Do not invent feature names, ECMAScript editions, proposal statuses, or MDN BCD paths. If a feature cannot be confidently mapped to BCD, keep the best candidate path only when it exists in the current BCD JSON. Otherwise leave it out or mark it for manual review before committing.

## JSON Shape

`data/edition-mapping.json` must remain a JSON array of edition objects:

```json
[
  {
    "name": "ES2026 (Candidate)",
    "year": 2026,
    "features": [
      {
        "name": "Feature name",
        "kind": "Feature category",
        "description": "Short plain-English description.",
        "paths": [
          "javascript.builtins.Example"
        ]
      }
    ]
  }
]
```

Required rules:

- Keep editions sorted newest first.
- Keep each `year` as a number.
- Keep `name`, `kind`, `description`, and every `paths` entry as strings.
- Keep descriptions short and factual.
- Use BCD paths from the `javascript` namespace unless there is a clear reason not to.
- Preserve valid JSON formatting with 4-space indentation.
- Do not include comments in the JSON file.

## Update Workflow

1. Read TC39 `finished-proposals.md` and identify finished proposals by edition year.
2. Compare the proposal list against existing entries in `data/edition-mapping.json`.
3. Add missing finished features, remove incorrect entries, and update candidate editions when proposals move.
4. Resolve each feature to one or more existing paths in the current `@mdn/browser-compat-data` JSON.
5. Prefer specific BCD paths for the feature surface that users care about, such as built-in constructors, methods, syntax, or operators.
6. When a proposal maps to multiple APIs, include all relevant BCD paths.
7. Validate that every configured path exists in the downloaded BCD JSON and has a `__compat` object.
8. Re-run the app locally and check that unresolved paths are expected.

## Suggested OpenAI GPT Prompt

Use a prompt like this when asking an OpenAI GPT model to update the mapping:

```text
Update data/edition-mapping.json for this ECMAScript compatibility table.

Use only these authoritative sources:
- TC39 finished proposals: https://github.com/tc39/proposals/blob/HEAD/finished-proposals.md
- @mdn/browser-compat-data data.json

Requirements:
- Keep the JSON schema exactly as-is: edition objects with name, year, and features; feature objects with name, kind, description, and paths.
- Keep editions sorted newest first.
- Map each feature to existing @mdn/browser-compat-data paths under the javascript namespace whenever possible.
- Include multiple paths when a feature has multiple public APIs.
- Do not invent BCD paths. If unsure, list the item for manual review instead of adding an invalid path.
- Keep descriptions short, factual, and neutral.
- Return only the complete updated JSON file content.
```

## Manual Review Checklist

Before accepting an LLM-generated update:

- The JSON parses successfully.
- Edition years match TC39 finished proposal data.
- Feature names are readable and stable enough for the UI.
- BCD paths exist in the current `@mdn/browser-compat-data` package.
- No unrelated formatting churn or renamed fields were introduced.
- Candidate editions are clearly labeled with `(Candidate)`.
