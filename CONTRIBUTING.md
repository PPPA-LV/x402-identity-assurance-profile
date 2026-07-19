# Contributing

This profile is developed in the open. Issues and pull requests are welcome from
anyone, including implementers who disagree with the current design: a profile
that only its authors can implement is not a profile.

## Developer Certificate of Origin

Every commit **must** be signed off under the
[Developer Certificate of Origin](DCO) (DCO 1.1). Signing off certifies that you
wrote the contribution, or otherwise have the right to submit it under the
repository's licences.

Add the trailer with `git commit -s`, which produces:

```
Signed-off-by: Your Name <your.email@example.com>
```

Use your real name and an address you can be reached at. Pull requests whose
commits are not signed off cannot be merged. To fix an existing branch:

```
git rebase --signoff main
```

There is no separate Contributor Licence Agreement.

## Licences

By contributing you agree that your contribution is licensed under:

- **Apache License 2.0** — schemas, examples, code (`LICENSE`)
- **CC BY 4.0** — specification and documentation text (`LICENSE-SPECIFICATION`)

Apache-2.0 is used for code because it grants patent rights expressly, and
because it matches the licensing of the x402 project this profile extends.

## What makes a good change

The profile exists to state one thing precisely: **a caller's ability to pay
does not establish its right to buy.** Changes are easiest to accept when they:

- close a gap an implementer actually hit;
- remove an assumption that only one ecosystem can satisfy;
- strengthen a normative statement without narrowing who can conform;
- come with a conformance test, or a reason the existing tests are sufficient.

Changes that add vocabulary specific to one deployment, one jurisdiction or one
credential format belong in a downstream profile, not here.

## Normative changes

A change to a **MUST**, **MUST NOT** or **SHOULD**, to the processing order, to
the response precedence table, or to the conformance suite is normative. Open an
issue describing the problem before opening a pull request, and say which
conformance tests change as a result.

## Security

Do not report vulnerabilities through public issues. See [SECURITY.md](SECURITY.md).
