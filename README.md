# x402 Identity Assurance Profile

**Status:** Draft specification · v0.1

The x402 Identity Assurance Profile defines how a service combines machine-native payment with identity, assurance, authorisation and revocation checks.

> **A caller’s ability to pay does not establish its right to buy.**

The profile introduces an assurance gate before the x402 payment exchange. A service open to unidentified callers may return `402 Payment Required`. A service requiring an identified or authorised caller must first establish that requirement and return `403 Forbidden` when it is not met. No price should be disclosed to a caller that is ineligible to receive the service.

## Why this profile exists

x402 standardises how a caller pays for an HTTP resource or service. It deliberately does not determine:

- who operates the calling agent;
- which person or organisation the agent represents;
- whether the agent holds a valid mandate;
- the assurance level of the presented identity;
- whether a credential, delegation or membership has been revoked;
- whether the requested data may legally or contractually be supplied.

## Core processing rule

1. Determine whether the service is free or paid.
2. Establish the caller’s identity and assurance level.
3. Check credential, delegation and participant status.
4. Determine whether the caller is authorised to obtain the service.
5. Only then offer or verify payment.
6. Invoke the underlying service only after all checks succeed.
7. Create an audit record binding identity, policy decision, payment and delivery.

This gives rise to the defining behaviour:

- eligible anonymous caller, payment absent → `402`;
- caller lacks required identity or assurance → `403`, without price disclosure;
- caller is revoked or suspended → `403`, regardless of payment;
- identity and policy satisfied, payment settled → normal service response.

## Repository contents

- [`specification.md`](specification.md) — normative profile
- [`schemas/assurance-extension.schema.json`](schemas/assurance-extension.schema.json) — JSON Schema
- [`examples/tool-annotation.json`](examples/tool-annotation.json) — paid tool declaration
- [`examples/pricing-document.json`](examples/pricing-document.json) — signed pricing document
- [`conformance.md`](conformance.md) — conditional conformance suite
- [`SECURITY.md`](SECURITY.md) — security reporting guidance
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — how to contribute, and the DCO sign-off requirement
- [`MAINTAINERS.md`](MAINTAINERS.md) — who reviews changes

## Relationship to x402

This profile does not modify x402. The x402 payment requirement remains the settlement source of truth. The `x-assurance` extension declares identity, assurance, mandate and status conditions that must be satisfied before payment is offered or accepted.

## Design principles

- Payment is consideration, not authorisation.
- Identity and status are evaluated before price.
- A service must not settle a payment it already knows it cannot honour.
- Facilitators need not take custody of funds or private keys.
- Settlement rails remain pluggable.
- Prices are discoverable before invocation.
- Every paid or refused call is auditable.
- A refusal tells an identified caller why, and tells an anonymous caller nothing.

## Implementation status

This is an experimental interoperability profile. Implementations should not claim conformance until they pass the conformance suite.

**No implementation currently conforms.** The AI LV Exchange Node Starter Kit
implements the same identity-before-payment rule and is the intended first
reference implementation, but it does not yet emit the `x-assurance` extension
or publish `/.well-known/x402-assurance.json`, so it does not pass this suite
today. That work is tracked separately and this statement will change only when
a conformance report exists.

## Governance

The profile is maintained as an open initiative of the Public-Private Partnership
Association of Latvia (PPPA). Cyberfort SIA is the legal entity contributing the
work. Substantive changes should be proposed through GitHub issues and pull
requests; see [`CONTRIBUTING.md`](CONTRIBUTING.md).

Every commit must carry a [Developer Certificate of Origin](DCO) sign-off
(`git commit -s`). There is no Contributor Licence Agreement.

## Licence

- Specification text and documentation: **CC BY 4.0** ([`LICENSES/CC-BY-4.0.txt`](LICENSES/CC-BY-4.0.txt))
- Schemas, examples and code: **Apache License 2.0** ([`LICENSE`](LICENSE))

Apache-2.0 is used for code because it grants patent rights expressly, and
because it matches the licensing of the x402 project this profile extends.
