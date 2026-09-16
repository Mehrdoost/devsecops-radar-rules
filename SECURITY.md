# Security Policy — Pipeline Sentinel Community Rules

This document describes the security model of the community rules repository maintained by ReverseForge: how releases are signed, how clients verify them, and how to report a vulnerability.

---

## Table of contents
1. [Trust model](#trust-model)
2. [Release signing](#release-signing)
3. [Trusted fingerprints](#trusted-fingerprints)
4. [Client-side verification](#client-side-verification)
5. [Rotating a maintainer key](#rotating-a-maintainer-key)
6. [Revoking a compromised key](#revoking-a-compromised-key)
7. [Verifying a release manually](#verifying-a-release-manually)
8. [Reporting a vulnerability](#reporting-a-vulnerability)
9. [Scope](#scope)

---

## Trust model
Community rules are **data**, not code. They cannot execute anything on the client. They are consumed by Pipeline Sentinel, which merges them with scanner output and displays the result in the dashboard.

Because rules are data, the primary risk is **supply-chain tampering**. A compromised repository could inject a rule whose `id` collides with a real finding, whose `description` misleads an operator, or whose `severity` causes the merge step to hide a genuine issue.

This repository mitigates that risk with **GPG-signed release tags** and a **narrow, explicit trust anchor** on the client side.

---

## Release signing
1. Community pull requests are reviewed and merged into `main`.
2. When the maintainers are ready to publish, they create a Git tag of the form `vMAJOR.MINOR.PATCH` (for example `v1.4.0`). The client's release resolver only recognises tags matching this pattern; a tag in any other format is ignored when resolving the "latest" release.
3. The tag is signed with the maintainer's GPG key.
4. Pushing the tag triggers the project's GitHub Actions workflow, which verifies the signature and publishes a GitHub Release.

Clients fetch the **tag**, never the tip of `main`. A compromised `main` branch — for example, through a stolen contributor account — therefore cannot reach a client until it has been reviewed, merged, and re-signed by a maintainer.

---

## Trusted fingerprints
The maintainer's GPG fingerprints are published **below and on the top of every GitHub release page**. Both locations must agree; if they do not, treat the release as suspect and report it privately.

```text
Primary fingerprint (current signing key):
REPLACE-WITH-MAINTAINER-FINGERPRINT

Secondary fingerprint (previous key, during rotation grace period):
(empty unless a rotation is in progress)
```

A fingerprint is formatted as 40 hexadecimal characters (SHA-1, the standard for GPG v4 keys). Spaces are optional; clients normalise whitespace before comparison.

## Client-side verification
Clients set the trusted fingerprint in their `.env`:
```bash
TRUSTED_GPG_FINGERPRINTS=REPLACE-WITH-MAINTAINER-FINGERPRINT
```

When the client runs `--update-rules`:
* It resolves the highest `vX.Y.Z` tag advertised by the repository (the release workflow pushes only semver tags; any other tag is ignored).
* It clones that tag into `~/.devsecops-radar/community-rules/` (or the value of `SENTINEL_DATA_DIR`, if set).
* It runs `git verify-tag --raw <tag>` inside the clone.
* It requires a `VALIDSIG` line whose fingerprint is present in the allow-list.
* It refuses to proceed if the signature is missing, if the fingerprint does not match, or if the GPG status output contains any of `BADSIG`, `ERRSIG`, `EXPSIG`, `EXPKEYSIG`, `REVKEYSIG`, `NO_PUBKEY`, `KEYEXPIRED`, `SIGEXPIRED`, `NO_SECKEY`, or `NODATA`.

When verification fails, the rejected repository is moved to `~/.devsecops-radar/rejected/` for forensic inspection. It is never silently deleted.

If `TRUSTED_GPG_FINGERPRINTS` is unset, `--update-rules` refuses to install anything and logs a clear error. There is no hard-coded fallback fingerprint; the operator must opt in explicitly. This is deliberate: a fallback fingerprint would let a compromised release reach a client whose operator never configured the trust anchor.

## Rotating a maintainer key
To rotate the signing key without breaking existing clients:
1. Generate a new GPG key.
2. Sign the new key with the old one (cross-signing). This proves continuity to any existing client that already trusts the old key.
3. Publish both fingerprints in this document — the new one under "Primary", the old one under "Secondary" — and in the release workflow.
4. Continue signing releases with the new key. Existing clients that have only the old fingerprint in their `.env` will still accept the new releases only if the cross-signature is verifiable through the old key. Operators are encouraged to add the new fingerprint at their earliest convenience.
5. After a grace period of at least 90 days, remove the old key from this document and from the release workflow, and delete the "Secondary" section.

Clients that do not update their `.env` within the grace period will continue to trust the old key as long as the cross-signature remains valid. This is intentional: key rotation must not break existing installations that cannot be updated promptly.

## Revoking a compromised key
If a maintainer signing key is compromised:
1. Immediately revoke the key on `keys.openpgp.org` and any other keyserver the project publishes to.
2. Publish a security advisory in this repository's Security tab. Include the fingerprint, the affected release tags, and the recommended operator action.
3. Cross-sign a new key with the previous non-compromised key if one exists. This limits the damage to releases published under the compromised key.
4. Update this document, the "Primary" and "Secondary" sections, and the release workflow.
5. Do not force-push or delete the compromised release tags. Preserve them for forensic analysis.

Existing clients that have the compromised fingerprint in their `.env` will continue to trust signatures made with it. The advisory must therefore instruct operators to remove that fingerprint manually. The client does not and cannot revoke trust on its own.

## Verifying a release manually
To verify a release outside of Pipeline Sentinel:

```bash
# 1. Import the maintainer's public key.
gpg --keyserver keys.openpgp.org --recv-keys REPLACE-WITH-MAINTAINER-FINGERPRINT

# 2. Clone the repository and check out the tag.
git clone [https://github.com/ReverseForge/devsecops-radar-rules.git](https://github.com/ReverseForge/devsecops-radar-rules.git)
cd devsecops-radar-rules
git fetch --tags
git checkout v1.4.0   # replace with the release you are verifying

# 3. Verify the tag signature.
git verify-tag v1.4.0
```

The output must contain `Good signature from ...` and the fingerprint reported by `git verify-tag` must match the value published in this document. A `Good signature` line alone is not sufficient: the signing key must also be the one whose fingerprint is in the trusted set.

## Reporting a vulnerability
If you find a security issue in this repository — a malicious rule, a signing problem, a bug in the release workflow, or anything else — please report it privately.

**Preferred channel:** GitHub Security Advisories
[Report a Vulnerability]

**Alternative: email**
If you prefer email, or if GitHub is not available to you, write to `security@reverseforge.io`. If you want the message encrypted, encrypt it to the GPG key whose fingerprint is published in this document.

Do not open a public issue for a security report.

You will receive an initial response within 72 hours. Please include:
* A description of the issue and its potential impact.
* Steps to reproduce, if applicable.
* The affected rule file(s) and release tag(s), if known.
* Any suggested mitigation or fix.
* Whether you intend to publish the finding, and if so, on what timeline (this helps us coordinate disclosure).

We follow a 90-day coordinated disclosure policy. We will credit reporters in the advisory unless they ask to remain anonymous.

## Scope
This policy covers the community rules repository only, i.e. the contents of `ReverseForge/devsecops-radar-rules` and the release workflow that signs it.

Vulnerabilities in the main Pipeline Sentinel project — the CLI, the dashboard, the scanners, the analyzers, and the Docker and Kubernetes artifacts — should be reported through the corresponding `SECURITY.md` in the main repository:
[Pipeline Sentinel SECURITY.md]

Vulnerabilities in third-party dependencies (Trivy, Semgrep, Gitleaks, Zizmor, Poutine, Ollama, and so on) should be reported to the upstream maintainers of those tools, not to ReverseForge. We are happy to help coordinate a disclosure if the issue affects Pipeline Sentinel's use of the dependency, but we do not maintain the dependency itself.
