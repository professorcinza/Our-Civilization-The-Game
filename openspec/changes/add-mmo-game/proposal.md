# add-mmo-game

## Why

Product direction decision: the game itself will be a Role-Playing MMORPG based on the lore specified in the specs. Until now the specs described a single-player narrative engine and its research program; the multiplayer end-state existed only implicitly (MUD/CyberCode lessons, d3wasm engine path). This change records the commitment: a persistent multiplayer world on top of the specified engine, browser-first via the d3wasm path, narrative-first progression preserved.

## What Changes

**New spec: mmo-game (vision-level contract)**
- 6 requirements:
  1. Final product is a lore-based MMORPG — canonical world derives from the specs' lore (O Cidadão do Futuro, military training worlds, doctrine regiments); the engine stays the simulation core.
  2. Persistent multiplayer world — off-screen ticks continue while players are offline; other players' consequences surface through narrative means (MUD lessons applied).
  3. Browser client on the d3wasm engine path — prototype → final engine with the documented GPL-3.0 trade-off.
  4. Narrative-first progression — no HP/mana/grind loops leak into multiplayer systems; progression is memory, journal, relationships, standing.
  5. Social layer with roleplay integrity — presence, in-character speech/emotes, separated OOC channels; age-banding trays and avatar-mirror consent/LGPD govern multiplayer visibility.
  6. Community contribution channel — CyberCode lesson: contributions enter through the moderated scenario-authoring pipeline, never mutating canon directly.
  7. Scale targets for the open world (v1) — 1k–3k concurrent players per map, ~100 visible per client at 30+ FPS via interest management, thousands of routine NPCs, tens–~200 LLM-alive minds per region, ~US$ 0.01–0.03 per narrated turn (bottleneck order: LLM cost → client render → world sim).
  8. Hybrid simulation layers — deterministic moment-to-moment (no LLM), LLM narrative events on significant beats only, per-region NPC-mind pools with witness filter and LLM budgets with graceful degradation.
  9. d3wasm netcode gap as headline risk — client prediction, server authority, snapshotting and interest management built from scratch and load-tested before scale sign-off.
  10. Cultural shards over the same canon — GTA V RP lesson: regional/community shards (own rules, tone, language) as configuration over shared canon, never forked lore; community gates cannot override age-banding or avatar-mirror consent.
  11. Player-run institutions — institutional roles (peacekeeping, medical, legal, press) occupied by players with persisted minds (player-minds), community-operated world state under the same audit and canon rules.
  12. Closed player-driven economy — no infinite NPC faucets; value produced by players/simulation, drained by lifelike sinks (taxes, rent, insurance); regional markets with divergence usable as tick signals.
  13. Carry-only material consequence — robbery transfers what is carried; banked/stored/insured assets safe; character life remains sacred per RP rules; insurance as sink.
  14. Declared territory wars via player institutions — territory income flows through the economy; takeovers require declaration, time window and engagement rules recorded in the event store.

- From: no MMO requirement; multiplayer implied by research lessons only.
- To: explicit vision-level contract; detailed mechanics (sharding, networking, economy, scale) arrive as future changes against this spec.

## Impact

- Affected specs: none modified; adds `mmo-game`. worldbuilding-research is referenced (engine path, MUD/CyberCode lessons), not changed.
- Non-breaking: vision-level requirements; implementation plan unchanged until a future change picks it up.
- The "engine, not a single game" framing in the project context is refined: the engine remains the core, and the committed product target built on it is this MMORPG.
