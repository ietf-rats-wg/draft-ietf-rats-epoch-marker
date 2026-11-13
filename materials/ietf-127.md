---
title: "Epoch Markers Update"
footer: IETF 127, San Francisco, RATS WG
slidenumbers: true
slidecount: true
slidenumber-style: alignment(right)
autoscale: true
theme: Next
---

# Epoch Markers 08

## draft-ietf-rats-epoch-markers-08

### RATS WG, IETF 127

---

## Summary of changes

| Rev | Main changes |
|-----|--------------|
| -04 | Epoclet; `etime` imported from RFC 9581; `em-type` parameter and CoAP Content-Formats; interaction-model text |
| -05 | Privacy Considerations |
| -06 | Epoch Bell EKU; "Epoch Bell-signed" clarified; epoch-tick-list signing |
| -07 | CBOR time tags; stricter EKU; counter state; epoclet section reworked |
| -08 | CWT `em` claim value type fixed |

### All issues closed

---

## New: Epoclet (-04, restructured in -07)

- Problem: per-session nonce state hurts scalability in highly available Verifiers
- Time-synchronised servers share a symmetric key and mint/consume nonces
  - a nonce minted by one server is accepted by any other: no session stickiness
- New Epoch ID type, CBOR tag 26985: `[ [KeyID, Timestamp, Pad], AuthTag ]`
  - `AuthTag` = HMAC-SHA-256 over the deterministic CBOR of `TimeToken`
  - 44 to 64 bytes: fits the 64-byte challenge limit of common Evidence ABIs
- Key handling follows NIST SP 800-57 part 2

---

## New: Epoch Marker media type parameter and CoAP C-Fs (-04)

- `application/epoch-marker+cbor` has an optional `em-type` parameter
  - value is the CBOR tag number, e.g. `em-type=26982` is an Epoch Tick
- One CoAP Content-Format per Epoch Marker type (TBD2..TBD10), plus the generic one (TBD1)
- Lets constrained deployments label the type without parsing the payload

---

## [fit] Changed: `etime` is imported, not redefined (-04)

- Previously: the draft redefined tag 1001 in its CDDL
- Now: `etime` imported from RFC 9581; this document only profiles it
  - key 1 (POSIX time) is mandatory
  - optional accuracy guarantee (key -8), no string keys
  - extra keys must be negative integers; critical (unsigned) keys are discouraged: they hurt interoperability
- No new CBOR tag is introduced

---

## [fit] New: explicit statement on CBOR Time Tags (-07)

- Tags 0, 1 and 1001 are reused as-is; no dedicated Epoch Marker tag is allocated
- The context (`em` claim, media type) identifies the item as an Epoch Marker
- That should be enough, we don't need an explicit tag

---

## New: Epoch Bell certification (-06, tightened in -07)

- Carl Wallace; Relying Parties need to know that a signing key was authorised as an Epoch Bell
- New X.509 EKU `id-kp-epochBell` (with ASN.1 module), for X.509-certified Bells (aligned with RFC3161bis)
  - EKU extension MUST be present and critical
  - MUST NOT include `anyExtendedKeyUsage` or any other purpose
  - RPs that require X.509-certified Bells MUST reject non-conforming certificates
- CBOR-native authorisation (e.g. CWT/COSE trust structures) is out of scope
- New IANA requests: SMI EKU and module identifier

---

## Clarified: "Epoch Bell-signed Epoch Marker" (-06)

- An Epoch Marker has no intrinsic signature
- "Epoch Bell-signed Epoch Marker":
  conveyed in a structure signed by an Epoch Bell
  - required whenever the recipient must establish issuance by an authorised Bell
- Extracting an Epoch ID into another signed message does not make the new signer a Bell
- Epoch Tick List: the whole list MUST be Bell-signed; ticks are then used individually without signatures

---

## Changed: reuse the Handle CMW Indicator (next revision)

- -06 to -08 requested two CMW Indicators: Epoch ID and Nonce
- Dropped: draft-ietf-rats-reference-interaction-models (-20) registers a CMW Indicator for Handles
- An Epoch Marker used as a `handle` is already covered; no registration needed here
- Removes two IANA requests (TBD13, TBD14)

---

## New: counter state requirements (-07)

- Strictly monotonic counter: the Epoch Bell MUST NOT emit a value $$≤$$ one already emitted
  - including across crashes and restarts
- SHOULD persist state; or resume with a safe margin (technique from RFC 8613, Appendix B.1.1)
- If safety cannot be guaranteed: MUST start a new counter context (e.g., new key or Bell identity)
- MUST NOT wrap on overflow

---

## New: Privacy Considerations (-05)

- Predictable emission makes Epoch Markers a system-wide synchronisation signal
  - observers can infer epoch boundaries, correlate messages, link users of the same Bell
- Mitigations: variable emission timing, variable counter increments, scoped markers
- Coupled with an acceptance-window note: variability reduces effective epoch lifetime

---

## New: use with Passport and Background-Check (-04)

- Epoch Marker used as the `handle` in the RATS interaction models
- Verifier MUST apply the Epoch Marker acceptance policy
  - epoch windows, replay and rollback state, skew on the Evidence path
- Background-Check: account for the extra hop via the Relying Party
- Passport: Attestation Results stay bound to the Epoch Marker (or its `handle`)

---

## Fixed: `em` CWT claim value type (-08)

- Claim value type was "CBOR array";
  now "CBOR-tagged Epoch ID"
- IANA registration for the claim and media type name aligned
  (`application/epoch-marker+cbor`)

---

## Editorial

- Ref updates & co

---

## Next steps

- Request allocations:
  CBOR tag, CWT claim, media type, Content-Format, SMI
- Editors believe that the draft is ready for WGLC
