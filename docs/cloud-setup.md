# Loading the Destiny tools and skill in a cloud (web/mobile) session

*Instructions an agent can execute directly: if you are a Claude session running in the repo, perform steps 1, 2 and 5 yourself — including the commit and push, since a cloud VM is throwaway and only pushed changes survive. Only step 3 needs a human (it is a settings UI). The plugin-install route structurally cannot work in a cloud session — as of Claude Code v2.1.195, declaring a marketplace in `.claude/settings.json` registers it but does not install a plugin from an external source, and a laptop's prompted `userConfig` values never reach a cloud VM. A cloud session needs three things checked into the repo it runs in, plus two values in the cloud environment config.*

## Step 1 — root `.mcp.json` (checked into the repo)

```json
{
  "mcpServers": {
    "destiny2": {
      "type": "http",
      "url": "${D2_MCP_URL}",
      "headers": { "Authorization": "Bearer ${D2_MCP_TOKEN}" }
    }
  }
}
```

Keep the `${...}` indirection literally as written — never hardcode the URL or token; the token is a credential and the file is committed.

## Step 2 — vendor the skill (checked into the repo)

`.mcp.json` gets the tools; the skill is the guidance layer, one Markdown file, vendored not forked:

```bash
mkdir -p .claude/skills/destiny-build
curl -sL https://raw.githubusercontent.com/guusvanw/destiny-build/main/plugin/skills/destiny-build/SKILL.md \
  -o .claude/skills/destiny-build/SKILL.md
```

Re-run the same `curl` to update. Do not edit the copy — it is byte-identical to what the plugin ships; editing it is how a skill drifts.

**Now commit and push both files** — this is part of the setup, not housekeeping. In a cloud session, an uncommitted file dies with the VM:

```bash
git add .mcp.json .claude/skills/destiny-build/SKILL.md
git commit -m "Load the Destiny tools and skill in cloud sessions"
git push
```

## Step 3 — the two values, in the cloud environment config (NOT in the repo)

In the Claude Code cloud environment settings for the repo, set:

| Variable | Value |
|---|---|
| `D2_MCP_URL` | your own server's MCP endpoint, `https://<your-app>.fly.dev/mcp` |
| `D2_MCP_TOKEN` | your own server's `D2_MCP_TOKEN` |

**Each player has their own deployment and their own grant** — never another player's URL or token. If you don't hold your values, the operator who deployed your server can read them from its Fly secrets.

Note the documented caveat: the cloud environment config has no secrets store — anyone who can use that environment can read these values.

## Step 4 (optional) — if a laptop with the plugin also opens this repo

The plugin (user-wide) and this repo's `.mcp.json` + vendored skill would both load there — every tool and the skill twice, noisy and a known way real failures hide behind duplicates. If that applies, also commit a `.claude/settings.json` disabling the plugin for this repo only:

```json
{ "enabledPlugins": { "destiny-build@destiny-tools": false } }
```

The plugin stays active everywhere else on that laptop.

## Step 5 — verify, in a fresh session

1. Start a **new** cloud session (a running one won't pick up new env vars).
2. The tool list should show **one** `destiny2` server (15 tools, `d2_profile` … `d2_triage`).
3. Call `d2_profile` — it should answer with *your* characters.
4. The very first call after a server deploy can take noticeably long (the machine wakes from suspend and may rebuild its manifest cache); a timeout there is a retry, not an outage. Steady state is a couple of seconds after a wake.

Triage: tools missing → env vars unset or stale session · `401` → wrong token · someone else's characters → wrong URL.

## Two traps that have already cost time

- **Pick one route per repo.** If the same repo also has the plugin enabled, the same server loads twice under two names — every tool duplicated, and a duplicate name is how a real connection failure once stayed invisible for a week. Laptops keep the plugin for everywhere else; inside a repo carrying `.mcp.json`, the plugin should not also be enabled.
- **Don't "fix" an empty tool list by hardcoding values into `.mcp.json`.** The fix is always the environment config.
