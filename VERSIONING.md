# AI⁴ Proof Binding Versioning v0.1

## Identifiers

| Item | Value |
| --- | --- |
| Protocol name | AI⁴ Proof Binding |
| Protocol version | `0.1` |
| Release stage | RC1 (release packaging status) |
| Release artifact | `v0.1.0-rc.1` |
| Manifest `schema_version` | `ai4.proof-manifest.v1` |
| Conformance suite | `conformance/v0.1/` (`conformance_version`: `0.1`) |
| Status vocabulary | closed five-value enum (see `STATUS_MODEL.md`) |

## Protocol version and release artifact

Protocol semantics are version `0.1`. RC1 is the packaging status of this release candidate. It is not a separate protocol version.

The release artifact identifier is `v0.1.0-rc.1`. That label identifies the packaged publication. It does not change the protocol version, the manifest `schema_version`, or `conformance_version`.

A later patch to an implementation does not automatically change protocol semantics. Implementation fixes can ship without a protocol version bump only when expected vector statuses, digest rules, schema, and transport-to-status mapping stay the same. A change to those semantics requires a protocol and conformance version bump, as in the compatibility rules below.

## Compatibility rules

1. **Patch / editorial** within 0.1: documentation clarification that does not change expected vector statuses MAY ship without bumping `conformance_version`.
2. **Semantic change** to status outcomes, digest rules, schema, or transport-to-status mapping MUST bump protocol and conformance versions (for example to `0.2`) and MUST add or revise vectors.
3. **New status strings** MUST NOT be introduced under version `0.1`. A future version may add statuses only with a version bump and new vectors.
4. **Schema:** `ai4.proof-manifest.v1` is closed. A new manifest schema MUST use a new `schema_version` const. Digests that parse as JSON with a non-v1 `schema_version` remain `unsupported` under this protocol version.
5. **Name syntax** MUST NOT be broadened without a version bump; invalid names remain input errors.

## Conformance claims

An implementation may claim "AI⁴ Proof Binding Protocol v0.1" only if it passes all vectors in `conformance/v0.1/vectors.json` (33) with matching `expected.status`.

## Implementations (informative)

Independent implementations are cross-checked by status agreement on `conformance/v0.1/`. Passing an implementation does not replace the vector suite. An implementation patch does not by itself change protocol version `0.1`.

## Non-goals for version bumps

Versioning here does not cover UNS record write APIs, custody, ERC-8004, or product UI.
