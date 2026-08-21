## ADDED Requirements


### Requirement: Reverse Engineering of Albion Online Mechanics

The research system SHALL document, from public sources (official wikis, patch notes, dev blogs, Sandbox Interactive), the Albion Online world mechanics relevant to world-building: player-driven economy (resources, crafting, regional markets), territories and guilds, full-loot and risk zones by band (Blue/Yellow/Red/Black), faction travels, and the seasons cycle. Each mechanic SHALL produce a lesson card with: the original mechanic, why it works (emergent effect), and a candidate translation to the narrative engine (or a justified discard).

#### Scenario: Translated Economy Lesson

- **WHEN** the research documents Albion's regional markets
- **THEN** a candidate translation SHALL exist (e.g., prices/scarcity as a world tick trigger) or a discard with rationale

#### Scenario: Verifiable Source

- **WHEN** a lesson card states a number or game rule
- **THEN** the card SHALL cite the public source (URL) and verification date

### Requirement: Reverse Engineering of GTA San Andreas Mechanics

The research system SHALL document the systemic mechanics of GTA San Andreas that sustain the feeling of a living world: CJ's stats (respect, stamina, muscle, driving skill), gang territories with war and takeover, NPC and traffic routines, the wanted level with escalating police response, and the stack of worlds (city → countryside → desert) with progressive story gating. Each mechanic SHALL generate a lesson card in the same format as Albion.

#### Scenario: Translated NPC Routine

- **WHEN** the research documents the daily routine of pedestrians/NPCs
- **THEN** a mapping to npc-minds NPC agendas (schedules, objectives) or a justified discard SHALL exist

#### Scenario: Response Escalation

- **WHEN** the research documents the wanted level (1–6 stars)
- **THEN** the card SHALL propose how an escalating consequence of player actions could appear in world ticks

### Requirement: Reverse Engineering of MUD (Multi-User Dungeon) Mechanics

The research system SHALL document, from public sources (documentation and wikis of the DikuMUD/MOO/MUSH families, RPI MUDs), the text-based multi-user world mechanics relevant to world-building: a 24/7 persistent world that evolves while the player is offline, the world as a network of rooms with named exits and descriptions revealed on demand (`look`), presence and social communication (`who`, `say`, `emote`, channels), enforced roleplay (RPI), and collaborative world authoring (online OLC/builders, programmable MOO/MUSH worlds). Each mechanic SHALL generate a lesson card in the same format as the other tracks.

#### Scenario: The World Evolves Without the Player

- **WHEN** the research documents the persistent multi-user world (events occurring while the player is offline)
- **THEN** a candidate translation to world-simulation off-screen ticks or a discard with rationale SHALL exist

#### Scenario: The Room as the Unit of World

- **WHEN** the research documents rooms with named exits and descriptions on command
- **THEN** the card SHALL propose a mapping to LOCATION story cards with on-demand inspection (scenario-authoring/narrative-engine) or a justified discard

#### Scenario: Collaborative Authoring

- **WHEN** the research documents OLC/builders or programmable worlds (MOO/MUSH)
- **THEN** the card SHALL evaluate what the in-world authoring experience teaches about the scenario builder (frontend-ui/scenario-authoring)

### Requirement: Reverse Engineering of CyberCode Online Mechanics

The research system SHALL document, from public sources (the open-source repository dexterhuang/cybercodeonline — README, CONTRIBUTING, UpdateNote — plus the live game), the world-building-relevant mechanics of CyberCode Online (browser/mobile text-based cyberpunk MMORPG): the casual AFK/idle core loop (tasks, leveling, crafting advancing without continuous player attention), the procedural generation of enemies, dungeons and locations from community-contributed corpora (word lists, dungeon layout structure masks, procedural equipment names), and lore (item/scenario/dungeon) as a first-class, multilingual community contribution channel. Each mechanic SHALL generate a lesson card in the same format as the other tracks.

#### Scenario: Procedural World from Contributed Corpora

- **WHEN** the research documents enemies/dungeons/locations generated from user-contributed lists and structure masks
- **THEN** the card SHALL propose what this teaches about community-authored story card corpora and combinatorial variety (scenario-authoring/plot-generation) or a justified discard

#### Scenario: World Moves While AFK

- **WHEN** the research documents the idle/AFK progression loop
- **THEN** the card SHALL evaluate its alignment with off-screen ticks and timeskip (world-simulation) or discard with rationale

#### Scenario: Verifiable Source

- **WHEN** a lesson card claims a mechanic or number about the game
- **THEN** the card SHALL cite the public source (repository path or URL) and verification date

### Requirement: Albion-in-Life-RP Hybrid Synthesis

The research system SHALL document the cross-game synthesis of Albion Online systems transposed onto a real-life RP world (GTA San Andreas/RP style): risk bands reinterpreted as state presence per region (financial district = Blue with cameras and fast response; industrial/port = Yellow; periphery = Red; no-signal rural zones = Black with no reliable map/info), full loot domesticated as carry-only material loss, regional markets per neighborhood, guild territory reinterpreted as faction-controlled districts with protection/commerce income, seasons as elected government terms, spec-by-use as practice-based skill, and inter-city logistics as freight routes with ambush risk. The synthesis SHALL surface what each side fixes: Albion's closed economy solves RP inflation; RP's sacred character life domesticates Albion's cheap death; player institutions legalize scheduled territory wars.

#### Scenario: Hybrid Translation Cards

- **WHEN** the synthesis is documented
- **THEN** lesson cards SHALL exist for the three novel translations: closed economy with faucets/sinks (mmo-game), carry-only material consequence (mmo-game), and declared territory wars via player-run institutions (mmo-game)
- **AND** each card SHALL cite both source games and public sources

#### Scenario: Tension Resolution Recorded

- **WHEN** the synthesis identifies a design tension (cheap death vs. sacred life, scheduled vs. emergent conflict, systemic vs. character depth)
- **THEN** the resolution mechanism SHALL be recorded as part of the card

### Requirement: Reverse Engineering of GTA V RP Worldwide Mechanics

The research system SHALL document, from public sources (server sites and wikis: nopixel.net, cidadealta.gg, gta5rp.com, gta.world; platform browsers: rage.mp/servers, forge.plebmasters.de), the mechanics of the worldwide GTA V roleplay ecosystem (private RP cities on FiveM in the West/Brazil, RAGE MP in Russia/CIS): whitelist/allowlist gates (application, interview, paid tiers), the IC/OOC rule set (RDM/VDM, metagaming, powergaming, New Life Rule), player-run institutions (police, EMS, lawyer, judge, press), player-driven economies and gangs/factions, staff arbitration with seasonal storytelling arcs, and the per-country differentiation (NoPixel's story-first streamer culture, Brazil's streamer-founded cities with paid convenience tiers, Russia's voice-integrated massive servers, GTA World's strict text RP with 1M+ registered players). Each mechanic SHALL generate a lesson card in the same format as the other tracks, with candidate translations mapped where applicable to mmo-game.

#### Scenario: Cultural Shard Lesson

- **WHEN** the research documents per-country/per-community server differentiation (same world, different rules, tone and language)
- **THEN** the card SHALL propose a translation to cultural/regional shards over one uniform world (mmo-game) or a justified discard

#### Scenario: Player Institutions Lesson

- **WHEN** the research documents player-run institutions (police, EMS, press) as world state operators
- **THEN** the card SHALL evaluate institutional roles occupied by players with persisted minds (player-minds) instead of NPCs (mmo-game/npc-minds)

#### Scenario: New Life Rule Maps to Witness Filter

- **WHEN** the research documents the New Life Rule (dead characters forget the events of their previous death)
- **THEN** the card SHALL record the convergence with the engine's witness filter and memory pyramid, and what the RP implementation teaches

#### Scenario: Whitelist Maps to Protections

- **WHEN** the research documents whitelist gates as community quality/protection mechanisms
- **THEN** the card SHALL map them to age-banding trays and avatar-mirror consent gates

#### Scenario: Platform Risk Lesson

- **WHEN** the research documents platform dependency risk (Rockstar/Take-Two action against RAGE MP threatening the Russian scene)
- **THEN** the card SHALL record the argument for the self-owned engine path (d3wasm) as mitigation

#### Scenario: Verifiable Source

- **WHEN** a lesson card claims a mechanic about a specific server or country scene
- **THEN** the card SHALL cite the public source (URL) and verification date

### Requirement: d3wasm-Based Prototype of the Final Engine for World-Building

The project SHALL include a playable in-browser prototype built on **d3wasm** (github.com/gabrielcuvillier/d3wasm — the id Tech 4 / Doom 3 engine ported to WebAssembly via Emscripten with a full WebGL renderer backend, GPL-3.0) as the prototype of the game's final engine. The prototype serves as a world-building laboratory — dark corridors, dynamic flashlights, shadows, interactive lore terminals, positional audio, and script triggers — where every level design element SHALL teach a lesson mappable to the narrative engine (e.g., terminal with lore ≈ story card; script trigger ≈ plot seed; lighting that guides ≈ narrative emphasis). Studying and extending the d3wasm codebase (`neo/` engine sources) SHALL also produce engine-architecture lesson cards (renderer, asset pipeline, scripting, GUI/terminal systems) informing the final engine decision.

#### Scenario: The Prototype Loads in the Browser

- **WHEN** the prototype is opened in a modern browser (no native build, no plugins)
- **THEN** it SHALL render a first-person 3D scene with dynamic lighting at 30+ FPS on common hardware, running on the d3wasm WebAssembly/WebGL engine

#### Scenario: Interaction with Lore

- **WHEN** the player interacts with a prototype terminal
- **THEN** the displayed lore text SHALL be mapped to a world-building lesson card

#### Scenario: Engine Architecture Lessons

- **WHEN** the d3wasm codebase (id Tech 4 subsystems: scripting, GUI, asset pipeline, renderer) is studied
- **THEN** lesson cards SHALL capture which architectural decisions apply to the final engine of a narrative RPG (or a justified discard)

#### Scenario: GPL Boundary Is Respected

- **WHEN** the prototype incorporates d3wasm engine code (GPL-3.0)
- **THEN** the prototype's own code SHALL be licensed GPL-3.0-compatible
- **AND** no original Doom 3 game assets (maps, textures, models, sounds, `.pk4` content) SHALL enter the repository — original or freely licensed assets only
- **AND** the trade-off that a final engine derived from d3wasm inherits GPL-3.0 copyleft SHALL be documented before adoption

### Requirement: Versioned Lesson Cards

Lessons from the tracks (Albion, GTA SA, GTA V RP, MUDs, CyberCode, Doom 3) SHALL be persisted in a versioned dataset (`data/worldbuilding/lessons.json`) with fields: source game, mechanic, evidence/source, candidate translation, status (proposed/accepted/discarded), and target spec. Accepted cards SHALL reference the target spec requirement that absorbs the lesson.

#### Scenario: Traceable Accepted Card

- **WHEN** a card is marked as accepted
- **THEN** a reference to the target spec and requirement that incorporated it SHALL exist
- **AND** the dataset SHALL be loadable without network dependency

### Requirement: No Asset Violations

The research program SHALL use only documentary observation of mechanics (public sources) and original implementation in the prototype; no asset, code, model, texture, or audio extracted from the reference games SHALL enter the repository. The single licensed-code exception is the d3wasm engine itself (GPL-3.0, documented), used as the prototype's engine base; game assets remain original or free with a documented permissive license — the GPL-3.0 of the engine code does not extend to using proprietary game data.

#### Scenario: Asset Audit

- **WHEN** the prototype includes a model or texture
- **THEN** the provenance/license SHALL be documented in the repository
