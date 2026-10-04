# PF2e Spell Smoothing

## Purpose and features

Spell Smoothing improves several PF2e spell flows and tracks poisons, diseases, and curses as staged effects. It adds a target-aware Counteract flow and an optional way for the caster to roll against target save DCs. Spell changes appear on chat cards; afflictions appear as effects and chat cards.

## Setup

Foundry VTT 13 with PF2e 7.12.2 or newer 7.x, or Foundry VTT 14 with PF2e 8.x. The manifest sets Foundry minimum 13 and was verified with PF2e 8.5.1 on Foundry 14.

This is a free module. Install it with the [SpazzMods Installer](https://github.com/Spazzletopia-Studios/spazzmods-installer/releases/latest), or use the [public GitHub release](https://github.com/Spazzletopia-Studios/pf2e-spell-smoothing/releases/latest). Enable **PF2e Spell Smoothing** in Manage Modules.

## Quick start

1. Enable the module. Most spell changes work on the normal PF2e spell cards.
2. Cast a supported spell and use its new card buttons.
3. For the affliction tracker, have a GM open the world once and wait for the first index build to finish.
4. Use an exposure or stage card to roll a save, apply an affliction, treat it, or identify it.
5. Review **Configure Settings** before play, especially affliction visibility and the optional caster-roll setting.

## Detailed use

### Force Barrage

The card has **1 shard**, **2 shards**, and **3 shards** damage buttons. Each button combines shards aimed at one creature into one roll. Use a separate one-action roll for each target when splitting shards. The number of shards per action increases with spell rank.

**Optional house rule: charge reserves** (off by default):

1. In the module's settings, turn on **Force Barrage charge reserves**.
2. Cast Force Barrage from a fixed prepared, spontaneous, or flexible spellcasting entry. The **Force Barrage · Rank N** picker offers **1 action · N shards**, **2 actions · N shards**, and **3 actions · N shards**. Choose the actions to use now, or choose **Cancel**.
3. The spell slot is consumed when you confirm the cast. The actions you did not use become reserve charges for one minute. If you use all three actions, no reserve is left.
4. The **Force Barrage Reserve** row appears below that spell. It shows the rank, remaining charges, and time left. Click its **Cast** button, then choose how many actions to spend. A reserve cast uses no second spell slot.
5. The reserve disappears when empty or expired. Each fixed prepared slot keeps its own reserve. Spontaneous and flexible slots share a display only at the same rank; their charges keep separate expiry times and the oldest are spent first.

This charge rule is a house rule. It does not require PF2e Wand & Staff Casting.

### Other spell-card changes

- **Shield:** casting applies or refreshes PF2e's Shield effect on the caster at the cast rank.
- **Loose Time's Arrow:** targets selected at cast time receive Quickened until the end of the caster's next turn. If a target was missed, target it and click **Apply Quickened to Targets** on the card. Reapplying refreshes the effect. More than six targets prompts a warning.
- **Fear:** the target's save applies frightened 1 on success, frightened 2 on failure, or frightened 3 and fleeing for one round on a critical failure. A reroll replaces the prior result. If the rolling player cannot update the target, the GM receives the result to apply.
- **Acid Arrow:** after normal damage is applied, the exact damaged creature receives persistent acid damage for the cast rank. A critical hit does not double the persistent damage. Reverting that damage removes only the condition tied to that damage message.
- **Counteract spells:** target the creature with the effect and click **Counteract** on the spell card. Choose an eligible effect or condition. Review its rank and suggested DC; a GM can edit the DC. Confirm the roll. The counteract rank ladder determines whether the effect is removed. Candidates the spell cannot affect are shown disabled with a reason. If the target is no longer selected, the flow tries the spell's saved target, then a selected token, then the caster.
- **Cleanse Affliction:** choose an affliction. Its base effect reduces a case one stage if it is past stage 1, once per case. The counteract roll is available at rank 3+ for poison or disease and rank 4+ for curse. Below that rank the base reduction still applies. A successful counteract ends the affliction; failure does not undo the base stage reduction.
- **Something not listed…:** use this row for an effect that exists only in a stat block. Enter its rank and DC and roll. The chat result is advice for the GM; it does not delete or change anything automatically.
- **Clear Mind:** the picker includes its listed conditions and can use recent spell cards to suggest a rank and DC. A near miss can show the spell's suppression rider.

### Afflictions: first build

The tracker ships no compendium-derived affliction list. On the first GM login for a PF2e version, it reads the world's installed PF2e compendiums, parses affliction names, DCs, saves, onset, duration, stages, damage, and conditions, then caches the result in Foundry user data for that system version. A progress bar appears. Players use the built index; they do not build it. Homebrew and third-party entries can be parsed from their item descriptions.

After a PF2e update, the next GM login builds a new version cache. If players log in before a GM has built it, the browser can be empty until a GM connects and completes the build. Existing tracked effects still work. A GM can force a rebuild with `game.pf2eSpellSmoothing.afflictions.rebuild()`.

### Apply an affliction and resolve its saves

1. To apply from the **Affliction Browser**, target the affected creature, open `game.pf2eSpellSmoothing.afflictions.browser()`, search and select an entry, then choose **Apply (initial save)** or **Apply at Stage 1**. The first choice follows the initial-save setting; the second is a direct stage-1 application. **Custom…** creates a stageless item, hazard, or curse record; enter its name, kind, level, and DC, then click **Apply to Targets**.
2. A detected exposure normally posts a card with the save DC and **Roll Initial Save**. Roll the target's save. **Apply Anyway (GM)** applies the affliction without the initial save. The GM can set **Initial save handling** to automatic or manual only instead.
3. A matching save rolled from an affliction description can be detected when **Detect rolled affliction saves** is on. Check the card or chat result before applying it. Detection is intended to stop a second manual application of the same save.
4. Once applied, the creature gets a real effect with an onset or stage badge. Conditions and stage damage are applied to that creature, not to whichever token is selected. In combat, stage saves roll automatically at the end of the afflicted creature's turn by default. Set **Stage saves in combat** to card mode to let a player click **Roll Save**.
5. When a time jump passes pending saves, the default is to ask. Use the pending-save card's **Process** button to resolve them, or choose automatic blind rolls in settings.
6. The stage card offers **+1 Stage (GM)**, **−1 Stage (GM)**, and **Remove (GM)** for GM corrections. Removing the effect also removes conditions it granted. Stage damage already applied to HP is not rolled back when the effect is removed.

Onset, stage intervals, and maximum duration can advance with world time outside combat. A missing interval on a rounds-scale poison is treated as one round; slower entries without a printed interval need GM-led saves.

### Treat, identify, and coat

- To treat, control the healer or assign a character, target the patient where the PF2e action requires it, then click **Treat Poison** or **Treat Disease** on the stage card. The healer rolls Medicine against the affliction DC. Treat Poison is one action; Treat Disease takes eight hours under its normal rule. Critical success gives +4, success +2, and critical failure −2 circumstance to the patient's next save. The result appears as a visible effect and is removed after that save. Player requests route through the active GM.
- For an unidentified case, select or assign the identifying character and click **Identify**. Choose a Recall Knowledge skill in the dialog and click **Roll**. Poison and disease offer Medicine, Nature, Arcana, Occultism, and Religion; curses offer Occultism, Religion, Arcana, and Nature. A success identifies the case. A GM can use **Reveal (GM)** to identify without a roll. The optional Party Bestiary integration also shares already-revealed knowledge.
- To coat a weapon, use **Coat Weapon** on an injury-poison item card. Choose a weapon you own and click **Coat**. One dose is spent. The next successful Strike exposes its target and consumes the coating; if it does not hit within ten minutes, the coating expires.

### Optional caster-rolled saves

The world setting **Caster rolls against target save DCs** is off by default. When enabled, the save button on new spell cards becomes a caster roll for each target. A critical success by the caster maps to a target critical failure, and so on. The card reports the save result and the damage multiplier; roll and apply basic-save damage with the matching damage button.

**What the caster rolls** defaults to automatic: spell attack for PCs and spell DC minus 10 for NPCs. Cover is folded into the DC by default, but only for Reflex saves against area spells. The card labels the cover adjustment. `game.pf2eSpellSmoothing.coverProbe()` reports cover between the selected token and a target.

This changes who rolls. The target's degree adjustments, such as Evasion, do not apply automatically; the caster's attack-roll bonuses can affect the check. Some modules that listen for saving-throw outcomes will not see one. PF2e Toolbelt Target Helper also uses the save button; choose one system for that flow. **Also invert spells cast by the GM** is on by default.

## Settings

Important defaults:

| Setting | Default | Use |
|---|---|---|
| Force Barrage charge reserves | off | Optional one-minute house rule described above |
| Caster rolls against target save DCs | off | Changes who rolls for save spells |
| Also invert spells cast by the GM | on | Applies caster rolls to GM spell cards too |
| What the caster rolls | auto | PC spell attack; NPC spell DC − 10 |
| Fold cover into the target's save DC | on | Applies cover to Reflex area spells |
| Resolve mirror images automatically | on | Announces redirect and image loss; never applies damage |
| Track poisons and diseases as effects | on | Master switch for poison, disease, and curse tracking |
| Detect afflictions on Strike damage / rolled saves | on | Posts or resolves exposure from qualifying actions |
| Initial save handling | card | Card with a roll button; can be automatic or manual only |
| Stage saves in combat | automatic | Can instead post a card with **Roll Save** |
| Track affliction time outside combat | on | Advances onset, intervals, and maximum duration |
| After a time jump | ask | Ask before processing missed saves |
| Affliction cards visible to | GM only | Can be changed to everyone |
| New poison doses advance the stage | on | Applies the multiple-exposure rule |
| Apply afflictions unidentified | on | Hides identity from players until identified |
| Composite affliction icons | on | Writes small combined icons to world user data |
| Cleanse Affliction's base effect at stage 1 | as written | The house-rule option lets the base effect cure stage 1 |

## Limits and recovery

Affliction data depends on the PF2e compendiums installed in this world. The first GM build after a system update can take a minute or two. Curse coverage depends on parseable entries. Some afflictions have computed damage or prose-only stages; those display their text but cannot be automated. Unlinked tokens on inactive scenes are processed when their scene becomes active. The icon setting can be turned off for read-only hosting; without it, plain icons are used.

Affliction cards are whispered to GMs by default because disease details can be secret. A GM can make them public in settings. Players' requests pass through the active GM, so players do not need permission to change a monster; the GM client performs and validates the writes. Applying an unidentified case hides its identity from players, not its stage or damage. Use campaign permissions and table practice for any secrets.

The stage-1 Cleanse behavior follows the printed rule by default: its base effect does nothing to a stage-1 case. Turning on the alternative is a house rule. Likewise, Force Barrage reserves are a house rule. In read-only hosting, turn off icon composition to avoid file writes. A successful Strike consumes a weapon coating; a miss does not. Removing an effect does not undo damage already applied.

## API and development

The API `game.pf2eSpellSmoothing.afflictions` includes `apply`, `browser`, `list`, `find`, `getData`, `rollSave`, stage changes, `cleanse`, `treat`, `coatWeapon`, counteract helpers, `rebuild`, and `status`. Hooks report applied, changed, removed, and identified afflictions.

The source gate is `npm test` in `harness/`. Other useful source checks are `node harness/build-afflictions.mjs --check` and `node harness/test-runtime-build.mjs`. These development checks do not replace an installed Foundry smoke.

Source data note: `data/afflictions.json` quotes Paizo stat-block prose. It is a development artifact and is excluded from the release package; Paizo's Community Use Policy covers free projects only. `data/overrides.json` ships and is merged into each build. Affliction backgrounds come from the project SDXL artwork pipeline; `bg-poison-ingested.webp` was hand-picked from a reroll, so a full regeneration will differ.

## Credits and license

Module code: MIT License. Paizo rules text and marks are subject to Paizo's Community Use Policy and other applicable terms. The unshipped development data is not additional user content.

## Get help

[SpazzMods Support](https://github.com/Spazzletopia-Studios/spazzmods-support).
