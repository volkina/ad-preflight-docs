# CLI guide

Validate, fix, and package HTML5 ad creatives from the terminal - catch issues before they cause rejections on any platform.

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

All examples below use `npx @ad-preflight/cli` - it works with either install (and even
with none: npx fetches the package). With a global install, plain `ad-preflight` works too.

---

## Commands

| Command | What it does |
|---------|-------------|
| `npx @ad-preflight/cli package <folder>` | Validate and package an ad creative into an upload-ready ZIP |
| `npx @ad-preflight/cli preview <folder>` | Open the ad in a local publisher-style page (test layout, z-index, click-through) |
| `npx @ad-preflight/cli init-rules` | Add agent instructions (`AGENTS.md`, Cursor rules) so AI agents know to validate ads with ad-preflight |
| `npx @ad-preflight/cli buy` | Show how to buy the Pro Deep Audit plugin |
| `npx @ad-preflight/cli trial <token>` | Start the free Deep Audit trial (get the token at https://ad-preflight.com) |
| `npx @ad-preflight/cli plugin-install <token>` | Install the Deep Audit plugin with your purchase token - see [Deep Audit](DEEP.md) |
| `npx @ad-preflight/cli plugin-uninstall` | Remove the plugin and release this machine's seat |
| `npx @ad-preflight/cli license` | Show the license installed on this machine (`--refresh` to renew) |
| `npx @ad-preflight/cli recover-license <email>` | Email your purchase token to the address you bought with |
| `npx @ad-preflight/cli mcp` | Start the MCP server (stdio) for Cursor and other MCP clients |

---

## Package command

```bash
npx @ad-preflight/cli package <folder> [options]
```

This is the core command. It validates the ad creative, reports issues, and produces a platform-ready ZIP.

### Options

| Flag | Description |
|------|-------------|
| `--fix` | Auto-fix issues (click handler, HTTPS, ad.size meta). Backs up originals to `{folder}_original.zip` |
| `--type <type>` | Ad type: `standard` (default), `amp`, `app` |
| `--strict-dimensions` | Fail on non-IAB dimensions (default: warn only) |
| `--deep` | Run the ad in a headless browser: JS errors, CPU, network, visual, animation and behavior checks (Pro - see [Deep Audit](DEEP.md)) |
| `--report [file]` | With `--deep`: write a self-contained PDF report you can share with adops (default name: `ad-preflight-<folder>.pdf`) |
| `--out <dir>` | Where the `.zip` files go. Defaults to the folder containing the creative - for a multi-size campaign, that means one archive per size, next to the campaign |
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
| Validate only | `npx @ad-preflight/cli package ./my-ad` |
| Validate and auto-fix | `npx @ad-preflight/cli package ./my-ad --fix` |
| AMP ad | `npx @ad-preflight/cli package ./amp-ad --type amp --fix` |
| App campaign | `npx @ad-preflight/cli package ./app-ad --type app` |
| Strict IAB dimensions | `npx @ad-preflight/cli package ./my-ad --strict-dimensions` |
| Deep validation (Pro) | `npx @ad-preflight/cli package ./my-ad --deep` |
| Deep validation + PDF report | `npx @ad-preflight/cli package ./my-ad --deep --report` |
| JSON output for scripts | `npx @ad-preflight/cli package ./my-ad --json` |
| Skip browser preview | `npx @ad-preflight/cli package ./my-ad --fix --nopreview` |

---

## Local preview

Test your ad in a publisher-style page before uploading - check layout, z-index stacking, and click-through behavior:

```bash
npx @ad-preflight/cli preview ./my-ad
npx @ad-preflight/cli preview ./my-ad --port 8080   # custom port (default: 3750)
```

Opens a local browser page that mimics how the ad will appear on a publisher site.
Multi-size campaigns are detected automatically - every size gets an entry in the preview.

---

## Pro features

The Deep Audit runs your ad in a real headless browser and reports what the standard audit can't
catch: runtime JavaScript errors, CPU on pace for Google's Heavy Ad Intervention, network
problems, blank renders, slot overflow, animation-limit violations, and auto-redirects.
Multi-size packs get one rolled-up report, and `--report` produces a shareable PDF scorecard.

```bash
npx @ad-preflight/cli package ./my-ad --deep    # Requires the Pro plugin
```

**Try it free:** get a trial token at https://ad-preflight.com, then:

```bash
npx @ad-preflight/cli trial <token>
```

**License management:**

```bash
npx @ad-preflight/cli buy                        # How to buy
npx @ad-preflight/cli plugin-install <token>     # Install the plugin on this machine
npx @ad-preflight/cli license                    # Check status (--refresh to renew)
npx @ad-preflight/cli recover-license <email>    # Lost your token? Get it emailed
npx @ad-preflight/cli plugin-uninstall           # Remove and free this machine's seat
```

Plans, pricing and machine limits: https://ad-preflight.com. Core features remain free forever.

**Full guide: [Deep Audit (Pro)](DEEP.md)** - what it checks, where to buy, installation, and troubleshooting.
