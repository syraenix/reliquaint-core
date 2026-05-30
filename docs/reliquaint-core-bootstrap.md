# reliquaint-core Bootstrap Tasks

> **Status:** One-time setup. This document can be archived or deleted once v0.3 ships.
> **Goal:** Bring the official Reliquaint Core tap online as a separate repository, ready for the launcher to subscribe to as part of v0.3.
> **Repository to be created:** `syraenix/reliquaint-core` at `https://github.com/syraenix/reliquaint-core`.

This document covers the standing-up of the new `reliquaint-core` repository. It runs alongside the launcher's `docs/v0.3-tasks.md` Milestone 6, which handles the launcher-side cutover.

The new repository's content all comes from the launcher repo's existing `tap/` directory. The launcher repo's tap content stays in place during the bootstrap; only after the launcher has been switched to subscribe (v0.3-tasks Task 6.2) does the bundled copy come out. There will be a brief period when both repositories hold the same content — that's expected.

**Reading order:** ADR-0003 (the tap model) and `schema.md` (the tap schema) from the launcher repo. `docs/v0.3-tasks.md` for the sequencing context.

---

## Task 1: Create the GitHub repository

Manual step. Settings:

- **Owner:** `syraenix`
- **Name:** `reliquaint-core`
- **Description:** "The official catalog of classic DOS and Amiga games for Reliquaint — curated metadata, recommended emulator configurations, legal acquisition pointers."
- **Visibility:** Public.
- **Initialize with:** nothing — the migrated content arrives in Task 4.
- **Topics:** `reliquaint`, `dos`, `amiga`, `dosbox`, `fs-uae`, `retro-gaming`, `preservation`, `catalog`.
- **Default branch:** `develop`, matching the launcher repo's convention.

Apply the same branch protection rules used on the launcher repo (require PR review, require CI passing once CI exists).

**Done when:** The empty repository is reachable at the canonical URL with the right description, topics, and branch protections.

---

## Task 2: Initial repository hygiene

In a fresh local clone of the empty repository, add the foundation files:

- `.gitignore` — minimal. OS and editor cruft. No build artifacts since this is a content repo.
- `README.md` — describes what the tap is, who maintains it, how to subscribe to it from Reliquaint, and a pointer to the tap maintainer guide in the launcher repo (`docs/tap-maintainer-guide.md`, written as part of v0.3-tasks Task 7.3). Opening line: "The official Reliquaint Core catalog of classic DOS and Amiga games for Reliquaint."

The TOML and configuration files that make up the tap arrive in Task 4.

**Done when:** The repository has `.gitignore` and `README.md` as its initial commit on `develop`, pushed to GitHub.

---

## Task 3: Community and contributor files

Add the standard community files plus the agent/contributor guidance files, tailored for a content repository:

- **`LICENSE`** — CC-BY-SA-4.0, matching the launcher repo's `LICENSE-CONTENT`. The catalog's TOML manifests and emulator configs are creative content; share-alike protects community contributions to the catalog.
- **`CODE_OF_CONDUCT.md`** — copy verbatim from the launcher repo. Update the maintainer contact placeholder if a different contact is preferred for catalog-specific issues.
- **`SECURITY.md`** — narrowly scoped. The surface here is "malicious TOML that crashes a parser" or "path-traversal references in catalog entries." Most Reliquaint security reports should still go to the launcher repo; this file should be short and explicit about the scope, with a redirect for anything outside it.
- **`CONTRIBUTING.md`** — tap-maintainer-focused. Cover:
  - How to add a catalog entry (file naming, location, schema reference to the launcher repo's `docs/schema.md`).
  - How to validate locally before submitting (`reliquaint tap validate .` from the repository root; requires launcher v0.3+).
  - Policy on proprietary game files: never submitted, never linked, never referenced.
  - Policy on acquisition links: must point to legitimate legal sources (GOG, Steam, developer sites, Internet Archive releases authorized by the rights holder). No abandonware download sites; no DRM-bypass storefronts.
- **`AGENTS.md`** — repository guidelines for any contributor (human or AI agent). Covers project structure, validation commands, schema conventions, catalog entry process, and commit/PR conventions. A pre-written version is available in the launcher repo's setup artifacts under `reliquaint-core/AGENTS.md`.
- **`CLAUDE.md`** — operational guidance specifically for Claude Code. Highlights the content-only nature of the repository, the cross-repository dependency on the launcher's schema and ADRs, and the workflow for common operations. A pre-written version is available in the launcher repo's setup artifacts under `reliquaint-core/CLAUDE.md`.

**Done when:** All six files are present at the repository root, content-appropriate for a tap repository.

---

## Task 4: Migrate tap content

Move the catalog content from the launcher repo's `tap/` directory into this repository.

- **History preservation preferred.** Use `git filter-repo` or equivalent to bring across history where practical. If history preservation is too complex, a single clean import commit is acceptable — the commit message must cite the launcher repo's source commit hash.
- **Drop the wrapping `tap/` directory.** In the launcher repo the structure is `tap/tap.toml`, `tap/catalog/dos/...`. Here the structure is `tap.toml` at the repository root, `catalog/dos/...`. The new repo's root **is** the tap root.
- **Update `tap.toml`:**
  - Bump `version` to `0.1.0`.
  - Update `url` to point at this repository (`https://github.com/syraenix/reliquaint-core`).
  - Ensure `id` is `reliquaint-core`.
- **Do not yet delete the tap content from the launcher repo.** That deletion is part of launcher v0.3-tasks Task 6.2, which runs after this repository is live and tagged.

**Done when:** Every catalog entry from the launcher repo's `tap/` directory is present at the correct path in this repo, with sibling configs intact and `tap.toml` updated.

---

## Task 5: CI validation workflow

Set up CI to run schema validation against every catalog entry on every PR and push.

- Workflow path: `.github/workflows/validate.yml`.
- Job steps:
  1. Check out the repository.
  2. Install Reliquaint. Easiest path: download the latest `.deb` from the launcher repo's Releases page and `dpkg -i` it. Alternative: `cargo install --git https://github.com/syraenix/reliquaint`.
  3. Run `reliquaint tap validate .` against the repository root.
- The workflow depends on launcher v0.3-tasks Task 6.1 being implemented and released. Until then, the workflow will fail because the `tap validate` subcommand doesn't exist. **Note this in a header comment in the workflow file** so the reason for early-stage failures is unambiguous.
- Add issue templates under `.github/ISSUE_TEMPLATE/`:
  - `catalog_fix.yml` — broken entry, wrong metadata, config that no longer launches.
  - `new_game_request.yml` — proposing a new entry without contributing one. Adapt the launcher repo's `catalog_request.yml` for this repo's context.
  - `config.yml` — disable blank issues; redirect security to the launcher's security tab.
- Add `.github/PULL_REQUEST_TEMPLATE.md`. Cover: description of the change (new game, fix to existing entry, new collection), test evidence (the contributor has played the game with the updated entry), and the same proprietary-content attestation pattern as the launcher repo.

**Done when:** The workflow file exists with appropriate documentation of its prerequisite; the issue and PR templates are in place; once launcher v0.3 ships, the workflow passes against the migrated content.

---

## Task 6: Tag initial release

Once validation passes and the content matches what was in the launcher repo's `tap/`:

- Tag the repository `v0.1.0`.
- Write release notes summarizing what's in the initial catalog: count of DOS games, count of Amiga games, collections covered. Cite the launcher repo commit the content was sourced from.
- The launcher's v0.3 release notes will link to this tag, so this tag should be the first thing the launcher's new subscription default points users at.

**Done when:** `v0.1.0` is tagged and visible in the repository's Releases page, with release notes.

---

## Task 7: Hand-off

Once the new repository is live, tagged, and CI-green:

- Confirm the URL pattern from launcher v0.3-tasks Task 1.3 (`reliquaint-core` short-name resolution) matches this repository's canonical URL.
- Signal to the launcher work that the cutover (v0.3-tasks Task 6.2) can proceed.
- The launcher repo's `docs/tap-maintainer-guide.md` (created in v0.3-tasks Task 7.3) should link to this repository as the reference example of a well-formed tap.

**Done when:** The launcher can subscribe to `reliquaint-core` via its short name, fetch this repository, and surface its catalog entries. This document's job is finished; archive or delete it.

---

## Notes on the longer-term shape

This document is transient by design — its purpose is to bring the new repository online once. After v0.3 ships:

- Catalog contributions go through PRs to this repository, not the launcher.
- Releases of this repository happen on their own cadence, decoupled from the launcher's releases. A patched manifest can be tagged and released here without waiting for a launcher release; users get it on their next `reliquaint tap sync`.
- The launcher repo's `docs/tap-maintainer-guide.md` becomes the durable reference for the next maintainer who wants to set up *their own* tap. This document is bootstrap-specific and shouldn't be confused with that guide.
