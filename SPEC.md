# AI⁴ Proof Binding Protocol Specification v0.1

**Status:** Release Candidate 1 (RC1)
**Conformance:** `conformance/v0.1/` (33 vectors)
**Schema:** `schemas/ai4-proof-manifest-v1.schema.json` (`schema_version` = `ai4.proof-manifest.v1`)

This document describes current verified behavior of AI⁴ proof binding as exercised by two independent implementations against the v0.1 conformance suite. It does not add protocol features beyond that behavior.

## 1. Purpose and non-goals

### 1.1 Purpose

AI⁴ proof binding lets a resolver check that UNS records for a second-level `.ai4` name point at a manifest whose bytes match a declared SHA-256 digest, and that the manifest binds that name (and optionally the UNS owner address) in a closed JSON shape.

### 1.2 Non-goals (normative)

Verification MUST NOT be interpreted as:

- truth about an agent, artifact, or claim
- trust, reputation, or safety judgment
- authorization or access control
- proof of on-chain custody beyond comparing a supplied owner address to `issuer.owner`
- ERC-8004 registration or any registry write

Integrity and domain binding are structural checks only.

## 2. Terminology

| Term | Meaning |
| --- | --- |
| Name | Supported second-level `.ai4` string after trim and lowercase |
| Records | Map of UNS text records; protocol keys are `ai4.proof` and `ai4.proof.sha256` |
| Manifest | JSON document fetched at `ai4.proof` whose SHA-256 equals `ai4.proof.sha256` |
| Resolved owner | Caller-supplied UNS owner address context used for owner binding |
| Status | One of the closed enum values in Section 4 |
| Transport | Component that fetches HTTPS bytes under transport policy (Section 8) |

The keywords MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY are used as in RFC 2119.

## 3. Inputs

A verification invocation takes:

1. `name` (string): candidate `.ai4` name
2. `records` (map): may contain `ai4.proof` and/or `ai4.proof.sha256`
3. `owner` (string or absent): resolved UNS owner address
4. Transport (present or absent): if absent, only declaration syntax is evaluated

## 4. Closed status vocabulary

Implementations MUST return exactly one of:

| Status | Meaning (v0.1) |
| --- | --- |
| `unavailable` | No declaration records, or transport/fetch/policy failure |
| `declared` | Syntactically valid paired declaration without fetch; or post-fetch integrity and name binding ok with incomplete owner binding |
| `verified` | Digest, schema, subject, issuer name, and UNS owner binding all succeed |
| `conflict` | Incomplete/invalid declaration, digest mismatch, JSON/schema/subject/issuer/owner mismatch |
| `unsupported` | Digest matches JSON whose `schema_version` is not `ai4.proof-manifest.v1` |

Implementations MUST NOT invent additional status strings in v0.1.

## 5. Name normalization

1. Implementations MUST accept only second-level `.ai4` names matching:
   `^[a-z0-9](?:[a-z0-9-]{0,61}[a-z0-9])?\.ai4$` after normalization.
2. Allowed normalization MUST be trim of leading/trailing whitespace, then lowercase.
3. Implementations MUST NOT broaden name syntax in v0.1.
4. Names that fail this pattern after trim+lowercase are an **input error**, not a proof status. Implementations MUST surface this outside the status enum (for example by throwing). See Ambiguity C.

## 6. Declaration inspection (pre-fetch)

Let `has_proof` be true when `ai4.proof` is present and non-empty.
Let `has_digest` be true when `ai4.proof.sha256` is present and non-empty.

1. If neither is present: status MUST be `unavailable`.
2. If exactly one is present: status MUST be `conflict`.
3. `ai4.proof` MUST be an HTTPS URI with a host and MUST NOT contain URL userinfo (username/password). Otherwise status MUST be `conflict`.
4. `ai4.proof.sha256` MUST be exactly 64 lowercase hex characters (`^[0-9a-f]{64}$`). Uppercase, `0x` prefix, wrong length, or non-hex MUST yield `conflict`.
5. If both are syntactically valid and no transport is provided: status MUST be `declared` (Ambiguity B, sense a).

## 7. Manifest fetch and integrity

When transport is provided and declaration syntax is valid:

1. Transport MUST attempt to fetch `ai4.proof` under Section 8 policy.
2. If transport fails or rejects the request under policy: status MUST be `unavailable` (Ambiguity A). This is normative v0.1 behavior for both reference implementations.
3. Implementations MUST compute SHA-256 over the exact verifier-visible response body bytes.
4. If the hex digest (lowercase) does not equal `ai4.proof.sha256`: status MUST be `conflict`.
5. Body bytes MUST decode as UTF-8 JSON. Invalid JSON or a non-object root MUST yield `conflict`.
6. Digests are byte-exact. Semantically equivalent JSON with different whitespace MUST NOT validate against another encoding's digest.

## 8. Transport policy (normative minima)

Transport is separate from protocol status semantics, but policy failures map into the closed enum as `unavailable`.

Transport MUST:

1. Use HTTPS only (no `http://`).
2. Reject URLs with userinfo.
3. Reject literal non-public / special IP hosts in the URL (including common loopback, private, link-local, and IPv4-mapped forms exercised by conformance).
4. Reject redirects to non-public / special IP targets.
5. Enforce a maximum response size of 256 KiB (262144 bytes). Declared `Content-Length` above the cap MUST be rejected.
6. Accept `Content-Type` whose type/subtype (before parameters) is `application/json` or ends with `+json`. Charset parameters MAY be present. Other media types MUST be rejected.

Transport SHOULD:

1. Limit redirect hops (implementations MAY choose a finite limit; suite exercises one hop).
2. Apply a finite request timeout for live fetches.

Residual SSRF and DNS-rebinding limits are documented in `SECURITY.md`. They are not fully eliminated by v0.1.

## 9. Schema and binding

After digest match and JSON object parse:

1. If `schema_version` is not exactly `ai4.proof-manifest.v1`: status MUST be `unsupported`.
2. Otherwise the document MUST validate against `schemas/ai4-proof-manifest-v1.schema.json` (closed objects with `additionalProperties: false` on core objects). Failure MUST yield `conflict`.
3. `issued_at` MUST match RFC3339 with required `Z` or numeric offset:
   `^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(?:\.\d+)?(?:Z|[+-]\d{2}:\d{2})$`
   Date-only or timezone-less strings that some parsers accept MUST still be `conflict`.
4. `subject.name` MUST equal the normalized resolved name; else `conflict`.
5. `issuer.type` MUST be `uns-name` and `issuer.name` MUST equal the normalized resolved name; else `conflict`.
6. Owner binding:
   - If `issuer.owner` is absent or empty: status MUST be `declared` (Ambiguity B, sense b).
   - If resolved `owner` context is absent while `issuer.owner` is present: status MUST be `declared`.
   - If both are present: compare with EIP-55 checksum normalization. Mismatch or invalid address MUST be `conflict`. Match MUST be `verified`.
7. Case differences in hex address letters that checksum-normalize equally MUST still yield `verified` when otherwise valid.

## 10. Manifest shape (informative summary)

Required fields: `schema_version`, `subject`, `issued_at`, `issuer`, `artifact`.
Optional: `proof`, `metadata`.
See the schema file for patterns and `additionalProperties` rules. Extra top-level fields MUST fail validation (`conflict`).

## 11. Output

Implementations SHOULD return at least:

- `name` (normalized)
- `status` (closed enum)
- `manifest` (parsed object or null)
- `notes` (human-readable strings; not a machine enum)
- binding flags when applicable (`subject_name_match`, `issuer_type_ok`, `issuer_name_match`, `owner_present`, `owner_match`)

Notes MUST NOT claim truth, trust, reputation, safety, or authorization.

## 12. Conformance

An implementation claims AI⁴ Proof Binding Protocol v0.1 conformance when it produces the expected `status` for every vector in `conformance/v0.1/vectors.json` (33 vectors) under offline mocked transport/DNS as described in `conformance/v0.1/README.md`.

## 13. Documented ambiguities (no behavior change in v0.1)

### Ambiguity A - Transport rejects map to `unavailable`

Transport policy rejections (private/special IP direct, redirect to private IP, response larger than 256 KiB, non-JSON Content-Type) and other fetch failures all surface as `unavailable`. There is no distinct `rejected` status in the closed enum. **Normative for v0.1** for both reference implementations.

### Ambiguity B - `declared` overload

`declared` covers both:

1. Pre-fetch: syntactically valid paired records with no transport / no fetch performed
2. Post-fetch: integrity + subject + issuer-name success with missing owner binding (`ISSUER_OWNER_MISSING`)

Callers MUST treat `declared` as incomplete verification, not as `verified`.

### Ambiguity C - Invalid `.ai4` name is an input error

Out-of-syntax names after trim+lowercase are outside the status enum. Implementations raise an error rather than returning a proof status. The suite does not broaden name syntax.

## 14. Frozen production reference (informative)

Local fixture `conformance/v0.1/fixtures/valid-full-binding.json` is a byte-identical copy of:

- URI: `https://www.thesingulant.ai/proof/manifests/architect-ai4.json`
- SHA-256: `0d8e6572588523f262ed45598e72835f791922d9d18df0705702185a616c6fae`
- Owner: `0xA3Ac252805dbC452c232dFb2b5eEF7F3f274501D`
- Name: `architect.ai4`

The conformance suite MUST run offline and MUST NOT depend on live fetches of that URL. Production artifacts MUST NOT be modified by this protocol package.
