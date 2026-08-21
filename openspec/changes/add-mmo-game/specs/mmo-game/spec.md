## ADDED Requirements


### Requirement: Final Product Is a Lore-Based MMORPG

The final product SHALL be a Role-Playing MMORPG whose canonical world and content derive from the lore specified in the project (scenario lore cards, worldbuilding volumes, doctrine regiments, military forces catalog). The narrative engine (memory pyramid, world ticks, plot seeds, npc-minds, auditor) SHALL remain the simulation core; the MMO layer adds multiplayer presence on top of it, not a separate game.

#### Scenario: Lore Is Canonical

- **WHEN** any MMO content (zone, faction, NPC, item) is authored
- **THEN** it SHALL trace back to lore defined in the specs' source material (story cards, worldbuilding docs) or enter through the scenario-authoring pipeline
- **AND** content that contradicts established canon SHALL be rejected in review

#### Scenario: Engine Powers the MMO

- **WHEN** the MMO world simulates (memory, ticks, plots, NPC minds)
- **THEN** it SHALL use the specified engine systems rather than bespoke MMO logic

### Requirement: Persistent Multiplayer World

The world SHALL be persistent and shared: it continues to evolve off-screen (world-simulation ticks) while any given player is offline, and events caused by other players SHALL be observable later (rumors, journal entries, world changes) — applying the MUD lessons already captured in worldbuilding-research.

#### Scenario: World Moves While a Player Is Offline

- **WHEN** a player returns after an absence
- **THEN** the world state SHALL reflect ticks and other players' consequences that occurred in the interval
- **AND** the return SHALL surface those changes through narrative means (journal, world memory, NPC speech), not raw logs

#### Scenario: Player-Consequence Visibility

- **WHEN** one player's action changes the world (economy, territory, NPC fate)
- **THEN** other players SHALL be able to encounter that consequence in their own narration

### Requirement: Browser Client on the d3wasm Engine Path

The game client SHALL follow the engine path specified in worldbuilding-research: prototype on d3wasm (WebAssembly + WebGL id Tech 4) with a documented GPL-3.0 trade-off decision before the final engine is adopted; the MMO client remains browser-first (no native install required).

#### Scenario: Client Runs in the Browser

- **WHEN** a player opens the game in a modern browser
- **THEN** the client SHALL run without plugins or native installation

### Requirement: Narrative-First Progression

The MMORPG SHALL keep the engine's narrative-first rules: no HP bars, mana or grind; progression measured in memory (crystals), journal, relationships and world standing; combat resolved by the creativity score — even with many players online.

#### Scenario: No Grind Leaks In

- **WHEN** multiplayer systems are designed (grouping, shared quests, economy)
- **THEN** they SHALL NOT introduce numeric grind loops (XP bars, repetitive reward cycles) contradicting the narrative-first invariant

### Requirement: Social Layer with Roleplay Integrity

The social layer SHALL provide presence, communication and cooperation between players (seeing who is present, talking, acting together in a scene), informed by the MUD/RPI lessons: roleplay integrity expectations and consent boundaries, with avatar-mirror and age-banding protections applying to what other players can see and say to each other.

#### Scenario: Presence and Speech

- **WHEN** two players share a location
- **THEN** each SHALL perceive the other's presence and in-character speech/emotes in the narration
- **AND** out-of-character channels SHALL be clearly separated from in-world speech

#### Scenario: Bands and Mirror Protections Carry Over

- **WHEN** a minor-band player shares the world with adult-band players
- **THEN** the age-banding tray SHALL govern what content reaches them
- **AND** avatar-mirror consent and the LGPD deny-list SHALL apply to multiplayer visibility of personal data

### Requirement: Community Contribution Channel

Following the CyberCode Online lesson, the MMO SHALL treat community-contributed content (lore fragments, scenario seeds, procedural corpora) as a first-class, moderated channel entering through the scenario-authoring pipeline — never directly mutating canon.

#### Scenario: Moderated Contribution

- **WHEN** a community contribution is submitted
- **THEN** it SHALL pass scenario-authoring validation and review before becoming visible in the world

### Requirement: Scale Targets for the Open World (v1)

The MMO SHALL meet these v1 measurable scale targets on the d3wasm + narrative-engine hybrid (targets are engineering estimates recorded as contracts, revisable by future changes with measured data): 1,000–3,000 concurrent players per open map; per-client visible characters capped by interest management at ~100 rendered at 30+ FPS on common hardware; thousands of deterministic routine NPCs per map; tens up to ~1–2 hundred LLM-alive NPC minds per region; and a per-narrated-turn LLM cost envelope in the ~US$ 0.01–0.03 range, with the ~US$ 0.2–0.6 per active player-hour figure as the planning budget. The bottleneck order recorded: LLM throughput/cost first, client rendering second, world simulation last.

#### Scenario: Full Map Under Load

- **WHEN** 3,000 players are online in one open map
- **THEN** each client SHALL render at most ~100 characters in its area of interest at 30+ FPS
- **AND** the world simulation SHALL remain responsive (no synchronous LLM dependency in the moment-to-moment path)

#### Scenario: Per-Turn Cost Stays in Envelope

- **WHEN** a narrated turn completes (narrator + auditor + crystallization + tick)
- **THEN** its LLM cost SHALL be measured and tracked against the ~US$ 0.01–0.03 envelope, with prompt-caching zone hits reported

### Requirement: Hybrid Simulation Layers

The simulation SHALL be layered so scale does not route through the LLM: (a) a deterministic moment-to-moment layer (movement, presence, short speech) with server authority and no LLM calls; (b) an LLM narrative-event layer invoked on significant player decisions and world beats only; (c) a shared NPC-mind pool per region with the witness filter governing what each NPC knows about each player. The open world SHALL partition into scenes/regions (the MUD room-lattice model), each region carrying its own LLM call budget.

#### Scenario: Movement Never Calls the LLM

- **WHEN** a player moves, emotes briefly or perceives presence
- **THEN** the interaction SHALL be handled entirely by the deterministic layer
- **AND** no LLM call SHALL be triggered

#### Scenario: Region LLM Budget

- **WHEN** a region's LLM call budget is exhausted
- **THEN** narrative events in that region SHALL queue or degrade gracefully (deterministic narration fallback) instead of blocking the deterministic layer

### Requirement: d3wasm Netcode Gap Is the Headline Risk

Adapting d3wasm (single-player port, no networking) to the MMO SHALL require building from scratch: client prediction, server authority, snapshotting and interest management. This netcode layer is the largest single engineering risk of the engine path and SHALL be load-tested against the v1 scale targets before those targets count as met.

#### Scenario: Load Test Before Scale Sign-Off

- **WHEN** the v1 scale targets are claimed as met
- **THEN** a load test report (concurrent players, visible entities, FPS, LLM concurrency and cost) SHALL exist as evidence

### Requirement: Cultural Shards Over the Same Canon

Drawing from the GTA V RP worldwide lesson, the MMO SHALL support cultural/regional shards: communities playing the same canonical world with their own rules, tone and language, rather than one uniform world-for-all. Shard-specific behavior SHALL be configuration over the shared canon (never forked lore), and cross-shard consequences MAY be limited by design.

#### Scenario: Same Canon, Local Culture

- **WHEN** a regional shard defines its own tone, language and house rules
- **THEN** the canonical lore SHALL remain identical across shards
- **AND** shard differences SHALL be declared configuration, reviewable against canon

#### Scenario: Community Gate Mirrors Protections

- **WHEN** a shard admits players through a community gate (application, invitation or tier)
- **THEN** the age-banding tray and avatar-mirror consent SHALL remain non-negotiable beneath the community layer

### Requirement: Player-Run Institutions

The MMO SHALL allow institutional roles (peacekeeping, medical, legal, press) to be occupied by players with persisted minds — the player-minds variant of npc-minds — making the world state partially community-operated, per the GTA V RP lesson. Player-held institutions SHALL be subject to the same narrative audit and canon rules as every other actor.

#### Scenario: Player Institution Operates World State

- **WHEN** a player on duty performs an institutional action (patrol, triage, ruling, reporting)
- **THEN** the action SHALL enter the event store and affect the world like any actor's
- **AND** the institution's conduct SHALL be auditable by the narrative auditor

#### Scenario: Institution Handover

- **WHEN** an institutional role changes hands between players
- **THEN** the persisted mind and standing of the institution SHALL carry over without losing memory of prior events

### Requirement: Closed Player-Driven Economy

Per the Albion-in-life-RP lesson, the MMO economy SHALL be closed and player-driven: no value spawned by NPC shops or infinite NPC jobs; goods and services produced by players (or the world simulation) with sinks draining value through lifelike costs (taxes, rent, utilities, insurance, maintenance); markets regional (per district/neighborhood) with prices allowed to diverge, and price/scarcity divergence usable as world-simulation tick signals.

#### Scenario: No Infinite Faucet

- **WHEN** a player earns money
- **THEN** the value SHALL trace to another actor's spending or world production, never to an infinite NPC source
- **AND** sinks (taxes, rent, maintenance) SHALL exist that drain value at a tunable rate

#### Scenario: Regional Price Divergence Signals the World

- **WHEN** prices diverge between districts beyond a threshold
- **THEN** the world simulation MAY use that divergence as a tick trigger (shortage, conflict, blockade) surfaced through narration

### Requirement: Carry-Only Material Consequence

Per the hybrid synthesis, material consequence SHALL apply to what a character carries, never to the character's life: robbery under threat transfers carried goods (wallet, phone, purchases, vehicle); stored, banked or insured assets remain safe; character death remains governed by RP protections (sacred life) — the fear is losing the cargo, the car, the month's money, not the person.

#### Scenario: Robbery Transfers Carried Goods Only

- **WHEN** a robbery under threat concludes per RP rules
- **THEN** only carried items and the vehicle involved SHALL transfer
- **AND** banked, stored and insured assets SHALL be untouched

#### Scenario: Insurance as Sink

- **WHEN** a player insures goods or vehicles
- **THEN** premiums SHALL act as an economy sink and claims SHALL restore value without creating new money beyond the insured amount

### Requirement: Declared Territory Wars via Player Institutions

Faction-controlled territory SHALL grant passive income (protection/commerce) and SHALL change hands only through wars declared via the player-run institutions (mayoralty, judgeship, peacekeeping): declaration, time window and engagement rules recorded in the event store — legalizing scheduled conflict (the Albion GvG lesson) inside the RP frame instead of ad-hoc staff arbitration.

#### Scenario: War Requires Declaration

- **WHEN** a faction attempts a territory takeover
- **THEN** a declaration SHALL exist, approved through the competent player institution, with time window and engagement rules
- **AND** undeclared mass conflict SHALL be treated as a rule violation subject to audit

#### Scenario: Territory Income Is Simulation-Wired

- **WHEN** a faction holds a territory
- **THEN** its passive income SHALL flow through the economy (taxes/commerce), not spawn new value outside the closed economy
