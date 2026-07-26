# CLI guide

Validate, fix, and package HTML5 ad creatives from the terminal — catch issues before they cause rejections on any platform.

**Supported platforms:** Google Ads · DV360 · Doubleclick · Sizmek · Adform · any IAB-standard HTML5 ad platform

**See also:** [Deep Audit (Pro)](DEEP.md) for runtime checks in a real browser · [CI/CD guide](CI-CD.md) for pipelines · [MCP setup](MCP.md) for Cursor and AI agents

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
| `ad-preflight plugin-install <token>` | Install the Deep Audit plugin with your purchase token — see [Deep Audit](DEEP.md) |

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
| `--deep` | Run the ad in a headless browser: JS errors, CPU, network, visual, animation and behavior checks (Pro — see [Deep Audit](DEEP.md)) |
| `--report [file]` | With `--deep`: write a self-contained PDF report you can share with adops |
| `--nopreview` | Skip opening the browser after packaging |
| `--out <dir>` | Where the `.zip` files go. Defaults to the folder containing the creative — for a multi-size campaign, that means one archive per size, next to the campaign |
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
| Deep Audit (Pro) | `ad-preflight package ./my-ad --deep` |
| Deep Audit + PDF report (Pro) | `ad-preflight package ./my-ad --deep --report` |
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

The Deep Audit runs your ad in a real headless browser and reports what static checks can't catch: runtime JavaScript errors, CPU on pace for Google's Heavy Ad Intervention, network problems, blank renders, slot overflow, animation-limit violations, and auto-redirects. Multi-size packs get one rolled-up report, and `--report` produces a shareable PDF scorecard.

```bash
ad-preflight trial                               # Free trial: 3 Deep Audits, emailed token
ad-preflight plugin-install <token>              # One-time setup with your purchase token
ad-preflight package ./my-ad --deep --report     # Audit + shareable PDF report
```

**Full guide: [Deep Audit (Pro)](DEEP.md)** — what it checks, where to buy, installation, and troubleshooting.

Pro activates one machine; Team covers five. Core features remain free forever.

---

## Licence commands

```bash
ad-preflight trial [token]        # Request a free-trial token, or redeem the one emailed to you
ad-preflight license              # What this machine holds: plan, machine id, file locations
ad-preflight license --refresh    # Check in and renew after a long spell offline
ad-preflight license --show-token # Print the purchase token in full
ad-preflight recover-license      # Lost the token? We email it to the address that bought it
ad-preflight plugin-uninstall     # Remove the plugin and hand the machine slot back
```

A paid licence does not expire. The plugin re-verifies online from time to time, sending
only the licence token and a hashed machine id — never your creatives. A machine that
stays offline past the grace window shows `needs to reconnect` until it checks in again.
