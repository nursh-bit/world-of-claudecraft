# ROADMAP — World of Claudecraft

---

## Product Snapshot

- **Product name:** World of Claudecraft
- **Current version:** 0.1.0
- **Version confidence:** High — `package.json` declares `0.1.0`; single git tag `trailer-v01`
- **Product purpose:** A browser-playable micro-MMO and headless Gymnasium-compatible RL training environment, built on a deterministic TypeScript simulation core. Currently uses WoW-Classic mechanics; roadmap is pivoting to Pathfinder 2e mechanics with a fresh original world setting ("The Shattered Reaches").
- **Core users:** (1) Players who want a free, self-hostable D&D/PF2e-flavored MMO; (2) AI/RL researchers who want a fast, scriptable game environment with a well-documented rule system
- **Current major capabilities:** 9 classes, 3 zones, 3 dungeons, ~60 quests (WoW mechanics); multiplayer server with Postgres persistence; PBR graphics with CC0 assets; headless RL env with Gymnasium Python wrapper; admin dashboard; Docker Compose deploy
- **Current architecture summary:** TypeScript monorepo. `src/sim/` is the deterministic sim core. `src/render/` is Three.js. `server/` is Node.js/WebSocket. `headless/` is the RL env. `python/` is the Gymnasium wrapper. `tests/` is vitest (9 files, 131+ tests). Postgres persistence.
- **Current stability assessment:** Good for v0.1. Three active PRs in flight: security fix (PR #7, live exploit), delta snapshots (PR #8), spatial grid (PR #9). These must merge before v0.2 branch work begins.
- **Current documentation assessment:** Strong README (WoW-focused — will be rewritten in v0.3). Design docs in `docs/design/` cover the WoW expansion spec (archived). Feature design for PF2e conversion completed and documented.
- **Current testing assessment:** Solid coverage for WoW mechanics. All WoW sim tests will be replaced (not deleted) with PF2e equivalents during v0.2. Engine-level tests (party, trade, duel, social, spatial, snapshots) pass unchanged.
- **Current roadmap status:** Updated — previous WoW-focused v0.2–v0.5 plans archived; PF2e conversion plan now active.

---

## Roadmap Planning Model

- **Current major version:** v0.1.x
- **Fully planned major versions:** v0.2.x, v0.3.x
- **75% planned major version:** v0.4.x
- **50% planned major version:** v0.5.x
- **Idea backlog major versions:** v1.0.x, v1.1.x
- **Archive policy:** Completed and superseded minor versions move to the Archive section. Nothing is deleted.

---

## Active Roadmap Summary

- **Current line:** v0.1.x — security patch + performance (PRs #7, #8, #9)
- **Next fully planned line:** v0.2.x — PF2e engine foundation + class/ancestry system
- **Second fully planned line:** v0.3.x — The Shattered Reaches world content (3 zones, 3 dungeons)
- **75% planned line:** v0.4.x — social/community features + RL env update
- **50% planned line:** v0.5.x — platform and hosting maturity
- **Idea backlog:** v1.0.x (stable release milestone), v1.1.x (content + PvP)
- **Current decision:** PRs #7 → #8 → #9 are verified locally (141/141 on the combined tree) and ready to merge, but merging is blocked: the available GitHub credentials are read-only and `main` requires an approving review. A maintainer must approve + merge in order (#7, #8, #9). After that: commit CHANGELOG + version bump 0.1.2, open `feature/pf2e-engine`, begin v0.2.0 with `src/sim/types.ts`.

---

## Current Major Version: v0.1.x

**Planning level:** Current and fully planned

### v0.1.0 — Initial Open-Source Release

- **Status:** Delivered
- **Theme:** Open-source launch — WoW-Classic micro-MMO + RL training environment
- **Delivered:** 9 classes, 3 zones, 3 dungeons, ~60 quests, multiplayer server, Postgres persistence, Docker deploy, PBR graphics overhaul with CC0 assets, Gymnasium RL wrapper, admin dashboard, MIT license
- **Known issues at release:** Trade item-duplication exploit; server crash on malformed message; no CI; performance O(n²) in broadcastSnapshots and mobsInRadius
- **References:** Initial commit `3387f8b`; MIT license PR #6 `6974f85`; tag `trailer-v01`

---

### v0.1.1 — Security Hardening

- **Status:** Verified, awaiting merge — PR #7 tested locally (131/131, `tsc` clean) on 2026-06-12; merge blocked because the available GitHub account has read-only access and `main` requires an approving review from a maintainer
- **Theme:** Fix the live trade exploit and harden message dispatch
- **Scope:**
  - Aggregate per-item slot validation in `tradeSetOffer` and `tradeConfirm` (duplicate-slot exploit)
  - Input shape validation for trade slots (null/NaN/Infinity rejection)
  - Wrap message dispatch in try/catch — malformed payload drops one client, not the server
- **Release criteria:** All 131+ vitest tests pass; `tsc --noEmit` clean; `social_e2e.mjs` 10/10
- **Risk:** Low. Server-side only, backward-compatible, covered by three new test cases.
- **Note:** Merge before any other work. This is a live security issue.

---

### v0.1.2 — Performance Pass 1

- **Status:** Verified, awaiting merge — full integration (main + #7 + #8 + #9) tested locally on 2026-06-12: 141/141 tests, `tsc` clean, on branch `integration/v0.1.2-verify`; merge blocked on maintainer access (see v0.1.1)
- **Theme:** Eliminate O(n²) hot paths in the server's snapshot and AI loops
- **Scope:**
  - PR #8: Entity JSON cached once per tick; heavy self fields delta-compressed (−65% payload, −35% CPU at 10 idle clients)
  - PR #9: Spatial hash grid replaces all linear entity scans; O(entities) combat-flag pass replaces O(players × entities) scan (−14% tick time, −15% CPU at 30 clients)
- **Dependencies:** v0.1.1 must merge first (game.ts conflict)
- **Release criteria:** 138/138 vitest tests; `mp_integration.mjs` 26/26, `social_e2e.mjs` 10/10, `crypt_raid.mjs` 8/8; no tick-time regression
- **Risk:** Low-Medium. Manual JSON string-building in `selfWireJson` is a maintainability concern to watch.
- **Note:** After v0.1.2 merges, open `feature/pf2e-engine` from main.

---

## Next Major Version: v0.2.x

**Planning level:** Fully planned

### Version Theme
PF2e Engine Foundation. Replace the WoW-Classic mechanical layer — entity model, combat resolution, resource system, class data — with Pathfinder 2e equivalents. World content is *not* in scope for v0.2; a placeholder test zone is sufficient. Every PR lands on `feature/pf2e-engine`; no merge to `main` until v0.2 is complete and all engine-level tests pass.

### Goals
- `src/sim/` speaks PF2e: d20 roll-to-hit, 4-degree success, action pool accrual, conditions, spell slots, focus points
- All 11 MMO-simplified PF2e classes playable to level 5 in the test zone
- All 6 ancestries implemented with correct HP bonuses and ability boosts
- All engine-level (non-WoW) tests pass unchanged: party, trade, duel, dungeon instances, spatial grid, delta snapshots
- All WoW sim tests replaced with PF2e equivalents (same discipline, new formulas)
- CI pipeline gates every PR on `feature/pf2e-engine`

### Breaking changes and compatibility notes
- **Postgres character schema changes.** New columns (`ancestry`, `pf2e_class`, `ability_scores`, `spell_slots`, `focus_points`, `conditions` as JSONB). Lazy migration: characters missing new fields are initialized to class defaults on first load rather than crashing. A migration script (`migrations/pf2e_character_migration.sql`) must be idempotent and reversible.
- **WebSocket snapshot format changes.** New self fields (`actionPool`, `conditions`, `spellSlots`, `focusPoints`, `abilityScores`). Added additively — existing fields not renamed where possible. `ClientWorld.applySnapshot` updated in v0.2.2.
- **WoW content removed.** All WoW zone data, mob templates, quest definitions, item definitions, and class data are removed on this branch. The WoW versions are archived here in ROADMAP.md and preserved in git history.

### Architecture direction
- `src/sim/types.ts`: remove `resource`/`maxResource`/`resourceType`, `cooldowns`, `attackPower`, `critChance`, `dodgeChance`, `gcdRemaining`, `comboPoints`, `autoAttack`, `queuedOnSwing`, `eating`, `drinking`, `overpowerUntil`. Add `abilityScores`, `proficiencyRanks`, `actionPool`, `actionAccrual`, `reactionAvailable`, `spellSlots`, `focusPoints`, `conditions`, `classDC`, `keyAbility`.
- `src/sim/sim.ts`: replace `resolveAttack()` (WoW hit table) with `resolveStrike()` + `degreeOfSuccess()`; replace GCD/swing-timer tick logic with action pool accrual; add `tickConditions()`.
- New `src/sim/content/ancestries.ts` alongside the existing `classes.ts` replacement.
- `src/sim/obs.ts`: PF2e obs vector deferred to v0.4.x; keep a stub that compiles.

### Minor Version Plan

#### v0.2.0 — Entity Model + d20 Combat

- **Theme:** Replace the WoW mechanical foundation with PF2e primitives
- **Features / Technical work:**
  - Entity model update: add all PF2e fields; remove all WoW-specific fields (`src/sim/types.ts`, `src/sim/entity.ts`)
  - d20 combat resolver: `resolveStrike(attacker, defender, weapon)` → `degreeOfSuccess(total, dc, rawRoll)` → `'critSuccess' | 'success' | 'failure' | 'critFailure'`
  - Action pool accrual in tick loop: 1 action per 2000ms (40 ticks), caps at 3; ability dispatch checks pool cost before executing
  - Reaction timer: 1 reaction per ~6s round; Fighter's Attack of Opportunity auto-triggers on enemy movement through threatened square
  - Condition system (Phase 1 minimal set): frightened (N), slowed (N), prone, grabbed — each modifies the correct roll/stat; `tickConditions()` decrements per-round
  - TypeScript `strict: true` enabled at this step (all new code must be strict-clean; legacy files patched incrementally)
- **Tests:** T-001 (`d20_strike_four_degrees`), T-002 (`action_pool_accrual`), T-003 (`ability_cost_deduction`), T-004 (`reaction_aoo`), T-005 (`condition_frightened`); all party/trade/duel/social tests pass unchanged
- **Release criteria:** `vitest run` green; `tsc --noEmit` clean; tick duration ≤0.6ms/tick headless bench; action pool accrues and caps correctly

#### v0.2.1 — Class + Ancestry System

- **Theme:** All 11 PF2e classes and 6 ancestries playable to level 5
- **Features / Technical work:**
  - New `src/sim/content/ancestries.ts`: Human, Elf, Dwarf, Goblin, Gnome, Halfling — each with base HP, size, two free ability boosts, one ancestry feat (simplified to a fixed passive bonus for the MMO context)
  - Replace `src/sim/content/classes.ts` with PF2e class definitions: Fighter, Rogue, Wizard, Cleric, Druid, Ranger, Bard, Champion, Monk, Barbarian, Sorcerer — each with `keyAbility`, `hpPerLevel`, `proficiencyGrants`, `spellSlotProgression`, `focusPoints`, `abilities[]` (3–5 abilities at launch per class)
  - Spell slots: prepared casters (Wizard, Cleric, Druid, Ranger at 5+) vs. spontaneous casters (Sorcerer, Bard); cantrips never cost slots; class DC = `10 + proficiency + keyAbility mod`
  - Focus points: Champion (lay on hands), Druid (heal), Bard (inspire courage) — 1 focus point base; restored by 10-minute `refocus` action out of combat
  - Rest mechanic: `/rest` command (or 10 minutes standing idle out of combat) restores spell slots and focus points — avoids requiring 8 real hours
  - Postgres lazy migration script: character without `ancestry`/`pf2e_class` fields initializes to Human Fighter defaults at load
- **Tests:** T-006 (`spell_slots_prepared`), T-007 (`spell_slots_spontaneous`), T-008 (`class_dc`), T-009 (`ancestry_hp`); `/rest` restores slots test; all 11 classes boot without crash
- **Release criteria:** All 11 classes playable to level 5 in the test zone; `crypt_raid.mjs`-equivalent 5-bot PF2e party clears the test dungeon; T-006–T-009 pass

#### v0.2.2 — UI Overhaul (PF2e HUD)

- **Theme:** Replace the WoW HUD with PF2e-appropriate UI components
- **Features / Technical work:**
  - Action pool meter (replaces the mana/rage/energy bar): 3-pip visual that fills in real-time; abilities gray out when pool < cost; action cost badge (1/2/3) shown on each action bar button
  - Character sheet (`C` key): shows STR/DEX/CON/INT/WIS/CHA modifiers, proficiency ranks (Untrained/Trained/Expert/Master/Legendary), class DC, max HP, AC, saves (Fortitude/Reflex/Will), ancestry and class
  - Condition display: condition icons under the unit frame with value and timer (frightened 2, slowed 1, etc.)
  - Spell slot tracker: per-slot-level pip counters in the spellbook window
  - Focus point indicator: small pip in the portrait frame for classes with focus spells
  - Update `src/net/online.ts` `applySnapshot()` for new self fields
- **Tests:** Snapshot test: `actionPool`, `conditions`, `spellSlots` fields present in first snapshot; reference-identity preserved when fields unchanged (delta snapshot behavior verified)
- **Release criteria:** Visual tour shows correct HUD in the test zone; all snapshot delta tests pass; `tsc --noEmit` clean

#### v0.2.3 — CI + Developer Experience

- **Theme:** Automated quality gate and contributor onboarding for the PF2e branch
- **Features / Technical work:**
  - GitHub Actions CI: `npm test`, `tsc --noEmit`, `npm run build`, `npm run build:env` on every PR to `feature/pf2e-engine` and `main`
  - Extract `DELTA_SELF_KEYS`/`mergeSelf` from three bot scripts into `scripts/util.mjs`
  - Spatial grid cell cleanup on empty arrays (`SpatialGrid.remove` — deferred from v0.1.2 follow-up)
  - `CONTRIBUTING.md`: dev setup, test commands, PR process, PF2e content-addition guide
  - `docs/design/pf2e-rules.md`: document all intentional MMO adaptations (action pool accrual rate, simplified feat system, rest mechanic, 4-degree success in real-time context)
- **Release criteria:** CI green on `feature/pf2e-engine`; `CONTRIBUTING.md` and `docs/design/pf2e-rules.md` merged

### Risks

| Risk | Severity | Mitigation |
|---|---|---|
| All WoW sim tests break simultaneously | High | Replace incrementally per subsystem (combat, leveling, classes); never let vitest go red overnight |
| Postgres lazy migration initializes wrong defaults | Medium | Test migration on a database with WoW characters before any deployment |
| Action pool feel — players may feel sluggish waiting for pool | Medium | Make accrual rate configurable (server config); tune during v0.2.1 class playtesting |
| TypeScript strict mode surfaces widespread latent type errors | Medium | Enable incrementally; allow `@ts-expect-error` with TODO comments temporarily |
| `feature/pf2e-engine` diverges far from main | Low | Cherry-pick hotfixes from main promptly; rebase weekly during active development |

### Dependencies
- v0.1.1 and v0.1.2 must merge to main before `feature/pf2e-engine` is branched
- v0.2.1 depends on v0.2.0 (classes build on the action pool and d20 resolver)
- v0.2.2 depends on v0.2.1 (UI needs stable self fields from the class system)
- v0.2.3 can run in parallel with v0.2.1 and v0.2.2

### Release criteria
All four v0.2.x minor versions merged to `feature/pf2e-engine`; CI green; 100+ PF2e vitest tests pass; all engine-level (non-WoW) tests pass; 11 classes playable; PF2e HUD working; pf2e-rules.md written.

### Promotion criteria
v0.3.x (world content) begins after v0.2.1 (classes) is stable — Zone 1 content requires all 11 classes to be playable. v0.2.2 and v0.2.3 can overlap with v0.3.0 development.

---

## Second Next Major Version: v0.3.x

**Planning level:** Fully planned

### Version Theme
The Shattered Reaches — world content. Build the three-zone original world on top of the PF2e engine. Zone 1 (Ashfen Borderlands, levels 1–5), Zone 2 (The Thornwood, levels 5–10), Zone 3 (The Sundered Keep, levels 10–15), and three dungeons. Also includes the branch merge to `main` (v0.3.2) and the README rewrite.

### Goals
- A character can level 1–15 through The Shattered Reaches without getting stuck
- All three zones have hub NPCs, vendors, graveyards, quests, mob camps, and a dungeon
- Three-act story arc: The Shattered Reaches conspiracy (shadow-plane breach)
- PF2e treasure parcels replace WoW-style gear drops (appropriate magic items by level)
- Faction system: Wardens of Ashfen, Breach Seekers, Hollow Court — affects NPC attitudes
- Branch merged to `main`; README fully rewritten for PF2e + new world

### Breaking changes and compatibility notes
- `feature/pf2e-engine` merges to `main` at v0.3.2. This replaces the WoW codebase on main. Tag `v0.3.2` as the first PF2e release.
- All WoW content (zones, quests, mobs, items) is removed from main. Preserved in git history and in the Archive section of this document.

### Architecture direction
- `src/sim/content/` gains `zone_ashfen.ts`, `zone_thornwood.ts`, `zone_sundered.ts`, `dungeons_pf2e.ts` following the same `ZoneDef`/`MobDef`/`QuestDef`/`ItemDef` shape — no engine changes needed
- PF2e mob stat blocks sourced from the PF2e Monster Core level-to-stat tables (Attack bonus, AC, Fortitude/Reflex/Will saves, HP, damage)
- Items gain a `rarity` field (`common`/`uncommon`/`rare`/`unique`) replacing WoW-style item quality
- Faction attitudes stored per-character in JSONB; faction standing affects NPC dialogue and vendor access

### Minor Version Plan

#### v0.3.0 — Zone 1: The Ashfen Borderlands (Levels 1–5)

- **Theme:** The starting zone — Ashford town, shadow corruption spreading through farmland
- **Setting:** Ashford is a walled border town surrounded by farms being eaten by shadow-touched corruption. The three local factions are in uneasy alliance against the threat.
- **Hub:** Ashford (Warden Aldric — quest hub; Sister Maren — healer/vendor; Blacksmith Orvyn — weapons/armor; Merchant Tess — consumables; Scout Fenn — faction rep)
- **Mob families (levels 1–5):**
  - Shadow Rat (CR 1–2): scavengers corrupted by the breach; swarm behavior
  - Thornback Bandit (CR 2–3): human bandits taking advantage of the chaos
  - Ashfen Shade (CR 3–4): humanoid shadows; undead family; weak to positive energy (Cleric/Champion)
  - Dungeon trash: Ashford Cultist + Breach Wraith (elite, CR 4)
- **Quests (10+):** Shadow corruption investigation chain; missing farmers; bandit supply disruption; Warden loyalty quest; dungeon breadcrumb; rare spawn (The Pale Wanderer, a CR 4 unique shade)
- **Dungeon:** The Ashford Cellar (5-player, ~level 4) — cultists attempted to open a sub-breach beneath Ashford. Boss: Archon Malgrist, Shadow Caller (PF2e: summons shades at 50% HP; Darkness pulse every 8s)
- **Items:** Common weapons/armor by class archetype at vendors; uncommon drops in the dungeon
- **Tests:** T-013 (`zone1_leveling_1_to_5`): Fighter can level 1→5; all quests completable; dungeon beatable by a 5-player PF2e party bot test
- **Release criteria:** Zone 1 solo-playable 1→5; dungeon bot test passes; no T-poses in visual tour

#### v0.3.1 — Zone 2: The Thornwood (Levels 5–10)

- **Theme:** Ancient forest where a Druidic circle lost the breach-containment ritual; aberrations spreading outward
- **Hub:** Greenvale Outpost (Captain Thessaly — military quests; Elder Brynn — Druid circle NPC; Artificer Moll — upgraded gear; Witch Serafine — potion vendor)
- **Mob families (levels 5–9):**
  - Thornwood Stalker (CR 5–6): fey-touched wolf; nature family
  - Tainted Dryad (CR 6–7): corrupted fey; charm aura (Will save or fascinated)
  - Breach Hound (CR 7–8): aberration; sense of breach energy (aggro range +50%)
  - Elite: Void Tendril (CR 8–9, elite); paired ambush mechanic
  - Rare spawn: Mirefen the Deceiver (CR 8 unique aberration; polymorph ability mimics friendly NPCs)
- **Quests (15+):** Druid circle restoration chain; aberration containment; fey court diplomacy (faction: Greenvale Wardens vs. Breach Seekers disagreement); Zone 3 breadcrumb
- **Dungeon:** The Thornwood Depths (5-player, ~level 9) — the Druid circle's ritual chamber, now a nest of aberrations. Boss: The Unbound (CR 10, aberration; tentacle slam + mental blast AoE)
- **Items:** Uncommon weapons with elemental properties; rare dungeon drops
- **Tests:** Fighter levels 5→10 through Thornwood; all quests completable; dungeon bot test; faction standing system tested (quest outcome changes based on standing)
- **Release criteria:** Zone 2 completable; dungeon bot test passes; faction system functional

#### v0.3.2 — Zone 3: The Sundered Keep + Branch Merge

- **Theme:** The original breach site; three factions fight for control; the story climax
- **Hub:** The Breach Camp (a fortified survivor settlement at the gate's outer perimeter)
- **Mob families (levels 10–15):**
  - Hollow Court Revenant (CR 10–12): undead soldiers of the old kingdom; organized squad tactics (group aggro)
  - Breach Seeker Mercenary (CR 11–12): humanoid; will negotiate (faction standing check) or fight
  - Gate Horror (CR 12–14): large aberration emerging from active breach tears; AoE fear aura (Will save vs. frightened 2)
  - Elite: Void Knight (CR 13–14, elite); Champion-equivalent — uses Reactions and Shield Block
  - Boss: Grand Inquisitor Valdris (CR 15; the secret society leader; summons adds at 60%/30% HP; Sundering Strike crits on 19–20)
- **Quests (20+):** Three parallel faction quest chains converging on the breach; solo-able lead-up (level 13–14) → 5-player dungeon finale
- **Dungeon:** The Sundered Keep Vault (5-player, level 14–15) — the breach gate chamber beneath the Keep. Bosses: Gate Warden Kael (miniboss) → Valdris → The Breach Itself (environmental final phase: players must activate 3 sealing runes while fighting adds)
- **Merge:** `feature/pf2e-engine` merges to `main`; tag `v0.3.2`; README rewritten for PF2e + The Shattered Reaches
- **Release criteria:** Zone 3 completable 10→15; dungeon beatable with 5-bot party; README rewritten; branch merged; all 138+ tests pass on `main`

#### v0.3.3 — Content Polish + Balance Pass

- **Theme:** Tuning, quest balance, world detail, and economy
- **Features:**
  - XP curve audit: no dead zones between levels 1–15; verify PF2e milestone XP equivalent
  - Economy audit: vendor prices, treasure parcel values, copper drops reviewed against level progression
  - Rest mechanic tuning: adjust out-of-combat timer for rest; add campfire objects in each zone that speed rest
  - Rare spawn pass: verify all three zones have at least one rare spawn with unique drops
  - World event (first one): "The Breach Pulses" — periodic server event where breach tears spawn across all zones for 1 hour, boosting XP and dropping unique items
  - Soulbound rare items: epic drops from final dungeon are character-bound (not tradeable)
- **Tests:** XP curve continuity test; rest timer test; soulbound item rejected in trade
- **Release criteria:** Live playtest with 5 players completing 1→15 without major XP gaps; no reported soft-locks

### Risks

| Risk | Severity | Mitigation |
|---|---|---|
| PF2e mob stat tuning is wrong at launch | High | Use PF2e Monster Core level tables as baseline; tune in v0.3.3; bot tests catch outlier difficulty |
| Zone 1→2 breadcrumb breaks solo players | Medium | Level gate at 5 for Zone 2 breadcrumb quest; Zone 2 entrance always accessible |
| `feature/pf2e-engine` merge conflicts at v0.3.2 | Medium | Merge monthly hotfixes from main into the feature branch; use rebase not merge |
| World events cause server spike if too many breach tears spawn simultaneously | Low | Cap breach tear count per zone; use spatial grid for proximity aggro (already implemented) |

### Dependencies
- v0.2.1 (all 11 classes) must complete before v0.3.0 content development begins
- v0.3.0 before v0.3.1; v0.3.1 before v0.3.2 (content is sequential)
- v0.3.2 merge to main is the gate for all future work on main

### Release criteria
Zones 1–3 fully playable 1→15; three dungeons complete with bot tests; faction system functional; branch merged to main; README rewritten; CI green on main; all tests pass.

---

## Third Next Major Version: v0.4.x

**Planning level:** 75% planned

### Theme
Social + community layer and RL environment update. Now that the PF2e world is on `main`, v0.4 adds the multiplayer social features that make World of Claudecraft a community game — adventuring guilds, group finder, chat channels — and updates the headless RL environment to reflect the PF2e state space.

### Goals
- Guild system (create, invite, rank, guild chat, MOTD)
- Looking For Group board for dungeon coordination
- Chat expansion (whisper, channels, ignore)
- RL environment updated: `PF2eMMOEnv` replaces `WoWClassicEnv` (obs/action space for PF2e state)
- Multi-agent RL support (N agents in one sim)

### Likely Minor Versions

#### v0.4.0 — RL Environment Update

- **Theme:** Make the headless env speak PF2e
- **Scope:**
  - New `encodeObs()` for PF2e state: action pool fraction, spell slots remaining per level, conditions bitmask, ability score modifiers, nearby-enemy AC/saves, degree-of-success probabilities
  - Expanded action space: ability targeting (target a party member for heals), item-use action, `/rest` action
  - `headless/env_server.ts` updated; Python `WoWClassicEnv` kept as deprecated alias pointing to new `PF2eMMOEnv` class
  - New reward shaping: quest completion, dungeon boss kill, condition-infliction bonuses for control classes
  - Headless bench target: ≥200 steps/s (same as current WoW benchmark)
  - `obs_vector_pf2e` vitest test: deterministic `obsSize()`; all values in [-2, 2]

#### v0.4.1 — Guild System

- **Theme:** Persistent adventuring guilds with membership, ranks, and guild chat
- **Scope:**
  - New DB tables: `guilds (id, name, motd, created_at)`, `guild_members (guild_id, character_id, rank, joined_at)`
  - PF2e flavor: guild ranks named after PF2e archetypes (Squire, Adventurer, Veteran, Champion, Guildmaster)
  - Commands: `guild_create`, `guild_invite`, `guild_accept`, `guild_kick`, `guild_promote`, `guild_demote`, `guild_leave`, `guild_disband`, `guild_motd`
  - Guild name in nameplate; guild roster UI
  - New `tests/guild.test.ts`
  - Migration script (idempotent, reversible)

#### v0.4.2 — Group Finder + Chat Expansion

- **Theme:** Help players find dungeon groups; improve chat for larger populations
- **Scope:**
  - In-memory LFG board: post listing (dungeon, role, note), browse, invite from listing, 30-minute expiry
  - Whisper system (`/w PlayerName message`); trade channel (`/trade`); general channel (`/1`)
  - Ignore list (DB-persisted per character)
  - Server-enforced message length cap (255 chars)

#### v0.4.3 — Multi-Agent RL

- **Theme:** Enable N simultaneous RL agents in one sim for multi-agent research
- **Scope:**
  - `Sim` already supports N players; wire up per-agent `encodeObs` and per-agent action dispatch
  - Python `WoWPartyEnv` → `PF2ePartyEnv`: `num_agents` parameter; per-agent obs/reward/done from one step
  - Benchmark multi-agent throughput at N=5 before committing scope

### Technical direction
- Guild DB establishes the migration pattern for all v0.4.x+ schema work
- LFG is in-memory only (no DB); clears on server restart
- RL obs update must be backward-compatible with the Protocol (new fields additive)

### Known dependencies
- v0.3.2 must merge to main before v0.4 work begins (world content gives RL agents meaningful objectives)
- v0.4.0 and v0.4.1 are independent and can develop in parallel on separate branches
- v0.4.3 depends on v0.4.0 (multi-agent obs builds on single-agent obs)

### Major risks
- **PF2e obs dimensionality:** PF2e has more state than WoW (conditions, multiple save types, action pool, spell slots). Obs vector may be larger; verify learning still works.
- **Guild DB migration pattern:** Must be proven safe on v0.4.1 before adding more schema work.
- **Multi-agent learning difficulty:** 5-agent coordination in PF2e is significantly harder than solo. Curriculum learning or action masking may be needed.

### Open questions
- Should the LFG board show cross-zone listings or same-zone only? (default: same-zone in v0.4.2; cross-zone in a later patch)
- What training framework to target for PF2e RL? (Stable Baselines 3 is the default recommendation; framework-agnostic wrapper stays)
- Should multi-agent env support mixed human + AI sessions? (Architecturally feasible; defer design decision to v0.4.3 planning)

### Validation needed before full planning
- Benchmark PF2e multi-agent sim throughput at N=5 to confirm near-linear cost (the spatial grid from v0.1.2 makes this likely)
- Profile PF2e obs encoding cost at N=5 (condition bitmask + spell slot arrays may be heavier than WoW `cds` map)

---

## Fourth Next Major Version: v0.5.x

**Planning level:** 50% planned

### Strategic direction
Platform and hosting maturity. By v0.5, the game and RL env should be production-ready for community server operators and ML researchers at scale. This version addresses observability, scalability, and operational tooling.

### Candidate themes
- Horizontal scaling: evaluate multiple `Sim` instances per process (different zones or world seeds)
- CDN and static asset serving for the Vite client build
- Structured logging (replace `console.log/error` with pino or equivalent)
- Prometheus `/metrics` endpoint: player count, tick duration EMA, snapshot sizes, DB query latency
- Admin tooling expansion: ban/unban, character restoration, audit log UI
- Account-level rate limiting (prevent connection farming)
- WebSocket compression evaluation (permessage-deflate; delta snapshots from v0.1.2 reduce urgency)

### Major unknowns
- Whether the single-Postgres-process approach scales to 100+ concurrent players without sharding (likely yes with connection pooling; needs a load test at v0.5 planning time)
- Whether community operators want multi-world on one server vs. separate deployments
- Whether CDN requires asset fingerprinting changes to the Vite config

### Research needed before full planning
- Load test at 50 and 100 concurrent WebSocket clients with realistic movement patterns
- Profile Postgres autosave query latency under 100 concurrent saves (every 30s per player)
- Survey RL researcher needs for training observability (what metrics do they need from the env server?)

### Items not yet committed
- Chat history persistence (deferred from v0.3 as ephemeral-only)
- Mobile client (exploratory only; Three.js on mobile requires a full controls redesign)
- Billing or premium features (explicitly out of scope — MIT, self-hosted)

---

## Idea Backlog: v1.0.x and Beyond

### v1.0.x — Stable Release Milestone

- **Idea:** Declare a stable 1.0 with a versioned WebSocket protocol, documented migration guide from 0.x, and a compatibility promise
- **Value:** Gives community server operators and RL researchers a stable target; establishes trust
- **Category:** Stability and community trust
- **Confidence:** High
- **Keep:** Yes — natural milestone after v0.5 platform work

### v1.1.x — Content Expansion: Levels 15–20

- **Idea:** Extend the Shattered Reaches to level 20 with a post-Valdris epilogue arc
- **Description:** Valdris was working for something older. A fourth zone (The Void Margin — a semi-planar boundary region) and a fourth dungeon tier for levels 15–20, culminating in closing the breach permanently (or choosing not to).
- **Confidence:** High — the three-act structure leaves an obvious hook
- **Keep:** Yes

### v1.1.x — PvP: Breach War Battleground

- **Idea:** Two teams of up to 5 players fight over a breach control point (Warsong Gulch-style but PF2e-balanced)
- **Description:** Private `Sim` instance for the battleground; queue through a Battlemaster NPC; PF2e combat with modified conditions for PvP (no insta-kill conditions)
- **Confidence:** Medium — PF2e PvP balance is a significant ongoing commitment
- **Keep:** Yes, with the explicit caveat that PvP balance work is a separate sustained effort

### v1.2.x — Pre-Trained Agent Showcase

- **Idea:** Train and publish a Fighter agent that completes the Ashfen Borderlands (1→5) and demonstrate it running in the browser
- **Value:** Makes the RL angle of the project visible and compelling to researchers
- **Confidence:** High — the headless bench already achieves 236 steps/s; training to zone 1 completion is tractable
- **Keep:** Yes — prioritize after v0.4.x RL env is stable

### v1.2.x — Auction House

- **Idea:** Global auction house where players post items for sale; requires a mail system to deliver sold/returned items
- **Confidence:** Low-Medium — depends on whether player density reaches the level where a global AH is useful
- **Keep:** Yes, pending player density signal

---

## Deferred or Rejected Items

| Item | Original target | Decision | Reason | Revisit trigger |
|---|---|---|---|---|
| WoW ability rank completion (all 9 classes) | v0.2.0 (WoW plan) | Superseded | WoW mechanical layer replaced by PF2e; ability rank spec archived | Never — WoW content is fully replaced |
| WoW asset coverage for zone 2–3 mob families | v0.2.1 (WoW plan) | Superseded | New world uses different mob families; KayKit/Quaternius assets reused with new mappings | Never — WoW content is fully replaced |
| WoW guild system with WoW-named ranks | v0.3.0 (WoW plan) | Redesigned | Guild system moves to v0.4.1 with PF2e-appropriate rank names | Implemented in v0.4.1 |
| WoW world events (Harvest Festival) | v0.3.3 (WoW plan) | Redesigned | Replaced by "The Breach Pulses" world event in v0.3.3 with PF2e flavor | Implemented in v0.3.3 |
| Chat history persistence | v0.3.2 (WoW plan) | Deferred | Ephemeral chat is acceptable at small scale; adds DB write load | When concurrent players regularly exceed 50 |
| Offline whisper delivery | v0.3.2 (WoW plan) | Deferred | Requires mailbox system; disproportionate complexity for v0.3 scope | Implement when auction house mailbox is built |
| Mixamo character animations | UE5 plan | Rejected | Adobe license; incompatible with MIT/open-source goal | Do not revisit |
| KTX2 HDR textures | UE5 plan | Rejected | three.js r165 cannot transcode HDR; marginal size benefit | Revisit when upgrading to three.js r168+ |
| Draco mesh compression | UE5 plan | Rejected | meshopt is already in use and is better for animation-heavy low-poly | Do not revisit |
| Billing / premium tier | Any | Rejected | MIT license, self-hosted community focus | Do not revisit |

---

## Roadmap Risks and Dependencies

### Cross-version dependencies
- v0.1.2 (spatial grid, performance baseline) must land on main before `feature/pf2e-engine` branches — the performance work is engine-level and applies equally to PF2e
- v0.2.1 (all 11 PF2e classes) must be stable before v0.3.0 (world content) begins
- v0.3.2 (branch merge to main) is the gate for all v0.4.x and v0.5.x work
- v0.4.0 (PF2e RL env) must complete before v0.4.3 (multi-agent RL) begins
- Guild DB schema in v0.4.1 establishes the migration pattern for v0.4.2 and v0.5.x schema work

### Major technical risks

| Risk | Severity | Decision |
|---|---|---|
| WoW sim tests all break simultaneously during v0.2.0 | High | Replace incrementally per subsystem; never let vitest go fully red overnight |
| PF2e mob stat tuning produces fights that are too hard/easy | High | Use PF2e Monster Core level tables as baseline; bot tests catch outlier difficulty |
| Character Postgres migration corrupts data after branch merge | High | Idempotent migration script; lazy per-character migration; tested on clean Postgres 16 |
| `feature/pf2e-engine` diverges too far from main to merge cleanly | Medium | Rebase from main monthly; merge main hotfixes into the branch promptly |
| Action pool feel — players feel sluggish between actions | Medium | Make accrual rate a server config value; tune during v0.2.1 playtesting |
| PF2e obs vector too large for efficient RL training | Medium | Profile obs encoding at N=1 and N=5 before v0.4.3; apply sparse encoding if needed |

### Product risks
- **Community adoption:** README rewrite (v0.3.2) and `CONTRIBUTING.md` (v0.2.3) are the key enablers. The PF2e rules doc (`docs/design/pf2e-rules.md`) is critical for the TTRPG audience who will scrutinize rule fidelity.
- **RL research interest:** The PF2e 4-degree success system and condition model are richer than WoW mechanics — better for research. But the obs space changes (v0.4.0) mean any WoW-trained agents need to be retrained.
- **Content scope vs. team size:** The Shattered Reaches spans 3 zones, 3 dungeons, and 20+ quests. Each zone should have its own PR with a bot test before the next zone begins.

### Documentation risks
- README is currently WoW-flavored and must be rewritten by v0.3.2 (branch merge)
- `docs/design/pf2e-rules.md` must be written before v0.2.2 so contributors and TTRPG players understand intentional deviations
- Python wrapper rename (`WoWClassicEnv` → `PF2eMMOEnv`) must have a clear deprecation notice in v0.4.0

---

## Archive

### v0.1.0 — Initial Open-Source Release

- **Status:** Completed
- **Summary:** First open-source release. Full 1–20 WoW-Classic MMO with 9 classes, 3 zones, 3 dungeons, ~60 quests, multiplayer server, Postgres persistence, Docker deploy, PBR graphics (CC0 assets), Gymnasium RL env, MIT license.
- **Delivered:** 9 vanilla classes with real vanilla formulas; Eastbrook Vale, Mirefen Marsh, Thornpeak Heights; Hollow Crypt, Sunken Bastion, Gravewyrm Sanctum; party/trade/duel; GM characters; admin dashboard; scrypt auth; rate limiting; Docker Compose
- **Not delivered at v0.1.0:** CI pipeline; guild system; full spell rank progressions for all classes; complete glTF coverage for all zone 2–3 mob families
- **Known issues at time of archive:** Trade item-duplication exploit; server crash on malformed message; O(n²) entity scans
- **References:** Commit `3387f8b`; MIT license PR #6 `6974f85`; tag `trailer-v01`

---

### Superseded: WoW-Focused v0.2.x–v0.5.x Plans (First Roadmap)

- **Status:** Superseded
- **Summary:** The initial ROADMAP.md (created 2026-06-12) planned v0.2.x–v0.5.x around WoW mechanics: ability rank completion, WoW asset coverage, WoW guild system, WoW world events, WoW-flavored RL env. These plans are superseded by the PF2e conversion decision and replaced by the active roadmap above.
- **Key items superseded:**
  - v0.2.0: WoW ability rank implementation (Execute, Slam, Cleave for Warrior; Kidney Shot for Rogue; all 9 classes per `docs/design/spell-ranks.md`)
  - v0.2.1: glTF asset coverage for zone 2–3 WoW mob families (troll, ogre, dragonkin)
  - v0.2.2: CI + developer experience (carried forward into v0.2.3 of the PF2e plan)
  - v0.2.3: WoW content polish and quest balance
  - v0.3.0–v0.3.3: WoW guild system, group finder, WoW chat expansion, WoW world events (Harvest Festival)
  - v0.4.x: WoW single-agent RL improvements
  - v0.5.x: Platform/hosting maturity (carried forward unchanged)
- **Reason for supersession:** User directed a full pivot to Pathfinder 2e mechanics with a fresh original world setting. WoW-specific plans have no value in the PF2e direction.
- **Preserved in:** git history on `main`; `docs/design/` folder (WoW spec documents archived, not deleted); this Archive section
- **Date superseded:** 2026-06-12

---

## Next Roadmap Actions

1. **Immediate (maintainer action required):** Approve + merge PR #7 (trade exploit — live security issue), then #8, then #9. All three are verified: the combined tree passes 141/141 tests with a clean typecheck (local branch `integration/v0.1.2-verify`). Exact commands from a write-access account: `gh pr review 7 --approve && gh pr merge 7 --merge` (repeat for 8, then 9).
2. **Post-merge:** Commit the prepared CHANGELOG.md (move 0.1.1/0.1.2 from Unreleased to released), bump `package.json` to 0.1.2, then open `feature/pf2e-engine` from post-merge main.
3. **v0.2.0 start:** Begin with `src/sim/types.ts` — add PF2e entity fields, remove WoW fields. Fix all TypeScript compilation errors before touching any other file.
4. **v0.2.0 parallel:** Write `docs/design/pf2e-rules.md` documenting MMO adaptations (action pool accrual rate, simplified feats, rest mechanic, real-time 4-degree resolution). Do this during v0.2.0 while the combat resolver is being designed.
5. **v0.2.1 prep:** Before writing any class data, write T-006–T-009 tests first (TDD for the class system). This ensures the class definitions are correct before playtesting.
6. **v0.3.0 prep:** Draft Zone 1 content (Ashfen Borderlands) while v0.2.1 classes are being built. Zone content uses the same `ZoneDef`/`MobDef`/`QuestDef` shapes — no engine dependency.
7. **v0.3.2 merge gate:** Do not merge `feature/pf2e-engine` to `main` until: (1) all 138+ tests pass, (2) 5-bot party clears all three dungeons, (3) README is rewritten.
8. **Ongoing:** Cherry-pick any hotfixes from `main` into `feature/pf2e-engine` promptly. Rebase the branch from main before each minor version completes.

---

## Roadmap Decision Output

- **Current roadmap status:** Updated and current
- **Current product version decision:** v0.1.0 — High confidence
- **Versions fully planned:** v0.1.x, v0.2.x, v0.3.x
- **Version 75% planned:** v0.4.x
- **Version 50% planned:** v0.5.x
- **Idea backlog versions:** v1.0.x, v1.1.x, v1.2.x
- **Archived versions:** v0.1.0 (delivered); WoW-focused v0.2.x–v0.5.x plans (superseded)
- **Major corrections made:** Entire v0.2.x–v0.3.x plan replaced — WoW ability ranks, WoW asset coverage, WoW guild system all superseded by PF2e engine conversion and The Shattered Reaches world content
- **Major assumptions:**
  - PF2e SRD is freely available and used as the canonical rules source
  - The team is small; minor version scope is set conservatively (one zone per PR, one class subsystem per PR)
  - The Three.js renderer and CC0 assets are setting-agnostic and require no changes for PF2e
  - Character data migration is one-way after v0.2.1; a rollback script must exist before any deployment
- **Risks:** WoW sim tests all break simultaneously in v0.2.0; PF2e mob tuning may require multiple passes; `feature/pf2e-engine` branch must be kept in sync with main hotfixes
- **Next best action:** Merge PR #7 today. Open `feature/pf2e-engine` after v0.1.2 merges. Start with `src/sim/types.ts`.
- **Confidence level:** High
