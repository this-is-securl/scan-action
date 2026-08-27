# SecURL Website Posture Scan

Add passive external website posture evidence to a GitHub Actions workflow. SecURL checks
public responses, redirects, TLS, headers, cookies, DNS and other bounded public signals. It
does not require a SecURL account, repository access, target credentials or an API token.

## Start with visible evidence

```yaml
name: External posture evidence

on:
  workflow_dispatch:
  release:
    types: [published]

permissions:
  contents: read

jobs:
  securl:
    runs-on: ubuntu-latest
    steps:
      - uses: this-is-securl/scan-action@v2
        with:
          url: https://example.com
```

The action writes a human-readable job summary and a full JSON report. The summary includes
an optional continuation into the interactive SecURL scanner, prefilled with the same target.
Starting the hosted scan or saving a watch remains an explicit user action.

## Add a policy gate when the evidence is stable

```yaml
      - uses: this-is-securl/scan-action@v2
        with:
          url: https://example.com
          fail-if-score-below: "75"
          fail-on: critical
```

Do not choose a threshold simply to make a build green. Start non-blocking, observe the
target across releases, then select a policy the team is prepared to maintain.

## Scan several public targets

```yaml
      - uses: this-is-securl/scan-action@v2
        with:
          targets: https://app.example.com https://api.example.com https://docs.example.com
          scan-mode: quiet
```

## Optional GitHub integrations

The default action needs only `contents: read`. PR comments and Code Scanning uploads are
off by default because they require write permissions.

```yaml
permissions:
  contents: read
  pull-requests: write
  security-events: write

steps:
  - uses: this-is-securl/scan-action@v2
    with:
      url: https://example.com
      post-pr-comment: "true"
      upload-sarif: "true"
```

## Inputs

| Input | Description | Default |
| --- | --- | --- |
| `url` | One public URL or hostname | none |
| `targets` | Space-separated public URLs or hostnames | none |
| `scan-mode` | `quiet`, `standard`, or `deep-passive` | `quiet` |
| `fail-on` | `critical`, `warning`, or `info` | disabled |
| `fail-if-score-below` | Minimum score from 0 to 100 | disabled |
| `baseline` | Path to a previous SecURL JSON report | none |
| `fail-on-regression` | Fail on regression against `baseline` | `false` |
| `upload-sarif` | Upload findings to Code Scanning | `false` |
| `post-pr-comment` | Create or update a PR summary | `false` |
| `node-version` | Node.js version, 22 or newer | `22` |

## Outputs

| Output | Description |
| --- | --- |
| `score` | Score for the first target |
| `grade` | Grade for the first target |
| `critical-count` | Critical findings across every target |
| `warning-count` | Warning findings across every target |
| `passed` | Whether the assessment and configured policies passed |
| `report-path` | Full JSON report path |
| `sarif-path` | Generated SARIF document path |

## Privacy and safety

- The npm package runs in the workflow runner and contacts only the public target and the
  bounded public data sources used by the selected scan mode.
- Repository names, commits, actors, source code and workflow secrets are not sent to the
  hosted SecURL service.
- The hosted scanner is contacted only if a person follows the continuation link and starts
  a separate scan.
- Use the action only for public targets you own or are authorised to assess.
- SecURL is passive-first posture evidence, not a pentest or a promise that a site is safe.

The action pins `securl@1.28.6` and its third-party GitHub Actions dependencies. Version
updates are reviewed and released deliberately instead of floating through `latest` tags.

## More

- [SecURL website](https://securl.online)
- [Interactive scanner](https://app.securl.online)
- [SecURL engine and CLI](https://github.com/this-is-securl/securl)
- [npm package](https://www.npmjs.com/package/securl)
