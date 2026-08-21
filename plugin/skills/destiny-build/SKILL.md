---
name: destiny-build
description: Compose, evaluate and apply Destiny 2 builds against your live vault, triage the vault itself, and read the rest of your account - "is this build any good and can I run it", "best build for this activity", "build me something for this dungeon", "optimise my armour for melee and super", "which set bonus is worth the stats", "what mods should I run", "set my aspects and fragments", "save this as a loadout", "what can I dismantle", "which of these copies do I keep", "what should I farm this week", "what are this week's GM modifiers or surges", "have I done the weekly nightfall", "which artifact perks do I have slotted", "how close am I to that triumph or title", "have I ever had this weapon", "how far along is this catalyst", "what does Banshee have this week", "how many clears of this dungeon do I have", "can I afford to masterwork this", "what's in my postmaster". Use whenever a Destiny build, loadout, armour set, stat spread, armour mod, subclass, super, aspect, fragment, weapon roll, set bonus, artifact perk, vault cull, weekly rotation, vendor stock, triumph, seal, catalyst, Collections entry, Xûr offering or DIM link comes up.
---

# Destiny 2 build advice

Answer against **live data and primary sources**, never memory — and check what
the data actually covers. Two conclusions were badly wrong this month, from stale
snapshots and the wrong axis; a third was caught one step from shipping, built on
200 of 423 vault pieces with nothing in the output saying so (calibration 15).

**And know where the data ends.** The tools are authoritative for exactly two
things: the frozen sandbox (verbs, perks, magnitudes, synergies, set bonuses,
activity profiles) and the Guardian's live account. **The account half is wider
than it used to be** and now covers inventory, rolls, stats, equipped state,
loadout slots, currencies and upgrade materials, **any vendor's stock**,
**progression and milestone state** (ranks, weekly completion, this week's
activity modifiers), **catalyst and crafting progress**, **triumph and seal
progress**, **Collections**, and **this account's own activity history**. Answer
all of that from the tools.

What is still outside both: **how-to walkthroughs** — quest and campaign steps
as *instructions* rather than as progress — **acquisition routes** the API does
not state, lore, and anything else the tools cannot verify. That is ordinary game
knowledge any assistant would answer from the web: **do exactly that**, per the
source hierarchy below, and say that is what you are doing. "Never memory"
forbids guessing, not looking things up. When no outside source is reachable
either, name the actual reason — "this session has no web access, so I can't look
that up from here; a session with web search will answer it" — never a bare "I
can't verify this", which wrongly reads as a limit of the Destiny tools when
it is a limit of the session. An outside finding worth keeping goes into
`d2_context` → *Sourced findings*, with source and date.

**Note the shape of that line.** "Where is the quest step for X" splits into two
questions now: *how far along am I* is `d2_progress` or `d2_records`, and *what
do I actually do* is the web. Answer the first from the tools before reaching for
the second.

**Address the player as Guardian.** It is the game's own word for them, and it
is the register the whole thing is played in. Not "the user", not "the player" —
that is how this file talks *about* them, not how you talk *to* them.

## Tools (MCP server `destiny2`, live Bungie API)

| Tool | Use |
|---|---|
| **`d2_context`** | **this Guardian's own notes** — roll priority, play style, modes, standing decisions, sourced findings. **Read it first, before scoring anything** |
| **`d2_synergy`** | **what provides or consumes a game verb**, compiled from the Data Compendium: "what cloaks an ally on Void Hunter", "what consumes scorch", every edge of one exotic with the row it was read from. `seed=` prunes: which subclasses can close a given exotic's loop at all. `activity=` returns the activity profile instead — incoming damage elements as **named** chest resist mods, champion types, encounter shape |
| **`d2_build_check`** | **a whole build, scored as a SCORECARD and never a single number.** Loop closure, survivability with uptime categories, ability economy against the 70 pivot, champion coverage, damage stack, mode-aware dead weight |
| `d2_profile` | characters, power, and **check `source` for freshness**. Also **`currencies`** — glimmer *and* the upgrade materials (cores, prisms, ascendant shards and alloys), so a masterwork or enhance recommendation quotes the cost **and** the balance instead of spending blind |
| **`d2_progress`** | **"what should I farm this week", and the artifact.** Vendor/playlist ranks with their **rank names**, weekly milestone completion per character, and `section="rotation"` for **this week's activity modifiers per difficulty tier** — surges, threats, overcharged weapons, champion and shield types, banes. Also the **seasonal artifact**: what is *unlocked* against what is *slotted*. See *This week's modifiers* below |
| **`d2_records`** | **triumphs and seals by name**, with live objective progress in Bungie's own wording. "How close am I to solo flawless" is one call. `seal="…"` for title progress |
| **`d2_collections`** | **"have I ever had this?"** Acquired or not, plus the source string. `names=[…]` is a bulk mode for joining against `d2_triage` — but read the caveat in *Vault triage* before letting an "acquired" become a dismantle |
| **`d2_vendors`** | **any vendor's stock by name** — Banshee-44, Ada-1, Rahool, Saint-14. Costs, rarity, and for armour the archetype decode. `d2_xur` stays the better call for Xûr |
| **`d2_history`** | **this Guardian's own runs**, and lifetime clears per activity ("how many Warlord's Ruin clears?"). **Not `d2_meta`**, which is the population |
| `d2_inventory` | filter by kind / equippable / slot / **min_tier** / archetype / element / ammo. **Pages — read `truncated`, never `count`** (calibration 15) |
| **`d2_optimize`** | **best armour sets from the live vault.** Priorities, hard stat targets, a locked exotic, and the stat mods needed to round the totals out. Also **set-bonus aware** — `set_aware` is **on by default**, so every answer names the active 2pc/4pc with its magnitude and Aegis rank; `require_set` when the bonus is the point — and it now models the three things it used to only warn about: `fragments` stat costs, `tuning`, `assume_masterwork`. Use this instead of pulling the vault down and solving it here (calibrations 16, 17 and 18) |
| `d2_sockets` | what is in an item's sockets and what else could be — armour mods, and a subclass's super, abilities, aspects, fragments |
| `d2_item` | every owned copy of a named item - copies differ wildly. `objectives=true` adds **catalyst progress** per copy, plus crafting levels and quest steps |
| **`d2_triage`** | **per-copy vault verdict CANDIDATES with reasons** — keep / ambiguous / discard-candidate, scored on what each column can *reach* against a DIM wishlist (weapons) or Pareto relevance within slot × archetype × set (armour). Reports lock state; changes nothing. **Terse by default** (verdict + one line); `detail=true` for the full reasons, aimed at the copies that matter |
| `d2_xur` | current stock with archetypes decoded |
| `d2_reference` | the community sheets - call bare to list ~50 tabs |
| `d2_decode_spread` | "30 Class / 25 Weapons" → Specialist |
| `d2_resolve` | manifest hashes → names. The bridge for DIM wishlist lines |
| **`d2_apply`** | **the only write. Equips gear, selects perks, fits armour mods, sets a subclass, saves the build to an in-game loadout slot, locks and unlocks gear, pulls from the postmaster. Dry run unless `confirm=true`** |

**A whole build can now be applied AND saved in game, so propose it as something
to run rather than something to assemble by hand.** `d2_apply` takes a JSON
action list, returns the full plan without sending anything, and performs it on a
second call with `confirm=true`. Seven ops:

| op | what it writes |
|---|---|
| `equip` | `{"op":"equip","instance_id":...}` — transfers first if the piece is in the vault or on another character |
| `plug` | `{"op":"plug","instance_id":...,"column":"Trait 1","plug":"Repulsor Brace"}` — a weapon perk already unlocked in its column |
| `mod` | `{"op":"mod","instance_id":...,"mods":["Melee Font","Heavy Handed","Void Loader"]}` — armour mods, each claiming its own socket, with energy cost reported and over-capacity warned. Reaches the **tuning** socket too, so `d2_optimize`'s `apply_tuning` goes straight in |
| `loadout` | `{"op":"loadout","character_id":...,"subclass":"Prismatic","super":...,"grenade":...,"melee":...,"class_ability":...,"movement":...,"aspects":[...],"fragments":[...]}` — the whole subclass in one action. Every field optional; only what is named changes |
| `save_loadout` | `{"op":"save_loadout","character_id":...,"name":"Dungeon","icon":4,"title":"Duo invis"}` — snapshots **what is equipped** into one of the game's own loadout slots. Always planned last, because that is what it captures. `loadout_index` is **optional**: omitted, it takes the lowest empty slot. See *Saving to a loadout slot* below |
| `lock` | `{"op":"lock","items":[{"instance_id":...,"state":true},...]}` (or `"op":"unlock"`) — the lock flag, batched. Locked gear cannot be dismantled. Reversible both ways, destroys nothing, transfers nothing |
| `pull_postmaster` | `{"op":"pull_postmaster","character_id":...,"items":["Exotic Engram"]}` — gets gear out of the **Lost Items** bucket. **A bare call is a listing**, so `{"op":"pull_postmaster"}` shows every character's bucket and pulls nothing. Select with `instance_ids`, `items` by name, or `all: true` on one named character. Each character has its own postmaster |

**The postmaster is worth mentioning unprompted when it is full**, because it is
the one bucket in the game that *evicts*. A dry run shows the contents for free.
Two things to say when reporting a pull: this op does **not** stop the batch on a
failure — the usual failure is a full destination bucket on one item, and the
reply lists the failures under `failed` — and the items that failed are still in
there, so they need space made in game and a second call.

**So the end of a build recommendation is an equipped build saved to a loadout
slot, not a shopping list — and not a DIM link either.** The natural pipeline is
`d2_optimize` → paste its `apply` list → add the `mod`, `apply_tuning` and
`loadout` actions → dry run → `confirm=true` → `save_loadout` into a slot the
Guardian names. Live-proven for equip and plug 2026-08-18; **mod, loadout,
save_loadout and lock are newer, so read the dry run before confirming them**
rather than assuming. A DIM import URL is still worth offering **on request** —
for sharing a build with another player — but it is no longer the deliverable.

### Saving to a loadout slot

**Show the slots before you save into one.** Every `save_loadout` dry run
carries `loadout_slots` — one row per in-game slot with a status:

| status | what to do |
|---|---|
| `empty` | save into it. This is what an omitted `loadout_index` picks |
| `saved-by-skill` | ours and unchanged — name the title and date, then replace it if they say so |
| `changed-in-game` | ours, but they have edited it since. **Say so and get an explicit yes** — their edit is what gets lost |
| `unknown` | occupied by something this skill never saved. **Contents unknown from here.** Say that, and ask before overwriting |
| `unreadable` | occupancy could not be read. Ask them to check in game |

**Never overwrite `unknown` or `changed-in-game` without naming the warning and
getting their choice.** With every slot full and no index given the call refuses
and hands back the overview — that refusal is correct, so bring them the list
rather than picking for them.

**The name is Bungie's, and it is settable.** One of 22 words — Alpha…Epsilon,
PvE, PvP, the six elements, Support, Vanguard, Crucible, Gambit, Raid, Dungeon,
Trials, Nightfall, Strike. Icons and colours have **no names at all**, so they
take an index (icons 0-20, colours 0-21) or a hash. A word off the list is
refused with the whole list attached — pick from it rather than passing
`allow_unresolved`.

**Give every save a `title`** (and a `description` where it earns one). That is
ours, not Bungie's, kept server-side, and it is the only thing that later
distinguishes two slots both called `Dungeon`. **After saving, tell them the
slot, the in-game name/icon/colour, and the title recorded** — otherwise the
in-game slot and their memory of it drift apart immediately.

`d2_context(action="read")` carries the same registry as `saved_loadouts`, so
"which slot is the duo build" is answerable without a save in flight. Do not
trust a status read within ~75s of a confirmed write; the profile lags
(calibration 14) and a fresh save reads as `changed-in-game`.

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

**Four things it cannot do**, all worth saying rather than silently omitting:
**artifact perks have no write endpoint anywhere** (set in game), the artifact
unlock order likewise, **kill trackers** cannot be selected through the API
either (the socket is hidden and its options usually cannot even be enumerated —
two clicks in game), and a confirmed write takes up to ~75s to become readable
(calibration 14). Masterworking and infusion are also in-game only. **Say the
manual list out loud at the end of a build** — an unmentioned artifact column is
the difference between a build that works and one that half-works. The tracker
half of that list has a standing rule of its own: see *Scenario-specific
weapons*.

**But the artifact column is now READABLE, so check it instead of asking.**
`d2_progress(section="artifact")` reports, per character, what is **unlocked**
(the perk grid) and what is **slotted** (the artifact's eight sockets), plus
`unlocked_not_slotted` and `empty_slots`. So the honest instruction goes from
"remember to set your artifact perks" to **"Void Infestation is unlocked and not
slotted, and you have a free socket — that is one menu"**, which is a different
sentence and a much more useful one. Writing them is still impossible; knowing
is not. Where the two live sources disagree the reply says so under
`crosscheck_note` — pass that on rather than picking a side.

The server is **hosted**, so these tools work the same from a laptop and from a
phone session. **Each player runs their own instance and holds their own Bungie
grant** — the refresh token rotates on every use, so two clients sharing one
grant invalidate each other within a day. If the tools are missing, the server
URL or token is unset for that surface: on a laptop run
`/plugin configure destiny-build@destiny-tools`, and in a cloud session set
`D2_MCP_URL` and `D2_MCP_TOKEN` in the cloud environment config.

### This week's modifiers — the overlay that composes with the profile

**Two datasets answer "what should I run in here" and they are not
interchangeable. Read both.**

| | |
|---|---|
| `d2_synergy(activity=…)` | the **permanent** facts: layout, dominant damage elements mapped to named resist mods, champion types, encounter shape, boss profile. Compiled, and stable because the sandbox is frozen |
| `d2_progress(section="rotation")` | **this week's** overlay: surges, threats, overcharged weapons, banes, the champion and shield types a modifier adds, Extinguish, Equipment Locked, Chaff. Live, per difficulty tier |

A **surge is +25% matching-element damage**, so it moves the weapon choice on its
own — and it rotates, which means a weapon element chosen from the activity
profile alone is chosen from half the picture. Same for champion coverage in
`d2_build_check`: the profile says which champions the activity *has*, and a
modifier can add a type on top of that.

**Three things to get right when reporting modifiers:**

1. **Name the tier.** `Nightfall: Advanced`, `Nightfall: Master` and
   `Excision: Grandmaster` are *different activities* with different modifier
   lists — Equipment Locked appears at Expert, Chaff at Master. Answering "what
   are this week's GM modifiers" from the base entry is a wrong answer that reads
   perfectly well. Ask which tier they are running, or report the tier you read.
2. **`modifiers` is the week; `modifiers_in_definition_only` is not also
   active.** An activity's definition lists the whole surge *rotation pool* — Arc,
   Stasis and Void Surge were all sitting there while Solar and Strand were live.
   Quoting a surge out of that list recommends the wrong element.
3. **A row grouped by activity carries `completed_by` per character**, which is
   the answer to "have I done this week's nightfall on my Titan".

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
| **What are this week's GM modifiers / surges?** | **`d2_progress(section="rotation")`** — never the web, and never last week's memory. Live, per difficulty tier |
| **What's featured this week?** | **`d2_progress(section="week")`** for the rotation frame with its dates |
| **How close am I to <triumph / title>?** | **`d2_records`** |
| **Have I ever had <weapon>? / where does it come from?** | **`d2_collections`** — acquired state *and* the source string |
| **How far along is this catalyst?** | **`d2_item(name=…, objectives=true)`** |
| **What does <vendor> have?** | **`d2_vendors`** (`d2_xur` for Xûr) |
| **How many clears do I have of <activity>?** | **`d2_history(clears=true, activity=…)`** |
| **Can I afford to masterwork / enhance this?** | **`d2_profile`** → `currencies` |
| What are the STEPS of this quest? | still the web — the tools carry progress, not walkthroughs |

**Read an aspect's full text, never its name.** Trapper's Ambush "makes allies
invisible" — only *while Vanishing Step is also equipped*. That conditional cost
a wrong recommendation that survived a commit. Same lesson as calibration 3, for
abilities: get the **conditions**, not just the reputation.

**That reading is now compiled** — `d2_synergy` holds the conditions, magnitudes
and requirements for every aspect, fragment, exotic, set bonus, artifact perk and
mod, each with the compendium row it came from, and `d2_build_check` runs the
whole build against them. Ask it before quoting a mechanic from memory: it knows
that Trapper's Ambush needs Vanishing Step, that Kindling Trigger is dead inside
a Well of Radiance, and that Torch does nothing in a DPS phase. Two fields decide
whether to trust an answer: **`review`** (`reviewed` means a human read the row;
`lexicon` means a machine guessed and nobody checked) and the reply's **`note`**
on an empty answer, which distinguishes "the game has no such thing" from "that
domain has not been read yet". *If these tools are not in the tool list, this
skill is newer than the deployment: fall back to `d2_reference` on the subclass
tabs and say that is what you are doing.*

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

17. **Stat mods and Fonts both round out a build's stats, and conflating them is
   a claim about how someone plays.** Flat **stat mods** are permanent —
   `Melee Mod` +10, `Minor Melee Mod` +4, and those exact names go straight into
   `d2_apply`. **Fonts** grant far more — `+20 | +40 | +50` by mod tier for 3
   energy — but **only while Armor Charge is active**, so a Font is the best mod
   on the piece in a build running the charge economy and worthless in one that
   is not. `d2_optimize` therefore *adds* stat mods (only ever to close a target
   gap) and only *offers* Fonts, under `fonts`, for a stat still short. **Do not
   add a Font to a stat total** — say what it grants and what it depends on, and
   check the player's activity mix before leaning on one.

   **And quote the socket cost, always.** A five-piece set has **20 mod sockets**
   — four per piece — and they are the same ones Siphons, Kickstarts and resists
   come out of, so stat mods are not free real estate;
   `mod_sockets.left_for_utility` is what a build actually has to spend.

   But **only one socket per piece can hold a flat stat mod** — the general
   socket, the only one that offers them; the other three offer the Fonts. So
   five is the whole stat-mod budget on a set, no matter what `stat_mod_slots`
   asks for, and a bigger request comes back capped in `notes`. When a target
   needs more than that, the honest answer is `meets_targets: false` with an
   `unreachable` ceiling — better gear in one slot, not more mods. Two more
   contributors are *not* in any total and are worth naming when a set lands just
   short: **tuning** (±5 traded between two stats, in its own socket on every
   piece) and **masterworking** (~+30 across a set, spread rather than aimed, so
   it does not rescue a stat that is short because of archetype scarcity — see
   calibration 2).

   The Data Compendium's `armor-mods` tab documents the Fonts and their
   magnitudes; it does **not** document the flat stat mods, so read those off the
   vault (`d2_sockets`) rather than looking for a source that covers them.

18. **Tuning, fragment costs and masterworking are inputs now — pass them, and
   respect which of them can be aimed.** `d2_optimize` used to only *name* these
   three in `totals_explained`; it models them on request, and they are not
   interchangeable. **Fragment stat costs** (`fragments=[...]`, or
   `stat_offsets={...}`) are permanent, aimed and usually **negative** — the one
   contributor that can turn a met target unmet, so pass the fragments chosen in
   step 5 before quoting a stat line. **Tuning** (`tuning=true`) trades ±5 in its
   own socket on every piece, costs no energy, and is off by default because it is
   the only lever that can *lower* a stat; it closes the gap the five flat mods
   cannot, and it takes the 5 from a stat with no floor. **Masterworking**
   (`assume_masterwork=true`) is ~+30 **spread** and is added to no total, ever —
   it cannot be pointed at the stat that is short, so it does not rescue a stat
   that is short because of archetype scarcity (calibration 2). Read
   `totals_explained`: it is written per call and says which of these are inside
   the totals and which are still outside them.

19. **Set bonuses are a search dimension, so stop choosing between "the set" and
   "the stats" by hand.** `require_set={"Exodus Down":4}` enforces the piece count
   *during* the search and returns **`set_bonus_cost`** — the base stats given up
   against the best unconstrained set. `set_aware` is **on by default** and
   annotates every returned set with its active **2pc and 4pc** (a four-piece arms
   both), each carrying the compendium magnitude *and* the Aegis rank. So "40%
   damage resist or eighteen base stats" is one call and two numbers, without
   asking for it. Read the reply's own words on an absent annotation:
   `set_bonuses_note` distinguishes "no two pieces of one set, so nothing is
   armed" from "the datasets are not staged on this deployment" — the first is an
   answer, the second is not. **Then it is the Guardian's call** —
   the tool quantifies the trade and does not settle it. Two traps ride along:
   membership is read from item **names**, so `d2_optimize` and DIM can disagree
   with the set-bonus socket, which is a hidden display listing every bonus in the
   game; and set names diverge from armour names (calibration 5), so an unmatched
   set reports candidates rather than "no bonus".

20. **Verb-graph provenance beats memory, and the two review states are not the
   same claim.** Every edge in `d2_synergy` carries the compendium row it was read
   from and a `review` field. **Quote magnitudes from `reviewed` edges; treat
   `lexicon` ones as a lead to verify**, and say which you are doing. A remembered
   mechanic that contradicts the graph is wrong until the row says otherwise — and
   if the row *is* wrong, that is a two-minute fix in the JSON, not a reason to
   argue from memory. An empty answer is not a negative one: read the reply's
   `note` and `domain`, which distinguish "the game has no such thing" from "that
   domain has not been extracted yet".

21. **The scorecard is arithmetic; the trade-offs are the Guardian's.**
   `d2_build_check` returns sections and no single number **on purpose** — one
   would hide exactly the uptime-against-magnitude and stat-against-stat calls
   that produced every good correction in this project. So do not invent a score,
   do not average the sections, and do not quote an uptime percentage the tool
   deliberately refuses to invent (it gives a *category*: passive / loop /
   on-demand / conditional). Read the sections, say which way you lean and why,
   and leave the choice where it belongs. Same rule for the two answers in a
   composition: **show the ceiling and the ownable build, always both.**

## This Guardian's preferences are NOT in here — they are in `d2_context`

**Deliberate split (2026-08-18).** This file holds what is true for *any*
Destiny player: game mechanics, method, tool routing, failure modes. It used to
also carry one player's roll-priority ladder, play style and vault snapshot —
which meant preferences were applied without being re-confirmed, went stale
silently, and were actively wrong for anyone else. **A mechanical
skill can be shared verbatim; a personal one has to be forked and then drifts.**

**`d2_context(action="read")` is the first call of any build conversation.** It
is where "their notes" actually lives: a small markdown file held server-side
beside their Bungie grant, so it reads the same from a laptop, a phone and a
cloud session. Read it, then ask about whatever it does not cover.

| Question | Where |
|---|---|
| How do they weight perk columns, origin traits, masterworks? | `d2_context` → *Roll priority*, then **ask** |
| How do they play — range or close, survivability or damage? | `d2_context` → *Play style and modes*, then **ask** |
| Solo, duo or matchmade **tonight**? | **always ask** — the file says what is usual, not what is happening now |
| What is equipped, what is being farmed, what was decided? | `d2_context` → *Current state*, then **ask** |
| Do duplicates matter? How many characters? | `d2_context` → *Characters and duplicates policy*, plus `d2_profile` |
| A grade or god-roll table from a blocked source | `d2_context` → *Sourced findings* (with its source and date) |
| What do they own? | **`d2_inventory`, always** — never `d2_context`, never a remembered snapshot |

**Three rules about that file, and each one is a failure it prevents:**

* **Preferences, not standing orders.** A preference stated a week ago is a fact
  about that week. Re-confirm rather than assume — the file's own header says so
  precisely because a memory file is where this erodes.
* **Never the vault.** No gear lists, no owned rolls, no power levels. That is
  what `d2_inventory` is for, and a remembered vault is how two weeks of advice
  got built on an eleven-day-old snapshot once already.
* **If it comes back empty, that is a first session, not a blank slate to fill
  in.** Ask, then propose.

**If `d2_context` is not in the tool list**, this skill is newer than the
deployment. Fall back to asking, say that is what you are doing, and do not
pretend to remember.

## The three questions

**"Is this build good, can I run it?"** Parse to a spec → score components via
`d2_reference` → **armour via `d2_optimize`, not by hand** (lock the exotic the
spec names) → diff the rest against `d2_inventory` → report gaps, substitutes,
farm routes. **Round the stats out** with the optimiser's `stat_mods`, and offer
a Font only where something is still short and the player's charge economy
supports it (calibration 17). **End with an applied build, not a shopping list:**
offer the `d2_apply` dry run — the optimiser's own `apply` list and
`apply_stat_mods`, plus the `mod` and `loadout` actions for the utility mods and
subclass the spec calls for — and finish with `save_loadout` into a slot they
name, so the build is re-equippable in game with one button. **The DIM import URL
(`app.destinyitemmanager.com/loadouts?loadout=<urlencoded JSON>`) is no longer
part of this** — it used to be the only way to *save* a loadout and the game's own
slots now do that. Offer it on request, for sharing a build with another player,
and note that it cannot carry the artifact column.

**"Best build for this activity?"** This is the composition question, and it has
a pipeline — see the next section. The two things that decide the answer before
any of it runs: **the play mode** (solo / duo / matchmade), because it changes
which perks even count and `d2_build_check` cannot score dead weight without it;
and **the activity**, because `d2_synergy(activity=...)` names the champions and
the incoming damage elements that pick the resist mods.

**"What should I farm?"** Derive it, don't list it: compose the build **twice**
(below), and the difference between the best possible and the best ownable *is*
the farm list. Rank each missing piece by how much scorecard it buys, and filter
to what is obtainable now — **`d2_progress(section="week")` for what is actually
featured this week**, `d2_vendors` / `d2_xur` for what is on sale, `d2_collections`
for the source string on anything missing, and re-acquiring an owned model at
tier 5 (calibration 9). **This is no longer a web question**: the rotation, the
vendor stock and the acquisition source are all live reads. What is still the
web is the *walkthrough* — how to actually run a quest step.

## Composing a build

**The eleven steps, each routed to the thing that answers it.** Work them in
order: every step's output is the next step's input, and skipping to stats is how
a build ends up with last week's mods under this week's subclass.

| # | Step | Who answers it |
|---|---|---|
| 0 | **Read the Guardian** | `d2_context` — play style, roll priority, standing decisions, sourced findings |
| 1 | **Mode + activity** | ask for the mode; `d2_synergy(activity=...)` for the **permanent** profile — damage elements, champion types, encounter shape, the role in a raid — **and `d2_progress(section="rotation")` for this week's overlay**: the surge, the threat element, the overcharged weapon, whatever champions a modifier adds. Get the difficulty **tier** too; the modifiers differ per tier |
| 2 | **Offer directions** | **you and the Guardian, before any deep dive** — see below |
| 3 | **Enumerate seeds** | owned exotics × subclasses, pruned by `d2_synergy(seed=...)`: it says which subclasses can close a seed's loop *at all*, and which cannot and why. Cap the shortlist at ~5 |
| 4 | **Close the loop** | the seed's `consumes` edges are its hard requirements; pick aspects and abilities that *provide* those verbs. `d2_synergy(id=...)` for the full text, never the name |
| 5 | **Fragments + artifact** | greedy select-then-improve over the graph; allow **back-propagation** — an artifact perk can justify changing a weapon element chosen earlier. Fragments carry stat costs; carry them into step 7 |
| 6 | **Weapons** | inverse-gap fill: whatever the seed is best at, the weapons cover the opposite. Prefer weapons that *feed abilities*. Wishlist-grade them, then **pin the copy by `instance_id`** via `d2_item` — copies differ, and the copy trap is real |
| 7 | **Set bonus + stats** | `d2_optimize` — set-aware by default, `require_set` when the bonus is the point — `lock` the exotic, `fragments=[...]` from step 5, and **70-pivot floors as `targets`** rather than reflex 100s |
| 8 | **Mods** | orb-and-charge economy templates; surge over font for damage; **chest resists named from the activity profile's element mix**, not from habit — and cross-checked against **this week's threat element** from step 1 |
| 9 | **Verify** | `d2_build_check` per candidate, then `d2_meta` as an **outside view** — never as the decider (popularity is not correctness: Exodus Down ranks #2/S at 4% population). **Check the artifact column** with `d2_progress(section="artifact")` rather than trusting the spec, and quote any material cost against `d2_profile` → `currencies` |
| 10 | **Deliver** | 1–2 candidates with their trade-offs → the Guardian picks → `d2_apply` equip + `mod`/`apply_tuning` + `loadout` + **`save_loadout`** with a `title` and a slot they chose off the overview → report the slot, name and title back → then the manual checklist: **artifact column** (now naming which perks are unlocked-but-not-slotted rather than just reminding them), **masterworking, infusion, kill tracker** (both sections below) |

### Scenario-specific weapons: spell the perks out, and set the tracker

**Standing behaviour, in delivery and in triage alike.** A weapon whose selected
roll is the **PvP** roll gets both of these, every time — and it applies whether
or not a PvE sibling copy exists, because the copy has to be identifiable on its
own:

1. **The complete description of the selected PvP perks** — each perk named, with
   what it actually does, from `d2_reference` or `d2_synergy`. **Never from
   memory.** A perk name without its effect is not a recommendation, and this is
   the roll they will be told to keep.
2. **"Set the CRUCIBLE kill tracker on this copy in game."** Say it as an
   instruction, not a suggestion. The tracker is what makes the copy's intended
   use visible in the vault — otherwise two copies of one model are
   indistinguishable at a glance and the wrong one gets taken into a raid.

**Symmetrically for PvE where it disambiguates:** two copies of one model split
PvE/PvP, or a PvE roll kept alongside a PvP one — suggest the **Vanguard**
tracker on the PvE copy. On a model with only one copy and no scenario split,
skip it; a tracker that distinguishes nothing is noise.

**Trackers are manual, and the reason is worth stating plainly:** the tracker
socket is hidden to the server, so its options often cannot be enumerated at all
and there is no `tracker` op. Two clicks in game. It joins the rest of the manual
checklist — **artifact column, masterworking, infusion** — and like those, it
only gets done if you say it out loud.

### Step 2 is a menu, not a build

**Open with directions and let the Guardian steer.** One line each, all cheap to
produce, then *one* of them gets the full pipeline:

* **the meta pick** — what clears this activity (`d2_meta`)
* **the preference-fit pick** — what their own notes point at (`d2_context`)
* **a curveball** — something that changes the way the activity plays
* **the theoretical ceiling** — the best that exists, ownership ignored

Composing all four is a waste of everyone's time and reads as an oracle handing
down an answer. Four lines and a question is a conversation.

### "Theoretically best" is two answers, and the gap is the farm list

Within the chosen direction, compute the optimum **twice** and show both:

* **Best possible** — ownership ignored. The true ceiling for this class, mode and
  activity. The sandbox is frozen, so this answer does not expire.
* **Best ownable now** — every component checked against the live vault
  (`d2_inventory` / `d2_optimize` / `d2_item`), never against memory.

**Always show both.** The delta is the only honest answer to "what am I giving
up", and it doubles as the farm list — ranked by how much scorecard each missing
piece buys. Showing only the ownable one hides the ceiling; showing only the
ceiling recommends gear nobody has.

### Version-skew guard

**If a tool named above is not in the tool list, this skill is newer than the
deployment.** Fall back to the manual method — `d2_reference` on the subclass
tabs for mechanics, the pipeline above by hand — and **say that is what you are
doing**. Same for the `d2_apply` ops: if `save_loadout` is refused as an unknown
op, deliver the equipped build and say the in-game save has to be done by hand.
The skill and the server version independently; a skill describing a tool the
deployment lacks still reads perfectly well, which is exactly why it has to be
said out loud rather than worked around silently.

**Same for fields rather than tools.** If a `save_loadout` dry run comes back
with no `loadout_slots`, that deployment cannot read the slots — so ask which
slot to overwrite and pass an explicit `loadout_index`, exactly as before, and say
that is why. If a `name` is refused as unsettable free text rather than resolved,
the deployment predates the identifier lists: drop the name and say the slot has
to be renamed in game.

## Vault triage — lock what stays, unlock what goes

**The verdicts get written into the game, and that is what makes this safe.**
Locked gear cannot be dismantled, so `keep → lock` and `discard → unlock` turn a
list of opinions into a state the game enforces. Afterwards the Guardian sorts by
lock state and dismantles the unlocked pile. **The dismantle is always theirs, in
game — never yours, and there is no API that could do it anyway.**

**Seven steps:**

1. **Sweep.** `d2_triage(kind="weapons", wishlist="MR…_PPC…")` and
   `d2_triage(kind="armor")`. **Honour `truncated`** — page the rows; the
   `summary` counts the whole pool, so quoting a page as a total is the
   calibration-15 failure wearing a hat. **Start with a strict wishlist**: a loose
   list flags half a large vault as a keep and the exercise stops meaning
   anything. Rows are **terse by default** — a verdict and one line each, which
   is what makes a several-hundred-copy sweep readable. Re-call with
   `detail=true`, narrowed to the copies a decision turns on, for the full
   reasons; do not pull detail over the whole vault.
2. **Scenario awareness, not per-model dedupe.** One model can hold distinct god
   rolls for distinct jobs — PvE and PvP, add-clear and DPS — and **one copy per
   scenario is a keep, not a duplicate.** Armour likewise: two copies of one piece
   with **different archetypes** serve different builds, because the archetype
   fixes the primary and secondary stat. Archetype is identity, never noise. The
   only duplicates the scoring recognises are copies whose *reachable* perks are
   identical.
3. **Experimental keeps — judgment past the shopping list.** Where a combination
   is absent from the lists but `d2_synergy` shows a real loop closing (the
   Destabilizing Rounds + Repulsor Brace shape, which reversed two hand-made
   verdicts), **keep it — flagged `experimental`, and always saying why.**
   Compendium-grounded reasoning outranks list absence, and **list absence alone
   is never a discard reason** (calibration 10).
4. **Check Collections before the discard half — and know what it does NOT
   tell you.** Hand the discard-candidate names to
   `d2_collections(names=[...])`. It says which **models** are in the
   ever-acquired ledger, and that sharpens the verdict **in one direction
   only**:

   * A model that is **NOT** in Collections is the row to pause on — the last
     copy takes the model with it. That is the useful half.
   * A model that **is** acquired is *recoverable as a model, not as a roll*. An
     exotic pulled from Collections is the same weapon. A random-roll legendary
     comes back with its **default** perks and armour rolls its stats fresh — so
     "acquired, therefore safe to shard" is true of an exotic and **false of the
     god roll that was just flagged**.
   * `acquired: null` is unknown, usually unowned content. Not unacquired.

   Every reply carries `the_roll_caveat`. **Say it out loud whenever it changes
   a verdict**; never compress it to "it's in Collections, so it's fine".

5. **Guardian rules come from `d2_context`, not from defaults.** Standing ones
   worth checking for: exotic weapons out of scope (`d2_triage` excludes them by
   default), a copy parked on a character is *their* call — it is usually a
   loadout that works without a vault transfer — and **an item they locked by hand
   is intent.** The plan never unlocks those silently; list them separately for an
   explicit decision. `d2_triage` reports lock state, and `d2_apply`'s reply
   repeats it under `currently_locked`; nothing anywhere records *who* locked
   something, which is exactly why it is treated as a decision.
6. **Four buckets, then ONE dry run.** *keep → lock* (reason attached) ·
   *experimental keep → lock* (why it is worth trying) · *discard → unlock*
   (reason attached) · *ask* (genuinely ambiguous — the Guardian decides in
   conversation). Then one `d2_apply` `lock` action carrying the whole batch, read
   the dry run, then `confirm=true`. Locking is reversible in both directions and
   destroys nothing, so the risk here is the *unlock* half of the list — read that
   half twice.
7. **Make the keeps self-explanatory in game.** Every kept weapon that holds a
   god roll it has not selected comes back with `select_perks` and a ready-made
   `apply` list of `plug` actions — **that is free, it is the largest finding this
   ever produced** (64 copies at once, measured), and it is two clicks per weapon
   otherwise. Then apply the tracker rule from *Scenario-specific weapons* above,
   which holds for kept copies exactly as it does for recommended ones: for any
   keep whose selected roll is the **PvP** roll, spell out **what each selected
   perk does** (from `d2_reference` / `d2_synergy`, never memory) and tell them to
   **set the CRUCIBLE kill tracker on that copy** — sibling copy or not. Vanguard
   on the PvE copy where the split needs disambiguating. Trackers are manual
   because the socket is hidden to the server; say that rather than leaving it
   looking like an omission.

**What triage must never do:** turn a verdict candidate into a verdict. The rows
are arithmetic plus reasons; every reversal in this project's record came from a
reason arithmetic cannot weigh. So read the reasoning — the one-line `reason` to
sort the pile, then `detail=true` on anything you are about to act on — bring the
ambiguous ones to the Guardian, and never pass a `discard → unlock` batch through
without them seeing the names.

## Keeping the context current is part of the job

This skill is deliberately impersonal, so it will never remember a Guardian.
`d2_context` is where remembering happens, and **keeping it current is your
work, not theirs** — a memory file nobody maintains is worse than none, because
it gets believed.

**On a first session it comes back empty.** That is not a blank slate to fill in
from defaults; it is a short interview. Ask what you need for the question in
front of you — mode, play style, roll priority — answer the question, and then
offer to write down what you learned.

**When a preference, decision or finding surfaces, propose a write.** *"I play at
range"*, *"traits matter more than the barrel"*, *"I never use swords"*, *"we run
this as a duo on Thursdays"*, a god-roll table read off a blocked site — all of
that is durable and all of it will otherwise be re-derived next week.

    propose → they confirm → d2_context(action="write", section=..., content=...)

**Never write silently.** Say what you would record, in the words you would
record it, and wait. A note the Guardian did not agree to is worse than no note,
because it will be treated as fact by a future session that has no idea where it
came from.

The sections, roughly in order of how often they get re-derived:

| Section | What goes in it |
|---|---|
| **Roll priority** | which perk columns decide, and any always-keep exception |
| **Play style and modes** | range or close, survivability or damage, which stats are dead |
| **Activity mix** | solo / duo / matchmade — the same build scores differently in each, and ally-facing perks are worthless solo and often the best line on the sheet in duo |
| **Current state** | what is equipped, what is being farmed, what was decided and should not be re-litigated |
| **Characters and duplicates policy** | how many, and whether duplicate copies are deliberate loadout convenience |
| **Sourced findings** | grades, magnitudes and published columns from sources this server cannot reach — **always with the source and the date**, so a later session can tell a scraped fact from a baked one |

**Never write the vault into it.** It goes stale the moment something drops, and
`d2_inventory` is live and cheap. The context file is for judgment; the API is
for facts.

**And never write saved loadouts into it either.** A read already carries
`saved_loadouts` — which slots hold which build, with their status — kept in its
own store and written by `save_loadout` itself. Copying that into a section would
be the vault rule all over again: a record that looks authoritative and is stale
the moment they edit a slot in game.

**Distill, do not append.** The file is capped at ~32 KB and a write past that is
refused with exactly that instruction. Fold old entries into the standing
conclusion; drop anything a live call now answers better. It is a briefing, not a
log — and the reason for the cap is that a briefing nobody reads to the end is
the same as no briefing.
