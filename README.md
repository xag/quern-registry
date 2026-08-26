# quern-registry

The package registry for [quern](https://github.com/xag/quern) — a Python library that keeps a domain's rules and vocabulary as data and publishes them as versioned packages. The packages in `packages/` are such bundles; the `quern` CLI (from that repo) is how you publish to a registry and consume from one.

A registry is not a service — it is a directory on git, and this repo is that directory in the open, so the format has a reference anyone can read, run, and copy.

## Thirty seconds

- **A registry is a directory.** `packages/<name>/<version>.json` plus `blobs/<sha256>`, and nothing else. There is no format here that `quern` does not already own: the repo is a published `quern.library.Library`, and git is only its transport.
- **A digest discipline.** Identity is not the git ref — it is the content hash a pin carries, re-checked on every machine that consumes it. Git moves bytes; the digest decides whether they mean what a pin says they mean.
- **Two doors.** *In:* `quern publish` — proof-gated, a package must demonstrate itself to enter. *Out:* `quern pin` → `quern sync`, and the proof re-runs on your own machine. A registry is trusted for transport, never for validation.
- **Root your own — an invitation, not a footnote.** This registry is canonical by convention, not by mechanism. A fork is a directory copy, and your fork is a registry just as real as this one: point your pins at it and publish. A hosted canonical registry may exist somewhere someday; this repo stays the format's reference and the free path, and that split is deliberate.
- **What CI refuses, and why.** A published version is never edited or removed — republishing `name@version` with different bytes would let a pin quietly change meaning. And artifacts enter only by the publish door: a pull request that hand-adds a package is refused, because a hand-added file has not passed the proof gate. See [the doors](#the-doors).

---

This working tree **is** a `quern.library.Library` root: `packages/<name>/<version>.json` plus `blobs/<sha256>`. There is no format here that quern does not already own — the repo is nothing but a published Library, and git is its transport.

Git is transport, **not identity**. Identity is the content digest a pin carries (`PackageRef.sha256`, the hash of the package's canonical serialization), checked at resolve time on every machine that consumes it. A checkout that mangles bytes cannot lie about meaning; a registry that drifted would be caught by the first pin it fails.

## The doors

- **In:** `quern publish <artifact> --registry <this checkout>` — and nothing else. Publishing is proof-gated: every rule exercised by the package's own examples, every rule refuted by its counter-examples staged alone, every wasm blob true to its hash and ABI. A package that cannot demonstrate itself does not enter. A pull request that adds a package JSON by hand has not been through this door, and CI refuses it (see [Contributing](#contributing)).
- **Out:** `quern pin name@version` → `quern.lock` (the flattened closure, every entry hash-bearing), then `quern sync` → a local Library cache. Sync re-enters through `publish`, so **the proof re-runs on the consumer's machine**: this registry is trusted for transport, not for validation.

Versions are immutable — republishing `name@version` with different content is refused at the door, and CI refuses any diff that modifies or deletes an existing artifact. To evolve a package, publish a new version; to disagree with one, fork-or-republish. No ranges, no resolver algebra.

Native contracts never live here: they are host code, registered in-process (`--natives MODULE`), and a package that names one travels as data while its implementation travels as Python. Meaning is data; safety is code.

Centralized as convention, not mechanism: anyone can root another registry, and a fork is a directory copy.

## Contributing

There is no hand-editing door into a registry. Artifacts arrive one way — `quern publish`, which proves a package against its own examples and counter-examples before writing it. CI enforces the same on pull requests:

- **A published version is never edited or removed.** A PR that modifies or deletes an artifact is refused: a pin resolves to bytes, and changing them under a fixed `name@version` is how a registry would come to lie.
- **A package is not added by hand.** A PR that adds a package JSON is refused too — it has not passed the proof gate, and a registry that accepted unproven artifacts by PR would be trusting GitHub review in place of the proof it exists to run.

So a pull request is for the README, the workflow, and the housekeeping around the registry — not for its contents. To publish a package, run the door. To carry your own packages, root your own registry: a fork is a directory copy, and it is a first-class registry the moment you push it.

The design decisions behind this posture — and the deferred full-validation check that would let the door itself live in CI — are journaled as data under [`ledger/`](ledger/).

---

Apache-2.0. © 2026 Xavier Grehant.
