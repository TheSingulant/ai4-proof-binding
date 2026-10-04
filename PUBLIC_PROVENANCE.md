# Public provenance (AI⁴ Proof Binding v0.1 RC1)

## What is normative

Protocol semantics are version 0.1.

- `SPEC.md` is the canonical prose specification.
- `schemas/ai4-proof-manifest-v1.schema.json` is the canonical closed schema (`schema_version` = `ai4.proof-manifest.v1`).
- `conformance/v0.1/vectors.json` and `conformance/v0.1/fixtures/` are the canonical conformance suite. A v0.1 conformance claim requires every vector `expected.status`.
- `STATUS_MODEL.md` records the closed status vocabulary used by the spec. It does not add statuses.

## What is informative

These files explain the package. They do not add statuses, digest rules, or schema requirements beyond the normative files:

- `README.md`
- `SECURITY.md` (threats and residual limits; where it restates transport rules, `SPEC.md` Section 8 is canonical)
- `VERSIONING.md`
- `IMPLEMENTATION_GUIDE.md`
- `PUBLIC_PROVENANCE.md`
- `conformance/v0.1/README.md`
- `examples/`
- `LICENSE.md` and `LICENSES/`
- `PUBLICATION_INTEGRITY.txt` (file digests only)

## How conformance relates to the spec

Each vector fixes a name, record map, owner context, and optional mocked HTTP response, including fixture bytes. The expected status is the outcome `SPEC.md` requires for that input. Implementations compare status only. Notes inside vectors are informative and are not an extra status enum.

The suite runs offline. It does not fetch live network resources and it does not modify production manifests.

## Independent cross-check

Implementation behavior was cross-checked independently in TypeScript and in Python against this same public vector set. Both produced 33/33 status agreement. Agreement means matching `expected.status`. It does not certify live transport parity, product quality, or on-chain custody.

## Publication scrub

The publication scrub changed no protocol semantics. Vector identifiers, inputs, record values, fixture bytes, and expected statuses are unchanged. Publication metadata names this protocol and version 0.1 only. Informative notes that named implementation functions were reworded without changing expected results.

## Non-claims

Integrity, namespace, issuer-name, and owner binding are structural checks. `verified` is not truth, reputation, authorization, safety, or endorsement. Residual DNS rebinding and TOCTOU limits for live fetches are in `SECURITY.md`.

## Integrity

`PUBLICATION_INTEGRITY.txt` lists SHA-256 digests for the public files, excluding itself. Compare those digests after checkout.

## License

Documentation and prose are CC BY 4.0. Machine-readable schema, vectors, fixtures, and examples are Apache-2.0. See `LICENSE.md`. This is a split license, not a dual license.

Recommended public repository name: `ai4-proof-binding`.
