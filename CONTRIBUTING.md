# Contributing to Pipeline Sentinel Community Rules

Thank you for considering a contribution. This repository hosts community-curated security rules for [Pipeline Sentinel](https://github.com/Mehrdoost/devsecops-radar).

## What we accept

- **Detection rules** for CI/CD, containers, Kubernetes, IaC, or cloud configurations.
- **Corrections** to existing rules (typos, outdated remediation advice, new CVE references).
- **Documentation** improvements.

We do **not** accept:

- Rules that are hidden advertisements for a commercial product.
- Rules that contain secrets, tokens, or credentials of any kind.
- Rules whose `description` field is empty or contains a single character.

## How to submit

See the ["Contributing a rule"](README.md#✍️-contributing-a-rule) section of the README for the full step-by-step process.

In short:

1. Fork this repository.
2. Add a JSON file under `rules/`.
3. Validate it: `python scripts/validate_rules.py rules/` (from the main repository).
4. Open a pull request against `main`.

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](https://github.com/Mehrdoost/devsecops-radar/blob/main/CODE_OF_CONDUCT.md). By participating you agree to uphold it.

## License

By contributing a rule, you agree that your contribution will be released under the MIT License (see [LICENSE](LICENSE)).

## Release process

Merged rules are published to clients through **signed release tags**. The maintainer creates a tag of the form `vX.Y.Z`, signs it with the project's GPG key, and pushes it. Clients running `devsecops-radar --update-rules` will pick it up on their next run.

If you need your rule to be available immediately, please say so in the pull request and the maintainer will prioritise a release.
