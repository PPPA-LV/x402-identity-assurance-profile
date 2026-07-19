# Security Policy

## Reporting a vulnerability

Please do not disclose exploitable vulnerabilities through a public issue.

Send a report to the maintainers using the private security-reporting channel configured for this repository. Include:

- affected component and version;
- reproduction steps;
- expected and observed behaviour;
- impact;
- suggested remediation, when available.

## High-priority vulnerability classes

Reports involving the following are particularly important:

- settlement before identity or status evaluation;
- payment replay;
- bypass of assurance or mandate requirements;
- forged or misbound identity evidence;
- price substitution;
- disclosure of payment details in a `403` response;
- invocation of the underlying service before settlement;
- private-key or wallet-custody exposure;
- audit-record tampering.
