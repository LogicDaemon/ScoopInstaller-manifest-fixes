---
description: "Use when creating or modifying Scoop manifests in this bucket. Covers schema-safe fields, checkver/autoupdate consistency, architecture blocks, hash updates, and validation workflow."
name: "Scoop manifest authoring (bucket fixes)"
applyTo: "bucket/*.json"
---
# Scoop Manifest Authoring Guidelines

Use these rules when creating or editing manifests in `bucket/*.json`.

## Scope and source of truth

- Only modify files in this repository (`ScoopInstaller-manifest-fixes`).
- Treat `Scoop/schema.json` as the schema authority (`additionalProperties: false`).
- Reuse patterns from existing manifests before inventing new structure.

## Editing principles

- Keep changes minimal and targeted to the requested fix.
- Preserve existing comments/notes and behavior unless the task requires changing them.
- Prefer consistency with the manifest being edited; do not reformat unrelated fields.
- Keep JSON valid and stable (no trailing commas, proper escaping in PowerShell strings).

## Field and structure conventions

- Required core keys: `version`, `homepage`, `license`.
- Common top-level order in this repo: metadata (`version`, `description`, `homepage`, `license`) then install/runtime fields (`architecture`, scripts, `bin`, `shortcuts`, `persist`) then update automation (`checkver`, `autoupdate`).
- For multi-arch apps, keep architecture-specific `url` and `hash` together under each arch entry.
- Prefer architecture key order `64bit`, `32bit`, `arm64` for new/rewritten blocks (unless preserving existing order is safer).

## checkver/autoupdate coherence

- If `version` extraction changes in `checkver`, verify `autoupdate` placeholders still resolve correctly.
- Keep capture names stable and explicit (for example `$matchTag`, `$matchCode`, `$cleanVersion`) when used by `autoupdate`.
- When `url` patterns include transformed versions, ensure the same transformation is represented in `checkver` (`replace`, regex captures) or built-in placeholders.

## Hash and URL integrity

- Every changed download URL must have a matching updated `hash` (or a valid `autoupdate.hash` extraction strategy).
- Keep hash algorithm format schema-compliant (typically SHA256 hex).
- Do not leave placeholder hashes in committed manifests.

## Script blocks

- Keep PowerShell snippets concise, idempotent where practical, and compatible with both Windows PowerShell and PowerShell (`pwsh`) when possible.
- Preserve escaping style used in the manifest (e.g., `\` in JSON strings, backtick escaping where already used).

## Validation workflow

- After edits, run repository wrappers from `bin/` as appropriate:
  - `formatjson.ps1` for formatting normalization
  - `checkver.ps1` for update-rule sanity
  - `test.ps1` for bucket tests
- Prefer fixing schema/format issues before changing logic.

## Practical patterns from existing manifests

- `python.json`: architecture-specific installers and hash extraction via `autoupdate.hash.regex`.
- `cursor.json`: named regex capture in `checkver` reused by `autoupdate` URLs.
- `main` examples (`7zip.json`, `git.json`): clear architecture blocks and `checkver`/`autoupdate` pairing.
