# Contributing to Reliquaint Core

Thank you for helping curate the official Reliquaint catalog. This repository is the content side of [Reliquaint](https://github.com/syraenix/reliquaint): it holds TOML catalog manifests and emulator configs, and no application code. Launcher code, bugs, and features belong in the launcher repository.

For broader, durable guidance on tap maintenance, read the launcher repo's [`docs/tap-maintainer-guide.md`](https://github.com/syraenix/reliquaint/blob/develop/docs/tap-maintainer-guide.md). This repository is its reference example. Agent and tooling conventions live in [`AGENTS.md`](AGENTS.md) and [`CLAUDE.md`](CLAUDE.md).

## The two hard rules

1. **No proprietary content, ever.** Never commit, link to, or reference game binaries, ROMs, Kickstart files, scanned manuals, music, or any other rights-holder content. This catalog only *describes* games and points users at where they can legally acquire them. The `.gitignore` blocks common emulator-image extensions as a backstop, but the responsibility is yours.
2. **Legal acquisition links only.** The `[acquisition]` table must point to legitimate sources: GOG, Steam, developer or publisher sites, or Internet Archive items the rights holder has explicitly released. Abandonware download sites and DRM-bypass storefronts are never acceptable.

## Repository layout

```
tap.toml                 # tap identity (id, version, maintainer, url, license)
catalog/
  dos/<id>.toml          # DOS catalog entry
  dos/<id>.conf          # DOSBox-Staging config (no [autoexec] section)
  amiga/<id>.toml        # Amiga catalog entry
  amiga/<id>.fs-uae      # FS-UAE config (optional)
```

The authoritative TOML schema for every file here is the launcher repo's [`docs/schema.md`](https://github.com/syraenix/reliquaint/blob/develop/docs/schema.md). Architectural reasoning lives in its ADRs (`docs/adr/`). If the schema needs to change, that conversation happens in the launcher repo first.

## Adding a catalog entry

1. **Confirm a legitimate acquisition source.** If none exists, don't add the entry.
2. **Choose a stable id** — lowercase, hyphenated, matching the filename (`^[a-z][a-z0-9-]*[a-z0-9]$`, e.g. `qfg1-ega`, `kq1sci`). For series, follow the existing naming pattern.
3. **Copy an existing entry of the same platform** as a starting point; for DOS, copy the sibling `.conf` too.
4. **Fill in** `[game]`, `[meta]`, `[acquisition]`, `[install]`, and `[runtime]` per the schema. Metadata (year, developer, publisher) should be verifiable.
5. **DOS:** tune the `.conf` for this title (cycles, scalers, MIDI, machine type). Remove any `[autoexec]` section — the launcher composes that at runtime from `runtime.dosbox.entry`.
6. **Amiga:** set `runtime.fs_uae.model` and `runtime.fs_uae.floppies`. A sibling `.fs-uae` config is optional.

## Validate before you submit

From the repository root, with Reliquaint v0.3+ installed:

```bash
reliquaint tap validate .
```

CI runs the same command on every push and PR. Then test for real by subscribing to your checkout as a local tap and launching the game end to end:

```bash
reliquaint tap add "file://$PWD"
reliquaint run <id>
```

## Commits & pull requests

- Use short, imperative commit summaries (`Add Loom catalog entry`, `Fix MIDI routing in qfg2.conf`). Keep each commit focused on one entry or one fix.
- In the PR, describe the change, document your local testing (at minimum that `reliquaint tap validate .` passes and the game launches), reference any related issue, and complete the proprietary-content attestation in the PR template.
- After accumulating mergeable changes, the maintainer bumps `tap.toml`'s `version` (SemVer) and tags a release. Users receive it on their next `reliquaint tap sync`.
