# bom-registry

This working tree **is** a `bom.library.Library` root: `packages/<name>/<version>.json`
plus `blobs/<sha256>`. There is no format here that bom does not already own — the
repo is nothing but a published Library, and git is its transport.

Git is transport, **not identity**. Identity is the content digest a pin carries
(`PackageRef.sha256`, the hash of the package's canonical serialization), checked at
resolve time on every machine that consumes it. A checkout that mangles bytes cannot
lie about meaning; a registry that drifted would be caught by the first pin it fails.

## The doors

- **In:** `bom publish <artifact> --registry <this checkout>` — and nothing else.
  Publishing is proof-gated: every rule exercised by the package's own examples,
  every rule refuted by its counter-examples staged alone, every wasm blob true to
  its hash and ABI. A package that cannot demonstrate itself does not enter.
- **Out:** `bom pin name@version` → `bom.lock` (the flattened closure, every entry
  hash-bearing), then `bom sync` → a local Library cache. Sync re-enters through
  `publish`, so **the proof re-runs on the consumer's machine**: this registry is
  trusted for transport, not for validation.

Versions are immutable — republishing `name@version` with different content is
refused at the door, and CI refuses any diff that modifies or deletes an existing
artifact. To evolve a package, publish a new version; to disagree with one,
fork-or-republish. No ranges, no resolver algebra.

Native contracts never live here: they are host code, registered in-process
(`--natives MODULE`), and a package that names one travels as data while its
implementation travels as Python. Meaning is data; safety is code.

Centralized as convention, not mechanism: anyone can root another registry, and a
fork is a directory copy.
