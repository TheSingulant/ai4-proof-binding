# AI⁴ Proof Binding Implementation Guide v0.1

Language-agnostic guide. Do not treat library or product names as normative. Normative rules live in `SPEC.md`. Validate with `conformance/v0.1/`.

## Pseudo-flow

```
function verify_proof_binding(name, records, owner, transport):
  normalized = normalize_ai4_name(name)
    # trim + lowercase; if pattern fail -> INPUT ERROR (not a status)

  proof = records.get("ai4.proof")
  digest = records.get("ai4.proof.sha256")
  has_proof = proof is present and non-empty
  has_digest = digest is present and non-empty

  if not has_proof and not has_digest:
    return status unavailable

  if has_proof != has_digest:
    return status conflict

  if not is_https_uri_without_userinfo(proof):
    return status conflict
  if not matches /^[0-9a-f]{64}$/ (digest):
    return status conflict

  if transport is absent:
    return status declared   # Ambiguity B sense (a)

  try:
    response = transport.fetch(proof)   # enforce Section 8 policy
  catch transport_or_policy_error:
    return status unavailable           # Ambiguity A (normative v0.1)

  body = response.body_bytes
  if sha256_hex_lowercase(body) != digest:
    return status conflict

  try:
    doc = utf8_json_parse_object(body)
  catch:
    return status conflict

  if doc.schema_version != "ai4.proof-manifest.v1":
    return status unsupported

  if not validate_against_v1_schema(doc):  # includes RFC3339 issued_at
    return status conflict

  if doc.subject.name != normalized:
    return status conflict
  if doc.issuer.type != "uns-name" or doc.issuer.name != normalized:
    return status conflict

  if issuer.owner missing/empty:
    return status declared   # Ambiguity B sense (b)
  if owner context missing:
    return status declared
  if not eip55_equal(issuer.owner, owner):
    return status conflict

  return status verified
```

## Required building blocks

1. **Name gate:** second-level `.ai4` regex after trim+lowercase; failures are input errors.
2. **Declaration gate:** pair presence, HTTPS pointer, lowercase-64 digest.
3. **Transport:** policy from `SPEC.md` Section 8; failures -> `unavailable`.
4. **Integrity:** SHA-256 of exact body bytes only.
5. **Schema:** load `schemas/ai4-proof-manifest-v1.schema.json`; enforce RFC3339 with Z/offset for `issued_at`.
6. **Binding:** subject and issuer name equality; owner via EIP-55 checksum compare.
7. **Status:** only the five closed values.

## Offline conformance harness

For each vector in `vectors.json`:

1. Use `name`, `resolved_owner`, and `records` as inputs.
2. If `http` is null, call verify without fetch (or with a transport that must not be invoked).
3. If `http` is set, mock DNS/`resolved_ip`, serve `body_fixture` bytes with given status/headers, honor `redirect_chain` and `body_mode: oversized`.
4. Compare only `status` to `expected.status`.
5. Compute digests from fixture file bytes; never invent hashes.

## What not to implement as "features" in v0.1

- New status strings (for example `rejected`)
- Broader name syntax
- Treating `declared` as success UX equivalent to `verified`
- Live dependency on production manifest URLs inside the suite
- UNS writes, custody changes, or ERC-8004 registration from the verifier

## Testing checklist

- [ ] 33/33 offline vectors pass
- [ ] Fixture SHA-256 values unchanged from published suite
- [ ] Invalid name raises input error (spot check outside suite)
- [ ] Notes never claim truth/trust/authorization
