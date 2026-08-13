# MCP setup

Let AI agents validate and fix HTML5 ad creatives without leaving the editor. ad-preflight runs as an MCP server in Cursor (or any MCP-compatible client), validating against IAB standards and platform requirements for Google Ads, DV360, Sizmek, Adform, and more.

**See also:** [CLI guide](CLI.md) for full command reference · [CI/CD guide](CI-CD.md) for pipelines · [Deep Audit (Pro)](DEEP.md) for runtime checks

---

## How it works

- Exposes one tool: **`validate_ad_creative`** - validates and optionally auto-fixes an HTML5 ad creative
- Also registers two prompts (`analyze_ad`, `fix_ad`) and three reference resources (platform specifications, file size limits, allowed file types) - your MCP client lists them alongside the tool
- Runs over stdio - Cursor starts it on demand, no separate server process
- Same validation rules as the CLI: click handlers (clickTag, Enabler API, Exit API), ad.size meta, HTTPS, file types, size limits, IAB dimensions

---

## Configuration

Add ad-preflight to your MCP config: `.cursor/mcp.json` (project-level) or `~/.cursor/mcp.json` (global).

### Recommended (npx)

Works whether the package is installed locally or not:

```json
{
  "mcpServers": {
    "ad-preflight": {
      "command": "npx",
      "args": ["-y", "@ad-preflight/cli", "mcp"]
    }
  }
}
```

### Alternative (explicit path)

Use when you prefer a fixed install path or want to avoid npx:

```json
{
  "mcpServers": {
    "ad-preflight": {
      "command": "node",
      "args": ["node_modules/@ad-preflight/cli/dist/mcp-server.js"]
    }
  }
}
```

**Restart Cursor after changing the config.**

---

## Tool: `validate_ad_creative`

Validates and packages an HTML5 ad creative according to platform specs.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `folderPath` | string | yes | - | Absolute path to the creative folder - a single creative, or a campaign folder with one subfolder per size |
| `type` | string | no | `"standard"` | Ad type: `standard`, `amp`, or `app` |
| `fix` | boolean | no | `false` | Auto-fix issues (inject ClickTag, HTTPS, ad.size). Rewrites the creative in place and backs the original up to `<folder>_original.zip` |
| `strictDimensions` | boolean | no | `false` | Fail when the ad size is not an IAB standard dimension (default: warn only) |

### What the agent sees

The tool returns a structured result the agent can act on:

| Field | Meaning |
|---|---|
| `success` | `false` means the creative must not be uploaded |
| `issues` | Blocking problems that made validation fail |
| `warnings` | Non-blocking problems worth reviewing |
| `fixLogs` | What was changed when `fix` was `true` |
| `packages` | Every ZIP produced - one entry per size for a multi-size campaign |
| `packagesSkipped` | Sizes that failed validation and produced no ZIP |
| `originalZipPath` | Backup of the pre-fix creative (only when `fix` was `true`) |
| `logs` | Progress messages from the run |

A creative that fails validation is a normal answer, not a tool error: the agent gets
`success: false` plus the findings. Tool errors are reserved for runs that can't produce a
verdict at all (for example, the folder doesn't exist).

---

## Example prompts

Once MCP is configured, ask the agent naturally:

| Prompt | What happens |
|--------|-------------|
| "Validate this ad creative" | Runs validation only (`fix: false`) |
| "Check and fix issues in this ad" | Validates and auto-fixes (`fix: true`) |
| "Validate this AMP ad" | Validates with `type: "amp"` |
| "Validate every size in this campaign" | Validates each size subfolder and reports which ones passed |
| "Is this ad ready for Google Ads?" | Validates against platform requirements |

The agent calls `validate_ad_creative` with the appropriate parameters based on your prompt.

---

## Agent suggestions (optional)

Write agent instructions into your project so agents recognise HTML5 ad creatives and know
to validate them:

```bash
npx @ad-preflight/cli init-rules
```

It writes three files, because no single format is read everywhere:

| File | Read by |
|---|---|
| `AGENTS.md` | the cross-tool convention most agent CLIs and IDEs look for |
| `.cursor/rules/ad-preflight.mdc` | current Cursor |
| `.cursorrules` | older Cursor versions (legacy) |

Existing files are left alone; pass `--force` to overwrite. This adds project-level
instructions only - you still need the MCP config above for the tool itself to work.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Tool not appearing in Cursor | Restart Cursor after editing `.cursor/mcp.json` |
| Wrong path error (node variant) | Verify the path to `mcp-server.js` is correct from your project root. Use the npx config to avoid path issues |
| Want CLI fallback instead | Run `npx @ad-preflight/cli package ./ad --json` and parse the output. See [CI/CD guide](CI-CD.md) |
