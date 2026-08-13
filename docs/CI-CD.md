# CI/CD guide

Catch broken ad creatives in your pipeline - before they reach any ad platform. ad-preflight validates against IAB standards and platform requirements (Google Ads, DV360, Sizmek, Adform, and more), outputs machine-readable JSON, and uses standard exit codes.

**See also:** [CLI guide](CLI.md) for full command reference · [Deep Audit (Pro)](DEEP.md) for `--deep` in pipelines · [MCP setup](MCP.md) for Cursor and AI agents

---

## Install in CI

**As a dev dependency (recommended - locked version):**

```bash
npm install --save-dev @ad-preflight/cli
```

**Or run with npx (no install needed):**

```bash
npx -y @ad-preflight/cli package ./ad --json
```

---

## Machine mode

In machine mode, ad-preflight writes **exactly one JSON object** to stdout - no banners, no progress bars, no tips. Use the exit code for pass/fail.

Machine mode activates automatically when:

- You pass `--json` or `--json=pretty`
- stdout is not a TTY (pipes, CI runners, headless environments)

---

## Exit codes

| Code | Meaning | Action |
|------|---------|--------|
| **0** | Validation passed, ZIP created | Continue pipeline |
| **1** | Validation failed, Deep Audit failed, or runtime error | Fail the build |
| **2** | Invalid arguments or setup problem (e.g. `--deep` without the plugin installed, unsupported Node major) | Fix the command / finish setup |
| **3** | Licensing: system not supported, trial already used, or license no longer active | Needs a human - see the message |

Code **3** comes from the setup commands (`plugin-install`, `trial`). `package` itself only
ever exits 0, 1 or 2 - a licence that refuses the Deep Audit fails the run as exit **1**.

> **Multi-size campaigns:** a size that fails validation is skipped, but the run still exits
> **0** if the other sizes packaged. Check `packagesSkipped` in the JSON (and fail the build
> yourself if it isn't empty) so a dropped size can't slip through unnoticed.

---

## Non-interactive shells

No command hangs waiting for input when there is no terminal:

- `package`, `preview` - never prompt.
- `plugin-install <token>` - refuses to **replace** an existing license in a non-interactive shell; pass `--force` when that is intended.
- `recover-license <email>` - the email must be passed as an argument (exits 2 otherwise).
- `trial <token>` - the token is required; request it at https://ad-preflight.com first.

---

## JSON output

**Success** (exit 0):

```json
{
  "success": true,
  "outputPath": "compliance_my-ad.zip",
  "size": 12345,
  "type": "standard",
  "warnings": [],
  "fixLogs": [],
  "staticChecks": [ { "id": "clicktag", "label": "Click handler", "status": "pass" } ],
  "staticChecksBySize": { "...": [] },
  "packages": [
    { "name": "my-ad", "outputPath": "compliance_my-ad.zip", "size": 12345 }
  ]
}
```

Notes for parsers:

- **Collect artifacts from `packages`, not from `outputPath`.** For a multi-size campaign
  there is one entry per size; top-level `outputPath`/`size` describe only the first ZIP.
  Sizes that failed validation appear in `packagesSkipped` instead.
- `originalZipPath` is present only when `--fix` ran (path of the pre-fix backup ZIP).
- `deep` is present only when `--deep` ran (the Deep Audit report object).
- `staticChecks` / `staticChecksBySize` feed the human report checklist - safe to ignore in CI.

**Failure** (exit 1):

```json
{
  "success": false,
  "issues": ["Missing var clickTag", "HTTP resource detected"],
  "warnings": [],
  "error": "Validation failed: 2 issues found"
}
```

Check `result.success === true` and exit code `0` to confirm a pass.

---

## Pipeline examples

### Simple - fail on non-zero exit

```bash
npx -y @ad-preflight/cli package ./ad --json
# Exit code 0 = pass. That's it.
```

### Shell script - capture output

```bash
#!/bin/sh
set -e
npx -y @ad-preflight/cli package ./ad --json > result.json
echo "Ad validation passed"
```

### GitHub Actions

```yaml
- name: Validate ad creative
  run: npx -y @ad-preflight/cli package ./ad --json > result.json

- name: Check result
  run: |
    if [ "$(jq -r .success result.json)" != "true" ]; then
      echo "::error::Ad validation failed"
      jq . result.json
      exit 1
    fi
```

### GitLab CI

```yaml
validate-ad:
  script:
    - npx -y @ad-preflight/cli package ./ad --json > result.json
  artifacts:
    paths:
      - result.json
    when: always
```

### With auto-fix in CI

```bash
npx -y @ad-preflight/cli package ./ad --fix --json > result.json
# Produces compliance_ad.zip (fixed) and ad_original.zip (backup)
# Upload compliance_ad.zip as your build artifact
```

---

## Options reference

All [CLI options](CLI.md) work in machine mode. Common CI flags:

| Flag | Use in CI |
|------|-----------|
| `--json` | Machine-readable output (default: compact) |
| `--json=pretty` | Pretty-printed JSON (easier to read in logs) |
| `--fix` | Auto-fix issues and produce upload-ready ZIP |
| `--type amp` | Validate as AMP ad (600KB limit) |
| `--strict-dimensions` | Fail on non-IAB dimensions instead of warning |
| `--deep` | Headless browser validation (Pro license required) |
