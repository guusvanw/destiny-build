# destiny-build

A Claude Code plugin for Destiny 2 build advice, answered against **your own
live vault** rather than from memory.

It ships the build-advice skill and the MCP connection to a `d2-mcp` server.
**You need your own deployment** — see below.

## Install

```
claude plugin marketplace add guusvanw/destiny-build
claude plugin install destiny-build@destiny-tools
```

The install prompts for two values:

| | |
|---|---|
| `d2_mcp_url` | your deployment, e.g. `https://your-app.fly.dev/mcp` |
| `d2_mcp_token` | the shared secret guarding that endpoint (stored in the OS keychain) |

Change them later with `/plugin configure destiny-build@destiny-tools`.

**Environment variables override the prompted values** when set:

```
url = ${D2_MCP_URL:-${user_config.d2_mcp_url}}
```

That ordering exists for cloud sessions. A cloud session runs on a fresh VM and
never reads `pluginConfigs`, so phone and iPad take their values from the
environment variables in the cloud environment config, while laptops use the
install prompt.

**Cloud sessions also need the plugin declared in a repository**, because
`enabledPlugins` in a user's `~/.claude/settings.json` does not travel to a
fresh VM. Add this to the `.claude/settings.json` of any repo you open sessions
on:

```json
{
  "extraKnownMarketplaces": {
    "destiny-tools": { "source": { "source": "github", "repo": "guusvanw/destiny-build" } }
  },
  "enabledPlugins": { "destiny-build@destiny-tools": true }
}
```

## You need your own server

The plugin talks to a `d2-mcp` instance holding a Bungie OAuth grant. **You
cannot share someone else's**: Bungie rotates the refresh token on every use, so
two clients on one grant invalidate each other within a day. Each player runs
their own instance with their own grant, and can authorise against the same
Bungie *application* — a registration is authorised per user.

## The skill is impersonal on purpose

It carries what is true for any Destiny player: game mechanics, method, tool
routing, failure modes. It carries nothing about any particular player's roll
priority, play style or vault, and it asks you to record your own as they come
up. That is what lets several people install the same copy without forking it —
a forked skill drifts, and the drift is silent.

## Writes

`d2_apply` is the only tool that changes the account, and it is **dry run by
default**. Performing a write needs `confirm=true`, `D2_ALLOW_WRITE` set on the
server, and a Bungie app registered for *Move or equip Destiny gear*. Two things
no API can do: **artifact perks** (set in game) and reading a write back
immediately — Bungie takes up to ~75s to reflect it.

## Licence

MIT.
