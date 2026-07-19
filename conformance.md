# Conformance suite

This suite applies to implementations claiming conformance with the x402 Identity Assurance Profile.

| ID | Test | Pass condition |
|---|---|---|
| IA-01 | Call a paid AL0 service without payment | `402` with a well-formed x402 requirement |
| IA-02 | Compare discovery metadata with the `402` response | Scheme, network, asset, payee and amount are identical |
| IA-03 | Call with valid identity, sufficient assurance and valid payment | Normal response; underlying service invoked exactly once |
| IA-04 | Call an AL1 service without identity | `403 identity_required`; no payment details disclosed |
| IA-05 | Call an AL2 service with AL1 evidence | `403 insufficient_assurance`; no payment details disclosed |
| IA-06 | Call a mandate-protected service without a mandate | `403 mandate_required`; no payment details disclosed |
| IA-07 | Call with a revoked credential or participant status | `403 status_revoked`; no settlement attempted |
| IA-08 | Call with a valid payment but invalid mandate | `403 invalid_mandate`; payment not settled |
| IA-09 | Replay a settled payment authorisation | Rejected; underlying service not invoked |
| IA-10 | Call a free service | Normal response; no `402` |
| IA-11 | Fetch the signed pricing document | Signature verifies and declarations match service metadata |
| IA-12 | Inspect the audit record for a successful paid call | Identity, assurance, price, settlement and delivery outcome are bound |
| IA-13 | Inspect the audit record for an identity refusal | Refusal is recorded without a settlement reference |
| IA-14 | Present payment before identity to an AL1 service | Identity is evaluated first; response remains `403` |
| IA-15 | Suspend a caller after discovery but before invocation | Fresh status check prevents settlement and service invocation |

## Critical tests

IA-04, IA-05, IA-06, IA-07, IA-08 and IA-14 are critical.

An implementation failing a critical test MUST NOT claim conformance.

## Test evidence

A conformance report should include:

- implementation name and version;
- profile version;
- timestamp;
- test environment;
- per-test result;
- relevant response status and error identifier;
- settlement reference for successful payment tests;
- redacted audit-record evidence;
- tester identity or automated runner signature.
