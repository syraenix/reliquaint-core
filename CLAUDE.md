# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

The official **Reliquaint Core** tap — the curated catalog of classic DOS and Amiga games that ships as the default subscription for [Reliquaint](https://github.com/syraenix/reliquaint).

This is a **content repository**, not a code repository. Its files are:

- Per-game catalog manifests (TOML) describing identity, runtime, metadata, and legal acquisition sources.
- Per-game DOSBox-Staging and FS-UAE configs that the launcher composes at runtime.
- A `tap.toml` describing this tap's identity and version.

The schema, ADRs, and architectural reasoning for this repository's structure all live in the launcher repo. Before adding or modifying anything here, read these documents in the launcher repo:

- `docs/schema.md` — authoritative TOML schema for everything in this repository.
- `docs/adr/adr-0001-two-layer-manifest-model.md` — catalog entry vs. installation record split.
- `docs/adr/adr-0002-split-dosbox-config-model.md` — why configs in `catalog/dos/*.conf` have no `[autoexec]`.
- `docs/adr/adr-0003-tap-based-distribution.md` — the tap model overall.
- `docs/tap-maintainer-guide.md` — broader guidance for tap maintainers; this repository is the reference example.

## Repository layout

```
tap.toml                      # this tap's metadata (id, version, maintainer, etc.)
catalog/
  dos/<id>.toml               # DOS catalog entries
  dos/<id>.conf               # DOSBox-Staging configs (NO [autoexec] section)
  amiga/<id>.toml             # Amiga catalog entries
  amiga/<id>.fs-uae           # FS-UAE configs (optional; launcher falls back to model template)
companion/
  <game-id>/                  # optional companion content (walkthroughs, maps, hints): Markdown + images
.github/
  workflows/validate.yml      # runs `reliquaint tap validate .` on PRs and pushes
  ISSUE_TEMPLATE/
  PULL_REQUEST_TEMPLATE.md
README.md
LICENSE                       # CC-BY-SA-4.0 (content license)
CODE_OF_CONDUCT.md
CONTRIBUTING.md               # tap-maintainer-focused
SECURITY.md                   # narrowly scoped; most reports redirect to the launcher repo
```

## Conventions when editing

- **No code lives here.** This repository carries no source beyond CI workflow YAML. Don't add Rust, Python, or shell scripts. Code lives in the launcher repo.
- **No proprietary game files, ever.** Never commit game binaries, ROMs, Kickstart files, scanned manuals, music, or any other proprietary content. The launcher orchestrates files the user has legally acquired; this catalog only describes how to do that.
- **Schema is the source of truth.** Every TOML file conforms to `docs/schema.md` in the launcher repo. If a schema change is needed, that conversation happens in the launcher repo first; this repository follows.
- **Validation must pass.** Before committing any TOML change, run `reliquaint tap validate .` from the repository root. CI runs the same check on every PR.
- **ID format.** Lowercase, hyphenated, must match the filename. Per the schema: `^[a-z][a-z0-9-]*[a-z0-9]$`.
- **Legal acquisition links only.** The `[acquisition]` table points users at legitimate sources: GOG, Steam, developer sites, Internet Archive items where the rights holder has explicitly released the game. Abandonware download sites and DRM-bypass storefronts do not appear here.
- **DOSBox configs ship without `[autoexec]`.** Per ADR-0002 in the launcher repo, the launcher composes the autoexec at runtime from the catalog entry's `runtime.dosbox.entry`. Configs in `catalog/dos/<id>.conf` carry only the per-game tuning (cycles, scalers, MIDI routing, machine type).

## Common operations

### Add a new catalog entry

1. Confirm a legitimate acquisition source for the game. If none exists, do not add an entry.
2. Choose a stable id following the format rules. For series, follow the existing pattern (`qfg1-ega`, `kq1sci`, `sq3`, etc.).
3. Copy an existing entry of the same platform as a starting point. For DOS, also copy the corresponding `.conf`.
4. Fill in `[game]`, `[meta]`, `[acquisition]`, `[install]`, and `[runtime]` per the schema.
5. For DOS: tune the `.conf` for this title. Remove any `[autoexec]` section if one ended up in your copy.
6. For Amiga: set `runtime.fs_uae.model` and `runtime.fs_uae.floppies`. A sibling `.fs-uae` is optional.
7. Run `reliquaint tap validate .` locally.
8. Test by subscribing Reliquaint to this directory as a local tap (`reliquaint tap add /path/to/this/repo`) and confirming the game launches.

### Fix or update an existing entry

Same flow, but applied to the existing files. Validation and local launch test still required. The commit message should reference the issue if one exists.

### Tag a release

After accumulating changes worth releasing:

1. Bump `tap.toml`'s `version` field per SemVer.
2. Commit the version bump.
3. Tag the commit (e.g., `v0.2.0`).
4. Push the tag. Users get the changes on their next `reliquaint tap sync`.

## What lives where

- **This repo:** catalog content. Manifests, emulator configs, `tap.toml`.
- **Launcher repo (`syraenix/reliquaint`):** the launcher code, the schema definition, the ADRs, the tap maintainer guide, the user manifest creation wizard.
- **User taps (created by `reliquaint add`):** stay on the user's machine in `${XDG_CONFIG_HOME}/reliquaint/tap/`. These are never submitted here directly. Users contribute via the workflow in `CONTRIBUTING.md` (manifest export, then PR).

## When you need information this file doesn't cover

The launcher repo's `docs/` directory is the next stop. Specifically:

- `docs/schema.md` for any TOML structure question.
- `docs/tap-maintainer-guide.md` for broader maintenance guidance.
- `docs/adr/` for architectural decisions that shaped this repository.

If a question doesn't have a clear answer from those documents, surface it as a discussion on the launcher repo rather than guessing here. The schema is shared; divergence creates problems for every tap, not just this one.
