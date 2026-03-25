# CI/CD guide

Catch broken ad creatives in your pipeline — before they reach any ad platform. ad-preflight validates against IAB standards and platform requirements (Google Ads, DV360, Sizmek, Adform, and more), outputs machine-readable JSON, and uses standard exit codes.

**See also:** [CLI guide](CLI.md) for full command reference · [MCP setup](MCP.md) for Cursor and AI agents

---

## Install in CI

**As a dev dependency (recommended — locked version):**

```bash
npm install --save-dev @ad-preflight/cli
```

**Or run with npx (no install needed):**

```bash
npx -y @ad-preflight/cli package ./ad --json
```

---

## Machine mode

In machine mode, ad-preflight writes **exactly one JSON object** to stdout — no banners, no progress bars, no tips. Use the exit code for pass/fail.

Machine mode activates automatically when:

- You pass `--json` or `--json=pretty`
- stdout is not a TTY (pipes, CI runners, headless environments)

---

## Exit codes

| Code | Meaning | Action |
|------|---------|--------|
| **0** | Validation passed, ZIP created | Continue pipeline |
| **1** | Validation failed or runtime error | Fail the build |
| **2** | Invalid arguments | Fix the command |

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
  "originalZipPath": "my-ad_original.zip",
  "deep": false
}
```

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

### Simple — fail on non-zero exit

```bash
ad-preflight package ./ad --json
# Exit code 0 = pass. That's it.
```

### Shell script — capture output

```bash
#!/bin/sh
set -e
ad-preflight package ./ad --json > result.json
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
ad-preflight package ./ad --fix --json > result.json
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
