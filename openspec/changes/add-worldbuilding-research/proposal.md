# add-worldbuilding-research

## Why

Project Lunar's world-building today derives from design intuition and from A/B invariants. Reference games have solved analogous problems (off-screen living world, systemic consequence, environmental lore) with mature mechanics that are widely documented in public sources. Documentary reverse engineering of these mechanics — combined with a playable prototype to feel the lessons in practice — accelerates the evolution of the world-simulation, npc-minds, plot-generation and scenario-authoring specs with verifiable lessons instead of guesswork.

## What Changes

**New spec: worldbuilding-research**
- From: no structured world-building research program; implicit and untraceable lessons.
- To: 6 requirements — reverse engineering of Albion Online (player-driven economy, territories, risk bands, seasons), reverse engineering of GTA San Andreas (CJ stats, gang war, NPC routines, wanted level, progressive gating), reverse engineering of MUDs (offline persistent world, room+look network, social channels, RPI, OLC/MOO), a Doom 3-style WebGL prototype as a world-building laboratory, versioned lesson cards in `data/worldbuilding/lessons.json` with target-spec traceability, and an asset safeguard (documentary observation only + original implementation).

**Translation, not copying**
- Each researched mechanic SHALL produce a card with: the original mechanic, why it works (emergent effect), and a candidate translation to the narrative engine (or a justified discard). Examples of candidate translations: Albion's regional markets → scarcity/price as a tick trigger; GTA SA pedestrian routines → npc-minds agendas; wanted level → consequence escalation in ticks; Doom 3 lore terminals → story cards.

**Doom 3-like prototype**
- To: a browser prototype (WebGL/Three.js, no native build, no plugins) with dark corridors, a dynamic flashlight, interactive terminals and script triggers — each element mapped to a lesson card. Assets 100% original or free with a documented license; nothing extracted from the reference games.

## Impact

- Affected specs: none modified; adds `worldbuilding-research`. Accepted translations will later enter as changes in the target specs.
- Non-breaking; versioned static data + isolated prototype (independent frontend or static route).
- Legal risk mitigated by the asset requirement: research is documentary (public sources), implementation is original.
