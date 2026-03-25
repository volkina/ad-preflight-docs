# CLI guide

Validate, fix, and package HTML5 ad creatives from the terminal — catch issues before they cause rejections on any platform.

**Supported platforms:** Google Ads · DV360 · Doubleclick · Sizmek · Adform · any IAB-standard HTML5 ad platform

**See also:** [CI/CD guide](CI-CD.md) for pipelines · [MCP setup](MCP.md) for Cursor and AI agents

---

## Installation

**Global (recommended):**

```bash
npm install -g @ad-preflight/cli
ad-preflight --help
```

**Project dependency:**

```bash
npm install --save-dev @ad-preflight/cli
npx @ad-preflight/cli package ./ad
```

---

## Commands

| Command | What it does |
|---------|-------------|
| `ad-preflight package <folder>` | Validate and package an ad creative into an upload-ready ZIP |
| `ad-preflight preview <folder>` | Open the ad in a local publisher-style page (test layout, z-index, click-through) |
| `ad-preflight init-rules` | Add `.cursorrules` to your project so AI agents can suggest ad-preflight |
| `ad-preflight buy` | Purchase Pro or join the waitlist |
| `ad-preflight activate <key>` | Activate a Pro license on this machine |
| `ad-preflight license` | Show current license status |

---

## Package command

```bash
ad-preflight package <folder> [options]
```

This is the core command. It validates the ad creative, reports issues, and produces a platform-ready ZIP.

### Options

| Flag | Description |
|------|-------------|
| `--fix` | Auto-fix issues (click handler, HTTPS, ad.size meta). Backs up originals to `{folder}_original.zip` |
| `--type <type>` | Ad type: `standard` (default), `amp`, `app` |
| `--strict-dimensions` | Fail on non-IAB dimensions (default: warn only) |
| `--deep` | Headless browser validation — CPU profiling, network analysis, visual snapshots (Pro) |
| `--nopreview` | Skip opening the browser after packaging |
| `--json [format]` | Machine-readable JSON output. Use `--json=pretty` for formatted output. See [CI/CD guide](CI-CD.md) |
| `-h, --help` | Show help |

### How `--fix` works

1. Zips the folder as-is to `{folder}_original.zip` (your backup)
2. Applies fixes on disk (injects click handler, ad.size meta, converts HTTP to HTTPS)
3. Zips the fixed result as `compliance_{folder}.zip` (upload-ready)

You keep the backup. The compliance ZIP is what you upload.

---

## Examples

| Goal | Command |
|------|---------|
| Validate only | `ad-preflight package ./my-ad` |
| Validate and auto-fix | `ad-preflight package ./my-ad --fix` |
| AMP ad | `ad-preflight package ./amp-ad --type amp --fix` |
| App campaign | `ad-preflight package ./app-ad --type app` |
| Strict IAB dimensions | `ad-preflight package ./my-ad --strict-dimensions` |
| Deep validation (Pro) | `ad-preflight package ./my-ad --deep` |
| JSON output for scripts | `ad-preflight package ./my-ad --json` |
| Skip browser preview | `ad-preflight package ./my-ad --fix --nopreview` |

---

## Local preview

Test your ad in a publisher-style page before uploading — check layout, z-index stacking, and click-through behavior:

```bash
ad-preflight preview ./my-ad
```

Opens a local browser page that mimics how the ad will appear on a publisher site.

---

## Pro features

Deep validation runs your ad in a headless browser and reports what static checks can't catch: CPU performance, network requests, and visual rendering issues.

```bash
ad-preflight package ./my-ad --deep    # Requires Pro license
```

**License management:**

```bash
ad-preflight buy                       # Purchase or join waitlist
ad-preflight activate <license-key>    # Activate on this machine
ad-preflight license                   # Check status
```

Each Pro license supports up to 2 machines. Core features remain free forever.
