# Ad-Preflight

<img src="https://ad-preflight.com/demo.gif?v=3" alt="Ad-Preflight validating and auto-fixing an HTML5 ad, then unlocking the Pro Deep Audit" width="600" />

▶ [Watch the full Deep Audit walkthrough](https://ad-preflight.com/#demo-video) (36s, on ad-preflight.com)

**Stop getting HTML5 ads rejected.** Validate, auto-fix, and package creatives before you upload - works with any platform that accepts HTML5 ads.

> ### 🔒 Everything runs on your machine
> **Creatives are never uploaded.** No portals, no accounts, no NDA risk. Every validation, auto-fix and package step is local - and so is the Pro Deep Audit, which drives a headless browser on your own computer.

**Works with:** Google Ads · DV360 · Doubleclick · Sizmek · Adform · and any platform following IAB HTML5 ad standards

[![npm version](https://img.shields.io/npm/v/@ad-preflight/cli)](https://www.npmjs.com/package/@ad-preflight/cli)

---

## Quick start

```bash
npx @ad-preflight/cli package ./my-ad --fix
```

```
✔ Ad validated
✔ Auto-fixed: clickTag injected, ad.size meta added
✔ Zip created: compliance_my-ad.zip (48 KB - under 150 KB limit)
✔ Original backed up: my-ad_original.zip
```

---

## What it catches - and what it fixes

| Validation check | Auto-fix (`--fix`) |
|---|---|
| Missing or incorrect `<meta name="ad.size">` | ✅ Injects from CSS dimensions |
| Missing click handler (clickTag, Enabler API, Exit API) | ✅ Injects clickTag variable |
| HTTP references (rejected by ad platforms) | ✅ Converts to HTTPS |
| Bundle exceeds size limit (150KB / 600KB / 5MB) | - |
| Non-standard IAB dimensions | - (warns; fails with `--strict-dimensions`) |
| Spoofed file extensions (magic-number check) | - |

With `--fix`, originals are backed up to `{folder}_original.zip` so nothing is lost.

---

## Works wherever you work

| Use case | How | Guide |
|----------|-----|-------|
| **Terminal** | `npx @ad-preflight/cli package ./ad --fix` - human-friendly output with optional browser preview | [CLI guide](docs/CLI.md) |
| **CI/CD** | `npx @ad-preflight/cli package ./ad --json` - single JSON on stdout, exit codes 0/1/2/3, auto machine mode | [CI/CD guide](docs/CI-CD.md) |
| **AI agents (Cursor / MCP)** | `validate_ad_creative` tool - agents validate and fix ads without leaving the editor | [MCP setup](docs/MCP.md) |

Same validation logic everywhere. Output adapts automatically: human-friendly in a TTY, machine-readable JSON when piped or with `--json`.

---

## Free vs Pro

| Capability | Free | Pro |
|---|:---:|:---:|
| Standard audit (meta tags, ClickTag, HTTPS, file types, size limits) | ✅ | ✅ |
| Auto-fix mode (`--fix`) | ✅ | ✅ |
| IAB dimension validation | ✅ | ✅ |
| ZIP packaging (upload-ready output) | ✅ | ✅ |
| CI/CD and MCP integration | ✅ | ✅ |
| Local preview (publisher-style page) | ✅ | ✅ |
| Deep Audit in a real browser (`--deep`) | - | ✅ |
| Runtime JavaScript error detection (with file:line) | - | ✅ |
| CPU profiling & Google Heavy Ad risk (low-end device emulation) | - | ✅ |
| Network waterfall, HTTPS & load-budget analysis | - | ✅ |
| Visual checks: screenshots, blank render, slot overflow, border rule | - | ✅ |
| Animation limit (30s) & auto-redirect/popup detection | - | ✅ |
| Multi-size pack reports + shareable PDF report (`--report`) | - | ✅ |

> **Get started free. Upgrade when you need runtime validation.**
>
> `npx @ad-preflight/cli buy` or visit [ad-preflight.com](https://ad-preflight.com)

**License management:**

```bash
npx @ad-preflight/cli buy                        # How to buy
npx @ad-preflight/cli plugin-install <token>     # Install the Pro plugin on this machine
npx @ad-preflight/cli license                    # Check license status
npx @ad-preflight/cli recover-license <email>    # Lost your token? Get it emailed
npx @ad-preflight/cli plugin-uninstall           # Remove and free this machine's seat
```

Plans, pricing and machine limits: [ad-preflight.com](https://ad-preflight.com). Core (static)
features are free forever. Full guide: [Deep Audit](docs/DEEP.md).

---

## Try Deep Audit free

Installing the CLI gives you all the **free** static tools right away. The **Deep Audit**
(`--deep`) - real-browser CPU / network / visual checks plus the shareable PDF report - is
Pro, but you can try it free, no purchase:

```bash
# 1. Get your trial token at https://ad-preflight.com - we email it to you
npx @ad-preflight/cli trial <token>            # 2. activate the trial on this machine
npx @ad-preflight/cli package ./my-ad --deep   # 3. run your first Deep Audit
```

The trial needs an internet connection; its limits and terms are on
[ad-preflight.com](https://ad-preflight.com). Running `--deep` without the plugin also
prints these steps.

Ready for whole-campaign (multi-size) audits and clean, unwatermarked reports?
See plans and pricing at [ad-preflight.com](https://ad-preflight.com).

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

`npx @ad-preflight/cli` works with either install - and with none at all (npx fetches
the package on first use).

For pipeline setup, see the [CI/CD guide](docs/CI-CD.md). For Cursor/MCP, see [MCP setup](docs/MCP.md). For Pro runtime validation, see [Deep Audit](docs/DEEP.md).

---

## Documentation

- [CLI guide](docs/CLI.md) - Commands, options, examples, local preview
- [Deep Audit (Pro)](docs/DEEP.md) - What `--deep` checks, install, reports, troubleshooting
- [CI/CD guide](docs/CI-CD.md) - Machine mode, exit codes, JSON schema, pipeline examples
- [MCP setup](docs/MCP.md) - Config, `validate_ad_creative` tool, Cursor examples

---

## License

See [LICENSE.md](LICENSE.md).
