# AI⁴ Proof Binding Security Model v0.1

## Scope

This document covers threats to **proof binding verification** (declaration records, HTTPS fetch of a manifest, digest check, schema and name/owner binding). It does not claim to secure UNS custody, agent behavior, or artifact content beyond digests recorded in the protocol.

Binding success is not a trust, truth, reputation, safety, or authorization verdict.

## Assets

| Asset | Why it matters |
| --- | --- |
| UNS records `ai4.proof` / `ai4.proof.sha256` | Pointer and integrity commitment |
| Manifest bytes | Bound document; digested exactly |
| Resolved name and owner context | Domain binding inputs from the resolver |
| Verifier host network / metadata | Must not be leveraged via SSRF |

## Protocol vs transport

| Layer | Responsibility |
| --- | --- |
| **Protocol** | Pair semantics, digest format, exact SHA-256 of body bytes, JSON Schema, subject/issuer/owner binding, closed status enum |
| **Transport** | HTTPS, no userinfo, public-IP policy, redirects, 256 KiB cap, JSON/`+json` Content-Type, timeouts |

Transport policy failures MUST map to status `unavailable` in v0.1 (no separate `rejected` status). See SPEC Ambiguity A.

Implementations SHOULD keep transport and protocol verification separable so offline conformance can mock fetch/DNS without changing status semantics.

## Threat model

### T1 - Manifest substitution / tampering

**Threat:** Attacker serves different bytes at the URI.
**Mitigation:** Exact SHA-256 of verifier-visible body MUST match `ai4.proof.sha256`. Whitespace-equivalent JSON with different bytes MUST fail (`BYTE_LEVEL_CHANGE`).

### T2 - Incomplete declaration

**Threat:** Only URI or only digest is published.
**Mitigation:** Both records MUST be present together; else `conflict`.

### T3 - Weak pointer

**Threat:** Cleartext HTTP or credentialed URL.
**Mitigation:** HTTPS only; userinfo forbidden; else `conflict` at declaration inspection.

### T4 - Digest format confusion

**Threat:** Uppercase, `0x` prefix, truncated, or non-hex digests accepted loosely.
**Mitigation:** Closed pattern `^[0-9a-f]{64}$` only; else `conflict`.

### T5 - Schema / binding confusion

**Threat:** Extra fields, wrong subject/issuer, missing timezone in `issued_at`, wrong schema version.
**Mitigation:** Closed v1 schema; `unsupported` for other `schema_version`; `conflict` for validation/binding failures.

### T6 - Owner spoof without custody check

**Threat:** Manifest claims an owner that is not the resolved UNS owner.
**Mitigation:** When both addresses are present, EIP-55-normalized compare MUST succeed for `verified`; mismatch is `conflict`. Missing owner yields `declared`, not `verified`.

### T7 - SSRF via proof URI / redirects

**Threat:** URI or redirect targets private/special IPs (loopback, RFC1918, link-local, metadata).
**Mitigation:** Transport MUST reject non-public literal hosts and redirects to such targets; result is `unavailable`.

### T8 - Resource exhaustion

**Threat:** Huge response bodies.
**Mitigation:** 256 KiB maximum; oversize is `unavailable`.

### T9 - Content-Type confusion

**Threat:** HTML or other non-JSON payloads accepted as manifests.
**Mitigation:** Require `application/json` or `*/*+json` (charset parameters allowed); else `unavailable`.

### T10 - Name syntax abuse

**Threat:** Unexpected name forms treated as verified bindings.
**Mitigation:** Second-level `.ai4` pattern only; invalid names are input errors outside the status enum.

## Known security limitations (v0.1)

1. **DNS rebinding / TOCTOU:** Checking that DNS answers are public before connect does not always pin the TCP peer address. Residual rebinding risk remains for live transports.
2. **No CA pinning:** Live HTTPS uses the platform trust store.
3. **Redirect hop limit:** Finite but not numerically fixed by the vector suite (suite exercises one hop).
4. **Owner context is supplied:** The protocol compares addresses; it does not itself prove live chain custody or prevent a caller from supplying a wrong owner context.
5. **`declared` is not verified:** Callers that treat `declared` as success over-trust incomplete bindings (Ambiguity B).
6. **Transport failures collapse to `unavailable`:** Operators cannot distinguish policy reject from network failure via status alone (Ambiguity A); notes MAY help but are not a machine enum.
7. **Artifact bytes beyond the manifest digest** are out of scope of binding verification.
8. **No claim of transport parity** between TypeScript and Python live clients beyond offline conformance.

## Operational guidance

- Prefer offline conformance and pinned fixtures for CI.
- Label live networked results explicitly.
- Do not write UNS, custody, or ERC-8004 state as part of verification.
- Do not modify frozen production manifests under `thesingulant.ai` from verifier tooling.
