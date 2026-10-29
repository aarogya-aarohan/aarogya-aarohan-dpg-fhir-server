# Contributing

Thank you for helping improve Aarogya Aarohan. Please read this guide before opening a pull request.

## Ways to contribute

- **Report a bug:** open a GitHub issue with what you ran, what you expected and what happened. Remove hostnames, tokens and any patient data.
- **Propose a change:** for anything larger than a small fix, open an issue first so the approach can be agreed before you write code.
- **Security issues:** do **not** open a public issue. Follow [SECURITY.md](SECURITY.md).

## Development setup

1. Fork the repository and clone your fork.
2. Install JDK 17, Maven 3.9 and Docker.
3. Start a local server with `docker compose up --build` (see the [README](README.md#quick-start-local)).

## Branches and commits

- Branch from `main`: `feat/<short-name>`, `fix/<short-name>`, `docs/<short-name>`.
- Write commit messages in the imperative, for example `binary: make S3 storage partition-aware`.
- Keep one logical change per pull request.

## Before you open a pull request

```bash
mvn -B verify                   # build and tests
scripts/test-multi-tenant.sh    # if you touched auth, partitions or storage
```

Changes to authentication, authorisation, partitioning or image storage need a test that shows one tenant cannot read or write another tenant's data.

## Pull request checklist

- [ ] The change is described in plain language: what changes and why
- [ ] Tests pass and new behaviour is tested
- [ ] New settings are documented in `docs/configuration.md` with safe defaults
- [ ] No secrets, tokens, internal hostnames or patient data are included

## Review and merge

At least one maintainer approval is required. Squash merge is preferred.

## Code of conduct

Participation is governed by the [Code of Conduct](CODE_OF_CONDUCT.md).

## License

By contributing, you agree that your contributions are licensed under the [Apache License 2.0](LICENSE).
