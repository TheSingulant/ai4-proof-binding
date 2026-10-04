# AI⁴ Proof Binding v0.1 RC1

AI⁴ Proof Binding v0.1 RC1 binds a `.ai4` name to an exact machine-readable proof manifest using the records `ai4.proof` and `ai4.proof.sha256`.

## Purpose

Bind a `.ai4` name to an exact machine-readable proof manifest. `ai4.proof` is the HTTPS location of the manifest. `ai4.proof.sha256` is the SHA-256 digest of the exact manifest bytes.

## Core claims

- Integrity binding: the fetched manifest bytes match `ai4.proof.sha256`.
- Namespace binding: the manifest subject name is the resolved `.ai4` name.
- Issuer-name binding: the manifest issuer name is the resolved `.ai4` name.
- Owner binding: when both are present, the manifest issuer owner matches the resolved owner.
- Deterministic conformance behavior: the same inputs produce the closed status values in the vector suite.

## Boundary

This protocol does not establish truth, reputation, authorization, safety, or endorsement. Status `verified` means the binding checks succeeded. It does not mean the manifest content is true, safe, or endorsed.

## What this package contains

- Normative specification, status model, security notes, and versioning rules
- Closed JSON Schema for `ai4.proof-manifest.v1`
- Offline conformance suite: 33 deterministic vectors
- Informative examples and an implementation guide

## Start here

| Doc | Role |
| --- | --- |
| [SPEC.md](SPEC.md) | Normative behavior |
| [STATUS_MODEL.md](STATUS_MODEL.md) | Status meanings and vector map |
| [SECURITY.md](SECURITY.md) | Threats and known limits |
| [VERSIONING.md](VERSIONING.md) | Protocol version and release artifact |
| [IMPLEMENTATION_GUIDE.md](IMPLEMENTATION_GUIDE.md) | Language-agnostic flow |
| [conformance/v0.1/](conformance/v0.1/) | Vectors and fixtures |
| [schemas/](schemas/) | Manifest schema |
| [examples/](examples/) | Sample inputs and outputs |

## Conformance

Claim protocol v0.1 conformance only if all 33 vectors in `conformance/v0.1/vectors.json` match `expected.status` under offline mocked transport and DNS.

Two independent implementations, one TypeScript and one Python, each passed 33/33 on this public vector set.

## Release

Protocol semantics are version 0.1. RC1 is release packaging status for this candidate. The proposed release artifact tag is `v0.1.0-rc.1`. This package does not apply that tag by itself.

See `PUBLIC_PROVENANCE.md` and `PUBLICATION_INTEGRITY.txt`.

## License

Split license, not a dual license. Documentation and prose are CC BY 4.0. Machine-readable schema, conformance vectors, fixtures, and examples are Apache-2.0. See [LICENSE.md](LICENSE.md) and [LICENSES/](LICENSES/).

Recommended public repository name: `ai4-proof-binding`.
