# Repository Guidelines

## Project Structure & Content Organization

This repository is the content side of [Reliquaint](https://github.com/syraenix/reliquaint): the official `reliquaint-core` tap of curated DOS and Amiga game manifests. It contains no source code — only TOML manifests, emulator configs, and supporting documentation. The Rust + Tauri launcher lives in the `syraenix/reliquaint` repository.

**Tap root.** `tap.toml` at the repository root carries the tap's identity (`id`, `version`, `maintainer`, `url`, `license`). Bump `version` per SemVer when tagging a release.

**Catalog content.** Per-platform subdirectories under `catalog/`:

- `catalog/dos/<id>.toml` — per-game DOS catalog entries.
- `catalog/dos/<id>.conf` — DOSBox-Staging configs with no `[autoexec]` section.
- `catalog/amiga/<id>.toml` — per-game Amiga catalog entries.
- `catalog/amiga/<id>.fs-uae` — FS-UAE configs (optional; the launcher falls back to a model template when absent).

**CI and templates.** `.github/workflows/validate.yml` runs schema validation on every PR. Issue and PR templates under `.github/` cover catalog fixes, new game requests, and contributor attestation.

**Schema reference.** The authoritative schema for every TOML file in this repository is `docs/schema.md` in the launcher repository. Architectural reasoning lives in the ADRs there (`docs/adr/`).

## Validation & Local Testing Commands

Reliquaint v0.3+ provides the validation tool that CI uses; install it before contributing.

```bash
# Validate everything in this repo against the schema
reliquaint tap validate .

# Subscribe to this checkout as a local tap for live testing
reliquaint tap add "file://$PWD"

# Run a specific game (after installing it)
reliquaint run <id>

# Inspect host setup and installed games
reliquaint doctor
```

CI runs `reliquaint tap validate .` against the repository root. If a local run passes, CI will too.

## Schema & Naming Conventions

- **IDs.** Lowercase, hyphenated, must match the filename. Per the schema: `^[a-z][a-z0-9-]*[a-z0-9]$` (e.g. `qfg1-ega`, `kq1sci`).
- **Series grouping.** The `collection` field in `[game]` groups related entries (e.g. `quest-for-glory`, `kings-quest`). This drives UI grouping in the launcher.
- **TOML style.** Snake_case keys; one entry per file; trailing newline.
- **DOSBox configs.** Carry only per-game tuning (cycles, scalers, MIDI routing, machine type). No `[autoexec]` section — the launcher composes that at runtime from the catalog entry's `runtime.dosbox.entry`.
- **Acquisition links.** Legitimate legal sources only: GOG, Steam, developer or publisher sites, Internet Archive items released by the rights holder. Abandonware download sites and DRM-bypass storefronts are not acceptable here.

## Catalog Entry Guidelines

A complete entry includes `[game]`, `[meta]`, `[acquisition]`, `[install]`, and `[runtime]` tables. `[meta]` and `[acquisition]` are technically optional in the schema but expected on every entry in `reliquaint-core`.

When adding a new entry:

1. Confirm a legitimate acquisition source exists. If none does, don't add the entry.
2. Copy an existing entry of the same platform as a starting point.
3. Populate metadata accurately. Year, developer, publisher should be verifiable.
4. For DOS: tune the sibling `.conf` against a real install of the game and test until it launches and runs cleanly.
5. For Amiga: set `runtime.fs_uae.model` and `runtime.fs_uae.floppies`. Test against the actual disk images you have.
6. Run `reliquaint tap validate .`.
7. Subscribe to your checkout locally and launch the game through Reliquaint end to end.

## Commit & Pull Request Guidelines

Commits use short, imperative summaries (e.g. `Add Loom catalog entry`, `Fix MIDI routing in qfg2.conf`). Keep each commit focused on one entry or one fix.

Pull requests should:

- Describe what's changing (new entry, fix, metadata update, etc.).
- Document local testing — at minimum, confirmation that `reliquaint tap validate .` passes and that the game launches successfully when behavior changed.
- Reference any related issue.
- Complete the proprietary-content attestation in the PR template. **No proprietary game files, ROMs, Kickstart files, manuals, music, or other rights-holder content may appear in any PR.**

After accumulating mergeable changes, tag a release (`vX.Y.Z`) and update `tap.toml`'s `version` to match. Users receive tagged releases on their next `reliquaint tap sync`.
