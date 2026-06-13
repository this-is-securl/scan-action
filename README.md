# SecURL Security Posture Scan · GitHub Action

Passive external security posture analysis for public web targets — directly in your CI pipeline.

[![SecURL](https://img.shields.io/badge/powered%20by-SecURL-blue)](https://securl.online)

## What it does

- Scans any public URL for security headers, TLS, DNS/email trust, third-party surface, and passive intelligence signals
- Produces a posture **grade (A–F)** and **score (0–100)**
- Uploads **SARIF findings to GitHub Code Scanning** (shows in the Security tab)
- Posts a **PR comment** with the posture table on every pull request
- Writes a **job summary** with results visible directly in the Actions UI
- Supports **CI policy gates**: fail the job on findings above a severity, score drops, or regressions against a saved baseline

## Quick start

```yaml
- uses: securl/scan-action@v1
  with:
    url: https://example.com
```

## Common patterns

### Gate on critical findings

```yaml
- uses: securl/scan-action@v1
  with:
    url: https://your-app.com
    fail-on: critical
```

### Gate on score

```yaml
- uses: securl/scan-action@v1
  with:
    url: https://your-app.com
    fail-if-score-below: 75
```

### Detect regressions against a saved baseline

```yaml
- name: Save current posture as baseline
  uses: securl/scan-action@v1
  with:
    url: https://your-app.com
    format: json

- name: Check for regressions on next run
  uses: securl/scan-action@v1
  with:
    url: https://your-app.com
    baseline: ./security-baseline.json
    fail-on-regression: true
```

### Scan multiple targets

```yaml
- uses: securl/scan-action@v1
  with:
    targets: https://app.example.com https://api.example.com https://docs.example.com
    fail-on: warning
```

### Full pre-release gate

```yaml
name: Security Posture Gate

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  posture:
    runs-on: ubuntu-latest
    permissions:
      security-events: write   # for SARIF upload
      pull-requests: write     # for PR comment
      contents: read

    steps:
      - uses: actions/checkout@v4

      - name: SecURL posture scan
        uses: securl/scan-action@v1
        with:
          url: https://your-public-site.com
          scan-mode: quiet
          fail-on: critical
          fail-if-score-below: 70
          upload-sarif: true
          post-pr-comment: true
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `url` | Target URL to scan | — |
| `targets` | Space-separated list of URLs (multi-target) | — |
| `scan-mode` | `default` \| `quiet` \| `deep-passive` | `quiet` |
| `fail-on` | Fail on findings at this severity: `critical` \| `warning` \| `info` | — |
| `fail-if-score-below` | Fail when score drops below this value (0–100) | — |
| `fail-on-regression` | Fail when regression detected vs baseline | `false` |
| `baseline` | Path to a previously saved JSON report | — |
| `upload-sarif` | Upload SARIF to GitHub Code Scanning | `true` |
| `post-pr-comment` | Post posture summary on pull requests | `true` |
| `format` | Step summary format: `summary` \| `markdown` \| `json` | `summary` |
| `node-version` | Node.js version (must be ≥22) | `22` |

## Outputs

| Output | Description |
|--------|-------------|
| `score` | Posture score (0–100) |
| `grade` | Posture grade (A–F) |
| `critical-count` | Number of critical findings |
| `warning-count` | Number of warning findings |
| `passed` | `true` if all policy gates passed |
| `report-path` | Path to the full JSON report |
| `sarif-path` | Path to the generated SARIF file |

## Required permissions

```yaml
permissions:
  security-events: write   # SARIF upload to Code Scanning
  pull-requests: write     # PR comment posting
  contents: read
```

If you don't need SARIF upload or PR comments, set `upload-sarif: false` and `post-pr-comment: false` — no additional permissions required beyond `contents: read`.

## Scan modes

| Mode | When to use |
|------|-------------|
| `quiet` (default) | High-frequency CI gates. Checks headers, TLS, redirects, DNS/mail, CT summary, and public trust. Skips page-body analysis and active probe checks. |
| `default` | Pre-release or scheduled scans. Full passive enrichment including HTML analysis, exposure probes, and third-party surface mapping. |
| `deep-passive` | Scheduled posture reviews. Expanded CT host sampling, broader passive recon, and deeper API surface probing. |

## What SecURL checks

- HTTP security headers (HSTS, CSP, X-Frame-Options, Referrer-Policy, Permissions-Policy, COOP, CORP)
- TLS certificate validity, expiry, and protocol/cipher posture
- Cookie security flags (Secure, HttpOnly, SameSite)
- DNS and email trust chain (SPF, DKIM, DMARC, DNSSEC, MTA-STS, TLS-RPT, BIMI)
- Third-party script and asset surface
- Passive HTML signals (inline scripts, missing SRI, form hygiene)
- Public trust and disclosure signals (security.txt, HSTS preload)
- AI surface, analytics, and session replay vendor detection (30+ vendors)
- Infrastructure and CDN fingerprinting

All checks are **passive-only** — no active exploitation, fuzzing, or authentication probing.

## Safety

Use this action only against targets you own or are authorised to assess. SecURL is passive-first and production-conscious, but it does make DNS queries, TLS handshakes, and low-noise HTTP checks.

## Powered by

[SecURL](https://securl.online) — passive external security posture analysis.  
[`@ktbatterham/external-posture-core`](https://www.npmjs.com/package/@ktbatterham/external-posture-core) — the open-source engine.

---

*Found a bug or want a feature? Open an issue on the [SecURL repository](https://github.com/ktbatterham/external-posture-insight).*
