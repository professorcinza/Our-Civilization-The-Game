# add-worldbuilding-research

## Why

Project Lunar's world-building today derives from design intuition and from A/B invariants. Reference games have solved analogous problems (off-screen living world, systemic consequence, environmental lore) with mature mechanics that are widely documented in public sources. Documentary reverse engineering of these mechanics — combined with a playable prototype to feel the lessons in practice — accelerates the evolution of the world-simulation, npc-minds, plot-generation and scenario-authoring specs with verifiable lessons instead of guesswork.

## What Changes

**New spec: worldbuilding-research**
- From: no structured world-building research program; implicit and untraceable lessons.
- To: 9 requirements — reverse engineering of Albion Online (player-driven economy, territories, risk bands, seasons), the Albion-in-life-RP hybrid synthesis (risk bands as state presence, carry-only loss, closed economy vs RP inflation, declared territory wars — translations mapped to mmo-game), reverse engineering of GTA San Andreas (CJ stats, gang war, NPC routines, wanted level, progressive gating), reverse engineering of GTA V RP worldwide (whitelist gates, IC/OOC rules, player-run institutions, per-country scenes, with translations mapped to mmo-game), reverse engineering of MUDs (offline persistent world, room+look network, social channels, RPI, OLC/MOO), reverse engineering of CyberCode Online (AFK/idle loop, procedural world from community-contributed corpora, multilingual lore channel), a d3wasm-based prototype of the final engine as a world-building laboratory, versioned lesson cards in `data/worldbuilding/lessons.json` with target-spec traceability, and an asset safeguard (documentary observation only + original implementation, d3wasm GPL-3.0 engine as the single documented code exception).

**Translation, not copying**
- Each researched mechanic SHALL produce a card with: the original mechanic, why it works (emergent effect), and a candidate translation to the narrative engine (or a justified discard). Examples of candidate translations: Albion's regional markets → scarcity/price as a tick trigger; GTA SA pedestrian routines → npc-minds agendas; wanted level → consequence escalation in ticks; Doom 3 lore terminals → story cards.

**d3wasm-based prototype of the final engine**
- To: a browser prototype built on **d3wasm** (github.com/gabrielcuvillier/d3wasm — id Tech 4 / Doom 3 ported to WebAssembly via Emscripten with a full WebGL renderer, GPL-3.0) used as the prototype of the game's final engine: dark corridors, a dynamic flashlight, interactive terminals and script triggers — each element mapped to a lesson card — plus engine-architecture lessons from studying the `neo/` sources (scripting, GUI/terminals, asset pipeline, renderer) to inform the final engine decision.
- Licensing boundary: d3wasm engine code is the single licensed-code exception (GPL-3.0, documented) — prototype code becomes GPL-3.0-compatible; game assets remain 100% original or free with documented licenses; no original Doom 3 game data (`.pk4` content) enters the repo; the copyleft inheritance of a d3wasm-derived final engine is a documented trade-off to decide before adoption.

## Impact

- Affected specs: none modified; adds `worldbuilding-research`. Accepted translations will later enter as changes in the target specs.
- Non-breaking; versioned static data + isolated prototype (independent frontend or static route).
- Legal risk mitigated by the asset requirement: research is documentary (public sources), implementation is original; the GPL-3.0 engine exception is explicit and contained to prototype code.
