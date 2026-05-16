<!-- markdownlint-disable MD033 MD041 -->
<div align="center">

# 🧩 Pipeline Sentinel Community Rules
**The open‑source rule marketplace for Pipeline Sentinel.**

[![GitHub stars](https://img.shields.io/github/stars/Mehrdoost/devsecops-radar-rules?style=flat-square)](https://github.com/Mehrdoost/devsecops-radar-rules/stargazers)
[![License](https://img.shields.io/github/license/Mehrdoost/devsecops-radar-rules?style=flat-square)](LICENSE)
[![Contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen?style=flat-square)](CONTRIBUTING.md)

</div>

---

## 📖 What Is This?

This repository hosts **community‑curated security rules** for [Pipeline Sentinel](https://github.com/Mehrdoost/devsecops-radar). Think of it as the **Nuclei Templates** for CI/CD security — anyone can contribute a rule, and everyone benefits.

Pipeline Sentinel can automatically download and use these rules with a single command:

```bash
devsecops-radar --update-rules
```

The rules are then stored in `~/.devsecops-radar/community-rules/` and can be loaded alongside your scanner results:

```bash
devsecops-radar --trivy scan.json --rules ~/.devsecops-radar/community-rules/
```

---

## 🚀 Quick Start for Rule Users

### 1. Update your local rules
```bash
devsecops-radar --update-rules
```
*This clones (or pulls) the latest rules from this repository into your machine.*

### 2. Run a scan with community rules
```bash
devsecops-radar --trivy trivy.json --semgrep semgrep.json --rules ~/.devsecops-radar/community-rules/
```
*The dashboard will now include findings from the community rules alongside your scanner results.*

### 3. (Optional) Use your own fork
If you maintain a private rule set, set the environment variable before pulling:
```bash
export COMMUNITY_RULES_REPO=[https://github.com/your-org/devsecops-radar-rules.git](https://github.com/your-org/devsecops-radar-rules.git)
devsecops-radar --update-rules
```

---

## ✍️ Contributing a Rule

We welcome all security professionals to share their custom detection rules. Here’s how to add a new rule:

### 1. Fork this repository
Click the **Fork** button at the top right of this page.

### 2. Create a new JSON file in the `rules/` directory
The file name should be descriptive (e.g., `cis-kubernetes.json`).

### 3. Write your rule in the standard Pipeline Sentinel format
Each rule file contains an array of findings. Every finding must have these fields:

| Field | Description | Example |
| :--- | :--- | :--- |
| `tool` | Name of your scanner or rule source | `"My Custom Scanner"` |
| `target` | File, image, or asset where the issue was found | `"deployment.yaml"` |
| `id` | Unique rule identifier | `"CIS-K8S-001"` |
| `severity` | One of `CRITICAL`, `HIGH`, `MEDIUM`, `LOW` | `"HIGH"` |
| `title` | Short description of the finding | `"Privileged container detected"` |
| `description` | Detailed explanation and remediation advice | `"Containers should not run in privileged mode..."` |
| `line` *(optional)* | Line number where the issue occurs | `42` |

**Example rule (`rules/cis-kubernetes.json`):**
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

### 4. (Optional) Validate your JSON
Before submitting, make sure your file is valid JSON. You can use:
```bash
python -m json.tool rules/my-rule.json
```

### 5. Submit a Pull Request
Commit your changes, push to your fork, and open a Pull Request against the `main` branch of this repository. Our maintainers will review your rule and merge it if it meets the quality standards.

---

## 🧪 Testing Your Rule Locally

You can test your rule locally before even pushing to the community repo:

1. Create your rule file (e.g., `my-rule.json`) anywhere on your machine.
2. Run Pipeline Sentinel pointing to the directory containing your file:
   ```bash
   devsecops-radar --trivy sample_trivy.json --rules /path/to/your/rule/dir/
   
```
3. Open the dashboard and verify that your finding appears in the table and charts.

---

## 📂 Repository Structure

```text
devsecops-radar-rules/
├── README.md          # You are here
├── LICENSE            # MIT
├── CONTRIBUTING.md    # Contribution guidelines
└── rules/             # All rule files go here
    ├── example.json
    ├── cis-kubernetes.json
    └── ... (your rules)
```

---

## 🔗 Useful Links

* [Pipeline Sentinel Main Repository](https://github.com/Mehrdoost/devsecops-radar)
* [Report a Bug or Request a Feature](https://github.com/Mehrdoost/devsecops-radar-rules/issues)

---

## 📜 License

MIT — see [LICENSE](LICENSE).

<br>
<div align="center">
⭐ <strong>If you find a rule useful, drop a star on the main Pipeline Sentinel project — it helps the community grow!</strong>
</div>
