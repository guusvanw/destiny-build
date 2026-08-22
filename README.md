# destiny-build

A Claude Code plugin for Destiny 2 build advice, answered against **your own
live account** rather than from memory — the vault, and since **2.1.0** also
progression and weekly milestone state, **this week's activity modifiers per
difficulty tier**, the seasonal artifact, triumphs and seals, Collections, any
vendor's stock and your own clear history.

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

## Web and mobile sessions: the plugin cannot install itself

**Agent-followable version of this section: [docs/cloud-setup.md](docs/cloud-setup.md)** — fetchable raw at `https://raw.githubusercontent.com/guusvanw/destiny-build/main/docs/cloud-setup.md`.

**This install is laptop-only, and the reason is structural rather than a
misconfiguration.** Declaring the plugin in a repo's `.claude/settings.json`
registers the marketplace but does **not** install it: as of Claude Code
v2.1.195, adding a marketplace does not install a plugin that comes from an
external source, on any path that loads plugins. So a fresh cloud container
prints the install command and waits for a person who is not there. Three
sessions went into configuring around this before that paragraph of the docs got
read.

**What works is a plain `.mcp.json` at the repository root.** Cloud sessions load
project-scoped MCP servers without prompting, precisely because they cannot show
an approval dialog:

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

Set `D2_MCP_URL` and `D2_MCP_TOKEN` in the cloud environment config — the same
two values the install prompts for — and every repo carrying that file has the
tools, from a phone included.

**The plugin and `.mcp.json` are alternatives, not layers.** A laptop with both
loads the same server twice under two names: two copies of every tool in the
list, confusing to read and paid for in every request. Pick one per repo.

### Getting the skill into a web session too

`.mcp.json` gets you the tools, not the skill — that is the real trade-off. The
skill is a single Markdown file, so vendor it rather than forking anything:

```
mkdir -p .claude/skills/destiny-build
curl -sL https://raw.githubusercontent.com/guusvanw/destiny-build/main/plugin/skills/destiny-build/SKILL.md \
  -o .claude/skills/destiny-build/SKILL.md
```

A repo carrying both that and the `.mcp.json` above has the full thing in a
cloud session with no install step. Re-run the `curl` to update — and it is the
same file the plugin ships, so nothing drifts as long as you do not edit the
copy.

## You need your own server

The plugin talks to a **[d2-mcp](https://github.com/guusvanw/d2-mcp)** instance
holding a Bungie OAuth grant. It is a Dockerfile and a Python app, so host it
wherever you like. **You cannot share someone else's**: Bungie rotates the refresh token on every use, so
two clients on one grant invalidate each other within a day. Each player runs
their own instance with their own grant, and can authorise against the same
Bungie *application* — a registration is authorised per user, so a second player
does not register anything; only the grant differs.

**Register that application for read AND write** — *Read your Destiny 2
information* plus *Move or equip Destiny gear*. Equipping a build, fitting its
mods, saving it to a loadout slot and locking what a vault cull keeps are all
writes, and a read-only registration mints tokens that refuse them.

## The skill is impersonal on purpose

It carries what is true for any Destiny player: game mechanics, method, tool
routing, failure modes. It carries nothing about any particular player's roll
priority, play style or vault, and it asks you to record your own as they come
up. That is what lets several people install the same copy without forking it —
a forked skill drifts, and the drift is silent.

## Working on the skill itself

The skill and the server **version independently**. That is deliberate — they
live in separate repos so a public plugin does not drag a private deployment's
Fly config and OAuth notes along with it — and the standing cost is that a skill
can happily describe a tool the deployment does not have. So when a change
alters a tool's contract, change the skill's calibrations in the same pass.

The server lives at [guusvanw/d2-mcp](https://github.com/guusvanw/d2-mcp), and
its `HANDOFF.md` carries the implementation orientation: current state, a code
map, and the traps that have actually cost time. Start there rather than here
for anything beyond skill prose.

## Writes

`d2_apply` is the only tool that changes the account, and it is **dry run by
default**. It equips gear, selects weapon perks, fits armour mods and tuning,
sets a whole subclass — super, abilities, aspects, fragments — **saves the result
into one of the game's own loadout slots**, sets the lock flag a vault cull runs
on, and **pulls gear out of the postmaster**. So a recommended build can be run
*and kept* rather than assembled by hand, and DIM is not needed to save one.

`pull_postmaster` behaves slightly differently from the rest and both
differences are deliberate: a **bare call is a listing**, so asking for it shows
every character's Lost Items bucket and pulls nothing; and a failure does **not**
stop the batch, because the usual failure is one full destination bucket while
the bucket everything else is sitting in *evicts*. Failures come back per row.

**Where a piece currently sits does not limit what can be recommended.** Bungie's
transfer is character-to-vault or vault-to-character and has no third form, so a
copy held by another Guardian is planned as two hops through the vault, and a
full destination bucket has something moved out of it first — named in the plan,
with the rule that chose it, and moved to the *vault* rather than dismantled.
Two things genuinely refuse and both say what to do about it: gear **equipped on
another character** (Bungie will not transfer equipped gear) and a **full
vault**.

Performing a write needs `confirm=true` per action — that is the real gate, and
the plan is always shown first — plus `D2_ALLOW_WRITE` on **your own** server,
which is a kill switch for freezing every write at once rather than a per-feature
opt-in. Underneath both sits the Bungie side: your **grant** has to have been
minted while the app registration carried *Move or equip Destiny gear*. Widening
the app afterwards does not widen an existing token, and nothing can read a
token's scope back, so the first write either works or fails `NotAuthorized` —
and the cheapest way to find out is reversible: lock one item, then unlock it.
The refusal message names which lock stopped you and derives your app name from
the server's own environment rather than printing somebody else's.

Four things no API can do, and a build recommendation should name them rather
than trailing off: **artifact perks** (set in game, as is the artifact unlock
order), **masterworking and infusion**, **kill trackers** (the socket is hidden
and its options usually cannot even be enumerated, so it is two clicks in game),
and reading a write back immediately — Bungie takes up to ~75s to reflect it.

**Artifact perks are unwritable but no longer unreadable.** Since 2.1.0
`d2_progress(section="artifact")` reports, per character, which perks are
*unlocked* and which are *slotted*, so the manual step comes with the specifics
instead of a reminder — "this one is unlocked, not slotted, and you have a free
socket" rather than "don't forget your artifact".

## Licence

MIT.
