---
description: "for Scoop manifests fixes. Covers schema-safe fields, checkver/autoupdate consistency, architecture blocks, hash updates, and validation workflow."
name: "Scoop manifest authoring (bucket fixes)"
applyTo: "bucket/*.json"
---
# Scoop Manifest Authoring Guidelines

Use these rules when creating or editing manifests in `bucket/*.json`.

This is a multi-root project, consisting of
  * github.com/LogicDaemon/ScoopInstaller-manifest-fixes — the project to make the changes in
  * github.com/ScoopInstaller/Scoop — the source code of the scoop app, to be used as reference for available options and data formats
  * github.com/ScoopInstaller/Main — bucket with most commonly used and generally well maintained manifests, use as examples
  * github.com/ScoopInstaller/Extras — more manifests, some of them are occasionally broken, but in general they are reviewed by scoop maintainers before getting merged
  * maybe additional manifest buckets
check the specific workspace contents when starting working on it to find out the exact paths in the filesystem.

## Scope and source of truth

- Only modify files in this repository (`ScoopInstaller-manifest-fixes`).
- Treat `schema.json` as the schema authority (`additionalProperties: false`).
- Reuse patterns from existing manifests before inventing new structure.

## Editing principles

- Keep changes minimal and targeted to the requested fix.
- Preserve existing comments/notes and behavior unless the task requires changing them.
- Prefer consistency with the manifest being edited; do not reformat unrelated fields.
- Keep JSON valid and stable (no trailing commas, proper escaping in PowerShell strings).
- Use scripts to output properly escaped data, and avoid manually crafting `\\\\n`-infested strings.

## Field and structure conventions
// ToDo: update to actually list required fields and avoid duplication
- Required core keys: `version`, `homepage`, `license`.
- Common top-level order in this repo: metadata  (`version`, `description`, `homepage`, `license`) then install/runtime fields (`architecture`, `scripts`, `bin`, `shortcuts`, `persist`) then update automation (`checkver`, `autoupdate`).
- For multi-arch apps, keep architecture-specific `url` and `hash` together under each arch entry.
- Prefer architecture key order `64bit`, `32bit`, `arm64` for new/rewritten blocks (unless preserving existing order is safer).

## checkver/autoupdate coherence

- If `version` extraction changes in `checkver`, verify `autoupdate` placeholders still resolve correctly.
- Keep capture names stable and explicit (for example `$matchTag`, `$matchCode`, `$cleanVersion`) when used by `autoupdate`.
- When `url` patterns include transformed versions, ensure the same transformation is represented in `checkver` (`replace`, regex captures) or built-in placeholders.

## Hash and URL integrity

- Every changed download URL must have a matching updated `hash` (or a valid `autoupdate.hash` extraction strategy).
- Hash must match the actual downloaded file or CI fails.

## Script blocks

- Keep PowerShell snippets concise and preferrably idempotent.

## Preserving configuration

- If app config file is saved in the app's directory directly (not a subdir):
  * when installing, if there is a config in persist already, try symlinking it to the app dir, and if it fails, hardlink (symlinking requires admin privileges except when dev mode is enabled, but hardlinks can be lost if application deleted and re-writes the config file)
  * before uninstalling/updating, move the config from the app dir to the persist dir (except if it's a symlink)
- If in a subdir in the app dir, make a junction (reparse point) from the persist dir to the app dir (no need to move, junctions do not require admin privileges)
  * If the config subdir is installed along with the app, rename it before creating the junction, then move the contents to the persist subdir without overwriting any files, so if there were no config, the default one will be used, but otherwise, the existing one will be preserved.
- If app config is not in the app dir, make junctions from persist to their location

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
