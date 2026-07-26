# Deep Audit (Pro)

Run your creative in a real browser before adops uploads it — and catch the rejections static checks can't see: runtime errors, heavy CPU, oversized loads, blank renders, and policy violations like endless animations or missing borders.

**Everything runs locally on your machine.** Your creative is never uploaded anywhere during validation.

**See also:** [CLI guide](CLI.md) for the free checks · [CI/CD guide](CI-CD.md) for pipelines

---

## What it checks

The Deep Audit loads your ad in a headless Chrome, watches it like a real device would, and grades every size against platform rules (Google display rules by default):

| Check | What you find out |
|---|---|
| **JavaScript errors** | Uncaught errors and console errors, with the exact `file.js:line` — the #1 cause of ads that render blank after upload |
| **CPU usage** | Whether your ad is on pace for **Google Heavy Ad Intervention** (Chrome silently unloads heavy ads). Measured at 4x CPU throttle, so results reflect low-end devices, not your dev machine |
| **Long tasks** | Main-thread blocking over 50ms — the jank that publishers complain about |
| **Network behavior** | Full request waterfall: insecure `http://` calls, broken assets (404s), failed requests, and your **initial load vs subload** split against the 150 KB budget |
| **Visual render** | A screenshot of every size, plus automatic detection of **blank renders**, content **overflowing the ad slot**, and **missing borders** on white backgrounds (a Google requirement) |
| **Animation limits** | Animations that loop forever or run past Google's **30-second limit** |
| **Behavior** | Auto-redirects and popups fired without a user click — instant rejection (and advertiser blocklisting) on every major platform |

Every issue comes with a **suggested fix**, written for creative developers.

### Multi-size packs

Point it at one folder containing your size variants and get a single report:

```
my-campaign/
├── 300x250/index.html
├── 728x90/index.html
└── 160x600/index.html
```

Each size is audited separately and rolled up into one pass/fail summary.

---

## Where to buy

1. Go to **[ad-preflight.com](https://ad-preflight.com)** and choose a plan (Pro or Team — Team includes more machine activations).
2. After checkout you receive a **purchase token** on the confirmation page. **Save it** — it's your proof of purchase, and you'll reuse it to install on a new machine or after a Node upgrade.

---

## How to install

**Requirements:** Node.js 20 or 22, and an internet connection for the one-time install (the plugin downloads a dedicated headless Chrome, ~300 MB — allow a few minutes).

```bash
ad-preflight plugin-install <your-purchase-token>
```

The installer activates your license for this machine, downloads the plugin and browser, and self-tests the setup. When you see **"Plugin installed and verified"** you're done.

After installation, deep audit works **fully offline** — no account, no login, no network needed to validate.

**Good to know:**
- Licenses cover a fixed number of machines (1 on Pro, 5 on Team). Reinstalling on the same machine doesn't use an extra slot, and `plugin-uninstall` hands the slot back.
- If you upgrade Node to a different major version (e.g. 20 → 22), just run `plugin-install` again with the same token.

---

## How to use

### Validate a creative

```bash
ad-preflight package ./my-ad --deep
```

Static validation runs first (same as the free checks), then the deep audit loads every size in the browser. You get a per-size scorecard:

```
 PASS  300x250 (300x250)
    ✓ JavaScript errors   no runtime errors
    ✓ Long tasks          main thread never blocked >50ms
    ✓ CPU usage           avg 5.9% · peak 32.2% @4x throttle
    ✓ Network             1 request(s) · 0.7 KB · first paint 124ms
    ✓ Visual              renders correctly in slot
    ✓ Animation           static or completed within window
    ✓ Behavior            no auto-redirects or popups
```

When something is wrong, the finding tells you what, where, and how to fix it:

```
 FAIL  300x250 (300x250)
  ✗ [js-errors] Uncaught error — renderCampaign is not defined (script.js:13:1)
      ↳ Fix: Define or load the referenced symbol before it runs (check script
        order/timing) or guard the call. Uncaught errors typically make the ad
        render blank on the platform.
```

### Shareable PDF report

```bash
ad-preflight package ./my-ad --deep --report
```

Writes `ad-preflight-report.pdf` — a single self-contained file with the overall verdict, every size's scorecard, screenshots, and all findings with fixes. Send it to adops or attach it to the campaign ticket; it opens anywhere, no tools required. (Use `--report my-report.pdf` to pick the filename.)

### CI/CD

```bash
ad-preflight package ./my-ad --deep --json
```

Outputs the full result as JSON and sets the exit code from the audit: **0** when everything passes (warnings allowed), **non-zero** when any size has a blocking issue — so your pipeline fails before a broken creative reaches trafficking. See the [CI/CD guide](CI-CD.md).

### Ad types

By default creatives are audited against **Google display** rules. Use `--type amp` or `--type app` for AMPHTML ads and App campaign assets — size budgets adjust accordingly.

---

## Reading the results

| Status | Meaning |
|---|---|
| **PASS** | Ready for upload — all checks green for this size |
| **WARN** | Uploadable, but review the warnings — they're the things reviewers and publishers notice |
| **FAIL** | Fix before upload — at least one issue that causes rejection or breakage in the wild |

The pack summary rolls all sizes together: one FAIL anywhere fails the run.

---

## Troubleshooting

| Message | What to do |
|---|---|
| `Deep Audit requires the Pro plugin (not installed)` | Run `ad-preflight plugin-install <token>` — buy a token at [ad-preflight.com](https://ad-preflight.com) if you don't have one |
| `Plugin for Node XX is not shipped yet` | Switch to a supported Node version (`nvm use 20` or `nvm use 22`) and re-run `plugin-install` |
| Plugin built for a different Node major | You upgraded Node — re-run `plugin-install` with your token to get the matching build |
| `Chromium is not installed` | The browser download was interrupted; run `npx playwright install chromium` once, or re-run `plugin-install` |
| `MACHINE_LIMIT` during install | Your plan's activations are used up — deactivate an old machine or upgrade to Team |
| Deep audit skipped — static validation failed | Fix the listed static issues first (or run with `--fix`), then re-run `--deep` |

---

## FAQ

**Does my creative get uploaded for validation?**
No. The audit runs entirely on your machine in a local headless browser. Nothing about your creative leaves your computer.

**Why do the CPU numbers look higher than my machine's task manager?**
The audit deliberately runs at 4x CPU throttle to emulate the low-end devices where Google's Heavy Ad Intervention actually triggers. An ad that looks fine on a dev machine can still be unloaded on a budget phone — this is the number that matters.

**Do I need to be online to validate?**
Only for the one-time install. Validation itself is fully offline. (If your creative references external assets, those are fetched during the audit like a real impression would.)

**How long does an audit take?**
Roughly 5–10 seconds per size, since each size gets a full load, settle, and measurement window in the browser.
