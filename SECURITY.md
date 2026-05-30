# Security Policy

This repository is **content only** — TOML catalog manifests and emulator configuration files. It contains no executable code, so its security surface is narrow.

## In scope for this repository

Report issues here only if they concern the catalog content itself:

- **Malformed or malicious TOML** in a catalog entry or `tap.toml` that could crash, hang, or otherwise abuse a conforming parser.
- **Path-traversal or unsafe path references** in catalog entries (for example, `install` or `runtime` fields that point outside their intended directory).
- **Acquisition links** that point to malware, drive-by downloads, or otherwise hostile destinations.

## Out of scope — report to the launcher instead

Anything about how Reliquaint *executes* this content — the launcher binary, how it parses or sandboxes taps, how it invokes DOSBox-Staging or FS-UAE, IPC, or filesystem access — is a **launcher** concern, not a catalog concern. The launcher is also where a parser hardening fix would land even if a malformed manifest here triggered it.

Report those privately through the launcher repository's **Security** tab → **Report a vulnerability**:

https://github.com/syraenix/reliquaint/security/advisories/new

Please do not open a public issue for any security problem. When in doubt, use the launcher's private reporting channel above.

## What to include

- A description of the issue and why it's a security concern.
- A proof-of-concept entry or file layout, where applicable.
- Any suggested remediation.
