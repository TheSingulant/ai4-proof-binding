# AI4 Proof Binding Conformance Suite v0.1

Protocol-level deterministic test vectors for `ai4.proof` / `ai4.proof.sha256` binding.

Any AI4 resolver implementation can consume `vectors.json` to prove correct binding semantics. This suite tests the protocol, not a single product UI.

## Status vocabulary (closed)

Closed status enum for protocol v0.1:

| Status | Meaning |
| --- | --- |
| `unavailable` | No declaration, or fetch/transport policy failed |
| `declared` | Syntactically valid declaration, or integrity+name ok but owner binding incomplete |
| `verified` | Digest, schema, subject, issuer name, and UNS owner binding all succeed |
| `conflict` | Incomplete/invalid declaration, digest mismatch, schema/subject/issuer/owner mismatch |
| `unsupported` | Digest matches JSON whose `schema_version` is not `ai4.proof-manifest.v1` |

Integrity and domain binding are not truth, trust, or authorization judgments.

## Layout

```
conformance/v0.1/
  vectors.json
  fixtures/
  README.md
```

## How to consume vectors.json

For each entry in `vectors`:

1. Treat `name` as the resolved domain input (apply only the protocol's allowed name normalization: trim + lowercase of a valid second-level `.ai4` name).
2. Treat `resolved_owner` as the UNS owner context (checksum-normalize with the same rules as the protocol).
3. Treat `records` as the UNS record map (keys `ai4.proof`, `ai4.proof.sha256`).
4. If `http` is null, do not fetch; run declaration inspection / verify entry only.
5. If `http` is present, simulate HTTP offline:
   - Use `redirect_chain` when present (manual redirects).
   - Otherwise serve `body_fixture` bytes with the given `status` and `headers`.
   - Use `resolved_ip` for simulated DNS answers for hostnames.
   - For `body_mode: "oversized"`, enforce the protocol max size (256 KiB).
6. Compute digests only from exact fixture file bytes. Never invent hashes.
7. Compare the implementation's proof-manifest `status` to `expected.status`.

### Fixture digests

Every digest in `records["ai4.proof.sha256"]` that is intended to match a body was produced with:

```bash
sha256sum conformance/v0.1/fixtures/<file>
```

`fixtures/valid-full-binding.json` is a local byte-identical copy of the frozen production manifest:

- URI: `https://www.thesingulant.ai/proof/manifests/architect-ai4.json`
- sha256: `0d8e6572588523f262ed45598e72835f791922d9d18df0705702185a616c6fae`

The suite itself must not depend on live network fetches of that URL.

## Category map (1-25)

1. `VALID_FULL_BINDING` -> `verified`
2. `NO_RECORDS` -> `unavailable`
3. `ONLY_AI4_PROOF` -> `conflict`
4. `ONLY_AI4_PROOF_SHA256` -> `conflict`
5. `INVALID_PROOF_URI_HTTP` -> `conflict`
6. `INVALID_DIGEST_FORMAT_*` -> `conflict` (uppercase, 0x prefix, wrong length, non-hex)
7. `MANIFEST_DIGEST_MISMATCH` -> `conflict`
8. `INVALID_JSON` -> `conflict`
9. `UNSUPPORTED_SCHEMA_VERSION` -> `unsupported`
10. `SUBJECT_MISMATCH` -> `conflict`
11. `ISSUER_NAME_MISMATCH` -> `conflict`
12. `ISSUER_OWNER_MISSING` -> `declared`
13. `ISSUER_OWNER_MISMATCH` -> `conflict`
14. `EXTRA_SCHEMA_FIELDS` -> `conflict`
15. `INVALID_ISSUED_AT` -> `conflict`
16. `REDIRECT_PUBLIC_TO_PUBLIC` -> `verified`
17. `REDIRECT_TO_PRIVATE_IP` -> `unavailable` (transport rejection folded into unavailable)
18. `PRIVATE_IP_DIRECT_*` -> `unavailable`
19. `RESPONSE_TOO_LARGE` -> `unavailable`
20. `WRONG_CONTENT_TYPE*` -> `unavailable`
21. `JSON_CONTENT_TYPE_WITH_CHARSET` -> `verified`
22. `PLUS_JSON_MEDIA_TYPE` -> `verified`
23. `BYTE_LEVEL_CHANGE` -> `conflict`
24. `OWNER_CASE_NORMALIZATION` -> `verified`
25. `NAME_NORMALIZATION` -> `verified`

## Implementations

This package ships the vectors and fixtures. It does not ship an implementation. An implementation injects `records`, mocks fetch and DNS from the `http` object, and compares `status` to `expected.status`. The suite does not write name records.

## Protocol notes for implementers

- HTTPS only for `ai4.proof` URIs (no credentials in URL).
- Digest must be lowercase hex, length 64, no `0x`.
- Manifest JSON is validated against the closed v1 shape (`additionalProperties` false on core objects).
- `issued_at` must match RFC3339 with `Z` or numeric offset (Date.parse alone is not enough).
- Fetch policy: public hosts only, max 256 KiB, JSON media types (`application/json` or `*/*+json`, charset parameters allowed).
- Transport policy failures currently surface as `unavailable`, not a separate rejected status.
