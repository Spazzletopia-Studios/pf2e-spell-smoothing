# PF2e Spell Smoothing

Fixes the spells that play clunky in Foundry — Force Barrage, Shield, Fear,
and more — plus turns poisons, diseases, and curses into real tracked
effects instead of plain text. Built for the Pathfinder Second Edition
system.

## Install

**The easy way (Windows):** download the [SpazzMods Installer](https://github.com/Spazzletopia-Studios/spazzmods-installer/releases/latest),
run it, and click Install on PF2e Spell Smoothing. No account needed.

**Without the installer:** paste this into Foundry's **Install Module →
Manifest URL** box:
`https://github.com/Spazzletopia-Studios/pf2e-spell-smoothing/releases/latest/download/module.json`

## Using it

- Install it and it works — nothing to open, nothing to configure. Cast one
  of the fixed spells (Force Barrage, Loose Time's Arrow, Shield, Fear) and
  the chat card already has the right buttons.
- Any spell the system marks as a counteract spell (Dispel Magic, Cleanse
  Affliction, Quench, and more) gets a real **Counteract** button: target
  the creature, click, and pick which effect to counteract from a list —
  no more guessing DCs by hand.
- Poisons, diseases, and curses become real effects with stages, timers, and
  automatic saves — the first time a GM loads the world, it reads your own
  pf2e compendia to learn every affliction (a one-time build, a minute or
  two, shown with a progress bar; the world stays usable while it runs).
- The **Affliction Browser** (`game.pf2eSpellSmoothing.afflictions.browser()`,
  best wired to a macro) lets you search, target, and apply any known
  affliction by hand.
- An optional setting flips who rolls save spells: turn on **Caster rolls
  against target save DCs** and the caster rolls once per target instead of
  every target rolling their own save — good for tables that want fewer
  dice rolled by NPCs.
- The optional **Force Barrage charge reserves** setting consumes the spell
  slot at the first cast, then keeps any unused actions as charges for 1
  minute. The reserve appears directly below the normal spell row and casts
  without using another slot.
- **Acid Arrow** now applies its persistent acid damage when its normal damage
  is applied. The exact receiving creature gets a real PF2e persistent-damage
  condition, with the correct cast-rank dice and no critical-hit doubling.
  PF2e's Revert Damage action removes that exact condition too.
- Everything above is on by default and needs no setup; the settings tables
  below let you turn any of it off or tune it per table.

---

Per-spell quality-of-life smoothing for the Pathfinder 2e system. Each entry in
`scripts/smoothings.js` fixes one spell whose table handling is clunkier in
Foundry than at a real table.

## Entries

### Force Barrage

The system ships Force Barrage as a flat `1d4+1` — the chat card's Roll Damage
button rolls one shard no matter how many actions were spent. This module
replaces that button with one button per action count:

> ◆ 1 shard · ◆◆ 2 shards · ◆◆◆ 3 shards

Each button rolls every shard as **one combined damage roll** (3 shards →
`3d4+3` force), which is the RAW handling for shards aimed at the same target
(combine before applying bonuses, resistances, and weaknesses). Shard counts
scale with the cast rank per the spell's Heightened (+2) text — a rank 3 cast
shows 2/4/6 shards.

Shards split between different targets: click the 1-action button once per
target (each click is its own roll).

The roll goes through the system's own `SpellPF2e#rollDamage` on a clone with a
multiplied formula, so the damage card, apply buttons, and slug-keyed module
integrations (e.g. pf2e-graphics animations) behave exactly like a native roll.

#### Optional charge reserves

Turn on **Force Barrage charge reserves** in the module settings to use the
house rule. Before the normal cast, pick 1, 2, or 3 actions. The spell slot is
consumed immediately. Any actions you did not use become charges for 1 minute;
the compact reserve row casts them later without spending another slot.

- A fixed prepared slot owns its own reserve. Two prepared Force Barrage slots
  never merge.
- Spontaneous and flexible casters see one combined reserve per spell rank.
  Each consumed slot still keeps its own timer, and the oldest charges are
  spent first.
- Different ranks never merge. Empty or expired reserves disappear.
- The cast card shows only the chosen action total, plus the single-shard
  split-target button at ranks where one action fires more than one shard.

This feature copies no code and has no runtime dependency on Wand & Staff
Casting. It uses the same proven pattern: keyed charge packets on the actor,
PF2e's public slot-consumption method, and native spell chat cards.

### Acid Arrow

PF2e stores Acid Arrow's ongoing acid only in the spell description. When the
spell's normal damage is applied, Spell Smoothing reads PF2e's own
`damage-taken` message to identify the exact receiving creature and gives it a
real persistent acid condition. Rank 2 is 1d6, rank 4 is 2d6, and so on. A
critical hit still doubles only the initial damage, never this condition.
Reverting the applied damage removes only the persistent condition created by
that damage message; other persistent acid damage on the creature is untouched.

### Loose Time's Arrow

The system ships no spell-effect item for this spell — the card just links the
Quickened condition, leaving the GM to hand-apply it to up to 6 targets with no
duration tracking. This module adds an **Apply Quickened to Targets** button to
the card: target the tokens, click, and every target gets a module-built
"Spell Effect: Loose Time's Arrow" (shape copied from the system's Effect:
Haste — `GrantItem` of Quickened with `inMemoryOnly`), lasting 1 round with
turn-end expiry anchored to the caster ("until the end of your next turn").
Re-applying refreshes the duration instead of stacking; more than 6 targets
warns but applies. When the effect expires, the granted Quickened goes with it.

Casting with targets already selected applies the effect automatically — the
card button stays as the fix-up path for targets missed at cast time.

### Shield

Casting Shield now applies the system's own **Spell Effect: Shield** to the
caster automatically — no dragging from the card. The effect's level is set to
the cast rank, so its hardness rule (`5*ceil(@item.level/2)`) heightens
correctly. Re-casting refreshes the effect instead of stacking a second copy.

### Fear

Fear's frightened ladder applies itself. When the target's Will save resolves
— rolled by the target from the card, or by the caster under the "caster
rolls" toggle — the module applies the outcome: success → frightened 1,
failure → frightened 2, critical failure → frightened 3 plus fleeing for 1
round (a 1-round effect, so it ends on its own). Frightened takes the higher
value and never stacks with one already there. When the resolving client can't
write to the target (a player's caster against a monster), a chat note tells
the GM what to apply. Saves rolled from pf2e-toolbelt's Target Helper rows are
detected too — their results land in the card's flags rather than in a
message, and the module reads those the same way the affliction tracker does.

Rerolls re-resolve rather than stack: a hero-point reroll (or a Target Helper
reroll) replaces the first outcome — frightened is set to the new rung, and a
fleeing effect from a crit fail is withdrawn if the reroll beats it. One
caveat: a reroll that improves the save will also lower a frightened that came
from somewhere else in the meantime. When the client resolving the save can't
write to the target (a player's caster against a monster), the outcome is
relayed to the GM's client through a whispered message and applied there.

### Counteract spells (Dispel Magic, Cleanse Affliction, Quench, ...)

Applies to **every** spell the system marks with `counteraction` (58 in pf2e
8.4) — no per-spell setup. The system's Counteract button rolls with **no DC
and no target** — it prints the counteract rank ladder as advisory text and
leaves everything else to the table. This module replaces that button with a
real flow:

1. Target the creature carrying the effect, click **Counteract**.
2. A picker lists every magical effect and condition on the target (anything
   `fromSpell` or carrying a magic tradition trait, skipping granted children).
3. Each candidate shows its rank and a resolved DC — the origin spell's caster
   DC when the effect knows where it came from, the origin actor's best spell
   DC otherwise, or the GM Core level-based DC for its rank as a fallback. The
   DC field stays editable for GM overrides.
4. The roll uses the system's own counteract statistic vs that DC, and the
   result is adjudicated with the counteract ladder (crit success: rank +3,
   success: +1, failure: only lower ranks). A beaten effect is deleted
   automatically and the outcome is posted to chat.

The dialog names who it is counteracting on. If the crosshair target is gone
by the time you click, it falls back to whoever you had targeted **when the
spell was cast** (every cast remembers its targets), then your selected token,
then the caster. Things the spell cannot affect still appear, greyed out and
labelled ("not affected by Dispel Magic"), so an empty list always explains
itself.

**Clear Mind** lists the conditions it can counteract — fleeing, frightened,
stupefied at rank 2; confused, controlled, slowed at 4; doomed at 6; stunned
at 8 — including bare conditions with no spell effect behind them. For those,
the picker looks back through recent chat for a spell cast aimed at that
creature that imposes the condition, and pre-fills its rank and the DC from
that cast ("condition · from Fear · rank 1 · DC 24"). A near miss (would have
beaten an effect 2 ranks lower) posts the spell's suppression rider instead
of a flat failure.

The last row, **Something not listed…**, is the manual escape hatch for things
that only exist in a stat block: type a rank and DC, roll, and the chat verdict
tells the GM what to apply by hand — nothing auto-removes from it, since it is
not linked to any effect. Rank and DC stay editable for every pick.

Candidates carry a **kind** — `magic`, `poison`, `disease`, or `curse` — and
the registry can narrow a spell's scope: **Dispel Magic** lists only magic;
**Cleanse Affliction** lists only afflictions and runs both halves of the
spell in one click: the **base effect** (drop the picked affliction one
stage — once per case, only if it is past stage 1, no roll) always applies,
and the **counteract roll** is made only when the cast rank clears the gate
(poisons and diseases at rank 3+, curses at 4+ — under-rank picks show
"needs rank N+" and get the base effect alone). A won counteract ends the
affliction; a lost one still leaves the stage reduction in place. Tracked
afflictions (below) resolve their own counteract DC (= the affliction's save
DC) and rank (= half its level, rounded up), and a beaten one is removed with
a full condition sweep.

## Affliction tracker

Poisons, diseases, and curses in the pf2e system are plain text — no effect,
no stages, no timers, and nothing for counteract to target. This module turns
them into real, tracked effects:

- **Every affliction in your own packs, read on first load.** The module ships
  no affliction data. The first time a GM opens a world with the tracker on,
  their client reads the pf2e compendia THEY have installed — every alchemical
  poison, monster venom, disease, and curse — and parses DC, save, onset,
  maximum duration, and each stage's damage, conditions, and interval out of
  the stat blocks. On pf2e 8.4.1 that is 515 afflictions out of 71 compendia
  and takes a minute or two behind a progress bar — the world is usable the
  whole time, nothing waits on it. The result is cached as
  a file in your Foundry user data
  (`Data/pf2e-spell-smoothing/affliction-cache/`) and reused until the pf2e
  system version changes, so it is built once, not once per session. Players
  read the same cached file. Anything the packs don't have — homebrew items,
  third-party bestiaries — is parsed live from its description with the same
  parser.
- **A real effect item** lands on the afflicted creature: poison/disease
  traits (so poison-immune creatures block it automatically), the affliction's
  level, and a stage counter badge (`Onset`, `Stage 1`…`Stage N`).
- **Stages do what they say**: entering a stage applies its conditions
  (stamped `grantedBy`, so the picker skips them and removal cleans them up),
  starts persistent damage, and rolls the stage damage and **applies it to the
  afflicted creature directly** through the system's IWR pipeline — no apply
  button, so it can never land on whatever token happens to be selected. The
  stage card shows the rolled damage with its type ("4 poison (1d6) —
  applied"). Frightened, sickened, drained, doomed, stunned, and unconscious
  persist beyond their stage, per RAW.
- **Combat runs itself**: at the end of the afflicted creature's turn, when
  the stage interval is over, the save is rolled publicly, the stage moves,
  and the new stage's conditions and damage apply — zero clicks. A setting
  switches this to a card with a Roll Save button if your table prefers
  players rolling their own.
- **Saves run the rules**: initial save (crit fail starts at stage 2), stage
  saves (crit success −2 / success −1 / failure +1 / crit failure +2),
  **virulent** (two consecutive successes to improve; crit counts as one),
  **multiple exposures** (a new poison dose forces a save; failure drives the
  stage deeper), and **incapacitation** (over-levelled targets improve their
  degree one step, called out on the card).
- **Time is tracked in and out of combat**: onset periods, per-stage
  intervals (save prompted at the end of the afflicted creature's turn), and
  maximum duration. Jump the world clock 8 hours and the tracker either asks
  before blind-rolling the missed saves or rolls them automatically — your
  choice in settings.
- **Auto-detection**: a Strike whose attack effects carry a poison posts an
  exposure card against the struck target. Rolling the save from an
  affliction's description link (the NPC ability card's DC button — the flow
  the card's targeting UI drives) **applies the outcome immediately**: a
  failed fresh save afflicts the roller, and a roll by a creature already
  afflicted counts as its stage save. Works with **pf2e-toolbelt's Target
  Helper** too — its card-row saves never post a chat message (the result
  lands in the card's flags), and the tracker reads those directly. Every
  affliction item's chat card also gets an **Afflict Target(s)** button.
- **Treat Poison / Treat Disease**: use the system's own Medicine action with
  the patient targeted — the tracker finds that kind of affliction on them,
  judges the roll against its DC, and stores the result. (The stage card's
  Treat button does the same from the other direction: select the healer's
  token, click, and their Medicine is rolled against the affliction's DC (with `action:treat-poison` / `action:treat-disease` roll
  options, so feats that key off those apply): crit success +4, success +2,
  crit failure −2 circumstance to the patient's **next** save. The treatment
  is a **visible effect** on the patient (Medicine icon, "+2" in the name)
  carrying the real modifier, and it disappears when that save is rolled.
- **Poisoned weapons**: an injury poison's card gets a **Coat Weapon** button
  — pick one of your weapons, one dose is spent, and the next successful
  Strike with it exposes the target (initial save card) and cleans the blade.
- **Unidentified afflictions** (setting, on by default): new cases show to
  players as **"Unknown poison"** (or disease/curse) — generic icon, a
  description that says only what kind it is, but the real stage badge and
  the real damage. The GM sees everything. The stage card's **Identify**
  button rolls Recall Knowledge (Medicine for poisons and diseases, a magic
  skill for curses) against the affliction's DC; **Reveal (GM)** skips the
  roll. Success swaps in the real name, composite icon, and stat block, and
  fires `pf2eSpellSmoothing.afflictionIdentified`. With **PF2e Party
  Bestiary** installed (optional), a case starts identified when the party's
  bestiary already has that creature's affliction revealed, and revealing it
  later identifies open cases on the spot.
- **The effect states its interval**: stage labels read "Stage 2 · 1 round"
  (or "Onset · 1 minute"), so the effect itself says when the next save comes.
- **Affliction Browser**: `game.pf2eSpellSmoothing.afflictions.browser()` —
  search everything the build found, target tokens, apply (with or without the
  initial save).
  Wrap it in a macro for one-click access. Its **Custom…** button makes a
  stageless affliction — a curse from an item, ritual, or hazard — from a
  name, kind, level, and DC, so Cleanse Affliction (4th+), Break Curse, and
  the counteract picker can target it.
- **Icons that tell you what and who**: each effect gets a background by kind
  (injury / ingested / inhaled / contact poison, disease, curse — eight
  bespoke icons in `icons/`) with the **source creature's token art overlaid**
  in a small circle. Two monsters' venoms share the background and differ by
  face — which is also what lets pf2e draw them as two separate token icons
  (it merges status icons with the same image). Composites are written once
  as small `.webp` files to the world's user data
  (`pf2e-spell-smoothing/affliction-icons/`) and reused; anything that can't
  be composited (remote token art, read-only hosting) falls back to the plain
  background. A setting turns compositing off entirely.

A stage with no printed interval on a rounds-scale poison is treated as
1 round (stat-block convention); on slower afflictions it simply has no timer
and the GM drives the save from the card's buttons. Affliction cards are
GM-whispered by default (disease details are often secret) — a setting makes
them public.

Players' clicks route through the active GM client (native socket, no
libraries), so a player rolling their own save never needs permission over
the monster that poisoned them. Deleting the effect item by hand sweeps the
conditions it granted.

### Affliction settings

| Setting | Default | |
|---|---|---|
| Track poisons and diseases as effects | on | Master toggle for the tracker |
| Detect afflictions on Strike damage | on | Exposure card when a poisoned Strike hits |
| Detect rolled affliction saves | on | A rolled save applies its outcome immediately |
| Initial save handling | card | Card with a button / roll automatically / manual only |
| Stage saves in combat | auto | Roll + advance at turn end, or a card with a button |
| Track affliction time outside combat | on | Onset, intervals, max duration on world time |
| After a time jump | ask | Ask before blind-rolling missed saves, or auto |
| Affliction cards visible to | GM only | Public option for open tables |
| New poison doses advance the stage | on | The multiple-exposures rule |
| Apply afflictions unidentified | on | Players see "Unknown poison" until identified (roll, GM reveal, or bestiary) |
| Composite affliction icons | on | Background by kind + source token overlay; off = plain backgrounds, no files written |
| Cleanse Affliction's base effect at stage 1 | as written | RAW: a stage-1 case is untouched by the base effect; house rule: it is cured |

### API

`game.pf2eSpellSmoothing.afflictions` — `apply(itemOrEntry, actors, opts)`,
`browser()`, `list()`, `find(slug)`, `getData(effect)`, `rollSave(effect)`,
`advanceStage/reduceStage(effect, n)`, `cleanse(effect, caster?)` (once-per-case
base Cleanse), `treat(effect, treater?)`, `coatWeapon(poisonItem, weapon)`,
`remove(effect, reason)`, `counteractDC/counteractRank(effect)`,
`rebuild()` (GM: force a fresh read of the packs), `status()` (where the
current data came from: `cache`, `built`, `shipped`, or `none`).
Hooks: `pf2eSpellSmoothing.afflictionApplied / afflictionChanged /
afflictionRemoved / afflictionIdentified`.

### Where the affliction data comes from

Nothing is bundled. On the GM's client:

1. read the cache file for the installed pf2e version
   (`Data/pf2e-spell-smoothing/affliction-cache/afflictions-pf2e-<version>.json`);
2. if it is not there, read the packs, parse them, and write that file;
3. players fetch the same file — they never build. A player who logs in before
   any GM has built it gets one console warning and an empty index (tracked
   afflictions on their sheet still work), and picks the data up on its own
   when a GM connects or finishes a build.

A pf2e update changes the cache key, so the next GM login rebuilds by itself.
`game.pf2eSpellSmoothing.afflictions.rebuild()` forces it. Foundry gives
modules no way to delete files, so the cache file for an old pf2e version just
sits there (~1 MB) until you remove it by hand.

### Rebuilding the offline copy (development)

```
node harness/build-afflictions.mjs          # rebuild the dev copy data/afflictions.json + review report
node harness/build-afflictions.mjs --check  # CI-style drift check
node harness/test-runtime-build.mjs         # runtime builder == offline build, byte for byte
node harness/afflive.mjs                    # live world: build, cache, apply (Playwright, logs in as "Claude 1")
```

`data/afflictions.json` is a **development artifact and is not distributed** —
it quotes Paizo's stat-block prose, and the Community Use Policy covers free
projects only. `harness/deploy.mjs` keeps it out of the release zip; the
harness diffs against it, and the module falls back to it only if a working
copy happens to have one. `scripts/afflictions/parse-core.js` is the parser
pipeline both the harness and the in-world builder run, which is what
`test-runtime-build.mjs` checks: it replays the real packs through the module's
own `build.js` under a stubbed Foundry and requires identical bytes.

Background icons come from the project SDXL pipeline (`Skill Trees/harness/gen-icons.py` with `harness/icon-prompts.json` → `icons/bg-*.webp`). Note: `bg-poison-ingested.webp` was hand-picked from a re-roll under a temporary key (the key-derived seed for the real key renders a pale ground), so a full regeneration of that one file will differ — keep the shipped file.

The build report (`harness/fixtures/afflictions-report.md`) lists every parse
warning and same-name variant (e.g. the DC 14 and DC 17 Giant Centipede
Venoms — both are kept; the picker and browser show the canonical one,
variants carry their sources). Hand corrections live in `data/overrides.json`,
which DOES ship and is merged into every build, offline and in-world alike.

### Known limitations (afflictions)

- The first load in a world after a pf2e update spends a minute or two reading
  the packs on the GM's client (a progress bar says so). Players who connect
  during that window see an empty affliction index until it finishes; it fills
  in on its own when the GM's build lands.
- Curse coverage is thin (20) — curses mostly live in rituals and relics, not
  parseable stat blocks. Add hand-written entries via `data/overrides.json`.
- Afflicted **unlinked** tokens parked on non-active scenes are picked up for
  world-time processing when their scene next activates.
- Poisoned-weapon application (coating a blade with a vial) has no system
  hook yet — use the item card's Afflict Target(s) button on the hit.
- A handful of afflictions have computed damage formulas or prose-only stages
  ("sleep normally"); those stages show their text and automate nothing.

## Caster rolls against save DCs (optional, off by default)

A world setting that inverts save spells: instead of every target rolling a
save against the caster's spell DC, the **caster rolls once per target against
that target's save DC** — the same shape as Grapple being an Athletics check vs
the target's Fortitude DC. It keeps the dice with the person taking the action
and lets the GM roll defence for the party instead of five players rolling
saves every round.

With it on, a save spell's card hides the save button and offers
**Roll vs Reflex DC** (or Fortitude/Will). Target the creatures, click once, and
the module rolls for each in turn, then posts one summary translating every
result back into save language:

```
Fireball — Reflex saves, rolled by the caster
  Goblin Warrior     critical failure    double damage
  Orc Brute          failure             full damage
  Sarah              success             half damage
  Ancient Wyrm       no Reflex save — resolve by hand
Cover folded into the DC: Goblin Warrior +2 (standard).
Basic save: roll damage on the card, then apply with the matching button.
```

The caster's roll is the exact mirror of the save it replaces: the caster's
critical success is the target's critical failure, and so on. Damage stays
manual — the summary names the multiplier to click.

### Settings

| Setting | Default | |
|---|---|---|
| Caster rolls against target save DCs | off | The master toggle |
| Also invert spells cast by the GM | on | Off keeps players rolling their own saves against monsters |
| What the caster rolls | auto | See below |
| Fold cover into the target's save DC | on | |

**What the caster rolls** matters more than it looks. A creature's spell attack
modifier and its spell DC are separate numbers: for player characters they come
from one statistic and agree, but bestiary stat blocks author them
independently ("spell attack +18, DC 30"). `auto` uses the attack modifier for
PCs and spell DC − 10 for NPCs, so enemy spells land exactly as written. In
`attack` mode the module logs a one-time warning per caster when the two
numbers diverge by 2 or more.

### Cover

Cover is computed by this module — no dependency on any cover module. It casts
a line from caster to target: a wall gives standard cover, a creature in the
way gives lesser, and being boxed in from every angle gives greater. A
deliberately applied **Take Cover** effect is honoured too, whichever is
stronger.

Per the rules, cover raises the DC only for **Reflex saves against area
spells** — never Fortitude or Will, and never a single-target Reflex spell.
Lesser cover is +1 to AC and contributes nothing to a save. When cover applies,
the roll card shows the original DC struck through next to the raised one with
a labelled tooltip, and the summary names it outright.

`game.pf2eSpellSmoothing.coverProbe()` reports the cover between your selected
token and your target.

### Known limitations

- **Degree-of-success adjustments swap sides.** They belong to whoever rolls,
  and that is now the caster. So the target's own adjustments (Evasion and the
  like) do not apply — the summary flags any target that has them so you can
  adjust by hand — while the caster's adjustments on attack rolls now shift a
  result that is really the target's save. The incapacitation trait is the
  exception and comes out correct on its own.
- **True Strike and other attack-roll bonuses now apply to save spells.** The
  caster's roll is a spell attack roll as far as the system is concerned; the
  attack domains cannot be removed without losing target-DC resolution.
- `pf2e-modifiers-matter` highlights the caster's spell DC modifiers rather
  than the target's save modifiers. Cosmetic only.
- `pf2e-assistant` automations triggered by saving-throw outcomes stop firing,
  since no saving throw is rolled.
- If pf2e-toolbelt's Target Helper is enabled, both it and this feature drive
  the save button; the module warns the GM once at startup. Pick one.

## Adding an entry

Add a config to `SMOOTHINGS` in `scripts/smoothings.js` keyed by the spell's
system slug. See the comment block there for the `repeat` smoothing contract.
Note: `unitsPerAction` must encode the spell's heightening text itself — these
spells carry no `system.heightening` data, that's why they need smoothing.

## Install

Copy this folder to `Data/modules/pf2e-spell-smoothing` (or install the
release zip), then enable it in the world's Manage Modules. Requires the
`pf2e` system (≥ 8.0.0), Foundry 12–14. No library dependencies.
