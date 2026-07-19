# Conformance suite

This suite applies to implementations claiming conformance with the x402 Identity Assurance Profile.

| ID | Test | Pass condition |
|---|---|---|
| IA-01 | Call a paid AL0 service without payment | `402` with a well-formed x402 requirement |
| IA-02 | Compare discovery metadata with the `402` response | Scheme, network, asset, payee and amount are identical |
| IA-03 | Call with valid identity, sufficient assurance and valid payment | Normal response; underlying service invoked exactly once |
| IA-04 | Call an AL1 service without identity | `403 access_denied`; no payment details, and no indication of which gate failed |
| IA-05 | Call an AL2 service with valid AL1 evidence | `403 insufficient_assurance`; the specific identifier is permitted because the caller is identified |
| IA-06 | Identified AL1 caller calls a mandate-protected service with no mandate | `403 mandate_required`; no payment details disclosed |
| IA-07 | Call with a revoked credential or participant status | `403`; no settlement attempted. Revoked evidence does not identify a caller, so an otherwise unidentified caller receives `access_denied` |
| IA-08 | Identified caller sends a valid payment with an invalid mandate | `403 invalid_mandate`; payment not settled |
| IA-09 | Replay a settled payment authorisation | Rejected; underlying service not invoked |
| IA-10 | Call a free service | Normal response; no `402` |
| IA-11 | Fetch the signed pricing document | Signature verifies and declarations match service metadata |
| IA-12 | Inspect the audit record for a successful paid call | Identity, assurance, price, settlement and delivery outcome are bound |
| IA-13 | Inspect the audit record for an identity refusal | Refusal is recorded without a settlement reference |
| IA-14 | Present payment before identity to an AL1 service | Identity is evaluated first; response remains `403` |
| IA-15 | Suspend a caller after discovery but before invocation | Fresh status check prevents settlement and service invocation |
| IA-16 | As an unidentified caller, call (a) an AL1 service, (b) a mandate-protected service, (c) a service that does not exist | All three responses are **indistinguishable**: same status, same error identifier, and no field revealing which gate failed or whether the service exists |

## Critical tests

IA-04, IA-05, IA-06, IA-07, IA-08, IA-14 and IA-16 are critical.

IA-16 enforces section 6.1: a refusal must not tell an anonymous caller which
gate it failed. An implementation returning a distinct identifier per gate to an
unidentified caller fails, however correct its precedence order is.

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
