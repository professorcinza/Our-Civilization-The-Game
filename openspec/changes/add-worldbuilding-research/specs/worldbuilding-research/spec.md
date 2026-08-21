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

### Requirement: Doom 3-Style WebGL Prototype for World-Building

The project SHALL include a playable in-browser prototype (WebGL/OpenGL ES via Three.js or raw WebGL, no plugins) inspired by Doom 3 — dark corridors, dynamic flashlights, shadows, interactive lore terminals, positional audio, and script triggers — used as a world-building laboratory: every level design element SHALL teach a lesson mappable to the narrative engine (e.g., terminal with lore ≈ story card; script trigger ≈ plot seed; lighting that guides ≈ narrative emphasis).

#### Scenario: The Prototype Loads in the Browser

- **WHEN** the prototype is opened in a modern browser (no native build)
- **THEN** it SHALL render a first-person 3D scene with dynamic lighting at 30+ FPS on common hardware

#### Scenario: Interaction with Lore

- **WHEN** the player interacts with a prototype terminal
- **THEN** the displayed lore text SHALL be mapped to a world-building lesson card

### Requirement: Versioned Lesson Cards

Lessons from the tracks (Albion, GTA SA, MUDs, Doom 3) SHALL be persisted in a versioned dataset (`data/worldbuilding/lessons.json`) with fields: source game, mechanic, evidence/source, candidate translation, status (proposed/accepted/discarded), and target spec. Accepted cards SHALL reference the target spec requirement that absorbs the lesson.

#### Scenario: Traceable Accepted Card

- **WHEN** a card is marked as accepted
- **THEN** a reference to the target spec and requirement that incorporated it SHALL exist
- **AND** the dataset SHALL be loadable without network dependency

### Requirement: No Asset Violations

The research program SHALL use only documentary observation of mechanics (public sources) and original implementation in the prototype; no asset, code, model, texture, or audio extracted from the reference games SHALL enter the repository. The Doom 3-like prototype SHALL use original or free assets only with a documented permissive license.

#### Scenario: Asset Audit

- **WHEN** the prototype includes a model or texture
- **THEN** the provenance/license SHALL be documented in the repository
