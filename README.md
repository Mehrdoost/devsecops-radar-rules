# 🧩 Pipeline Sentinel Community Rules

**The open-source rule marketplace for [Pipeline Sentinel](https://github.com/Mehrdoost/devsecops-radar).**

[![Stars](https://img.shields.io/github/stars/Mehrdoost/devsecops-radar-rules?style=flat-square)](https://github.com/Mehrdoost/devsecops-radar-rules/stargazers)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)
[![Contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat-square)](CONTRIBUTING.md)

---

## 📖 What is this?
This repository hosts community-curated security rules for **Pipeline Sentinel**, the DevSecOps command centre for organisational networks.

Think of it as **Nuclei Templates for CI/CD security**: anyone can contribute a rule, and everyone benefits. Pipeline Sentinel can download and use these rules directly from this repository.

---

## 🚀 For rule users

### Prerequisites
Before the first `--update-rules`, you must:

1. **Set `COMMUNITY_RULES_REPO`** in your `.env`:
   ```bash
   COMMUNITY_RULES_REPO=[https://github.com/Mehrdoost/devsecops-radar-rules.git](https://github.com/Mehrdoost/devsecops-radar-rules.git)
   ```
2. **Set `TRUSTED_GPG_FINGERPRINTS`** in your `.env`. The maintainer's fingerprint is published in `SECURITY.md`. Without it, `--update-rules` refuses to install anything — this is deliberate, since rules are code that runs inside Pipeline Sentinel.
   ```bash
   TRUSTED_GPG_FINGERPRINTS=<40-or-64-hex-characters>
   ```
3. Ensure `git` is available on PATH and in a trusted directory (see `EXTRA_TRUSTED_BIN_DIRS` in the main project).

If any of these is missing, `--update-rules` exits with a clear error. Do not skip this step.

### Update your local rules
```bash
devsecops-radar --update-rules
```
This clones (or pulls) the latest signed release tag from this repository into `~/.devsecops-radar/community-rules/`, verifies its GPG signature against `TRUSTED_GPG_FINGERPRINTS`, and refuses to install anything that does not carry a valid signature.

Only release tags of the form `vX.Y.Z` are considered. A commit pushed to `main` is not visible to clients until the maintainer creates a signed release tag.

### Run a scan with community rules
```bash
devsecops-radar \
  --trivy trivy.json \
  --semgrep semgrep.json \
  --rules ~/.devsecops-radar/community-rules/
```
The dashboard will include findings from the community rules alongside your scanner results.

### Optional: use your own fork
If you maintain a private rule set, point `COMMUNITY_RULES_REPO` at your fork:
```bash
export COMMUNITY_RULES_REPO=[https://github.com/your-org/devsecops-radar-rules.git](https://github.com/your-org/devsecops-radar-rules.git)
devsecops-radar --update-rules
```
Your fork must follow the same release process as this repository: signed `vX.Y.Z` tags, verifiable against the fingerprints in your own `TRUSTED_GPG_FINGERPRINTS`.

## ✍️ Contributing a rule

We welcome all security professionals who want to share their detection logic. Here is how to add a new rule.

**1. Fork this repository**
Click the Fork button at the top right of this page.

**2. Create a JSON file under `rules/`**
The file name should be descriptive and end in `.json` (for example, `cis-kubernetes.json`).

> **Size limits:** Each file is limited to 10 MiB, the total repository content is limited to 50 MiB, and Pipeline Sentinel will not load more than 1000 files in a single run. A rule file that exceeds any of these limits is rejected by the client.

**3. Write the rule in the standard format**
Each file contains a JSON array of findings. Every finding must have the following fields:

| Field | Required | Description | Example |
| --- | --- | --- | --- |
| `tool` | ✅ | Name of your scanner or rule source | "CIS Kubernetes Benchmark" |
| `target` | ✅ | File, image, or asset where the issue was found | "k8s/deployment.yaml" |
| `id` | ✅ | Unique rule identifier | "CIS-K8S-001" |
| `severity` | ✅ | One of CRITICAL, HIGH, MEDIUM, LOW, UNKNOWN | "HIGH" |
| `title` | ✅ | Short description of the finding | "Privileged container detected" |
| `description` | ✅ | Detailed explanation and remediation advice | "Containers should not run in privileged mode..." |
| `line` | ⬜ | Line number where the issue occurs (optional) | 15 |

Example (`rules/cis-kubernetes.json`):
```json
[
  {
    "tool": "CIS Kubernetes Benchmark",
    "target": "k8s/deployment.yaml",
    "id": "CIS-K8S-001",
    "severity": "HIGH",
    "title": "Privileged container detected",
    "description": "Containers should not run in privileged mode. Set `securityContext.privileged` to `false` in your pod specifications.",
    "line": 15
  }
]
```

**4. Validate your JSON before submitting**
The main repository ships a validator that enforces the same schema the client uses:
```bash
python scripts/validate_rules.py rules/
```
If you do not have the main repository checked out, at minimum run:
```bash
python -m json.tool rules/my-rule.json
```

**5. Submit a pull request**
Commit your changes, push to your fork, and open a pull request against the main branch of this repository. The CI workflow `validate-pr.yml` in the main repository runs the same validator, so a rule that passes locally passes CI.

Maintainers review every PR. Once merged, the rule becomes available with the next signed release tag — see the release section below.

## 🧪 Testing a rule locally

You can test a rule before opening a PR.
1. Create your rule file anywhere on your machine.
2. Point Pipeline Sentinel at the directory that contains it:
   ```bash
   devsecops-radar --trivy sample_trivy.json --rules /path/to/your/rule/dir/
   ```
3. Open the dashboard and verify that your finding appears in the table and in the charts.

The `--rules` flag accepts any directory that contains `*.json` rule files. Rules are loaded in addition to scanner output.

## 📦 Release process (for maintainers)

Releases are what clients actually see. Every release is a signed tag.

```bash
# 1. Make sure your signing key is available.
gpg --list-secret-keys

# 2. Create an annotated, signed tag.
git tag -s v1.0.0 -m "Community rules release v1.0.0"

# 3. Push the tag.
git push origin v1.0.0
```

The `--update-rules` command resolves the highest `vX.Y.Z` tag advertised by the remote and verifies its signature. A branch tip, a lightweight tag, or an unsigned tag is never accepted.
Rules merged to `main` are not visible to clients until the next signed release.

## 📂 Repository structure

```text
devsecops-radar-rules/
├── README.md          # You are here
├── SECURITY.md        # GPG key, threat model, disclosure policy
├── LICENSE            # MIT
├── CONTRIBUTING.md    # Contribution guidelines
└── rules/             # All rule files
    ├── cis-kubernetes.json
    └── ... (your rules)
```

## 🔐 Security

Community rules execute inside Pipeline Sentinel's rule-fusion engine. They are code, not just data: a malicious rule can silently change which findings are reported, what severity they carry, and how the dashboard behaves. For that reason:

* Every release tag is GPG-signed by the maintainer.
* The client verifies the signature against a fingerprint allow-list before installing anything.
* A repository with an invalid signature is moved to `~/.devsecops-radar/rejected/` and not executed.

See `SECURITY.md` for the maintainer's GPG fingerprint, the disclosure process, and the threat model.
Do not open a public issue for a security vulnerability. Use the process described in `SECURITY.md`.

## 🔗 Useful links

* Pipeline Sentinel — main repository
* Report a bug or request a feature
* ReverseForge on GitHub

## 📜 License

The rule files in this repository are released under the MIT License — see `LICENSE`. This is separate from the license of Pipeline Sentinel itself, which is AGPL-3.0-only. The MIT license applies only to the contents of this repository.

⭐ If you find a rule useful, drop a star on the main Pipeline Sentinel project — it helps the community grow.
