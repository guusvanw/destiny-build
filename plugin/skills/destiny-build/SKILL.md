---
name: destiny-build
description: Evaluate Destiny 2 builds against your live vault, and apply them - "is this build any good and can I run it", "best build for this activity", "optimise my armour for melee and super", "what mods should I run", "set my aspects and fragments", "what should I farm this week". Use whenever a Destiny build, loadout, armour set, stat spread, armour mod, subclass, super, aspect, fragment, weapon roll, set bonus, artifact perk, Xûr offering or DIM link comes up.
---

# Destiny 2 build advice

Answer against **live data and primary sources**, never memory — and check what
the data actually covers. Two conclusions were badly wrong this month, from stale
snapshots and the wrong axis; a third was caught one step from shipping, built on
200 of 423 vault pieces with nothing in the output saying so (calibration 15).

## Tools (MCP server `destiny2`, live Bungie API)

| Tool | Use |
|---|---|
| `d2_profile` | characters, power, and **check `source` for freshness** |
| `d2_inventory` | filter by kind / equippable / slot / **min_tier** / archetype / element / ammo. **Pages — read `truncated`, never `count`** (calibration 15) |
| **`d2_optimize`** | **best armour sets from the live vault.** Priorities, hard stat targets, a locked exotic. Use this instead of pulling the vault down and solving it here (calibration 16) |
| `d2_sockets` | what is in an item's sockets and what else could be — armour mods, and a subclass's super, abilities, aspects, fragments |
| `d2_item` | every owned copy of a named item - copies differ wildly |
| `d2_xur` | current stock with archetypes decoded |
| `d2_reference` | the community sheets - call bare to list ~50 tabs |
| `d2_decode_spread` | "30 Class / 25 Weapons" → Specialist |
| `d2_resolve` | manifest hashes → names. The bridge for DIM wishlist lines |
| **`d2_apply`** | **the only write. Equips gear, selects perks, fits armour mods, sets a subclass. Dry run unless `confirm=true`** |

**A whole build can now be applied, so propose it as something to run rather
than something to assemble by hand.** `d2_apply` takes a JSON action list,
returns the full plan without sending anything, and performs it on a second call
with `confirm=true`. Four ops:

| op | what it writes |
|---|---|
| `equip` | `{"op":"equip","instance_id":...}` — transfers first if the piece is in the vault or on another character |
| `plug` | `{"op":"plug","instance_id":...,"column":"Trait 1","plug":"Repulsor Brace"}` — a weapon perk already unlocked in its column |
| `mod` | `{"op":"mod","instance_id":...,"mods":["Melee Font","Heavy Handed","Void Loader"]}` — armour mods, each claiming its own socket, with energy cost reported and over-capacity warned |
| `loadout` | `{"op":"loadout","character_id":...,"subclass":"Prismatic","super":...,"grenade":...,"melee":...,"class_ability":...,"movement":...,"aspects":[...],"fragments":[...]}` — the whole subclass in one action. Every field optional; only what is named changes |

**So the end of a build recommendation is a `d2_apply` plan, not a shopping
list.** The natural pipeline is `d2_optimize` → paste its `apply` list → add the
`mod` and `loadout` actions → dry run → `confirm=true`. Live-proven for equip
and plug 2026-08-18; the mod and loadout ops are newer, so **read the dry run
before confirming them** rather than assuming.

**When it says something "is not selectable", call `d2_sockets` before telling
the player they do not own it.** Armour mod and subclass sockets are backed by
plug *sets*, and every option comes back tagged with the `source` it was found
in — so an empty option list means the server could not see the data, which is a
different answer from "not unlocked". Guessing between those two is how a
correct build gets talked out of existence.

**Aspects before fragments, and the tool already knows.** How many fragment
slots exist depends on which aspects are equipped, so a `loadout` action orders
its own writes. What this means for you: if a fragment is refused while aspects
are still pending in the same plan, apply the plan and dry-run the rest again —
do not conclude the fragment is unowned.

**Three things it cannot do**, all worth saying rather than silently omitting:
**artifact perks have no write endpoint anywhere** (set in game), the artifact
unlock order likewise, and a confirmed write takes up to ~75s to become readable
(calibration 14).

The server is **hosted**, so these tools work the same from a laptop and from a
phone session. **Each player runs their own instance and holds their own Bungie
grant** — the refresh token rotates on every use, so two clients sharing one
grant invalidate each other within a day. If the tools are missing, the server
URL or token is unset for that surface: on a laptop run
`/plugin configure destiny-build@destiny-tools`, and in a cloud session set
`D2_MCP_URL` and `D2_MCP_TOKEN` in the cloud environment config.

## The sandbox is frozen

**Destiny 2 stopped receiving updates.** `9.7.0 / Monument of Triumph` on
**2026-06-09** was the final live-service content update (and a huge ability +
Armor 3.0 overhaul); `9.7.0.3` on **2026-07-07** was the final patch. Bungie may
still hotfix for stability, nothing more.

So **freshness is now a property of the source, not the game.** A build cannot be
nerfed out from under you — "is this still viable" is permanently answerable.
The only question about any source is **does it cover the post-2026-06-09
state**. Anything older describes a materially different game, especially on
armour: **take the subclass loop from a guide, take the armour from the vault.**

## Sources — route the question, don't browse

Vetted 2026-08-16, extended 2026-08-18.

| Question | Go to |
|---|---|
| Is this weapon roll worth keeping? | **DIM wishlist** (below), then `d2_item` for the owned copies |
| Is this perk combo worth grinding? | [D2 Foundry](https://d2foundry.gg/) |
| Which set bonus is better? | `d2_reference("set-bonuses")` for the **rank**, then `("armor-perks")` for the **exact text** — never rank alone |
| What does this artifact perk do? | `d2_reference("artifact-perks")` — the layer DIM cannot apply |
| Which build should I run? | `d2_meta` + builders.gg. **Not** the sheets: their Builds tab was never started |
| **Aspect / fragment / subclass mechanics** | **Data Compendium subclass tabs** (`d2_reference("void")` etc), **never Endgame Analysis** — its aspect/fragment tabs stopped Dec 2025, before the June overhaul. The compendium is verified post-MoT |
| Weapon tier lists, archetypes, set bonuses | The sheets — Jul/Aug 2026, i.e. post-final and permanently current |

**Read an aspect's full text, never its name.** Trapper's Ambush "makes allies
invisible" — only *while Vanishing Step is also equipped*. That conditional cost
a wrong recommendation that survived a commit. Same lesson as calibration 3, for
abilities: get the **conditions**, not just the reputation.

**God rolls are a solved problem now — install the wishlist.**
`charlesxcaliber/DIMAegisWeaponWishlist` is generated from Aegis's own sheet and
updated ~weekly (last 15 Aug 2026):
`https://raw.githubusercontent.com/charlesxcaliber/DIMAegisWeaponWishlist/main/MrCharlesWishlist_MR{A-F}_PPC{0-3}.txt`
— `MR` minimum rank, `PPC` minimum good perks per column. **Start strict**; a
loose list flags half a large vault as a keep.

**Never quote numbers from boosting sites** (boostmatch, skycoach, epiccarry,
lfcarry, boosting-ground, playerauctions). They disagree on how many set
bonuses exist — 37 vs 56 vs the real 56 sets / 79 ranked bonuses.

**When a lookup misses, in order:** search the other name (calibration 5) →
pull the whole tab with a high `limit` and prove absence by naming its
alphabetical neighbours → check whether other owned sets are also uncovered (a
cluster means new content, a lone gap means lookup error) → web search →
**ask the player to read it off the screen**, which is the only route for
anything newer than the sheets.

**Blocked from cloud sessions** by the egress proxy — need a Home Desktop
Chrome session: `destiny2.science`, `destiny2armorstatcalculator.com`,
`destiny2.tools`. light.gg / builders.gg / Mobalytics 403 automation but load
fine in a browser.

## Standing calibrations — all learned the hard way

1. **Power is infusable, so it is never a ranking axis.** Filter and rank on
   tier and archetype instead. *How much a given player actually infuses is a
   preference — read their wiki.* Getting this wrong once produced a whole
   audit ranking exotics "usable vs stranded" by power, which was the wrong
   axis and the wrong conclusion. Exotic *armour* is tiered independently of
   power — a tier-5 piece can sit at power 74. **Exotic *weapons* have no tier at all** — all
   105 copies across 87 names report 0, with zero variance, while 186/187
   exotic armour pieces report >0. So "this exotic is tier 0" is never a
   finding; it is the only state that exists. Legendaries *do* tier, and tier 0
   there means untiered legacy gear.
2. **Stats are not freely allocatable.** Each piece is primary 30 / secondary 25
   / tertiary 20, and the *archetype fixes primary and secondary*. A target
   spread therefore fails on one specific stat, not on the total — a target of
   555 fitted inside a 555 ceiling and was still unreachable, because Super
   capped at ~181 and forcing Weapons to 130 dragged it to 142. **Do not derive
   this by hand any more:** `d2_optimize` reports `unreachable` per stat with the
   exact ceiling from owned gear, which is that whole finding as a field.
3. **Get magnitudes before rejecting anything.** Two B-tier set bonuses were
   recommended over an A-tier at 4× the damage resistance, because the cost was
   known and the benefit wasn't. Check `d2_reference("set-bonuses")` first.
4. **Community write-ups contain wrong numbers.** Three of seven stat spreads in
   one Xûr write-up were wrong. Verify against `d2_xur` / `d2_inventory`.
5. **Names diverge in two different ways — check both before saying "not
   documented".** Transcripts *mangle* names ("Queensfoil Censer" → "Queen's
   foil sensor", "To Shreds" → "Two Shreds", "AION Adapter" → "Ion Adapter").
   Separately, the compendium indexes armour sets by **set-bonus name** while
   the API returns **armour item names**, and they can be entirely different
   words: `Twisting Echo` armour carries the `Yearning Echo` bonus. A miss on
   the armour name proves nothing — search the bonus name and the source
   activity too.
6. **Don't put AI where a lookup works.** Archetype decoding, set-bonus ranking
   and stat maths are deterministic. Judgement and parsing unstructured build
   content are where the model earns its place. **Set optimisation is the
   clearest case and it now has a tool** — `d2_optimize` searches all five slots
   server-side. Re-deriving a solver in the conversation is the exact failure
   this calibration names, and it gets re-derived differently every session.
7. **Enhanced perks carry different plug hashes than their base versions.** A
   DIM wishlist lists *base* hashes, so an offline hash-subset match silently
   fails on anything crafted — measured 0/743 overlap on every `Enhanced *`
   socket versus 60% on Origin Traits. Names are shared between base and
   enhanced, so **match wishlists by perk NAME, never by hash.**
8. **Never dedupe on name — compare perk signatures.** Modern exotics carry
   **selectable catalysts**, so two copies of one exotic can be different
   weapons: Choir of One runs *Onslaught Refit* (damage) on one copy and
   *Subsistence Refit* (ammo) on the other. Ergo Sum goes further with ten
   distinct frame/guard/blade configurations. This surfaced only after a shard
   list was shipped on the assumption "exotics have fixed perks" — while the
   compendium had already listed Praxic Blade's four slottable catalysts. **Diff the
   perk set; assume nothing from rarity.** Of nine duplicated exotic names,
   six are genuinely identical and three are not.
9. **Tier 5 grants the enhanced traits automatically.** So a low-tier copy of a
   well-graded weapon is a **farm target, not clutter** — re-acquiring it at
   tier 5 upgrades the perks for free. That makes "S/A weapons owned only below
   tier 5" the standing answer to *"what should I farm?"* for weapons, ahead of
   chasing new models. 33 such models as of 2026-08-18, Perfect Paradox (S#3,
   8 copies, none above t3) the loudest.
10. **The wishlist has real holes — check all variants before blaming the grade
   floor.** Four owned weapons (Jurisprudent, Duty Bound, Precipial, Shoreline
   Dissident) appear in **none** of `MRS_PPC2` / `MRA_PPC{1,2,3}` /
   `MRB_PPC{1,2}`. Widening `MRA_PPC2` → `MRB_PPC2` was right and lifted
   coverage to 29 of 33 stacks, but it could never have reached these. One grep
   for `^// <name>$` across every variant settles it in a second.
   **Blueberries is the designated fallback and is egress-blocked from cloud
   sessions** — ask the player to read it and paste the grade. A pasted tier grade is *not* a per-column god roll,
   so rank the owned copies on trait quality yourself and say that is what you
   did.
11. **Score per wishlist line, then take the best — never against the union.**
   A weapon with more acceptable rolls would otherwise score *lower*, because
   the union of every line's perks inflates the denominator. This silently
   produced zero matches after the `MRB_PPC2` switch.
12. **A column is a SET, not a plug — and this dissolved the vault cull.**
   Component 310 (live since `d2-mcp` `9b95543`, deployed 2026-08-17) exposes
   the plugs each column can be switched between; 305 only ever showed the one
   inserted. **203 of 295 legendaries carry three options per trait column.**
   Rescored on reachable perks, **39 redundant duplicates became 0** — no two
   copies of any weapon share the same option sets. Never call two copies
   duplicates on selected perks. The useful output inverted from *dismantle
   these* to **64 copies at t5/540+ already hold an unselected god roll**.
13. **Never substitute a perk-list reading for the published columns.** With
   Blueberries blocked I ranked four weapons' copies on "trait quality" and
   called it a judgement call. Against the actual published columns, **two of
   four verdicts reversed** — Precipial's One-Two Punch + **Destabilizing
   Rounds** copy and Shoreline Dissident's Triple Tap + **Precision
   Instrument** copy are both 2/2 PvE god rolls I had written off. Both errors
   were pessimistic about a low-power copy, and one hid the leap in a
   parenthetical ("no Destabilizing *on the 550*" — true of that copy, false of
   the stack). **When the grading source is unreachable, name the copies and
   ask for the columns. Do not rank and relabel.** Especially before anything
   destructive.

14. **`d2_apply` works, and a confirmed write is not immediately readable.**
   Live-tested 2026-08-18 (equip Mistral Lift, revert to Taipan). `result: ok`
   means Bungie *accepted* the write; the profile endpoint served the **old
   state for ~75 seconds** afterwards, with fresh fetches throughout. Reading
   back straight away shows the previous value and means nothing — it looks
   like a silent failure and invites a pointless second write. **If the player
   is at the game, do not poll on their behalf** — just say it can take a
   minute. Character power lags the same way.

15. **A page of the vault is not the vault — check `truncated`.** `d2_inventory`
   caps at `limit` (default 200) while `count`/`total` reports everything that
   matched. One session got a five-piece Warlock loadout to the point of
   recommending it over **200 of 423 pieces**, and the only thing that caught it
   was noticing the two numbers disagreed. The reply now carries `truncated` and
   `next_offset` and says so in a `note`, so the only way left to hit this is to
   ignore them. **Read `truncated` before reasoning over `items`.** A 47% sample
   does not give a worse answer, it gives a confident wrong one — nothing in the
   output looks off.

16. **Set optimisation belongs to `d2_optimize`, and its output fields are not
   decoration.** Do not pull the vault and solve it here: 400+ pieces of one
   class overflows the tool-result limit, so it becomes a file plus `jq` plus a
   hand-rolled search, rebuilt differently every session. Pass `priorities` in
   order and `targets` as hard floors; a stat with a target is capped at it for
   ranking, so once Melee clears 100 the next priority gets to matter. Then read:
   **`reachable` is three-valued** — `false` only where an exact ceiling proves
   it, `null` where the bounded search found nothing and *nothing was proven*.
   Never report `null` as impossible; say the search did not find one and offer
   to loosen a target. **`approximate`** true means a search bound bit, named in
   a note. **Totals are base gear only** — no mods, fragments or tuning, so a 90
   with a free mod slot is usually a 100 once `d2_apply` fits the mods. And
   `lock` the exotic the player has already chosen: the free-slot answer and the
   locked answer are different sets, and only the second one is any use to them.
   *(For contrast, DIM's optimizer silently clamps an impossible minimum instead
   of refusing it, which is why a DIM result can violate your own floor.)*

## This player's preferences are NOT in here

**Deliberate split (2026-08-18).** This file holds what is true for *any*
Destiny player: game mechanics, method, tool routing, failure modes. It used to
also carry one player's roll-priority ladder, play style and vault snapshot —
which meant preferences were applied without being re-confirmed, went stale
silently, and were actively wrong for anyone else. **A mechanical
skill can be shared verbatim; a personal one has to be forked and then drifts.**

So before scoring anything, read the player's own pages:

| Question | Where |
|---|---|
| How do they weight perk columns, origin traits, masterworks? | their notes, or **ask** |
| How do they play — range or close, survivability or damage, solo/duo/matchmade? | their notes, or **ask** |
| What is equipped, what is being farmed, what was decided? | their notes, or **ask** |
| Do duplicates matter? How many characters? | `d2_profile`, then **ask** |
| What do they own? | **`d2_inventory`, always** — never a remembered snapshot |

**And ask rather than assume.** A preference stated a week ago is a fact about
that week. Anything that reads as *"this player prefers…"* belongs in a conversation
or their own notes, never here.

## The three questions

**"Is this build good, can I run it?"** Parse to a spec → score components via
`d2_reference` → **armour via `d2_optimize`, not by hand** (lock the exotic the
spec names) → diff the rest against `d2_inventory` → report gaps, substitutes,
farm routes. **End with an applied build, not a shopping list:** offer the
`d2_apply` dry run — the optimiser's own `apply` list, plus the `mod` and
`loadout` actions for the mods and subclass the spec calls for. A DIM import URL
(`app.destinyitemmanager.com/loadouts?loadout=<urlencoded JSON>`) is still worth
giving alongside it, because it *saves* the loadout for re-equipping later, which
`d2_apply` does not.

**"Best build for this activity?"** **Establish the play mode first** (solo /
duo / matchmade — see above); it changes which perks even count. Then match the
activity profile against build properties. Rank by fit **and** ownability -
show both, they're different answers.

**"What should I farm?"** Missing items across S-tier builds, ranked by how many
builds each unlocks, filtered to what's obtainable now (Xûr, featured dungeon).

## Keep your own notes — and offer to start them

This skill is deliberately impersonal, so it will never remember a player. Their
builds, decisions, farm targets and preferences belong in their own files.

**On a first session, there will be nothing to read.** The pointer table above
resolves to nothing, and the failure mode is quiet: advice gets given on
defaults nobody chose, and the same questions get asked every week.

So when a preference surfaces in conversation — *"I play at range"*, *"traits
matter more than the barrel"*, *"I never use swords"* — **offer to write it
down**, and say where. A single `destiny.md` alongside their other notes is
enough; a whole wiki is not required. Ask before creating a file, then keep it
current as things change.

What is worth capturing, roughly in order of how often it is re-derived:

| | |
|---|---|
| **Roll priority** | which perk columns decide, and any always-keep exception |
| **Play style** | range or close, survivability or damage, which stats are dead |
| **Activity mix** | solo / duo / matchmade — the same build scores differently in each, and ally-facing perks are worthless solo and often best in duo |
| **Current state** | what is equipped, what is being farmed, what was decided and should not be re-litigated |
| **Characters** | how many, and whether duplicate copies are deliberate loadout convenience |

**Never write the vault into notes.** It goes stale the moment something drops,
and `d2_inventory` is live and cheap. Notes are for judgement; the API is for
facts.

**And treat what is written as preferences, not standing orders.** A preference
recorded last month is a fact about last month — re-confirm before leaning on it.
Worth keeping in your own notes, because this skill will not: which builds you
run and why, what you are farming, decisions you have already made and do not
want re-litigated, and a log of corrections so the same mistake is not made
twice.
