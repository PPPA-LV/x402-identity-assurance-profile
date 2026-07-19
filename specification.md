# x402 Identity Assurance Profile

## Status

Draft v0.1. This specification is experimental.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** are to be interpreted as described in RFC 2119 and RFC 8174.

## 1. Scope

This profile defines how an x402-protected service binds payment to:

- caller identity;
- identity assurance;
- mandate or delegation;
- participant status and revocation;
- service entitlement;
- auditable delivery.

It does not alter x402 settlement semantics.

## 2. Core principle

A settled payment proves that consideration was supplied. It does not prove identity, authority or entitlement.

A service **MUST NOT** treat payment as a substitute for identification or authorisation.

Where a service requires identity, assurance, mandate or valid participant status, those requirements **MUST** be evaluated before a payment requirement is returned.

## 3. Roles

| Role | Function |
|---|---|
| Service provider | Publishes the service, sets conditions and receives settlement. |
| Caller | Requests the service and may present identity, mandate and payment evidence. |
| Principal | Person or organisation represented by the caller. |
| Identity issuer | Issues identity or assurance evidence. |
| Mandate issuer | Issues delegation or authority evidence. |
| Status authority | Publishes revocation or suspension state. |
| Facilitator | Verifies and broadcasts payment authorisations without necessarily taking custody. |

## 4. Assurance extension

A protected resource or tool **MUST** declare an `x-assurance` object when identity or policy conditions apply.

```json
{
  "x-assurance": {
    "minAssuranceLevel": "AL1",
    "requiresIdentity": true,
    "requiresMandate": false,
    "personalData": true,
    "legallySignificant": false,
    "acceptedCredentialTypes": ["OrganisationIdentityCredential"],
    "acceptedTrustFrameworks": ["https://example.org/trust-framework"],
    "invoiceMode": "aggregate-monthly"
  }
}
```

### 4.1 Fields

| Field | Requirement |
|---|---|
| `minAssuranceLevel` | `AL0`, `AL1` or `AL2`. Defaults to `AL0`. |
| `requiresIdentity` | Whether an identified caller is required. |
| `requiresMandate` | Whether evidence of authority to act for a principal is required. |
| `personalData` | Whether the response may contain personal data. |
| `legallySignificant` | Whether the response or action has legal significance. |
| `acceptedCredentialTypes` | Credential types accepted by the provider. |
| `acceptedTrustFrameworks` | Trust frameworks accepted by the provider. |
| `invoiceMode` | `per-call`, `aggregate-monthly`, or `none`. |
| `priceDisplay` | Optional human-readable display value without protocol meaning. |
| `assuranceFramework` | Optional URI of an external assurance framework (see 8.2). |
| `frameworkLevel` | Optional level identifier within `assuranceFramework` (see 8.2). |

A legally significant service **MUST NOT** use `AL0`.

A service returning personal data to an unidentified caller **MUST** have an explicit legal and policy basis. Implementations are RECOMMENDED to require at least `AL1`.

## 5. Processing order

The order is normative.

1. Resolve the requested service.
2. Determine whether it is free or paid.
3. Resolve any presented caller identity.
4. Establish assurance level.
5. Verify mandate where required.
6. Check credential, participant and mandate status.
7. Evaluate service entitlement.
8. Determine whether an invoice or contractual arrangement bypasses per-call settlement.
9. If payment is required and absent, return x402 payment requirements.
10. If payment is present, verify and settle it.
11. Invoke the underlying service.
12. Record the decision, settlement and delivery result.

The underlying service **MUST NOT** be invoked before all applicable checks and required settlement succeed.

## 6. Response precedence

The *condition* determines whether the request proceeds; §6.1 determines how
much the response is permitted to say about why it did not.

| Condition | Outcome |
|---|---|
| Service is free and policy is satisfied | Normal processing |
| Credential, mandate or participant is revoked or suspended | Refused |
| Required identity is absent | Refused |
| Assurance level is insufficient | Refused |
| Required mandate is absent or invalid | Refused |
| Caller is otherwise not entitled | Refused |
| Caller is eligible, payment is required and absent | `402` with x402 payment requirements |
| Payment fails verification or settlement | `402` with payment error |
| Policy and payment succeed | Normal processing |

A refusal is `403`.

A `403` response produced under this section **MUST NOT** disclose price, settlement asset, payee account, facilitator, payment network, or any other payment requirement.

### 6.1 Error granularity

A distinct error identifier per gate tells an unidentified caller which gate it
failed, and therefore what to acquire in order to pass. Repeated across a
catalogue it reveals which services exist, which are mandate-protected, and
which callers are revoked rather than merely anonymous. That is the
differential-error leakage §15 requires implementations to address, so this
profile does not permit conformant behaviour to produce it.

The rule is: **an identified caller may be told why; an unidentified caller may
not.**

| Caller state | Permitted refusal detail |
|---|---|
| Not identified, or identified below `AL1` | A single generic identifier: `access_denied`. The response **MUST NOT** distinguish absent identity from insufficient assurance, absent mandate, revoked status, or a service that does not exist. |
| Identified at `AL1` or above | The specific identifier from §14 **MAY** be returned, because the caller has already established who it is and the disclosure is made to a known party. |

A provider **MUST** record the specific reason in the audit record (§12)
regardless of what it returns on the wire. Refusals remain diagnosable by the
provider and, through a support channel, by the affected party; they are not
diagnosable by an anonymous prober.

A provider **MAY** return the specific identifier to an unidentified caller for
a service it has explicitly published as having no confidential existence, price
or policy. This is an opt-in for openly documented public services and
**MUST NOT** be the default.

## 7. Discovery

A caller **MUST** be able to discover both price and assurance requirements before invoking a paid service.

Where the service has a tool catalogue, the x402 declaration and `x-assurance` declaration SHOULD appear together in the tool metadata.

A provider MAY publish a signed pricing document at:

```text
/.well-known/x402-assurance.json
```

This path is **provisional**. It is not registered in the IANA Well-Known URIs
registry (RFC 8615) and is subject to change, including as a result of
registration. A provider **SHOULD** also advertise the document's location
alongside its service metadata, so that discovery does not depend on the path.

The document SHOULD contain profile version, provider identifier, provider signing key reference, issuance and expiry timestamps, service declarations, x402 requirements, assurance requirements, and a signature or proof.

## 8. Identity and assurance

The profile does not mandate a single identity technology.

Evidence MAY include verifiable credentials, OpenID Connect claims, OAuth client identity, workload identity, decentralised identifiers, legal-entity identifiers, organisational certificates, and ecosystem membership credentials.

The provider **MUST** document how assurance levels are established.

### 8.1 Baseline levels

| Level | Meaning |
|---|---|
| `AL0` | Caller is not identified by the service. |
| `AL1` | Caller or operating organisation is identified with verifiable evidence. |
| `AL2` | Identity plus stronger authority, contractual, regulated or high-assurance evidence. |

These levels are interoperability labels, not a replacement for a regulated assurance framework.

### 8.2 Declaring an external framework

The three baseline levels exist so that two parties with no shared regulator can
still agree on a floor. They are deliberately coarse and are **not** a new
assurance scheme.

A provider operating under an established framework **SHOULD** declare it, so a
caller can reason in the vocabulary it already holds evidence in:

```json
{
  "x-assurance": {
    "minAssuranceLevel": "AL2",
    "assuranceFramework": "https://eur-lex.europa.eu/eli/reg/2014/910/oj",
    "frameworkLevel": "high"
  }
}
```

`minAssuranceLevel` remains the interoperable floor. `assuranceFramework` and
`frameworkLevel`, where present, are authoritative for callers that understand
that framework; `minAssuranceLevel` governs for callers that do not.

Non-normative correspondence, offered as a starting point rather than as an
equivalence:

| This profile | eIDAS assurance | NIST SP 800-63-3 | ISO/IEC 29115 |
|---|---|---|---|
| `AL0` | - | - | LoA 1 |
| `AL1` | low / substantial | IAL1-IAL2 | LoA 2-3 |
| `AL2` | high | IAL3 | LoA 4 |

A provider **MUST NOT** present a mapping as conformity with the mapped
framework. Mapping is a discovery convenience; conformity is determined by that
framework's own authority.

## 9. Mandates

Where `requiresMandate` is true, the provider **MUST** verify that the mandate identifies the principal; identifies or constrains the agent or operator; covers the requested service and action; satisfies applicable value, volume and time limits; and has not expired or been revoked.

A payment made by the caller does not cure an absent or invalid mandate.

## 10. Revocation and status

Status **MUST** be checked before payment is offered or settled.

The provider **MUST NOT** settle a payment from a caller it has already determined it will refuse.

If status is discovered to have changed after settlement, the provider SHOULD refund where possible and MUST record the outcome.

## 11. Replay protection

A settled payment authorisation **MUST NOT** be accepted twice.

Settlement references or equivalent replay identifiers **MUST** be persisted and checked before invoking the underlying service.

## 12. Audit

For every paid-service attempt, the provider **MUST** record timestamp, service identifier, caller or payer identifier, principal identifier where applicable, assurance level, mandate identifier or digest where applicable, status decision, declared price and asset, settled amount, settlement reference, policy outcome, and delivery outcome.

Audit records SHOULD be tamper-evident.

Audit retention and access MUST comply with applicable law and contractual obligations.

## 13. Privacy

Identity and payment data MUST be minimised.

A wallet or settlement account may constitute personal data when linkable to a natural person.

A provider SHOULD avoid revealing service existence, price or commercial terms to a caller that fails an identity or entitlement gate.

## 14. Error model

Identifiers are grouped by who may receive them (§6.1).

Returnable to **any** caller:

- `access_denied` - the generic refusal, carrying no indication of which gate failed
- `payment_required`
- `payment_verification_failed`
- `payment_settlement_failed`
- `payment_replayed`

Returnable **only** to a caller identified at `AL1` or above, or for a service
explicitly published as having no confidential existence, price or policy:

- `identity_required`
- `insufficient_assurance`
- `mandate_required`
- `invalid_mandate`
- `status_revoked`
- `membership_suspended`

The payment identifiers are unrestricted because a caller only reaches them
after passing every policy gate, so they disclose nothing it was not already
entitled to learn.

Error descriptions SHOULD be understandable but MUST NOT disclose sensitive policy or payment details.

## 15. Security considerations

Implementations must address forged identity and mandate evidence, stale status information, payment replay, confused-deputy attacks, identity-to-wallet misbinding, price substitution, assurance downgrade, audit-log tampering, facilitator compromise, and privacy leakage through differential errors.

## 16. Conformance

An implementation conforms only if it passes all mandatory tests in `conformance.md`.

Failure of the identity-before-payment or revoked-caller tests is a critical failure.
