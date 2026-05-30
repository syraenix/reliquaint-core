# Reliquaint Core

The official Reliquaint Core catalog of classic DOS and Amiga games for Reliquaint.

This repository is the **content side** of [Reliquaint](https://github.com/syraenix/reliquaint): a curated tap of DOS and Amiga game manifests, recommended emulator configurations, and pointers to legal acquisition sources. It carries no application code — the Rust + Tauri launcher lives in the launcher repository. Reliquaint Core ships as the default subscription for every Reliquaint install.

## What's in here

```
tap.toml                 # this tap's identity (id, version, maintainer, url, license)
catalog/
  dos/<id>.toml          # DOS catalog entries
  dos/<id>.conf          # DOSBox-Staging configs (no [autoexec] section)
  amiga/<id>.toml        # Amiga catalog entries
  amiga/<id>.fs-uae      # FS-UAE configs (optional; launcher falls back to a model template)
```

Each entry describes a game's identity, metadata, runtime, and **how to legally acquire it** — it never contains or links to proprietary game files.

## Subscribing

Reliquaint v0.3+ subscribes to this tap by default. To add it manually:

```bash
reliquaint tap add https://github.com/syraenix/reliquaint-core
reliquaint tap sync
```

Tagged releases here are picked up on the next `reliquaint tap sync`, independent of the launcher's own release cadence.

## Contributing

Catalog contributions — new entries, metadata fixes, config tuning — are made via pull request to this repository. See [`CONTRIBUTING.md`](CONTRIBUTING.md) for the workflow, and the launcher repo's [`docs/tap-maintainer-guide.md`](https://github.com/syraenix/reliquaint/blob/develop/docs/tap-maintainer-guide.md) for broader guidance (this repository is the reference example of a well-formed tap). The authoritative TOML schema is the launcher repo's [`docs/schema.md`](https://github.com/syraenix/reliquaint/blob/develop/docs/schema.md).

**No proprietary content, ever** — no game binaries, ROMs, Kickstart files, scanned manuals, or music. Acquisition links must point to legitimate legal sources only.

## Maintainer

Derek Duncan <derek.b.duncan@gmail.com>

## License

Catalog content is licensed under [CC-BY-SA-4.0](LICENSE). The share-alike terms protect community contributions to the catalog.
