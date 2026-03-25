# MCP setup

Let AI agents validate and fix HTML5 ad creatives without leaving the editor. ad-preflight runs as an MCP server in Cursor (or any MCP-compatible client), validating against IAB standards and platform requirements for Google Ads, DV360, Sizmek, Adform, and more.

**See also:** [CLI guide](CLI.md) for full command reference · [CI/CD guide](CI-CD.md) for pipelines

---

## How it works

- Exposes one tool: **`validate_ad_creative`** — validates and optionally auto-fixes an HTML5 ad creative
- Runs over stdio — Cursor starts it on demand, no separate server process
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
| `folderPath` | string | yes | — | Absolute path to the ad creative folder |
| `type` | string | no | `"standard"` | Ad type: `standard`, `amp`, or `app` |
| `fix` | boolean | no | `false` | Auto-fix issues (inject ClickTag, HTTPS, ad.size) |

### What the agent sees

The tool returns structured results — issues found, fixes applied, ZIP path, and warnings — so the agent can report back or take follow-up actions.

---

## Example prompts

Once MCP is configured, ask the agent naturally:

| Prompt | What happens |
|--------|-------------|
| "Validate this ad creative" | Runs validation only (`fix: false`) |
| "Check and fix issues in this ad" | Validates and auto-fixes (`fix: true`) |
| "Validate this AMP ad" | Validates with `type: "amp"` |
| "Is this ad ready for Google Ads?" | Validates against platform requirements |

The agent calls `validate_ad_creative` with the appropriate parameters based on your prompt.

---

## Agent suggestions (optional)

Add `.cursorrules` to your project so agents automatically recognize HTML5 ad projects and suggest using ad-preflight:

```bash
ad-preflight init-rules
```

This adds project-level rules only — you still need the MCP config above for the tool to work.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Tool not appearing in Cursor | Restart Cursor after editing `.cursor/mcp.json` |
| Wrong path error (node variant) | Verify the path to `mcp-server.js` is correct from your project root. Use the npx config to avoid path issues |
| Want CLI fallback instead | Run `ad-preflight package ./ad --json` and parse the output. See [CI/CD guide](CI-CD.md) |
