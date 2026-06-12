# Changelog

All notable changes to World of Claudecraft are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/); versioning follows [SemVer](https://semver.org/).

## [0.1.2] — Unreleased (verified, awaiting merge of PRs #8 and #9)

Performance pass: eliminates the O(n²) hot paths in the server's snapshot
broadcast and sim AI loops.

### Changed
- Entity wire JSON is serialized once per tick and shared across all
  recipients whose interest radius contains it — snapshot serialization is
  now O(entities + clients) instead of O(entities × clients). (#8)
- Heavy self-state fields (`inv`, `equip`, `qlog`, `qdone`, `cds`, `stats`,
  `weapon`, `party`, `trade`, `duel`) are sent only when changed since the
  last snapshot for that session; absent means unchanged. A fresh session
  always receives a full snapshot. Self payload −65%, server CPU −35% with
  10 co-located idle clients. (#8)
- All entity radius queries (interest filtering, AoE effects, social aggro,
  tab-targeting, interact, nearest-player scans) now use a 32-unit spatial
  hash grid instead of scanning the full entity map. Sim tick −14% with 20
  moving players; server CPU −15% with 30 spread clients. (#9)
- The per-player combat-flag check is one O(entities) pass per tick instead
  of O(players × entities). (#9)

### Added
- `src/sim/spatial.ts` — spatial hash grid, kept roster-exact on
  spawn/despawn/teleport. (#9)
- `tests/snapshots.test.ts` (6 tests) and `tests/spatial.test.ts` (4 tests),
  including grid-vs-brute-force equivalence across the whole world. (#8, #9)

## [0.1.1] — Unreleased (verified, awaiting merge of PR #7)

Security hardening: fixes a live item-duplication exploit and a
single-message server crash. Merge before any other work.

### Fixed
- **Trade item-duplication exploit:** duplicate offer slots of the same item
  were validated independently against the bags, so N duplicate slots each
  passed alone and `addItem` credited the recipient unconditionally. Offers
  now merge duplicate slots per item and validate the cumulative total; the
  pre-swap check in `tradeConfirm` re-validates summed totals as
  defense-in-depth. (#7)
- **Server crash on malformed message:** any throw inside a command handler
  became an uncaught exception and took down the whole Node process. Command
  dispatch is now wrapped so a malformed payload is logged and dropped for
  that one client only. (#7)
- **Trade slot shape validation:** slots are rejected unless they are objects
  with a string `itemId` and a finite `count` — blocks `null` slots and
  `NaN`/`Infinity` counts that could corrupt item stacks. (#7)

### Added
- Three regression tests in `tests/social.test.ts` reproducing the exploit,
  the malformed-slot crash, and the legitimate split-offer case. (#7)

## [0.1.0] — 2026-06

Initial open-source release.

### Added
- WoW-Classic-style micro-MMO: 9 classes with vanilla formulas, 3 zones
  (levels 1–20), 3 five-player elite dungeons, ~60 quests, parties, trading,
  duels, tap rights.
- Authoritative multiplayer server: accounts (scrypt), persistent characters
  in Postgres, 20 Hz tick, interest-scoped snapshots, admin dashboard,
  per-IP rate limiting, Docker Compose deploy.
- Rendering: Three.js with rigged CC0 glTF characters (KayKit/Quaternius),
  PBR terrain splat, HDRI IBL, N8AO, procedural icons and WebAudio sound.
- Headless RL environment: NDJSON protocol, Gymnasium Python wrapper,
  23-action discrete space, configurable reward shaping.
- MIT license.
