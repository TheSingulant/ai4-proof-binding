# AI⁴ Proof Binding Status Model v0.1

Closed enum (do not extend in v0.1):

`unavailable` | `declared` | `verified` | `conflict` | `unsupported`

Integrity and domain binding are not truth, trust, reputation, safety, or authorization judgments.

## Status meanings

| Status | When it applies |
| --- | --- |
| `unavailable` | Neither record present; or transport/fetch/policy failure (including private IP, oversize, wrong Content-Type) |
| `declared` | Valid paired declaration without fetch; or post-fetch integrity+name ok but owner binding incomplete |
| `verified` | Digest, schema, subject, issuer name, and UNS owner binding all succeed |
| `conflict` | Incomplete pair, invalid URI/digest, digest mismatch, bad JSON, schema field failure, subject/issuer/owner mismatch |
| `unsupported` | Digest matches JSON whose `schema_version` is not `ai4.proof-manifest.v1` |

## Vector coverage (33/33)

| Vector id | Expected status | Brief reason |
| --- | --- | --- |
| `VALID_FULL_BINDING` | `verified` | Full pair, frozen-identical bytes, subject/issuer/owner bind |
| `NO_RECORDS` | `unavailable` | Neither `ai4.proof` nor `ai4.proof.sha256` |
| `ONLY_AI4_PROOF` | `conflict` | Digest record missing |
| `ONLY_AI4_PROOF_SHA256` | `conflict` | URI record missing |
| `INVALID_PROOF_URI_HTTP` | `conflict` | `http` not `https` |
| `INVALID_DIGEST_FORMAT_UPPERCASE` | `conflict` | Uppercase hex |
| `INVALID_DIGEST_FORMAT_0X_PREFIX` | `conflict` | `0x` prefix |
| `INVALID_DIGEST_FORMAT_WRONG_LENGTH` | `conflict` | Length not 64 |
| `INVALID_DIGEST_FORMAT_NON_HEX` | `conflict` | Non-hex character |
| `MANIFEST_DIGEST_MISMATCH` | `conflict` | Body digest != declared |
| `INVALID_JSON` | `conflict` | Digest matches non-JSON bytes |
| `UNSUPPORTED_SCHEMA_VERSION` | `unsupported` | `schema_version` not v1 |
| `SUBJECT_MISMATCH` | `conflict` | `subject.name` != resolved name |
| `ISSUER_NAME_MISMATCH` | `conflict` | `issuer.name` != resolved name |
| `ISSUER_OWNER_MISSING` | `declared` | Integrity+name ok; owner absent |
| `ISSUER_OWNER_MISMATCH` | `conflict` | Owner checksum differs |
| `EXTRA_SCHEMA_FIELDS` | `conflict` | Disallowed field (`additionalProperties` false) |
| `INVALID_ISSUED_AT` | `conflict` | Not RFC3339 with Z/offset |
| `REDIRECT_PUBLIC_TO_PUBLIC` | `verified` | Public redirect; final bytes match |
| `REDIRECT_TO_PRIVATE_IP` | `unavailable` | Transport reject -> unavailable |
| `PRIVATE_IP_DIRECT_127_0_0_1` | `unavailable` | Transport reject |
| `PRIVATE_IP_DIRECT_10_0_0_1` | `unavailable` | Transport reject |
| `PRIVATE_IP_DIRECT_169_254_169_254` | `unavailable` | Transport reject |
| `PRIVATE_IP_DIRECT___1` | `unavailable` | Transport reject (`::1`) |
| `PRIVATE_IP_DIRECT___ffff_7f00_1` | `unavailable` | Transport reject (IPv4-mapped) |
| `RESPONSE_TOO_LARGE` | `unavailable` | >256 KiB policy |
| `WRONG_CONTENT_TYPE` | `unavailable` | `text/html` |
| `WRONG_CONTENT_TYPE_SPOOF` | `unavailable` | Non-JSON media type |
| `JSON_CONTENT_TYPE_WITH_CHARSET` | `verified` | `application/json; charset=utf-8` ok |
| `PLUS_JSON_MEDIA_TYPE` | `verified` | `application/ai4+json` ok |
| `BYTE_LEVEL_CHANGE` | `conflict` | Whitespace-different body vs digest |
| `OWNER_CASE_NORMALIZATION` | `verified` | EIP-55 normalize equal |
| `NAME_NORMALIZATION` | `verified` | Trim+lowercase of valid name |

## Outside the enum

Invalid `.ai4` name syntax after trim+lowercase is an **input error**, not a status. See SPEC Ambiguity C.

## Decision sketch

```
normalize name (or input error)
if no records -> unavailable
if unpaired records -> conflict
if bad https URI or digest format -> conflict
if no transport -> declared
fetch under transport policy
  on policy/fetch failure -> unavailable
if body sha256 != declared -> conflict
if not UTF-8 JSON object -> conflict
if schema_version != v1 -> unsupported
if schema invalid -> conflict
if subject/issuer name bind fail -> conflict
if owner missing (manifest or context) -> declared
if owner mismatch -> conflict
else -> verified
```
