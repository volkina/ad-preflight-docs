# Ad-Preflight

<img src="https://ad-preflight.com/demo.gif" alt="Ad-Preflight demo" width="600" />

**Stop getting HTML5 ads rejected.** Validate, auto-fix, and package creatives before you upload — works with any platform that accepts HTML5 ads.

**Works with:** Google Ads · DV360 · Doubleclick · Sizmek · Adform · and any platform following IAB HTML5 ad standards

[![npm version](https://img.shields.io/npm/v/@ad-preflight/cli)](https://www.npmjs.com/package/@ad-preflight/cli)

---

## Quick start

```bash
npm install -g @ad-preflight/cli
ad-preflight package ./my-ad --fix
```

```
✔ Ad validated
✔ Auto-fixed: clickTag injected, ad.size meta added
✔ Zip created: compliance_my-ad.zip (48 KB — under 150 KB limit)
✔ Original backed up: my-ad_original.zip
```

---

## What it catches — and what it fixes

| Validation check | Auto-fix (`--fix`) |
|---|---|
| Missing or incorrect `<meta name="ad.size">` | ✅ Injects from CSS dimensions |
| Missing click handler (clickTag, Enabler API, Exit API) | ✅ Injects clickTag variable |
| HTTP references (rejected by ad platforms) | ✅ Converts to HTTPS |
| Bundle exceeds size limit (150KB / 600KB / 5MB) | — |
| Non-standard IAB dimensions | — (warns; fails with `--strict-dimensions`) |
| Spoofed file extensions (magic-number check) | — |

With `--fix`, originals are backed up to `{folder}_original.zip` so nothing is lost.

---

## Works wherever you work

| Use case | How | Guide |
|----------|-----|-------|
| **Terminal** | `ad-preflight package ./ad --fix` — human-friendly output with optional browser preview | [CLI guide](docs/CLI.md) |
| **CI/CD** | `ad-preflight package ./ad --json` — single JSON on stdout, exit codes 0/1/2, auto machine mode | [CI/CD guide](docs/CI-CD.md) |
| **AI agents (Cursor / MCP)** | `validate_ad_creative` tool — agents validate and fix ads without leaving the editor | [MCP setup](docs/MCP.md) |

Same validation logic everywhere. Output adapts automatically: human-friendly in a TTY, machine-readable JSON when piped or with `--json`.

---

## Free vs Pro

| Capability | Free | Pro |
|---|:---:|:---:|
| Static validation (meta tags, ClickTag, HTTPS, file types, size limits) | ✅ | ✅ |
| Auto-fix mode (`--fix`) | ✅ | ✅ |
| IAB dimension validation | ✅ | ✅ |
| ZIP packaging (upload-ready output) | ✅ | ✅ |
| CI/CD and MCP integration | ✅ | ✅ |
| Local preview (publisher-style page) | ✅ | ✅ |
| Headless browser validation (`--deep`) | — | ✅ |
| CPU & performance profiling | — | ✅ |
| Network waterfall analysis | — | ✅ |
| Visual snapshots | — | ✅ |

> **Get started free. Upgrade when you need runtime validation.**
>
> `ad-preflight buy` or visit [ad-preflight.com](https://ad-preflight.com)

**License management:**

```bash
ad-preflight buy                    # Purchase or join waitlist
ad-preflight activate <license-key> # Activate on this machine
ad-preflight license                # Check license status
```

Each Pro license supports up to 2 machines. Core features remain free forever.

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

For pipeline setup, see the [CI/CD guide](docs/CI-CD.md). For Cursor/MCP, see [MCP setup](docs/MCP.md).

---

## Documentation

- [CLI guide](docs/CLI.md) — Commands, options, examples, local preview
- [CI/CD guide](docs/CI-CD.md) — Machine mode, exit codes, JSON schema, pipeline examples
- [MCP setup](docs/MCP.md) — Config, `validate_ad_creative` tool, Cursor examples

---

## License

See [LICENSE.md](LICENSE.md).
