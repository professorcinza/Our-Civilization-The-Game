# OpenSpec Consolidated — Project Lunar

> Generated on 2026-08-21 from `openspec/` (spec-driven schema, en).
> The source of truth remains the original `openspec/` tree; this file is a consolidated snapshot for reading.

## Index

- [1. Project context](#1-project-context)
- [2. Specs (current requirements)](#2-specs-current-requirements)
  - [age-banding](#age-banding)
  - [avatar-mirror](#avatar-mirror)
  - [combat-system](#combat-system)
  - [event-persistence](#event-persistence)
  - [frontend-ui](#frontend-ui)
  - [game-api](#game-api)
  - [inventory-system](#inventory-system)
  - [journal-system](#journal-system)
  - [knowledge-graph](#knowledge-graph)
  - [llm-routing](#llm-routing)
  - [memory-system](#memory-system)
  - [military-forces-catalog](#military-forces-catalog)
  - [narrative-audit](#narrative-audit)
  - [narrative-engine](#narrative-engine)
  - [npc-minds](#npc-minds)
  - [opening-generation](#opening-generation)
  - [plot-generation](#plot-generation)
  - [prompt-caching](#prompt-caching)
  - [scenario-authoring](#scenario-authoring)
  - [world-simulation](#world-simulation)
  - [worldbuilding-research](#worldbuilding-research)
- [3. Changes (work in progress)](#3-changes-work-in-progress)
  - [add-engine-core](#add-engine-core)
  - [add-military-forces-catalog](#add-military-forces-catalog)
  - [add-worldbuilding-research](#add-worldbuilding-research)
  - [fix-auditor-agency-false-positive](#fix-auditor-agency-false-positive)

## 1. Project context

# Project Lunar — local-first narrative RPG engine

Language: en
All artifacts must be written in English.
Keep OpenSpec structural headings and SHALL/MUST keywords in English.
Game content (scenarios, prompts, crystals, journals, tags) remains
bilingual (en + pt-br) by design — that is content, not artifacts.

## What the system is
Project Lunar is a narrative RPG ENGINE (not a single game): authors create
scenarios (lore, NPCs, locations, factions, setup questions) and players
live adventures narrated by LLMs, with 4-level persistent memory (crystal
pyramid), a reactive world, creativity-based combat and AI-generated
cold-opens. Bilingual (en + pt-br) in every prompt, crystal, journal and
tag. No HP, mana or grind — narrative only.

## Stack
- Frontend: React 19 + Vite 7 + Zustand 5 + Tailwind 3 (frontend/)
- Backend: Python 3.10+ + FastAPI (backend/app/) — central orchestrator is
  GameSession (services/game_session.py)
- Persistence: SQLite event-sourced (events.db, scenarios.db, traces.db)
- Graph: Neo4j 5 (Docker) + Graphiti-core (experimental)
- LLM: litellm — DeepSeek (1M ctx), Anthropic Claude (1M/200k), OpenAI
  gpt-5.6-sol (372k); optional CLIProxyAPI proxy (port 8318)
- Narrative delivery: SSE (POST /api/game/action) with inline control tags
  ([MODE], [JOURNAL], [INVENTORY], [POWER], [CRYSTAL], [PLOT_AUTO],
  [TRUNCATE_CLEAN], [USAGE], [TRACE], [DONE])

## Capability map (openspec/specs/)
- scenario-authoring: scenarios, setup questions, story cards, interpolation
- opening-generation: fixed vs AI-generated cold-open
- narrative-engine: mode detection, prompts, streaming, open scene window
- prompt-caching: zones 0/1/2 + cloaking (PHASE 2)
- memory-system: SHORT→MEDIUM→LONG→MEMORY pyramid + RAG + witness filter
- combat-system: 40/40/20 creativity score + power levels
- npc-minds: private thoughts, decay, knowledge boundaries
- world-simulation: off-screen world ticks + timeskip
- plot-generation: automatic triggers, cooldowns, plot lock
- journal-system: automatic detection of significant events
- inventory-system: inline tags [ITEM_ADD|USE|LOSE]
- knowledge-graph: Neo4j + canonical name resolution
- llm-routing: multi-provider, model policy, token accounting
- narrative-audit: context-aware post-hoc auditor (PHASE 3b)
- game-api: REST + SSE contract
- event-persistence: event sourcing + full state rebuild
- frontend-ui: panels, wizard, builder, devtools
- military-forces-catalog: real catalog of units/squadrons/specializations
  of the Brazilian Armed Forces + world elites + ideal soldier model, with
  sources
- avatar-mirror: the player as an avatar of themselves (all ages):
  mirroring levels with consent, narrative translation (CTN), LGPD
  deny-list, context budget (~1k tokens) always in the volatile zone
- deployment: Docker Compose stack (uv backend + nginx frontend with /api
  and SSE proxy, Neo4j in optional profile), credentials outside the
  image, persistence across restarts, host access for the Hermes proxy

## Project invariants (learned in A/B — docs/fase3a_ab.md, fase3b_ab.md)
- Never feed the narrator's own raw prose back as history beyond the open
  scene (tic auto-conditioning).
- Pink-elephant anti-pattern: never give literal examples of vices to
  avoid.
- [ITEM_*] tags and @Name mentions are load-bearing side effects: the
  Auditor must never alter them in a rewrite.
- All in-memory state is rebuilt from the event store on every GET.
- Feature flags (LUNAR_FEATURE_*, default ON) degrade to legacy behavior
  instead of breaking.

## 2. Specs (current requirements)

<!-- source: specs/age-banding/spec.md -->

### age-banding

#### Purpose

Content trays per age band (A: up to 12; B: 13–17; C: 18+) applied to the existing scenarios: which scenarios each band may play, which narrative adaptations each band receives, and how the Avatar Mirror band automatically governs the experience — same engine, complete worlds, proportional protection.

#### Requirements

##### Requirement: Bands and scenario compatibility

The system SHALL declare, per scenario, the per-band compatibility (A/B/C) with full or adapted mode, automatically applying the tray of the Mirror profile's band when the band differs from the scenario's native one.

###### Scenario: Native band

- **WHEN** the profile belongs to the same band as the scenario's native tray
- **THEN** the scenario SHALL run in full mode, with no extra injunctions

###### Scenario: Scenario blocked for the band

- **WHEN** a Band A profile tries to open a scenario classified B/C-only
- **THEN** the system SHALL refuse with a friendly explanation and suggest scenarios for the band

##### Requirement: Per-tray narrative injunctions

Each tray SHALL carry content injunctions injected into the narrator's prompt (volatile zone, never cached): Band A — no explicit violence, no on-screen deaths, redeemable antagonists, fears resolved with agency, accessible vocabulary; Band B — real tension allowed, no graphic torture or detailed cruelty, moral dilemmas with room for choice; Band C — full, per the scenario's tone.

###### Scenario: Child in a military scenario

- **WHEN** a Band A profile plays Brasil em Armas (adapted mode)
- **THEN** the narrator SHALL receive the Band A injunction alongside the scenario's tone
- **AND** selection and training SHALL appear as personal overcoming, never as humiliation or harm

###### Scenario: Teenager in Exercício Convergência

- **WHEN** a Band B profile plays Guerra das Mentes (adapted mode)
- **THEN** coercive historical cases (KUBARK and the like) SHALL stay out of the text
- **AND** the doctrinal mechanics (TAA, OPSEC, clean MILDEC) SHALL remain intact

##### Requirement: Mechanics preservation across trays

Per-band adaptation SHALL change representation, never mechanics: the regiment's processes (TAA before product, the 5 OPSEC steps, the MILDEC objective) remain identical across all trays — what changes is the narrative surface.

###### Scenario: Same doctrine, different surface

- **WHEN** two profiles from bands A and C play the same scenario adapted
- **THEN** both SHALL exercise the same decision-making process
- **AND** neither SHALL receive the other tray's content

##### Requirement: Mirroring limited by band

The tray SHALL limit the mirroring level per the avatar-mirror spec (A: levels 0–1; B: 0–2; C: 0–3), and scenarios with dimensions in the setup SHALL replace dimension questions with neutral defaults when the band does not allow them.

###### Scenario: Setup with a blocked dimension

- **WHEN** a Band B profile opens A Comitiva (the strong_dimension question)
- **THEN** the question SHALL remain (it is fictional, about the character)
- **AND** no Mirror profile data beyond what the band allows SHALL fill it in

##### Requirement: Auditable classification table

Scenario × band compatibility SHALL be versioned data (age_bands.json), with a per-scenario rationale, and every change SHALL be recorded in the openspec change.

###### Scenario: Tray lookup

- **WHEN** the frontend lists scenarios for a Band B profile
- **THEN** the list SHALL mark native/adapted/blocked per age_bands.json

##### Requirement: No condescension

Band A and Band B injunctions SHALL elevate, not impoverish: the worlds remain complete (jungle, selection, espionage) with proportional treatment of the theme — the yardstick is children's and young-adult adventure literature, not the infantile one.

###### Scenario: Real adventure for a child

- **WHEN** Band A plays O Cidadão do Futuro adapted
- **THEN** the world's fractures (the snow, the Janela) SHALL appear as legitimate questions and choices
- **AND** the narrator SHALL NOT dilute the theme into irrelevance

<!-- source: specs/avatar-mirror/spec.md -->

### avatar-mirror

#### Purpose

The Avatar Mirror: the real player enters the game as an avatar of themselves — for all ages — with self-declared status translated into narrative traits. It defines HOW MUCH information about the person enters as game context, in which consent layer, with what absolute prohibitions, how it crosses the LLM boundary and how it is forgotten. Master principle: the game asks, never deduces — no behavioral profiling.

#### Requirements

##### Requirement: Mirroring layers with granular consent

The system SHALL offer four player mirroring levels, chosen explicitly on first boot and changeable at any time: Level 0 (anonymous — fully fictional avatar), Level 1 (playful essential: display name, language, age band), Level 2 (narrative traits: interests, opt-in fears, strengths on a playful 1–5 scale) and Level 3 (dimensions: physical, mental, cognitive and psychological self-assessment, each with separate opt-in).

###### Scenario: Anonymous start

- **WHEN** the player chooses Level 0
- **THEN** no real personal information SHALL enter any prompt, event or card — the avatar is a fictional character like any NPC

###### Scenario: Opt-in per dimension

- **WHEN** the player enables Level 3 but refuses the psychological dimension
- **THEN** the game context SHALL receive only the consented dimensions
- **AND** the refusal SHALL NOT degrade any game mechanic

##### Requirement: Narrative Translation Layer (CTN)

Every consented personal data point SHALL be translated into a fictional trait before entering any prompt — the real data stays in the backend; only the narrative trait leaves. The CTN uses the universe's vocabulary of dignity (Zero Exclusion): reduced mobility becomes "Remote Interface", hyperfocus becomes "Pattern Scan", never raw clinical terms.

###### Scenario: Translated trait in the prompt

- **WHEN** the player declared reduced mobility (Level 3, physical dimension)
- **THEN** the narrator SHALL receive "avatar operates via Remote Interface — zero latency, its own body wisdom" and SHALL NOT receive diagnosis, clinical term or raw data

###### Scenario: The translation is a firewall

- **WHEN** any LLM request is assembled
- **THEN** the payload SHALL contain only fictional traits from the CTN
- **AND** real personal data fields SHALL remain in the backend, outside the request

##### Requirement: Absolute data prohibitions

The system SHALL reject at profile validation — regardless of consent — the fields: clinical diagnosis or named health condition, medications, biometric data and body measurements, real location (beyond country/language), financial data, third-party names or data, identity documents and verifiable school/professional content.

###### Scenario: Prohibited field submitted

- **WHEN** any deny-list field appears in the profile
- **THEN** validation SHALL reject with a message explaining the policy
- **AND** no deny-list field SHALL exist in the schema as a hidden optional

##### Requirement: Age bands and consent

The system SHALL apply age bands with distinct rules: Band A (up to 12 years — only Levels 0–1, mandatory guardian consent, protective narrative, no traits or dimensions), Band B (13–17 — Levels 0–2, guardian consent when required by the jurisdiction) and Band C (18+ — up to Level 3 with granular opt-in). The band is declared, never inferred.

###### Scenario: Child attempts to enable dimensions

- **WHEN** a Band A profile requests Level 3
- **THEN** the system SHALL refuse with an explanation in child-friendly language
- **AND** SHALL suggest the anonymous or essential mode as the path

##### Requirement: Avatar context budget

The avatar context SHALL be compact and volatile: an Avatar Card of at most 400 tokens, an Avatar Crystal (memory of who the avatar is, distilled from events) of at most 600 tokens, and the set SHALL not exceed 5% of the active model's context window.

###### Scenario: Avatar in the volatile zone

- **WHEN** the zoned prompt is assembled
- **THEN** all avatar context SHALL stay in zone 2 (volatile)
- **AND** no avatar trait SHALL enter the cached zones 0/1, because ephemeral `cache_control` would create copies of the personal data at the provider for up to 1 hour

###### Scenario: Avatar Crystal grows

- **WHEN** the avatar's events accumulate identity ("I face fear of heights", "I protect my teammate")
- **THEN** the Avatar Crystal SHALL distill these facts within the budget limit, pruning the oldest
- **AND** the Crystal is the avatar's GAME memory — a game profile, never the person's behavioral profile

##### Requirement: Memory and forgetting boundary

The system SHALL treat the right to erase as first-class citizenship: deleting the avatar removes the profile, crystals and linked campaign events; the event store keeps only the translated traits (never the real data), and the retention policy publishes what exists and for how long.

###### Scenario: Erasing the mirror

- **WHEN** the player triggers "delete my avatar"
- **THEN** profile, consent, avatar crystal and mirrored campaigns SHALL be removed
- **AND** the system SHALL confirm the removal without retaining a derived copy

###### Scenario: Avatar travels between scenarios

- **WHEN** the same avatar enters another scenario
- **THEN** the consented traits SHALL be reused without new collection
- **AND** the new scenario SHALL NOT gain access to categories beyond those already consented

##### Requirement: Sensor boundary (self-contained game)

The system SHALL be self-contained: device sensors (camera, GPS, microphone, biometrics, motion) SHALL NOT be a source of game context at any mirroring level or age band. The only entry door for reality is text typed by the player ("player-as-sensor"): observations of the WORLD, opt-in, labeled as such — never data about the person or about third parties.

###### Scenario: Typed world observation

- **WHEN** the player records an observation of reality (a rumor, a headline, an event)
- **THEN** the system SHALL treat it as world context, opt-in and removable
- **AND** the record SHALL NOT contain or request location, image, audio or identification of third parties

###### Scenario: Feature attempts to activate a sensor

- **WHEN** any component requests access to camera, GPS, microphone or biometrics on the game's behalf
- **THEN** the system SHALL refuse by design (the boundary is binary, with no partial mode)
- **AND** the decision documented in world/fronteira_realidade_decisao.md is the architecture reference

<!-- source: specs/combat-system/spec.md -->

### combat-system

#### Purpose

Combat without HP, mana, or levels: actions evaluated by creativity/coherence/context with weights 40/40/20, anti-griefing against meta-gaming, a dynamic power scale anchored to the NPCs from the story cards, and an outcome imposed on the narrator (FAIL is FAIL).

#### Requirements

##### Requirement: Action evaluation on three axes

The system SHALL evaluate each combat action on creativity (40%), coherence (40%), and context (20%), producing a single final quality.

###### Scenario: Creative and anchored action

- **WHEN** the player describes a physically plausible, original action that uses the environment
- **THEN** the final quality SHALL reflect the three weighted scores

##### Requirement: Anti-griefing rejects meta-gaming

The system SHALL reject actions that claim victory by narrative fiat, authorial power, or god-modding, and physically impossible actions; surrender, withdrawal, and yielding ground are valid choices.

###### Scenario: "I kill them all instantly"

- **WHEN** the player declares instant victory over all opponents
- **THEN** the system SHALL reject the action with a reason in the language of the player's action
- **AND** the rejection SHALL be persisted as the narrator's reply for history context

###### Scenario: Surrendering is valid

- **WHEN** the player surrenders or retreats
- **THEN** the system SHALL accept the action as a legitimate combat choice

##### Requirement: Power scale anchored to story cards

When the scenario provides NPC story cards with power, the system SHALL build a WORLD POWER SCALE (top 25 + bottom 25 NPCs as anchors) to calibrate the estimation of new opponents on a 1–10 scale.

###### Scenario: New opponent calibrated

- **WHEN** an unlisted opponent appears in combat
- **THEN** their power SHALL be resolved anchored to the world scale
- **AND** the known power of opponents already faced SHALL be reused in future encounters

###### Scenario: Player power persists

- **WHEN** the player's power is evaluated with full context (once per campaign)
- **THEN** the value SHALL persist and be rebuilt from the event store after restart
- **AND** power changes SHALL be emitted as a `[POWER]` control tag

##### Requirement: Outcome imposed on the narrator

The system SHALL convert quality into an outcome (CRIT_SUCCESS, SUCCESS, FAIL, CRIT_FAIL) and inject the result into the narrator's input irrevocably.

###### Scenario: Failure is failure

- **WHEN** the outcome is FAIL or CRIT_FAIL
- **THEN** the narrator SHALL receive an explicit injection that the action FAILS and cannot describe victory
- **AND** CRIT_FAIL grants 2 extra actions to the NPC and CRIT_SUCCESS grants 1 extra action to the player

###### Scenario: Outcome logging

- **WHEN** an outcome is rolled
- **THEN** a COMBAT_RESULT event SHALL be persisted with outcome, quality, opponent, and powers

##### Requirement: Per-campaign toggle

Each campaign SHALL have a `combat_enabled` flag; when false, the mode detector never routes to COMBAT and the prompt omits the combat rules.

###### Scenario: Enabling combat mid-campaign

- **WHEN** the player toggles the flag in the campaign settings
- **THEN** the new preference SHALL take effect from the next turn onward

<!-- source: specs/event-persistence/spec.md -->

### event-persistence

#### Purpose

The persistence foundation: event sourcing in SQLite (events.db) with 12 canonical event types plus AI_OPENING_GENERATED, immutable events with witnesses, narrative time delta, and full reconstruction of all in-memory state (history, minds, journal, crystals, plot, powers) from the log.

#### Requirements

##### Requirement: Event store append-only

Every game effect SHALL be persisted as an immutable append-only event in SQLite, with type, payload, narrative time delta, location, entities and witnesses.

###### Scenario: Immutable event

- **WHEN** any code attempts to mutate an already recorded event
- **THEN** the mutation SHALL raise an error (events are a frozen namedtuple)

###### Scenario: Rewind removes only the turn pair

- **WHEN** rewind is executed
- **THEN** only the last PLAYER_ACTION + NARRATOR_RESPONSE pair SHALL be removed
- **AND** structural events derived from the removed pair SHALL be ignored in reconstruction

##### Requirement: Canonical event types

The log SHALL use exclusively the types PLAYER_ACTION, NARRATOR_RESPONSE, WORLD_TICK, COMBAT_ACTION, COMBAT_RESULT, PLOT_GENERATION, NPC_THOUGHT, JOURNAL_ENTRY, MEMORY_CRYSTAL, TIMESKIP, INVENTORY and POWER_LEVEL_UPDATE, plus AI_OPENING_GENERATED.

###### Scenario: New game effect

- **WHEN** an uncovered effect needs to be persisted
- **THEN** it SHALL be modeled as one of the existing types or SHALL require explicit extension of the enum

##### Requirement: Full state reconstruction

When accessing a campaign after a restart, the system SHALL reconstruct history, NPC minds, journal, memory crystals, plot lock state, player and opponent powers from the events.

###### Scenario: Backend restarted

- **WHEN** the backend restarts and the player resumes the campaign
- **THEN** all state SHALL be identical to the pre-restart state

###### Scenario: Crystallization cursor respected

- **WHEN** the history is reconstructed
- **THEN** the open scene window SHALL apply the same cursor-based cut applied in the live session

##### Requirement: Separate databases per responsibility

The system SHALL maintain separate SQLite databases: events.db (events), scenarios.db (scenarios, cards, campaigns, setup responses) and traces.db (per-turn LLM traces), each with a path overridable by environment variable.

###### Scenario: Custom paths

- **WHEN** `EVENT_DB_PATH`, `SCENARIO_DB_PATH` or `LLM_TRACE_DB_PATH` are set
- **THEN** the backend SHALL use the given paths

##### Requirement: Witnesses on events

Scene events SHALL carry the list of NPC witnesses of the represented scene; the player's presence is implicit and never recorded.

###### Scenario: Scene with two NPCs

- **WHEN** a narrator response has two NPCs present
- **THEN** the corresponding event SHALL list both as witnesses

<!-- source: specs/frontend-ui/spec.md -->

### frontend-ui

#### Purpose

The player's React 19 interface: a game canvas rendering prose and control tags, an action selector (DO/SAY/CONTINUE/META) with @-mention autocomplete, inspection panels (journal, minds, crystals, inventory, plot, map) and trace devtools.

#### Requirements

##### Requirement: Game canvas with control tags

The GameCanvas SHALL render the prose from the SSE stream and translate control tags into UI elements (mode badge, journal card, inventory toast, crystal alert, combat overlay).

###### Scenario: Combat overlay

- **WHEN** the stream delivers `[MODE]COMBAT`
- **THEN** the canvas SHALL display the combat overlay

###### Scenario: Unknown tag

- **WHEN** the stream delivers an unrecognized tag
- **THEN** the canvas SHALL render the text as ordinary prose without breaking

##### Requirement: DO/SAY/CONTINUE/META action selector

The ActionInput SHALL offer the four action verbs, with SAY inserting the speech verbatim before NPC reactions.

###### Scenario: Player speech

- **WHEN** the player uses SAY with text
- **THEN** the text SHALL appear literally in the narrative before the reactions

##### Requirement: @-mention autocomplete

The ActionInput SHALL offer NPC autocomplete with @, rendering mentions as @Full Name.

###### Scenario: Partial mention

- **WHEN** the player types @El
- **THEN** the autocomplete SHALL suggest NPCs whose names match the prefix

##### Requirement: Inspection panels

The frontend SHALL offer panels for the journal (with category filter), NPC minds (with editing), memory crystals (4 tiers), inventory, plot (manual generation) and world map (force graph).

###### Scenario: Mind editor

- **WHEN** the player saves the edit of a thought in the NpcInspector
- **THEN** the value SHALL be persisted via PUT to the API

###### Scenario: Force map

- **WHEN** the player opens the WorldMapModal
- **THEN** the graph SHALL be rendered with nodes colored by type

##### Requirement: Scenario builder and wizard

The frontend SHALL offer a ScenarioBuilder (metadata, tone, lore, setup questions, cards) and a SetupWizard (question flow with choice/text) before the game.

###### Scenario: Wizard with required question

- **WHEN** the player tries to start without answering a required question
- **THEN** the wizard SHALL block advancement

##### Requirement: Settings panel and devtools

The frontend SHALL offer a SettingsPanel (provider, model, temperature, max_tokens, per-campaign combat) and a DevtoolsPanel (per-turn usage summary and persisted LLM traces).

###### Scenario: Model switch

- **WHEN** the player changes the model in the SettingsPanel
- **THEN** the next action SHALL send the new model in the request

###### Scenario: Post-restart trace

- **WHEN** the devtools queries traces from previous turns
- **THEN** the panel SHALL read from the persisted traces route

<!-- source: specs/game-api/spec.md -->

### game-api

#### Purpose

The backend's HTTP contract: REST routes for scenarios, campaign, state, and devtools + the action SSE stream with inline control tags, configurable per request (provider, model, temperature, max_tokens, combat).

#### Requirements

##### Requirement: Action endpoint with SSE

`POST /api/game/action` SHALL stream the narration as Server-Sent Events (`text/event-stream`), ending with `[USAGE]`, `[TRACE]`, and `[DONE]`.

###### Scenario: Full flow of a turn

- **WHEN** the player sends an action
- **THEN** the stream SHALL deliver prose in `data:` lines followed by `data: [USAGE]...`, `data: [TRACE]...`, and `data: [DONE]`

###### Scenario: Per-request settings

- **WHEN** the action request includes provider/model/temperature/max_tokens/combat_enabled
- **THEN** the backend SHALL apply these values for this turn only
- **AND** the raw tone template SHALL be re-interpolated before reaching the narrator

##### Requirement: Inline control tags in the stream

The SSE stream SHALL use control tags for structured events: `[MODE]`, `[JOURNAL]`, `[INVENTORY]`, `[POWER]`, `[CRYSTAL]`, `[PLOT_AUTO]`, and `[TRUNCATE_CLEAN]`, each with a single-line JSON payload.

###### Scenario: Combat overlay

- **WHEN** the turn is classified as COMBAT
- **THEN** the frontend SHALL receive `[MODE]COMBAT` before the prose to display the overlay

###### Scenario: Crystal created during the turn

- **WHEN** crystallization fires during the turn
- **THEN** the stream SHALL emit `[CRYSTAL]` with tier and event count

###### Scenario: Truncation cleanup signaled

- **WHEN** the narrator's response arrives truncated and the system trims or completes the prose before delivering it
- **THEN** the stream SHALL emit `[TRUNCATE_CLEAN]` signaling to the frontend that the prose underwent truncation cleanup

##### Requirement: Per-campaign state routes

The backend SHALL expose per-campaign state reads: history, journal (filterable by category), npc-minds (with PUT/DELETE per NPC), characters, memory-crystals, inventory, world-graph, and graph-search.

###### Scenario: Reconstruction on GET

- **WHEN** any state route is queried after a backend restart
- **THEN** the session SHALL be rebuilt from events before responding

##### Requirement: Per-campaign game routes

The backend SHALL expose: rewind, timeskip, manual crystallize, generate (NPC/event/plot), inject-npc-seed, setup-answers, regenerate-opening, campaign settings (PATCH), and resolved scenario-view.

###### Scenario: Rewind

- **WHEN** the player triggers a rewind
- **THEN** the last pair of events (action+reply) SHALL be removed and all state SHALL be rebuilt consistently

##### Requirement: Scenario CRUD

`/api/scenarios` SHALL expose creation, listing, detail, deletion, JSON import/export, preview-opening, story-cards (POST/GET), and campaigns (POST/GET/DELETE).

###### Scenario: Creation with var_name validation

- **WHEN** creation receives duplicate var_names
- **THEN** the backend SHALL reject with a validation error

##### Requirement: Global settings and health

The backend SHALL expose `GET/POST /api/settings` (provider, model, temperature, max_tokens) and health probes (`/api/health`, `/api/health/neo4j`).

###### Scenario: Neo4j probe

- **WHEN** Neo4j is down
- **THEN** `/api/health/neo4j` SHALL return unavailable status with the error
- **AND** `/api/health` SHALL remain ok

##### Requirement: Per-campaign traces

The backend SHALL expose `GET/DELETE /api/game/{campaign_id}/traces` for reading and clearing persisted LLM traces.

###### Scenario: Paginated query

- **WHEN** the devtools requests the latest 25 traces
- **THEN** the route SHALL return the campaign's most recent traces

<!-- source: specs/inventory-system/spec.md -->

### inventory-system

#### Purpose

Narrative inventory: items are added, used and lost via the inline tags `[ITEM_ADD]`, `[ITEM_USE]`, `[ITEM_LOSE]` parsed from the LLM's prose, with event-sourced persistence, deduplication and manual adjustment through the panel.

#### Requirements

##### Requirement: Inline item tags

The narrator SHALL emit inline tags in the prose for item effects: `[ITEM_ADD:nome|categoria|origem]`, `[ITEM_USE:nome]` and `[ITEM_LOSE:nome]`, parsed as events after the response.

###### Scenario: Item acquired

- **WHEN** the prose contains `[ITEM_ADD:Chave de Cobre|chave|encontrada no baú]`
- **THEN** the item SHALL be added to the inventory with status carried
- **AND** the acquisition SHALL be emitted as an `[INVENTORY]` control tag to the frontend

###### Scenario: Item used

- **WHEN** the prose contains `[ITEM_USE:Chave de Cobre]` and the item is carried
- **THEN** the status SHALL change to used

###### Scenario: Item lost

- **WHEN** the prose contains `[ITEM_LOSE:Chave de Cobre]` and the item is carried
- **THEN** the status SHALL change to lost

###### Scenario: Bracket-resistant parser grammar

- **WHEN** an item name contains `]` or `|`
- **THEN** the parser SHALL use the grammar that prevents these characters from hiding a change of category or origin

##### Requirement: Deduplication

Adding an already carried item (same name, case-insensitive) SHALL be ignored.

###### Scenario: Double ADD

- **WHEN** the prose adds the same item twice with no use between the tags
- **THEN** only one entry SHALL exist with status carried

##### Requirement: Inventory in the prompt

The current inventory SHALL be injected into the PLAYER INVENTORY section of the narrator's prompt every turn.

###### Scenario: Item context

- **WHEN** the player has carried items
- **THEN** the prompt SHALL list them so the narrator can use them in the fiction

##### Requirement: Manual adjustment

The system SHALL allow manually adding and removing items via API and panel.

###### Scenario: Manual removal

- **WHEN** the player removes an item through the panel
- **THEN** the corresponding inventory event SHALL be persisted

##### Requirement: Tags are load-bearing in the audit

An audit rewrite SHALL preserve verbatim the multiset of item events from the original prose; changing any `[ITEM_*]` tag invalidates the rewrite.

###### Scenario: Auditor cannot drop a tag

- **WHEN** the auditor's rewrite omits an `[ITEM_ADD]` tag present in the original
- **THEN** the rewrite SHALL be rejected and the original prose kept

<!-- source: specs/journal-system/spec.md -->

### journal-system

#### Purpose

The campaign's automatic journal: an LLM evaluator identifies significant events in the narrative — discoveries, relationship changes, combat, decisions and world events — and records them with category, a summary in the campaign's language, and witnesses.

#### Requirements

##### Requirement: Automatic detection of significant events

After each turn, the system SHALL evaluate the narrative and record journal entries only for significant events.

###### Scenario: Common event

- **WHEN** the narrative contains no significant event
- **THEN** no entry SHALL be created

###### Scenario: Discovery

- **WHEN** the player discovers something important (a secret location, information, an item)
- **THEN** a DISCOVERY entry SHALL be recorded

##### Requirement: Canonical categories

Entries SHALL use exclusively the categories DISCOVERY, RELATIONSHIP_CHANGE, COMBAT, DECISION and WORLD_EVENT.

###### Scenario: Combat victory

- **WHEN** a combat ends
- **THEN** a COMBAT entry SHALL summarize the outcome

###### Scenario: Off-screen world change

- **WHEN** a world tick generates changes
- **THEN** a WORLD_EVENT entry SHALL record them in the journal

##### Requirement: Campaign language

Summaries SHALL be written in the campaign's configured language.

###### Scenario: pt-br campaign

- **WHEN** the campaign is in pt-br
- **THEN** the journal summary SHALL be written in Brazilian Portuguese

##### Requirement: Filter by category

The system SHALL allow filtering entries by category on read.

###### Scenario: Filtered query

- **WHEN** the panel requests only COMBAT entries
- **THEN** only entries of that category SHALL be returned

##### Requirement: Inherited witnesses

Each entry SHALL inherit the witness list from the narrator response that originated it.

###### Scenario: Entry with no NPC present

- **WHEN** the origin scene had no NPCs present
- **THEN** the entry SHALL have an empty witness list

##### Requirement: Player action log

When the action log is enabled, the system SHALL evaluate the player's action as a potential journal entry before narration.

###### Scenario: Pre-narration logging

- **WHEN** the player action log is enabled
- **THEN** the action SHALL be evaluated and emitted as a `[JOURNAL]` tag before the turn's prose

<!-- source: specs/knowledge-graph/spec.md -->

### knowledge-graph

#### Purpose

The world knowledge graph in Neo4j: entities (NPC, LOCATION, FACTION, ITEM, EVENT) extracted from the narrative with canonical name resolution, textual search, a snapshot for the world map, and experimental semantic search via Graphiti.

#### Requirements

##### Requirement: Narrative entity extraction

The system SHALL extract entities and relations from each narrator response and write them as nodes and edges in the campaign's graph.

###### Scenario: NPC mentioned for the first time

- **WHEN** the narrative introduces a new NPC with relations
- **THEN** the graph SHALL create the corresponding node and the edges to existing entities

##### Requirement: Canonical node types

Nodes SHALL use exclusively the types NPC, LOCATION, FACTION, ITEM and EVENT.

###### Scenario: New faction

- **WHEN** a faction is extracted from the narrative
- **THEN** the created node SHALL have type FACTION

##### Requirement: Canonical name resolution

The system SHALL resolve short names to canonical names before writing to the graph, avoiding duplicates of the same character.

###### Scenario: First name vs full name

- **WHEN** the narrative mentions "Elise" and the graph already has "Elise Halbrecht"
- **THEN** the mention SHALL resolve to the existing canonical node

##### Requirement: Graph snapshot for the map

The system SHALL provide a complete snapshot of the campaign's graph (nodes + edges) for visualization in the frontend's world map.

###### Scenario: Updated map

- **WHEN** the player opens the world map
- **THEN** the snapshot SHALL reflect all entities extracted so far

##### Requirement: Graph search

The system SHALL offer textual search for entities in the graph, with a local fallback when Graphiti is not available.

###### Scenario: Search without Graphiti

- **WHEN** graphiti-core is not installed or fails
- **THEN** the search SHALL degrade to local search on the Neo4j graph without breaking the route

##### Requirement: Relations as narrator context

A summary of the graph's relations SHALL be injected into the narrator's prompt for who-knows-whom consistency.

###### Scenario: WORLD RELATIONSHIPS section

- **WHEN** the graph has relevant relations
- **THEN** the prompt SHALL include the WORLD RELATIONSHIPS section

<!-- source: specs/llm-routing/spec.md -->

### llm-routing

#### Purpose

The multi-provider router over litellm: DeepSeek, Anthropic, and OpenAI (with an optional CLIProxyAPI proxy), a narrative vs. auxiliary model policy, per-model context windows, sampling guards, transient-failure retry, per-call token accounting, and an optional forensic dump.

#### Requirements

##### Requirement: Multiple provider support

The system SHALL support DeepSeek (deepseek-v4-flash/pro, 1M ctx), Anthropic (Claude 4.6/5 1M; 4.x 200k), and OpenAI (gpt-5.6-sol, 372k), with runtime switching via the settings panel without restart.

###### Scenario: Provider switch

- **WHEN** the player changes provider in the settings panel
- **THEN** the next action SHALL use the new provider
- **AND** no restart SHALL be required

##### Requirement: Narrative vs. auxiliary model policy

The router SHALL run narration on the chosen model and all other calls (audit, memory, journal, combat, NPC, plot, opening) on a cheaper auxiliary model from the same provider.

###### Scenario: Narration on Opus, the rest on Sonnet

- **WHEN** the player picks claude-opus-5 as the narration model
- **THEN** auxiliary calls SHALL run on claude-sonnet-5

##### Requirement: Per-model context window

The router SHALL know the context window of each supported model and expose it to size history, cards, and crystals; unknown models SHALL use a 200k fallback.

###### Scenario: Uncatalogued model

- **WHEN** a model with no catalog entry is selected
- **THEN** the window SHALL default to 200,000 tokens

##### Requirement: Per-model sampling guard

The router SHALL omit `temperature` for models that reject non-standard sampling parameters (the claude-opus-4-7/4-8/5 family, claude-sonnet-5, fable-5, mythos-5, gpt-5.6-sol).

###### Scenario: Call to a sensitive model

- **WHEN** a call is made to a model on the no-sampling list
- **THEN** the request SHALL omit temperature, top_p, and top_k

##### Requirement: Transient failure retry

The router SHALL retry calls that fail due to transient proxy/upstream failure with backoff (0.5s and 1.5s), totaling 3 attempts.

###### Scenario: Unstable proxy

- **WHEN** the first call to the proxy fails due to a transient timeout
- **THEN** the router SHALL retry up to 2 more times before propagating the error

##### Requirement: Per-action token accounting

The router SHALL accumulate per action the number of calls, input/output tokens, cache reads/creates, and time, displayed in the devtools via the `[USAGE]` tag.

###### Scenario: Per-turn summary

- **WHEN** a turn completes
- **THEN** the summary SHALL be emitted in the SSE stream and logged
- **AND** fire-and-forget calls that complete after the end of the stream SHALL be accounted for outside the turn snapshot

##### Requirement: Optional forensic dump

The router SHALL offer a forensic dump of every LLM call (full request + response + timing) in one JSON per call under `logs/llm_calls/`, enabled by `LUNAR_DUMP_LLM_CALLS=1`.

###### Scenario: Cost investigation

- **WHEN** the dump is active
- **THEN** each call SHALL produce exactly one JSON file with the messages, the model and max_tokens sent, and the response received

##### Requirement: Persistent devtools trace

The router SHALL capture prompt and output sections per call (limited by `LUNAR_DEVTOOLS_TRACE_MAX`, default 20k chars) and the backend SHALL persist the per-turn trace in a traces SQLite database for post-restart inspection.

###### Scenario: Panel after restart

- **WHEN** the backend restarts and the devtools queries old traces
- **THEN** the persisted traces SHALL be available for reading and per-campaign removal

<!-- source: specs/memory-system/spec.md -->

### memory-system

#### Purpose

Project Lunar's long-term memory: a 4-level crystal pyramid (SHORT→MEDIUM→LONG→MEMORY) that distills events into structured JSON preserving facts, with crystal RAG, a witness filter to prevent perspective leakage, and proper-noun integrity.

#### Requirements

##### Requirement: 4-level crystallization pyramid

The system SHALL distill a SHORT crystal from every 4 player events, consolidate 4 SHORTs into 1 MEDIUM, 4 MEDIUMs into 1 LONG, and 4 LONGs into 1 MEMORY (permanent world facts), with an automatic cascade after each crystallization.

###### Scenario: Automatic volume trigger

- **WHEN** the number of not-yet-crystallized events reaches 4
- **THEN** a SHORT crystal SHALL be created covering exactly those events
- **AND** the crystallization cursor SHALL advance to the last covered event

###### Scenario: Consolidation cascade

- **WHEN** a SHORT crystal is created and 4 unconsumed SHORTs already exist
- **THEN** the system SHALL consolidate them into 1 MEDIUM and mark the sources as consumed
- **AND** the cascade SHALL continue to LONG and MEMORY as long as quartets remain

###### Scenario: Manual crystallization

- **WHEN** the player triggers manual crystallization from the interface
- **THEN** the system SHALL crystallize the pending events immediately

##### Requirement: Structured, fact-preserving crystal schema

Each crystal SHALL carry `ai_content` in structured JSON with events (who/action/where/result), characters (description, state, relation to the player), items (name/owner/status), textual promises or missions, and lasting world facts, plus a short `summary` for the player.

###### Scenario: Open promises survive

- **WHEN** an unresolved promise or mission is consolidated
- **THEN** it SHALL appear ipsis litteris in the destination crystal until explicit resolution

###### Scenario: Proper nouns preserved exactly

- **WHEN** a crystal mentions a character named "Lena"
- **THEN** consolidation SHALL preserve the name exactly
- **AND** SHALL NOT replace it with a near variant ("Lana") or with a canonical pop-culture name

###### Scenario: Lossless fallback

- **WHEN** LLM consolidation fails
- **THEN** the destination crystal SHALL store the sources' `ai_content` verbatim as a JSON array (no loss rather than loss)

##### Requirement: Pyramidal WORLD MEMORY context

The system SHALL assemble the WORLD MEMORY section of the prompt with all MEMORY crystals (global canon, never filtered), LONG/MEDIUM/SHORT crystals ranked by relevance when there is query context, and a DELTA section with the latest non-crystallized events.

###### Scenario: Per-level headers

- **WHEN** the context is assembled
- **THEN** the levels SHALL appear under the PRMNT_MEM, ARC_MEM, MID_MEM, and RCNT_MEM headers
- **AND** recent non-crystallized events SHALL appear under DELTA as compact lines

###### Scenario: Crystal RAG

- **WHEN** there is query text, active NPCs, or an active location and the `LUNAR_FEATURE_RAG_CRYSTALS` flag is active
- **THEN** crystals SHALL be ranked by relevance and limited by a token budget proportional to the context window

##### Requirement: Witness filter (perspective)

Each crystal SHALL record which NPCs witnessed the source events; NPC-specific facts SHALL NOT leak to characters who were not present.

###### Scenario: Player solo scene

- **WHEN** the player crosses a forest alone and the passage is crystallized
- **THEN** the crystal SHALL have an empty witness list
- **AND** no NPC SHALL gain knowledge of the content via the minds pipeline

###### Scenario: MEMORY is global canon

- **WHEN** a crystal reaches the MEMORY level
- **THEN** it SHALL ignore the witness filter (global canon of the world)

##### Requirement: Post-restart reconstruction

All in-memory memory state SHALL be rebuilt from the persisted events on restart, with no loss of already-created crystals.

###### Scenario: Backend restart

- **WHEN** the backend restarts and an existing campaign is accessed
- **THEN** all persisted crystals SHALL be reloaded
- **AND** the crystallization cursor SHALL reflect the last crystallized event

<!-- source: specs/military-forces-catalog/spec.md -->

### military-forces-catalog

#### Purpose

The catalog of real military data that powers the game: units, squadrons, specializations, and the organizational structure of the Brazilian Armed Forces (Army, Navy, Air Force), elite forces from around the world, and the multidimensional model of the "ideal soldier" (physical, mental, cognitive, psychological). All data SHALL have a traceable source; the catalog feeds story cards and scenarios via import.

#### Requirements

##### Requirement: Brazilian Armed Forces Catalog

The system SHALL maintain a structured catalog of the three Brazilian Singular Forces, covering the chain of command (Military Area Commands and equivalent Navy and Air Force commands), major units (brigades, divisions, flotillas, wings/groups), operational units (battalions, squadrons with a war name) and education/training establishments.

###### Scenario: Unit with a War Name

- **WHEN** an Air Force unit has a war name (e.g., a squadron)
- **THEN** the record SHALL carry the numerical designation, war name, base/headquarters, primary mission, and aircraft/asset employed when applicable

###### Scenario: Explicit Subordination

- **WHEN** a unit is registered in the catalog
- **THEN** the record SHALL indicate the command it is subordinate to and the headquarters city

##### Requirement: Military Specializations and Courses

The catalog SHALL map military specializations (e.g., special forces, commandos, parachuting, diving, jungle warfare, search and rescue, fighter aviation, air traffic control) with the force that offers them, the responsible unit/training center, and the nature of the qualification.

###### Scenario: Specialization with an Associated Course

- **WHEN** a specialization requires a formal course (e.g., Estágio de Operações Especiais)
- **THEN** the record SHALL identify the training unit and the responsible force

##### Requirement: World Elite Forces with Selection Standards

The catalog SHALL cover international elite units with country, name, typical mission and — when publicly documented — selection numbers (pass rates, duration, key tests), plus the physical, mental, cognitive, and psychological dimensions assessed.

###### Scenario: Selection Number with a Source

- **WHEN** a pass rate or selection duration is recorded
- **THEN** the record SHALL carry the source of the information
- **AND** numbers without a confirmed source SHALL be marked as unverified instead of being silently omitted or invented

##### Requirement: Multidimensional Model of the Ideal Soldier

The system SHALL maintain a model of the "soldier closest to perfection" organized into the physical, mental, cognitive, and psychological dimensions, with measurable components per dimension and the documented trade-off that no single profile exists — optimal profiles differ by role.

###### Scenario: Profile by Role

- **WHEN** two distinct military roles are compared (e.g., special forces operator vs fighter pilot)
- **THEN** the model SHALL reflect differentiated requirements per dimension instead of a single "perfection" ranking

##### Requirement: Provenance of All Data

Every catalog record SHALL carry a source field (URL or documentary reference) and a verification date; data drawn from general knowledge without verification SHALL be marked as unverified.

###### Scenario: Record Without a Source

- **WHEN** a fact could not be confirmed in an accessible source
- **THEN** the record SHALL be marked `verified: false` with an explanatory note

##### Requirement: Export to Story Cards

The catalog SHALL be exportable as story cards of the types NPC, LOCATION, FACTION, ITEM, and LORE per scenario, ready for import in the scenario interchange format, with keywords extracted from unit names for RAG selection.

###### Scenario: Squadron as a LORE Card

- **WHEN** the author exports a force's units to a scenario
- **THEN** each unit SHALL become a card with name, type, descriptive text, and keywords including acronym and war name

##### Requirement: Ready-Made Military Training Scenarios

The system SHALL offer complete, importable scenarios built on the catalog: (a) training in the Brazilian Armed Forces with career progression and real specializations, and (b) international elite selection toward the ideal soldier, both bilingual (en + pt-br) and with interpolatable setup questions.

###### Scenario: Importable Brazilian Scenario

- **WHEN** the author imports the Brazilian Armed Forces scenario
- **THEN** the scenario SHALL arrive with lore, setup questions, and complete story cards, ready to create a campaign

###### Scenario: Specialization Progression in the Fiction

- **WHEN** the player chooses force and specialization in the training scenario setup
- **THEN** the answers SHALL interpolate into the lore and tone to steer the training narrative

<!-- source: specs/narrative-audit/spec.md -->

### narrative-audit

#### Purpose

The post-hoc quality network (Phase 3b): after the narrator's prose, a context-aware auditor rewrites only violations of player agency and world contradictions, in a flow of 3 drafts → critique → synthesis, with absolute safety over load-bearing tags and safe degradation on parse failure.

#### Requirements

##### Requirement: Optional post-hoc audit

The system SHALL run a post-hoc auditor over the narrator's prose when `LUNAR_FEATURE_NARRATOR_AUDIT` is active, with a configurable timeout (`LUNAR_AUDIT_TIMEOUT_S`, default 210s) and fallback to the original prose on timeout or failure.

###### Scenario: Auditor off

- **WHEN** `LUNAR_FEATURE_NARRATOR_AUDIT=0`
- **THEN** the original prose SHALL pass to the player without audit

###### Scenario: Timeout

- **WHEN** the audit exceeds the configured timeout
- **THEN** the original prose SHALL be delivered intact

##### Requirement: Three-draft pipeline with critique and synthesis

When it decides to rewrite, the auditor SHALL follow the flow of three drafts, critique, and synthesis before producing the final prose.

###### Scenario: Synthesis as the only candidate

- **WHEN** the audit produces intermediate drafts
- **THEN** only the final synthesis SHALL be a candidate to replace the original prose
- **AND** intermediate drafts SHALL NOT reach the player

##### Requirement: Full context, not blind

The auditor SHALL receive the open scene window (continuity) and the world context (memory, cards, inventory, character sheet, NPCs) to judge continuity — never only the isolated turn input.

###### Scenario: Established ability preserved

- **WHEN** the player established an ability (e.g., electricity) in the previous turn
- **THEN** the auditor SHALL NOT excise it as excess agency

##### Requirement: Rewrite scoped to agency and continuity

The auditor SHALL rewrite only what the narrator invented beyond the player's input plus the established scene; NPC initiative is not agency, and world contradictions have a high bar.

###### Scenario: NPC proposes a plan

- **WHEN** an NPC proposes a plan to the player
- **THEN** the auditor SHALL NOT treat the NPC's speech as player agency

##### Requirement: Load-bearing tag safety

The auditor SHALL preserve exactly the multiset of `[ITEM_ADD|USE|LOSE]` tags from the original prose; any addition, removal, or alteration invalidates the rewrite.

###### Scenario: Item fingerprint

- **WHEN** the rewrite changes any item tag
- **THEN** the rewrite SHALL be discarded and the original kept

###### Scenario: @Name mentions are cosmetic

- **WHEN** the rewrite omits an @Name mention
- **THEN** the system SHALL log it without rejecting the rewrite

##### Requirement: Safe degradation on parse failure

When the auditor's response does not parse (long prose in escaped JSON), the system SHALL deliver the original prose and log the occurrence.

###### Scenario: Broken JSON

- **WHEN** the auditor returns malformed `final_prose`
- **THEN** the original prose SHALL be kept without interrupting the turn

##### Requirement: Reasoning budget

The auditor SHALL receive extra output-token headroom for models that spend max_tokens on reasoning (`LUNAR_AUDIT_REASONING_HEADROOM`, default 8000).

###### Scenario: Reasoning model

- **WHEN** the auxiliary model spends part of the budget on reasoning
- **THEN** the headroom SHALL prevent the final text from coming out truncated

<!-- source: specs/narrative-engine/spec.md -->

### narrative-engine

#### Purpose

The narration core: classification of the player's action into modes, construction of the narrator prompt (with an open scene window), SSE streaming of the prose, auto-continuation of truncated responses, and language consistency.

#### Requirements

##### Requirement: Action Mode Detection

The system SHALL classify each player action into exactly one mode — NARRATIVE, COMBAT, or META — also returning `ambush`, `narrative_time_seconds`, `opponent_name`, and `opponent_power` (1–10).

###### Scenario: Combat Action

- **WHEN** the action starts or continues a fight
- **THEN** the classifier SHALL return `mode = COMBAT` with the opponent's name and estimated power

###### Scenario: Out-of-Character Speech

- **WHEN** the player addresses the narrator out of character
- **THEN** the classifier SHALL return `mode = META`

###### Scenario: Calibration by Power Scale

- **WHEN** the classification context includes a WORLD POWER SCALE
- **THEN** the opponent's power SHALL be estimated with calibration against the scale's NPCs as anchors

##### Requirement: Mode Coercion with Combat Disabled

The system SHALL downgrade COMBAT to NARRATIVE when the campaign has `combat_enabled = false`.

###### Scenario: Purely Narrative Campaign

- **WHEN** the classifier returns COMBAT in a campaign with combat disabled
- **THEN** the turn SHALL be processed as NARRATIVE
- **AND** the combat rules SHALL be omitted from the prompt

##### Requirement: SSE Streaming of the Narrative

The system SHALL deliver the narrator's prose via Server-Sent Events, preserving paragraph breaks.

###### Scenario: Multiple Paragraphs

- **WHEN** the response contains line breaks
- **THEN** each line SHALL be sent as an individual `data:` line in the SSE stream
- **AND** the client SHALL faithfully reconstruct the paragraphs

###### Scenario: Mode Signal for the Frontend

- **WHEN** the turn's mode is determined
- **THEN** the stream SHALL emit a control tag `[MODE]<value>` before the prose

##### Requirement: Open Scene Window

The narrator SHALL receive as raw prose only the open scene — events after the cursor of the last SHORT crystal minus one overlap batch — and the distilled past only via crystals.

###### Scenario: Cuts at the Crystallization Boundary

- **WHEN** there are enough SHORT crystals to define the boundary
- **THEN** the raw prose history SHALL contain the events after the end of the second-to-last crystallized batch (an overlap of 1 batch)
- **AND** the short-term floor SHALL be 4 messages when the computed window is smaller

###### Scenario: Safe Degradation

- **WHEN** the window computation fails or there are no crystals yet
- **THEN** the system SHALL use the full history instead of erroring

###### Scenario: Disable Flag

- **WHEN** `LUNAR_FEATURE_OPEN_SCENE_WINDOW=0`
- **THEN** the narrator SHALL fall back to full-history behavior

##### Requirement: Sizing by Provider Context

The system SHALL size history slicing, card selection, and the crystal budget by the active model's real context window, with no fixed character limits.

###### Scenario: 1M-Context Model

- **WHEN** the active model has a 1,000,000-token window
- **THEN** the card and crystal budgets SHALL scale proportionally
- **AND** the history SHALL respect the message caps defined for the provider

##### Requirement: Auto-Continuation of Truncated Responses

When the response ends mid-sentence, the system SHALL ask the LLM for an exact continuation instead of merely trimming it.

###### Scenario: Continuation Without Inventing Player Actions

- **WHEN** the response is incomplete
- **THEN** the continuation SHALL resume from the exact stopping point without repeating text
- **AND** SHALL NOT take new actions, decisions, speech, or thoughts on behalf of the player

##### Requirement: Narrator Rules and Language

The narrator SHALL follow the scenario's tone instructions, render NPC names as `@Full Name` for consistency, and respond in the campaign's language (`en` or `pt-br`).

###### Scenario: Campaign Language

- **WHEN** the campaign is in `pt-br`
- **THEN** all narrated prose SHALL come out in Brazilian Portuguese

###### Scenario: Prohibition Without Anti-Examples

- **WHEN** the narrator rules list style vices to avoid (e.g., recap recursion, false metric)
- **THEN** the rules SHALL NOT include literal examples of the vice (anti-pink-elephant pattern)

##### Requirement: META Mode Prompt

The system SHALL build a distinct prompt for META turns, without combat rules and oriented toward answering about the state of the world.

###### Scenario: Question About the World

- **WHEN** the player asks something out of character about the state of the world
- **THEN** the narrator SHALL answer using the available memory context
- **AND** SHALL NOT advance narrative time or count the turn

##### Requirement: Single-Call Mode Disabled

The system SHALL use the streaming path for all providers; the single-call mode with structured JSON output remains disabled.

###### Scenario: Any Provider

- **WHEN** an action is processed with any configured provider
- **THEN** the narration SHALL follow the streaming path
- **AND** single-call mode SHALL be treated as dormant code

<!-- source: specs/npc-minds/spec.md -->

### npc-minds

#### Purpose

The inner life of NPCs: each character tracks private thoughts (feeling, goal, opinion about the player, secret plan) with decay of transient states, fuzzy name deduplication with LLM confirmation, perspective-based knowledge boundaries, and manual editing through the inspector.

#### Requirements

##### Requirement: Private thoughts per NPC

The system SHALL maintain, per NPC, a mind with key thoughts: feeling, goal, opinion_of_player and secret_plan, updated from the narrative.

###### Scenario: Post-turn update

- **WHEN** a narrator response completes
- **THEN** the minds of the NPCs active in the scene SHALL be updated asynchronously
- **AND** thoughts SHALL be persisted as NPC_THOUGHT events

##### Requirement: Decay of transient thoughts

Transient emotional thoughts (feeling, mood, emotion) SHALL decay after 5 turns; long-term motivations (goal, opinion_of_player, secret_plan) SHALL persist until rewritten.

###### Scenario: Emotion fades

- **WHEN** an NPC becomes "anxious" on turn 12 and no new emotion updates it
- **THEN** the anxious state SHALL expire after 5 turns

###### Scenario: Goal persists

- **WHEN** an NPC has a goal set
- **THEN** the goal SHALL remain until the narrative rewrites it

###### Scenario: Disable flag

- **WHEN** `LUNAR_FEATURE_NPC_DECAY=0`
- **THEN** the pipeline SHALL revert to the no-decay behavior (states never expire)

##### Requirement: Fuzzy name deduplication

The system SHALL unify references to the same NPC by fuzzy similarity, with LLM confirmation before merging minds.

###### Scenario: Name variation

- **WHEN** the narrative mentions "Kael" and a mind for "Kael Noir" exists
- **THEN** the system SHALL query the LLM to confirm identity before merging

##### Requirement: Knowledge boundaries

The system SHALL prevent NPCs from "knowing" off-screen events: only witnessed events (or public/role knowledge) feed their minds, and the narrator's prompt includes a per-NPC boundaries block.

###### Scenario: Absent NPC does not know

- **WHEN** an event occurs with a witness list that does not include the NPC
- **THEN** the NPC's mind SHALL NOT incorporate the fact

###### Scenario: Disable flag

- **WHEN** `LUNAR_FEATURE_PERSPECTIVE_FILTER=0`
- **THEN** the pipeline SHALL revert to the pre-filter omniscient behavior

##### Requirement: Inspection and manual editing

The system SHALL expose reading, editing and removal of minds per NPC via API and a panel in the frontend.

###### Scenario: Mind editor

- **WHEN** the player edits an NPC's secret_plan in the inspector
- **THEN** the change SHALL be applied and persisted immediately

###### Scenario: Dead or merely mentioned NPC

- **WHEN** the mind update encounters a dead or merely mentioned NPC (no interaction)
- **THEN** the system SHALL skip updating the corresponding mind

<!-- source: specs/opening-generation/spec.md -->

### opening-generation

#### Purpose

Defines how a campaign's opening (cold-open) is produced: `fixed` mode (authorial text) or `ai` (generated by an LLM weaving the player's setup responses), including re-roll, preview without saving, and truncation cleanup.

#### Requirements

##### Requirement: Opening mode selection

The system SHALL support two opening modes per scenario: `fixed` (uses the authorial `opening_narrative` text) and `ai` (generates a unique opening per campaign).

###### Scenario: Fixed mode

- **WHEN** a campaign starts in a scenario with `opening_mode = fixed`
- **THEN** the opening SHALL be the authorial text interpolated with the setup responses

###### Scenario: AI mode with directive

- **WHEN** a campaign starts in a scenario with `opening_mode = ai`
- **THEN** the system SHALL generate the opening respecting the scenario's `ai_opening_directive`, if any

##### Requirement: AI-generated cold-open constraints

The system SHALL generate cold-opens in second person, with 180–320 words in 4–8 short paragraphs, ending with an invitation to the player's first action.

###### Scenario: Format limits

- **WHEN** the generator produces the opening
- **THEN** it SHALL be in second person ("You ..."), have between 180 and 320 words, organized in 4–8 short paragraphs
- **AND** end with terminal punctuation (. ! ? …) and with a question, beat or line of speech that invites the player's first action

###### Scenario: Setup responses woven organically

- **WHEN** the player has answered the setup wizard
- **THEN** the opening SHALL reference the CHARACTER SETUP block organically
- **AND** SHALL NOT list it as a block of attributes

###### Scenario: No mid-sentence truncation

- **WHEN** the LLM output is cut off in the middle of a sentence by a token limit
- **THEN** the system SHALL trim back to the last complete sentence

##### Requirement: Complete inputs, no withholding

The opening generator SHALL receive the scenario's full tone and full lore, with no selective truncation of details.

###### Scenario: Scenario with extensive lore

- **WHEN** the scenario has very long lore
- **THEN** the generator SHALL still receive the full content
- **AND** SHALL NOT discard sections automatically

##### Requirement: Opening regeneration

The system SHALL allow regenerating the AI opening of an existing campaign, producing a new variant.

###### Scenario: Re-roll

- **WHEN** the player requests regeneration of the opening
- **THEN** the system SHALL generate and persist a new opening for the campaign
- **AND** the campaign history SHALL reflect the new opening

##### Requirement: Preview without persistence

The system SHALL offer an AI opening preview during scenario authoring without saving a campaign.

###### Scenario: Author tests the directive

- **WHEN** the author requests a preview with tone, lore, questions and directive
- **THEN** the system SHALL return a sample opening
- **AND** no campaign SHALL be created and no state persisted

<!-- source: specs/plot-generation/spec.md -->

### plot-generation

#### Purpose

Generation of plot elements — macro arcs, micro-hooks and NPC seeds — with automatic triggers by cooldown and minimum turn, plot lock (only one active generation at a time) and the NONE rule (not generating is always preferable to generating bad content).

#### Requirements

##### Requirement: Automatic generation with per-type rules

The system SHALL define automatic generation rules per element type, evaluated at the end of each turn: micro_hook (min. 5 turns, cooldown 6 turns / 2 narrative hours, max. 8 triggers), npc (min. 8 turns, cooldown 10 turns / 24 narrative hours, max. 6) and plot_arc — each type with its own limits for minimum turn, cooldown and maximum triggers.

###### Scenario: Micro-hook available

- **WHEN** 5+ turns have passed and the cooldowns have expired
- **THEN** the system MAY generate a micro-hook to weave into the next response

###### Scenario: Trigger limit

- **WHEN** a type has already reached its maximum triggers in the campaign
- **THEN** no additional generation of that type SHALL occur

##### Requirement: Plot lock of one generation at a time

The system SHALL keep only one active plot element at a time; new generations SHALL wait for the active one (plot lock) before running.

###### Scenario: Active lock

- **WHEN** an unconsumed active seed exists
- **THEN** the automatic generator SHALL postpone new generations until the lock releases

##### Requirement: NONE rule

The generator SHALL accept and prefer the NONE response when generating something at the current moment would be forced, unnatural, or would break the scene's flow.

###### Scenario: Tense scene

- **WHEN** the current scene is tense (combat, confrontation, ceremony)
- **THEN** the generator SHALL NOT introduce unrelated content
- **AND** answering NONE is always acceptable

###### Scenario: No second plotlines

- **WHEN** a main complication is already active
- **THEN** the generated content SHALL NOT add a second complication before developing or resolving the first

##### Requirement: NPC seed with exact name

When an NPC seed is injected, the narrator SHALL use the exact name provided, weaving the character naturally into the scene with the defined appearance, personality, goal and power.

###### Scenario: Post-response verification

- **WHEN** the narrator responds with a pending seed
- **THEN** the system SHALL verify the exact name in the response
- **AND** the seed SHALL remain pending until it appears with the correct name

###### Scenario: Seed knowledge boundary

- **WHEN** a seeded NPC enters the scene
- **THEN** their knowledge SHALL be limited to public lore, role expertise and visible/on-screen facts

##### Requirement: On-demand generation

The system SHALL allow manual generation of an NPC, event or plot on demand through the interface, subject to the same context rules.

###### Scenario: Generation panel

- **WHEN** the player requests manual generation
- **THEN** the system SHALL generate the element respecting the type's context rules and cooldowns

<!-- source: specs/prompt-caching/spec.md -->

### prompt-caching

#### Purpose

Zoned prompt layout to leverage providers' prompt caching (PHASE 2): a stable cached prefix, per-action volatile content, and instruction cloaking in the first `user` message, eliminating the costly re-feeding of raw prose.

#### Requirements

##### Requirement: Cacheable zoned layout

When prompt caching is active, the system SHALL assemble the prompt in three zones: zone 0 (static canon per scenario: role, language, tone, character setup, opening, narrator rules), zone 1 (quasi-static: LORE cards in stable order + permanent MEMORY crystals) and zone 2 (volatile per action: recent memory, inventory, NPCs, journal, hints, graph, RAG cards, size directive).

###### Scenario: Consecutive turns in the same scenario

- **WHEN** two consecutive actions occur in the same campaign without a scenario change
- **THEN** zones 0 and 1 SHALL be identical byte for byte across the two prompts
- **AND** only zone 2 and the player's message SHALL differ

###### Scenario: Volatile content never in the cached zone

- **WHEN** zone 1 is rendered
- **THEN** it SHALL contain only content that is stable across turns
- **AND** volatile memory, hints and RAG cards SHALL be restricted to zone 2

##### Requirement: Instruction cloaking in the user message

The system SHALL wrap the narrator instructions inside the first `user` message, between `<narrator-instructions>…</narrator-instructions>` tags, with content blocks marked with ephemeral `cache_control`.

###### Scenario: Cache markers at zone boundaries

- **WHEN** the zoned prompt is sent to the provider
- **THEN** the content blocks of the stable zones SHALL carry `cache_control` of the ephemeral type with a 1-hour TTL
- **AND** the per-request size directive SHALL stay outside the cached zones

##### Requirement: Observable cache metrics

The system SHALL record, per call, the cache read tokens (`cache_read_input_tokens`) and cache creation tokens (`cache_creation_input_tokens`), exposed in the per-action summary.

###### Scenario: Second turn onward

- **WHEN** the same cached prefix is resent on the following turn
- **THEN** the turn's usage summary SHALL report `cache_read_input_tokens > 0`

##### Requirement: Monolithic prompt restore flag

Prompt caching SHALL be disableable via flag without code changes.

###### Scenario: Deactivation

- **WHEN** `LUNAR_FEATURE_PROMPT_CACHE=0`
- **THEN** the narrator SHALL use the single monolithic prompt (pre-PHASE 2 behavior)
- **AND** no `cache_control` block SHALL be sent

<!-- source: specs/scenario-authoring/spec.md -->

### scenario-authoring

#### Purpose

Defines how authors create scenarios (worlds) in Project Lunar: metadata, setup questions with variable interpolation, RAG-selected story cards, import/export, and the scenario→campaign relationship. A scenario is the authoring container; the campaign is the played instance.

#### Requirements

##### Requirement: Scenario Creation with Setup Questions

The system SHALL allow authors to define scenarios with title, description, tone instructions (`tone_instructions`), fixed opening, language (`en` or `pt-br`), free-form lore text, and a list of setup questions.

###### Scenario: Choice-Type Question

- **WHEN** an author creates a setup question of type `choice`
- **THEN** the question SHALL include `var_name`, `prompt`, a list of `options` (each with `label` and an optional `description`) and a `required` flag
- **AND** the player SHALL choose exactly one option when starting the campaign

###### Scenario: Free-Text Question

- **WHEN** an author creates a setup question of type `text`
- **THEN** the player SHALL answer with free text
- **AND** an empty answer to a non-required question SHALL be accepted

###### Scenario: Unique Variable Names

- **WHEN** a scenario is created with two questions using the same `var_name`
- **THEN** the system SHALL reject the creation with a validation error

##### Requirement: Variable Interpolation

The system SHALL interpolate setup answers into the scenario's tone, lore, and opening using the `{var_name}` syntax.

###### Scenario: Simple Substitution

- **WHEN** the tone contains `{main_clan}` and the player answered `main_clan = Iron Wolves`
- **THEN** the narrator SHALL receive the text with `Iron Wolves` in place of the token

###### Scenario: Missing Variable Stays Literal

- **WHEN** a template references `{typo_name}` with no corresponding answer
- **THEN** the token SHALL remain literal in the final text
- **AND** the system SHALL log a warning only once per (context, variable)

###### Scenario: Escapes and Single Passes

- **WHEN** the author writes `{{`, `}}`, or `\{var}`
- **THEN** the system SHALL render literal `{`, `}`, and `{var}` respectively
- **AND** substituted values SHALL NOT be re-interpolated (single pass, immune to recursion)

###### Scenario: Tokens Never Reach the Narrator

- **WHEN** the frontend resends the raw tone template in any request
- **THEN** the backend SHALL re-interpolate against the saved answers before any LLM call

##### Requirement: Story Cards with Dynamic Selection (RAG)

The system SHALL store story cards per scenario (types NPC, LOCATION, FACTION, ITEM, LORE) and select them per turn via keyword overlap with the recent context, instead of dumping the entire library.

###### Scenario: Budget Proportional to the Context Window

- **WHEN** the narrator assembles a turn's prompt
- **THEN** the token budget for cards SHALL be 15% of the active model's context window
- **AND** the budget SHALL have a floor of 4,000 tokens and a ceiling of 200,000 tokens
- **AND** at most 300 cards SHALL enter the prompt

###### Scenario: Relevance Ranking

- **WHEN** two cards compete for the budget
- **THEN** the LORE card SHALL receive a 100 bonus, an active NPC card a 50 bonus, a card mentioned by name a 30 bonus, and 5 points per matched keyword
- **AND** only the cards above the budget cutoff SHALL enter

###### Scenario: Cached Zone Stability

- **WHEN** LORE cards are rendered for the quasi-static cache zone
- **THEN** the order SHALL be deterministic (by `created_at`, then `id`) byte-stable across turns

##### Requirement: Lore Extraction into Cards

The system SHALL extract NPCs, locations, and factions from the scenario's free-form lore text into story cards via LLM.

###### Scenario: Scenario Created with Lore

- **WHEN** an author saves a scenario with `lore_text` filled in
- **THEN** the system SHALL generate cards corresponding to the detected entities
- **AND** the generated cards SHALL be editable and removable like any manual card

##### Requirement: Scenario Import and Export

The system SHALL export a complete scenario (metadata, questions, cards) as JSON and import scenarios from that same format.

###### Scenario: Lossless Round-Trip

- **WHEN** a scenario is exported and the resulting JSON is imported
- **THEN** the new scenario SHALL preserve setup questions, cards, and tone instructions

##### Requirement: Campaigns per Scenario

The system SHALL allow multiple campaigns per scenario, each with its own persisted setup answers, effective language, and `combat_enabled` flag.

###### Scenario: Persisted Answers per Campaign

- **WHEN** two campaigns of the same scenario answer different questions
- **THEN** each campaign SHALL interpolate only its own answers

###### Scenario: Listing and Removal

- **WHEN** the author lists a scenario's campaigns or removes a campaign
- **THEN** the system SHALL return the existing campaigns or delete all events and answers of the target campaign

<!-- source: specs/world-simulation/spec.md -->

### world-simulation

#### Purpose

The off-screen world evolves: each action advances narrative time and triggers world ticks proportional to the elapsed time (MICRO→HEAVY), with a preference for observable changes that follow established agendas — without inventing new mysteries.

#### Requirements

##### Requirement: Narrative time per action

Each player action SHALL receive an estimated duration in seconds of story time, determined during mode classification.

###### Scenario: Short vs long action

- **WHEN** the player performs a brief action (looking at a map) vs a long one (traveling for days)
- **THEN** the narrative time delta SHALL reflect the action's realistic duration

##### Requirement: World ticks by time magnitude

The system SHALL map accumulated narrative time into ticks of magnitude MICRO (<1 hour, no change), MINOR (1 hour–1 day), MODERATE (1 day–1 week), MAJOR (1 week–1 month) and HEAVY (>1 month).

###### Scenario: Short interval

- **WHEN** the time elapsed since the last tick is less than 1 hour
- **THEN** no world event SHALL be generated

###### Scenario: Months of time

- **WHEN** the accumulated time exceeds 1 month
- **THEN** the HEAVY tick SHALL describe major transformations (wars, alliances, deaths)

###### Scenario: One main change per tick

- **WHEN** a tick generates a world change
- **THEN** there SHALL be a single main development
- **AND** quiet intervals MAY advance routine without escalation

##### Requirement: Changes follow existing agendas

Ticks SHALL prefer direct, observable changes that follow from schedules, goals and consequences already established in the world context.

###### Scenario: New mystery prohibition

- **WHEN** the world context does not make something active
- **THEN** the tick SHALL NOT create a new mystery, secret investigation, conspiracy or hidden threat

##### Requirement: Manual timeskip

The system SHALL allow manual advancement of narrative time, recording a TIMESKIP event and processing the tick corresponding to the given interval.

###### Scenario: Advancing days

- **WHEN** the player manually advances N seconds
- **THEN** the system SHALL process the tick of the corresponding magnitude and record the changes in the journal

##### Requirement: Asynchronous execution

Automatic ticks SHALL run as a fire-and-forget asynchronous task after narration, without blocking the response flow to the player.

###### Scenario: Turn without extra waiting

- **WHEN** the turn's narration finishes
- **THEN** the world tick SHALL be scheduled in the background
- **AND** the SSE response to the player SHALL NOT wait for the tick to complete

<!-- source: specs/worldbuilding-research/spec.md -->

### worldbuilding-research

#### Purpose

A research program of reverse engineering the mechanics of reference games (Albion Online, GTA San Andreas, MUDs) and a playable Doom 3-style WebGL prototype, with the goal of extracting verifiable world-building lessons that feed the Project Lunar specs for world-simulation, npc-minds, plot-generation, and scenario-authoring. Everything in English; structural headings and SHALL/MUST keywords in English.

#### Requirements

##### Requirement: Reverse Engineering of Albion Online Mechanics

The research system SHALL document, from public sources (official wikis, patch notes, dev blogs, Sandbox Interactive), the Albion Online world mechanics relevant to world-building: player-driven economy (resources, crafting, regional markets), territories and guilds, full-loot and risk zones by band (Blue/Yellow/Red/Black), faction travels, and the seasons cycle. Each mechanic SHALL produce a lesson card with: the original mechanic, why it works (emergent effect), and a candidate translation to the narrative engine (or a justified discard).

###### Scenario: Translated Economy Lesson

- **WHEN** the research documents Albion's regional markets
- **THEN** a candidate translation SHALL exist (e.g., prices/scarcity as a world tick trigger) or a discard with rationale

###### Scenario: Verifiable Source

- **WHEN** a lesson card states a number or game rule
- **THEN** the card SHALL cite the public source (URL) and verification date

##### Requirement: Reverse Engineering of GTA San Andreas Mechanics

The research system SHALL document the systemic mechanics of GTA San Andreas that sustain the feeling of a living world: CJ's stats (respect, stamina, muscle, driving skill), gang territories with war and takeover, NPC and traffic routines, the wanted level with escalating police response, and the stack of worlds (city → countryside → desert) with progressive story gating. Each mechanic SHALL generate a lesson card in the same format as Albion.

###### Scenario: Translated NPC Routine

- **WHEN** the research documents the daily routine of pedestrians/NPCs
- **THEN** a mapping to npc-minds NPC agendas (schedules, objectives) or a justified discard SHALL exist

###### Scenario: Response Escalation

- **WHEN** the research documents the wanted level (1–6 stars)
- **THEN** the card SHALL propose how an escalating consequence of player actions could appear in world ticks

##### Requirement: Reverse Engineering of MUD (Multi-User Dungeon) Mechanics

The research system SHALL document, from public sources (documentation and wikis of the DikuMUD/MOO/MUSH families, RPI MUDs), the text-based multi-user world mechanics relevant to world-building: a 24/7 persistent world that evolves while the player is offline, the world as a network of rooms with named exits and descriptions revealed on demand (`look`), presence and social communication (`who`, `say`, `emote`, channels), enforced roleplay (RPI), and collaborative world authoring (online OLC/builders, programmable MOO/MUSH worlds). Each mechanic SHALL generate a lesson card in the same format as the other tracks.

###### Scenario: The World Evolves Without the Player

- **WHEN** the research documents the persistent multi-user world (events occurring while the player is offline)
- **THEN** a candidate translation to world-simulation off-screen ticks or a discard with rationale SHALL exist

###### Scenario: The Room as the Unit of World

- **WHEN** the research documents rooms with named exits and descriptions on command
- **THEN** the card SHALL propose a mapping to LOCATION story cards with on-demand inspection (scenario-authoring/narrative-engine) or a justified discard

###### Scenario: Collaborative Authoring

- **WHEN** the research documents OLC/builders or programmable worlds (MOO/MUSH)
- **THEN** the card SHALL evaluate what the in-world authoring experience teaches about the scenario builder (frontend-ui/scenario-authoring)

##### Requirement: Doom 3-Style WebGL Prototype for World-Building

The project SHALL include a playable in-browser prototype (WebGL/OpenGL ES via Three.js or raw WebGL, no plugins) inspired by Doom 3 — dark corridors, dynamic flashlights, shadows, interactive lore terminals, positional audio, and script triggers — used as a world-building laboratory: every level design element SHALL teach a lesson mappable to the narrative engine (e.g., terminal with lore ≈ story card; script trigger ≈ plot seed; lighting that guides ≈ narrative emphasis).

###### Scenario: The Prototype Loads in the Browser

- **WHEN** the prototype is opened in a modern browser (no native build)
- **THEN** it SHALL render a first-person 3D scene with dynamic lighting at 30+ FPS on common hardware

###### Scenario: Interaction with Lore

- **WHEN** the player interacts with a prototype terminal
- **THEN** the displayed lore text SHALL be mapped to a world-building lesson card

##### Requirement: Versioned Lesson Cards

Lessons from the tracks (Albion, GTA SA, MUDs, Doom 3) SHALL be persisted in a versioned dataset (`data/worldbuilding/lessons.json`) with fields: source game, mechanic, evidence/source, candidate translation, status (proposed/accepted/discarded), and target spec. Accepted cards SHALL reference the target spec requirement that absorbs the lesson.

###### Scenario: Traceable Accepted Card

- **WHEN** a card is marked as accepted
- **THEN** a reference to the target spec and requirement that incorporated it SHALL exist
- **AND** the dataset SHALL be loadable without network dependency

##### Requirement: No Asset Violations

The research program SHALL use only documentary observation of mechanics (public sources) and original implementation in the prototype; no asset, code, model, texture, or audio extracted from the reference games SHALL enter the repository. The Doom 3-like prototype SHALL use original or free assets only with a documented permissive license.

###### Scenario: Asset Audit

- **WHEN** the prototype includes a model or texture
- **THEN** the provenance/license SHALL be documented in the repository

## 3. Changes (work in progress)

<!-- source: changes/add-engine-core/ -->

### add-engine-core

#### `proposal.md`

# add-engine-core

## Why

The 22 engine specs (narrative, events, memory, audit, LLM routing, API, frontend, avatar-mirror, age-banding...) describe the complete product, but not a single line of executable code exists. Before implementing the entire engine, we need a walking skeleton: the smallest end-to-end vertical that proves the architecture — scenario import → opening generation → SSE action loop → event store → rewind — playable in the terminal, without UI.

## What changes

**FastAPI backend with uv**
- From: no backend; scenarios and datasets are static artifacts.
- To: `backend/` with a FastAPI app (uv venv, Python 3.14), scenario routes (import/list/detail), campaign route (create from an imported scenario), action route with SSE ([USAGE]/[TRACE]/[DONE]) and rewind.
- Reason: FastAPI has native async/SSE and fits litellm; uv is the dependency manager available on the machine (PEP 668, no global pip).
- Impact: new versioned directory; no breakage of existing artifacts.

**SQLite event store (spec event-persistence)**
- From: no game state.
- To: `backend/events.db` append-only with the 13 canonical types (12 + AI_OPENING_GENERATED), immutable events (frozen namedtuple), narrative time delta, state reconstruction from the log, rewind that removes only the last action+response pair.
- Impact: first real consumer of the event-persistence spec.

**LLM router with litellm (spec llm-routing)**
- From: no model calls.
- To: router with narrative vs auxiliary policy, per-model windows (200k fallback), sampling guard (omit temperature on no-sampling models), retry with 0.5s/1.5s backoff (3 attempts), token accounting.
- Reason: the llm-routing spec already defines providers; the skeleton implements routing in a provider-agnostic way with a test provider (mock/echo) to allow development without a key.
- Impact: no keys in the repo; real providers configurable via settings.

**Minimal narrative engine (specs narrative-engine + opening-generation)**
- From: scenarios exist but nothing executes them.
- To: prompt assembly (interpolated tone + lore + RAG cards), AI opening generation (180-word floor, 4–8 paragraphs), turn loop (player action → streamed narration → persisted events).
- Impact: first real playable scenario (any of the 14 via import).

**Non-scope of this change**: React frontend, combat-system, npc-minds, knowledge-graph/Neo4j, memory-system (crystals), journal-system, zone-based prompt caching — all remaining specs are left for subsequent changes; the skeleton does not block them.

> **Evolution note (as-built)**: the change grew beyond the original walking skeleton in subsequent iterations, all recorded in `tasks.md` (sections 11–15) and in the spec deltas: real providers + hermes provider (llm-routing), complete post-turn pipeline — crystals/minds/journal/combat (sections 13), Phase 2/3b — cache zones/plot/auditor/graph (section 14), Docker Compose with Neo4j in an optional profile and the new `deployment` spec (section 15). The React 19 frontend (spec frontend-ui) came in at section 12. The non-scope above describes the INITIAL cut, not the final state.

## Order (tasks)

1. uv + FastAPI scaffold + health check (`/api/health`)
2. SQLite event store (canonical types, append-only, immutable, reconstruction)
3. Scenario CRUD (import/list/detail — validates against the scenario/1.0 schema)
4. Campaign (create from scenario; setup answers)
5. LLM router (mock provider + litellm structure; per-campaign settings)
6. AI Opening (tone interpolation, 180-word floor, AI_OPENING_GENERATED)
7. SSE action (PLAYER_ACTION → stream → NARRATOR_RESPONSE + [USAGE]/[TRACE]/[DONE])
8. Rewind (removes pair; consistent reconstruction)
9. End-to-end smoke test in the terminal with scenarios/try_harder.json
10. Update README + commit

#### `tasks.md`

# add-engine-core — Tasks

End-to-end walking skeleton: import → opening → SSE action → event store → rewind.

## 1. Scaffold

- [x] 1.1 `backend/` with uv (Python 3.14): fastapi, uvicorn, litellm, pydantic, httpx
- [x] 1.2 Minimal FastAPI app + `GET /api/health`
- [x] 1.3 Backend `.gitignore` (events.db, .venv) + run README

## 2. Event store (spec event-persistence)

- [x] 2.1 `app/events.py`: enum of the 13 canonical types
- [x] 2.2 Append-only SQLite store; FrozenEvent (frozen model + FrozenPayload — mutation raises TypeError)
- [x] 2.3 `rebuild(campaign_id)`: reconstructs opening, history, setup, narrative clock from the log
- [x] 2.4 Rewind: removes only the last PLAYER_ACTION+NARRATOR_RESPONSE pair; opening preserved

## 3. Scenarios (spec game-api: CRUD)

- [x] 3.1 Pydantic models of the scenario/1.0 schema (validates unique var_name, card types, placeholders)
- [x] 3.2 `POST /api/scenarios/import` — the 14 scenarios imported via tools/import_all_scenarios.py against the real server
- [x] 3.3 `GET /api/scenarios` and `GET /api/scenarios/{id}`

## 4. Campaign

- [x] 4.1 `POST /api/game/campaigns` (WORLD_TICK campaign_created with scenario_id)
- [x] 4.2 `POST /api/game/{cid}/setup-answers` (persisted as event)

## 5. LLM router (spec llm-routing)

- [x] 5.1 Deterministic mock provider (dev without key) + structure for litellm
- [x] 5.2 Narrative vs auxiliary policy + per-model windows (200k fallback) + NO_SAMPLING
- [x] 5.3 Retry 0.5s/1.5s (3 attempts) + token accounting + traces

## 6. AI Opening (spec opening-generation)

- [x] 6.1 Interpolation of tone_instructions ({language} + var_names)
- [x] 6.2 Generation with 180-word floor (validator rejects below) — mock generates 336
- [x] 6.3 Persistence as AI_OPENING_GENERATED

## 7. SSE action (spec game-api)

- [x] 7.1 `POST /api/game/{cid}/action` → StreamingResponse text/event-stream
- [x] 7.2 Flow: `data:` chunks + `[USAGE]` + `[TRACE]` + `[DONE]`
- [x] 7.3 PLAYER_ACTION before the stream; NARRATOR_RESPONSE with narrative delta at the end

## 8. Rewind (spec event-persistence)

- [x] 8.1 `POST /api/game/{cid}/rewind` — verified: removes 2, history clean, opening intact

## 9. Smoke test

- [x] 9.1 `backend/tests/smoke_test.py` (9 checks via ASGI) — 9/9 OK
- [x] 9.2 Real uvicorn server :8642 + import of the 14 scenarios + full turn — all 200 OK

## 10. Wrap-up

- [x] 10.1 Root README: "Run the backend" section
- [x] 10.2 Commit + push (UI, combat, crystals, npc-minds, Neo4j → next change)

## 11. Real providers + Hermes integration (spec llm-routing)

- [x] 11.1 `.env` + `.env.example` (MTG_PROVIDER/MTG_NARRATIVE_MODEL/MTG_AUXILIARY_MODEL/MTG_TEMPERATURE + keys per provider); python-dotenv at boot; safe fallback to mock
- [x] 11.2 LitellmProvider: stream with include_usage, sampling guard, windows, retry; transactional `set_provider` (validates before mutating — a failed switch has no partial effect)
- [x] 11.3 `GET/POST /api/settings`: runtime switch without restart; `api_key_set` boolean (key never exposed)
- [x] 11.4 Provider `hermes`: local OpenAI-compat endpoint (`hermes proxy start`, port 8645; MTG_HERMES_BASE_URL configurable); placeholder bearer (the proxy injects the OAuth credential); reachability validation in `set_provider` (422 with instruction if the proxy is down)
- [x] 11.5 Real tests: OpenAI-compat stub (tests/hermes_proxy_stub.py) + suite tests/test_hermes_provider.sh — verified: mock→hermes 200, 484-word opening via the hermes provider, SSE turn with [USAGE]/[TRACE]/[DONE], dead proxy → 422 with actionable message, return to mock; smoke 9/9 green

## 12. Frontend (spec frontend-ui) + full EN parity

- [x] 12.1 frontend/ Vite + React 19 + strict TS: green build (63KB gzip)
- [x] 12.2 GameCanvas: SSE prose + control tags rendered as a discreet line; opening in a highlighted block; autoscroll
- [x] 12.3 ActionInput: DO/SAY/CONTINUE/META verbs (SAY wraps in verbatim quotes), @ NPC autocomplete (unicode regex), clickable suggestions
- [x] 12.4 Inspector: turn/narrative clock/response counters + story card list; setup in UI (choice/text) calling setup-answers; rewind in the HUD
- [x] 12.5 Real E2E: vite dev (proxy /api → 8642) + backend — 15→16 scenarios imported through the proxy, campaign + 336-word opening via the UI route (tools/e2e_frontend.py)
- [x] 12.6 Full EN parity: scenarios/en/{o_mercado,inoculacao}.json validated (var_names, placeholders, cards, keywords PT∪EN, doctrinal tokens Lei 12.737/GEC/FIMI/McGuire/Bad News preserved); tools/validate_en.py now 8/8 pairs

## 13. Post-turn pipeline (specs memory-system, npc-minds, combat-system, journal-system)

- [x] 13.1 app/pipeline.py: MemoryPyramid (crystal every 4 events, SHORT→MEDIUM→LONG→MEMORY cascade, cursor, witnesses, lossless verbatim fallback, WORLD MEMORY with PRMNT_MEM/ARC_MEM/MID_MEM/RCNT_MEM + DELTA)
- [x] 13.2 NpcMinds (feeling/goal/opinion_of_player/secret_plan as NPC_THOUGHT; transient decay after 5 turns; fuzzy dedup ≥0.82; boundaries by witnesses; reset via event)
- [x] 13.3 Heuristic journal with canonical categories (DISCOVERY/RELATIONSHIP_CHANGE/COMBAT/DECISION/WORLD_EVENT; optional auxiliary LLM in production)
- [x] 13.4 Combat: classifier (grief/surrender/normal), anti-griefing rejects meta-gaming with reason persisted as response, 40/40/20 evaluation, outcome CRIT_SUCCESS..CRIT_FAIL imposed on the narrator via irrevocable injection, COMBAT_ACTION/COMBAT_RESULT persisted, [MODE]COMBAT signaled
- [x] 13.5 New routes: journal (filterable), memory-crystals, manual crystallize, npc-minds GET/PUT/DELETE, world-memory (prompt block)
- [x] 13.6 tests/pipeline_smoke_test.py 7/7: anti-griefing, automatic crystallization, WORLD MEMORY, journal+categories+filter, editable minds, combat outcome, post-restart reconstruction with crystals preserved; original smoke 9/9 green

## 14. Phase 2/3b — zone caching, plot, auditor, graph

- [x] 14.1 app/advanced.py ZonedPrompt: zone 0 (canon) + zone 1 (LORE+MEMORY) stable byte-for-byte between turns (sha256 fingerprint verified), volatile zone 2 (history, WORLD MEMORY, RELATIONSHIPS, active cards); `<narrator-instructions>` cloaking on the 1st user message; ephemeral cache_control TTL 1h; flag LUNAR_FEATURE_PROMPT_CACHE=0 → monolithic prompt
- [x] 14.2 PlotGenerator: rules per type (micro_hook 5/6/2h/8, npc 8/10/24h/6, plot_arc 12/20/48h/3), plot lock of one generation at a time, NONE rule (tense scene/active complication), manual generation POST /generate subject to the same rules, consumption releases lock
- [x] 14.3 Post-hoc auditor: flag LUNAR_FEATURE_NARRATOR_AUDIT + default timeout 210s, total context (scene + world), rewrite scoped to agency/continuity, multiset fingerprint of [ITEM_*] tags invalidates divergence, safe degradation on parse/timeout
- [x] 14.4 WorldGraph in memory (Neo4j optional by decision): canonical types NPC/LOCATION/FACTION/ITEM/EVENT, canonical resolution with token-containment ("Kovács"→"Instrutora Kovács" with alias), deduplicated snapshot, textual search with local fallback, WORLD RELATIONSHIPS in the prompt; entity extraction from story cards each turn with [GRAPH]
- [x] 14.5 Routes: graph, graph-search, generate, traces GET/DELETE (asdict)
- [x] 14.6 tests/advanced_smoke_test.py 5/5: stable fingerprints, plot lock/cooldown, auditor passthrough+fingerprint, canonical graph+search, integrated turn; regression 9/9 + 7/7 green

## 15. Docker Compose (deploy of the full stack)

- [x] 15.1 backend/Dockerfile: python:3.14-slim + uv (--frozen from uv.lock), MTG_DB_PATH=/data on a volume, healthcheck /api/health, .env out of the image (optional env_file in compose)
- [x] 15.2 frontend/Dockerfile: multi-stage (Vite build → nginx:alpine) with nginx.conf proxy /api → backend:8642 (SSE: proxy_buffering off), IPv4 healthcheck
- [x] 15.3 docker-compose.yml: backend (:8642) + frontend (:8080, depends_on healthy) + neo4j in an optional profile ("graph"); extra_hosts host.docker.internal for the hermes proxy on the host; volumes mtg-events/mtg-neo4j
- [x] 15.4 Real scenario persistence: scenarios table in SQLite + save on import + reload on startup (scenarios and campaigns survive restart/rebuild — verified with a restart mid-flow)
- [x] 15.5 Docker E2E verified: build/up --wait healthy, 16 scenarios imported via :8080, 336-word opening, restart → 16 scenarios + campaign intact, SSE turn through nginx with [USAGE]/[TRACE]/[DONE]; local regression 9/9+7/7+5/5

## 16. As-built openspec reconciliation

- [x] 16.1 Spec deltas in the change: game-api (scenario persistence + MTG_DB_PATH, world-memory route, [AUDIT]/[GRAPH]/[PLOT] tags), llm-routing (hermes provider with conditional probe, config via .env, transactional switch), deployment (NEW spec: Compose stack, Neo4j profile, credentials out of the image, host-gateway), memory-system (immutable crystal_consumed marker, no-LLM mode), npc-minds (NPC_THOUGHT by replay + reset), plot-generation (persisted seeds, manual generation), knowledge-graph (graph in events with Neo4j optional), narrative-audit ([AUDIT] in the flow, passthrough without router), frontend-ui (campaign flow, build/deploy, relative routes)
- [x] 16.2 proposal.md: as-built note documenting the evolution beyond the walking skeleton (sections 11–15)
- [x] 16.3 config.yaml: `deployment` capability registered
- [x] 16.4 Structural validation of the deltas (MODIFIED/ADDED format, WHEN/THEN)

#### `changes/add-engine-core/specs/deployment/spec.md`

## ADDED Requirements

### Requirement: Docker Compose stack

The project SHALL provide `docker-compose.yml` with backend (FastAPI/uv, port 8642, healthcheck at `/api/health`, event store on a named volume) and frontend (multi-stage Vite build served by nginx on port 8080 with `/api` proxy and SSE without buffering), orchestrated with `depends_on` conditioned on healthcheck.

#### Scenario: Full startup

- **WHEN** `docker compose up -d --build --wait` is executed
- **THEN** both services SHALL be healthy
- **AND** the game SHALL be accessible at `:8080` with the API responding via the nginx proxy

#### Scenario: Restart preserves state

- **WHEN** the backend container restarts
- **THEN** imported scenarios and campaigns SHALL persist (volume + SQLite)

### Requirement: Neo4j as an optional profile

The Neo4j service SHALL sit behind a profile (`--profile graph`), outside the default stack, faithful to the architecture decision of an in-memory graph with Neo4j as an optional upgrade.

#### Scenario: Default stack

- **WHEN** `docker compose up -d` without a profile
- **THEN** no Neo4j container SHALL be created

### Requirement: Credentials out of the image

No credential SHALL enter the Docker images; the backend receives `.env` via `env_file` (optional) and the compose defines no provider defaults that override the `.env`.

#### Scenario: Missing env_file

- **WHEN** the deploy runs without `backend/.env`
- **THEN** the backend SHALL come up with the mock provider and remain healthy

### Requirement: Host access for the Hermes proxy

The backend service SHALL resolve `host.docker.internal` (`extra_hosts: host-gateway`) to consume the Hermes Agent proxy running on the host machine.

#### Scenario: Proxy on the host

- **WHEN** `MTG_HERMES_BASE_URL=http://host.docker.internal:8645/v1` with the proxy active on the host
- **THEN** the backend container SHALL reach the endpoint

#### `changes/add-engine-core/specs/frontend-ui/spec.md`

## ADDED Requirements

### Requirement: Campaign flow in the UI

The UI SHALL offer: scenario selection (list by language), question setup (choice as buttons, text as input), opening generation, turn loop with SSE prose rendered incrementally, control tags displayed discreetly, rewind in the HUD, and return to the scenario list.

#### Scenario: Turn played through the UI

- **WHEN** the player submits an action
- **THEN** the prose SHALL appear streaming on the canvas
- **AND** the `[USAGE]/[TRACE]/[DONE]` tags SHALL be rendered without breaking the layout

### Requirement: Production build and deploy

The frontend SHALL build for production (`npm run build`) and be served by the Docker Compose nginx with an `/api` proxy to the backend; the UI consumes exclusively relative `/api` routes (compatible with the Vite dev-proxy and nginx).

#### Scenario: Dev and production identical

- **WHEN** the same UI runs in `vite dev` (5173) and in nginx (8080)
- **THEN** the calls SHALL use the same relative `/api` path

#### `changes/add-engine-core/specs/game-api/spec.md`

## ADDED Requirements

### Requirement: Persistence of imported scenarios

Imported scenarios SHALL be persisted in SQLite (`scenarios` table) and reloaded on backend startup, surviving container restarts and rebuilds; the linked campaign SHALL remain playable after restart.

#### Scenario: Restart with imported scenarios

- **WHEN** the backend restarts after scenarios have been imported
- **THEN** all scenarios SHALL be listed without a new import
- **AND** existing campaigns SHALL remain associated with their scenario

#### Scenario: Configurable database path

- **WHEN** the environment variable `MTG_DB_PATH` is set
- **THEN** the event store SHALL use that path (default: `backend/events.db`; in container: `/data` volume)

### Requirement: world-memory route

The backend SHALL expose `GET /api/game/{cid}/world-memory` returning the WORLD MEMORY block as assembled in the prompt (levels PRMNT_MEM/ARC_MEM/MID_MEM/RCNT_MEM + DELTA), for inspection and debugging.

#### Scenario: Block inspection

- **WHEN** the route is queried with a valid campaign
- **THEN** the returned block SHALL contain the per-level headers and the DELTA of non-crystallized events

### Requirement: Pipeline control tags in the SSE flow

Besides `[MODE]`, `[JOURNAL]`, `[CRYSTAL]` and `[TRUNCATE_CLEAN]`, the SSE flow SHALL emit `[AUDIT]` (auditor decision), `[GRAPH]` (entities extracted in the turn) and `[PLOT]` (generated plot seed), each on a single line.

#### Scenario: Auditor rewrites

- **WHEN** the post-hoc auditor produces a valid rewrite
- **THEN** the flow SHALL emit `[AUDIT] {"action": "rewrite"}` before `[USAGE]`

#### Scenario: Plot seed generated

- **WHEN** the automatic generator produces a seed at the end of the turn
- **THEN** the flow SHALL emit `[PLOT]` with the seed's type and name

#### `changes/add-engine-core/specs/knowledge-graph/spec.md`

## MODIFIED Requirements

### Requirement: Narrative entity extraction

The system SHALL extract entities and relations from each narrator response and record them as nodes and edges in the campaign graph. In the reference implementation the graph lives in memory reconstructed from events (`kind=graph`) with deduplicated upsert; the Neo4j backend remains an optional upgrade (Docker profile `graph`), and the per-turn heuristic extraction cross-references entities mentioned in the prose with the scenario's story cards.

#### Scenario: NPC mentioned for the first time

- **WHEN** the narrative introduces a new NPC with relations
- **THEN** the graph SHALL create the corresponding node and the edges to existing entities

#### Scenario: Graph reconstruction

- **WHEN** the backend restarts
- **THEN** the graph snapshot SHALL be identical to the pre-restart one (derived from events)

#### `changes/add-engine-core/specs/llm-routing/spec.md`

## ADDED Requirements

### Requirement: hermes provider (OpenAI-compat endpoint)

The router SHALL support the `hermes` provider: a configurable OpenAI-compatible endpoint (`MTG_HERMES_BASE_URL`, default `http://127.0.0.1:8645/v1` for the local Hermes Agent proxy) with an optional key (`MTG_HERMES_API_KEY`). When the endpoint is the local proxy, the OAuth credential is injected by the proxy and the game's bearer is a placeholder; when the endpoint is remote with an explicit key, the key is used directly and the boot performs no network probe (a slow network must not block startup).

#### Scenario: Local proxy without key

- **WHEN** `hermes` is selected without `MTG_HERMES_API_KEY`
- **THEN** the router SHALL validate endpoint reachability before activating and refuse with an actionable instruction if inaccessible

#### Scenario: Remote endpoint with key

- **WHEN** `hermes` is selected with `MTG_HERMES_API_KEY` set
- **THEN** the provider SHALL activate without a network probe at boot
- **AND** the key SHALL be sent as bearer on calls

### Requirement: Configuration via .env file

The backend SHALL load provider configuration from `backend/.env` (gitignored) at boot: `MTG_PROVIDER`, `MTG_NARRATIVE_MODEL`, `MTG_AUXILIARY_MODEL`, `MTG_TEMPERATURE`, plus the per-provider keys. Without `.env` or with a keyless provider, the backend SHALL come up in `mock` with a warning in the trace, never breaking the boot.

#### Scenario: Without .env

- **WHEN** the backend starts without a `.env` file
- **THEN** the active provider SHALL be `mock` and `/api/health` SHALL respond ok

#### Scenario: Transactional provider switch

- **WHEN** `POST /api/settings` receives an invalid provider or one without credentials
- **THEN** no partial mutation SHALL occur (validation before the switch)
- **AND** the response SHALL be 422 with instructions on what to configure

#### `changes/add-engine-core/specs/memory-system/spec.md`

## MODIFIED Requirements

### Requirement: 4-level crystallization pyramid

The system SHALL distill a SHORT crystal every 4 player events, consolidate 4 SHORTs into 1 MEDIUM, 4 MEDIUMs into 1 LONG, and 4 LONGs into 1 MEMORY (permanent world facts), with an automatic cascade after each crystallization. Upper-level consolidation marks the sources as consumed via a persisted consumption marker (`crystal_consumed` event), without mutating already-registered events; the consumption is respected in reconstruction and consumed SHORT crystals do not appear in the WORLD MEMORY.

#### Scenario: Consumption marker is immutable

- **WHEN** 4 SHORT crystals are consolidated into 1 MEDIUM
- **THEN** the source events SHALL remain intact in the log
- **AND** a `crystal_consumed` marker event SHALL record the consumed ids

#### Scenario: Automatic volume trigger

- **WHEN** the number of not-yet-crystallized events reaches 4
- **THEN** a SHORT crystal SHALL be created covering exactly those events
- **AND** the crystallization cursor SHALL advance to the last covered event

#### Scenario: Consolidation cascade

- **WHEN** a SHORT crystal is created and 4 unconsumed SHORTs already exist
- **THEN** the system SHALL consolidate them into 1 MEDIUM and mark the sources as consumed
- **AND** the cascade SHALL continue to LONG and MEMORY while quartets remain

#### Scenario: Manual crystallization

- **WHEN** the player triggers manual crystallization through the interface
- **THEN** the system SHALL crystallize the pending events immediately

## ADDED Requirements

### Requirement: LLM-free distillation mode

Distillation SHALL operate in a deterministic mode without an LLM (`use_llm=false`) for development and testing at no cost, keeping the auxiliary-model consolidation path active when available; on LLM failure the lossless verbatim fallback applies in both modes.

#### Scenario: Dev without provider

- **WHEN** crystals are created with the mock provider
- **THEN** the ai_content SHALL contain the sources verbatim without loss

#### `changes/add-engine-core/specs/narrative-audit/spec.md`

## ADDED Requirements

### Requirement: Auditor decision serialized in the flow

The auditor decision SHALL be exposed as an `[AUDIT]` tag in the turn's SSE flow (action: rewrite/passthrough with reason), and the no-LLM mode (`router=None`) SHALL degrade to explicit passthrough, never blocking the turn.

#### Scenario: No router configured

- **WHEN** the audit runs in an environment without an auxiliary LLM
- **THEN** the decision SHALL be passthrough with the reason recorded

#### `changes/add-engine-core/specs/npc-minds/spec.md`

## ADDED Requirements

### Requirement: On-demand and per-event mind updates

Minds SHALL be updatable via `NPC_THOUGHT` event (name + turn + fields) and reconstructed by replay; the reset SHALL be a `kind=reset` event that removes the mind during reconstruction, without mutating the log. The automatic post-turn update SHALL be optional (auxiliary LLM) with manual update via PUT always available.

#### Scenario: Mind reconstructed after restart

- **WHEN** the backend restarts with persisted NPC_THOUGHTs
- **THEN** the minds SHALL be reconstructed with the most recent state per NPC

#### `changes/add-engine-core/specs/plot-generation/spec.md`

## ADDED Requirements

### Requirement: Seed persistence and inspection

Plot seeds SHALL be `PLOT_GENERATION` events (`kind=generated`/`consumed`) reconstructed by replay; the generator state (active, triggers, cooldowns) SHALL derive exclusively from the event store, and manual generation SHALL use the same rules as the automatic one.

#### Scenario: Manual generation respects the lock

- **WHEN** `POST /generate` is called with an active, unconsumed seed
- **THEN** the response SHALL refuse with the plot lock reason

<!-- source: changes/add-military-forces-catalog/ -->

### add-military-forces-catalog

#### `proposal.md`

# add-military-forces-catalog

## Why

The game needs real military data (units, squadrons, specializations, the organizational structure of the Brazilian Armed Forces; world elite forces with selection numbers; the ideal soldier model across the physical, mental, cognitive and psychological dimensions) to feed military training scenarios with verifiable grounding — avoiding invented units or wrong designations (e.g.: confusing Pampa/Anápolis, calling the 1º/7º GAv "Corsário" when the official name is Orungan, or citing "FEsEx" which does not exist).

## What Changes

**Military forces catalog with provenance**
- From: no military dataset; scenario lore would depend on LLM memory (subject to hallucinating unit names).
- To: dataset `data/military/forces_catalog.json` with 149 verified facts (45 Exército, 48 Marinha/FAB, 56 world elite), each with `source_url` and verification date; unconfirmed items marked as unverified with a note (20 documented uncertainties, e.g.: "Corsário", "P-8A na FAB", "Batalhão São Mateus").
- Reason: web research with 3 subagents on primary sources (eb.mil.br, marinha.mil.br, fab.mil.br, planalto.gov.br, socom.mil, rand.org, DTIC, PMC/Frontiers/PLOS/ScienceDirect studies).
- Impact: non-breaking; versioned static data.

**Multidimensional ideal soldier model**
- From: no model; an implicit "perfect soldier" concept.
- To: `data/military/ideal_soldier_model.json` with a grounded thesis (no single profile exists — profiles per function), 4 dimensions (physical/mental/cognitive/psychological) with evidence and benchmarks, 9 attrition rates and 4 doctrines (SOF Truths, NATO HFM-171, H2F, CANSOF).
- Impact: non-breaking.

**Importable scenarios**
- From: no military-themed scenario.
- To: `scenarios/brasil_em_armas.json` (a career in the Brazilian Armed Forces: 40 cards — COpEsp, Bda Inf Pqdt, CIGS, Tonelero/COMANF, GRUMEC, FAB squadrons with correct war names, 6 fictional instructor NPCs, 7 real locations) and `scenarios/a_comitiva_soldado_ideal.json` (international elite selection with real numbers: 38 cards, 21 units from 12 countries, tests with verified attrition). Both follow the scenario-authoring format (unique var_names, choice/text options, NPC/LOCATION/FACTION/LORE cards with keywords for RAG).
- Impact: non-breaking; importable via POST /api/scenarios/import.

**New spec: military-forces-catalog**
- 7 requirements: Brazilian Armed Forces catalog, specializations/courses, world elites with sourced numbers, ideal soldier model, mandatory provenance, export to story cards, ready-made scenarios.

## Impact

- Affected specs: none modified; adds `military-forces-catalog`.
- No data migrated, no contract broken. Instructor NPCs are archetypal fictional characters — no living real person is a game character.

#### `tasks.md`

# Tasks: add-military-forces-catalog

## 1. Catálogo militar e modelo do soldado ideal

- [x] 1.1 Pesquisar estrutura do Exército (8 CMAs, brigadas, COpEsp, Bda Pqdt, CIGS, especializações) com fontes — 45 fatos
- [x] 1.2 Pesquisar Marinha/CFN (Divisão Anfíbia, Tonelero, GRUMEC, Aviação Naval, NAM Atlântico) e FAB (Alas, esquadrões com nome de guerra, PARA-SAR, formação) com fontes — 48 fatos
- [x] 1.3 Pesquisar elites mundiais (19 unidades, 12 países) com taxas de seleção e padrões físicos/mentais/cognitivos/psicológicos fonteados — 56 fatos
- [x] 1.4 Consolidar em data/military/forces_catalog.json (149 fatos únicos, dedup, incertezas documentadas)
- [x] 1.5 Gerar data/military/ideal_soldier_model.json (4 dimensões + benchmarks + doutrinas)

## 2. Cenários militares

- [x] 2.1 Gerar scenarios/brasil_em_armas.json (40 cards, 5 perguntas de setup)
- [x] 2.2 Gerar scenarios/a_comitiva_soldado_ideal.json (38 cards, 5 perguntas de setup)
- [x] 2.3 Criar spec openspec/specs/military-forces-catalog/spec.md
- [x] 2.4 Importar via POST /api/scenarios/import quando o backend estiver disponível (2026-08-21: 16 cenários PT+EN importados via `tools/import_all_scenarios.py` contra backend mock em :8642; turno de amostra ok)

## 3. Universo "O Cidadão do Futuro" (worldbuilding + cenário)

- [x] 3.1 Consolidar cânone do autor (pastes) em world/citizen_of_the_future/worldbuilding_vol1.md
- [x] 3.2 Desenvolver as duas contradições fundadoras (Dilema da Utilidade → "a neve"/Vetor Nulo; Pressão da Eficiência → Tirania da Manutenção) em worldbuilding_vol2.md
- [x] 3.3 Expandir as 3 fronteiras propostas pelo autor (Triagem/Infância, Relações Internacionais, Cidades/Vida Cotidiana) com contradições vivas próprias
- [x] 3.4 Desenvolver a Doutrina de Defesa Integral (a Malha) em worldbuilding_vol3.md: cinco autodefesas (corpo/dados/mente/direitos/bolso), espionagem/CI/OSINT universal, cybernética completa (ciber-guerra + corpo-máquina), pedagogia sem quartel (Colégios de Defesa, Manobra/Companhia Vermelha, Reserva Sentinela), 6 contradições novas (tradecraft universal, duas moedas, idade do treino, Famintos, inverno tranquilo, Guarda na rua)
- [x] 3.5 Gerar scenarios/o_cidadao_do_futuro.json (42 cards: 19 LORE, 7 FACTION, 9 NPC, 7 LOCATION; 5 perguntas de setup incluindo mesh_role)

## 3b. Regimento de Operações de Informação (doutrina pública PSYOPS/InfoWar)

- [x] 3b.1 Pesquisar doutrina US pública de PSYOPS/MISO (JP 3-53 2003, JP 3-13.2, FM 33-1 1979/1993, FM 33-1-1 1994, 4th/2nd PSYOP Group, OSS/Chieu Hoi/Coreia, inoculação de McGuire) — 40 fatos com URL
- [x] 3b.2 Pesquisar doutrina conjunta pública de IO/EW/MILDEC/OPSEC (JP 3-13 2006/2012, JP 3-85 EMSO, JP 3-13.4, JP 3-13.3/3-54, NATO StratCom COE/Cognitive Warfare, GEC 5 pilares, EUvsDisinfo/FIMI, FM 3-0 MDO) — 42 fatos com URL
- [x] 3b.3 Pesquisar doutrina brasileira aberta (Vitória nas Sombras/EMA-335/COMOPNAVINST 30-01, C 45-4/1999 público vs EB70-MC-10.230 restrito, LBDN 2012, END/PND 2020, 1º B Op Psc, CDCiber/MD31-M-07) — 35 fatos com URL
- [x] 3b.4 Consolidar data/military/psyops_infowar_doctrine.json (117 fatos únicos com URL, 15 incertezas documentadas — ex.: JP 3-53 2012 e FM 3-53 não públicos; rótulos IPA não doutrinários)
- [x] 3b.5 Escrever world/regimento_operacoes_informacao.md — 8 títulos, 30 artigos: MISO (TAA, branco/cinza/preto, credibilidade, contrapropaganda, SCAME), IO/IRCs, MILDEC (meta/objetivo/terminação), OPSEC (5 passos), EMSO, ciber MD31-M-07, doutrina BR (EMA-335, C 45-4, ameaças híbridas), guerra cognitiva/GEC/FIMI, inoculação + história aberta
- [x] 3b.6 Gerar scenarios/guerra_das_mentes.json (28 cards; 4 setup vars: io_role, exercise_day, principle, dilemma) — Exercício Convergência, time vermelho "Companhia Cinza" seguindo doutrina real

## 3c. Biblioteca de Inteligência e Contrainteligência (acervo público)

- [x] 3c.1 Pesquisar acervo CIA/FBI/MI5-MI6 (National Security Act 1947, MKULTRA/Family Jewels/Church Committee desclassificados, COINTELPRO, Hanssen, ISA 1994, histórias oficiais Andrew/Jeffery, Cambridge Five) — 26 fatos/obras com URL (recovery de subagente que abortou no sumário)
- [x] 3c.2 Pesquisar Mossad/KGB/MSS (Caesarea/Kidon, Eichmann, Ira de Deus/Lillehammer, Entebbe, Stuxnet; estrutura KGB PGU/2ª/8ª, Arquivo Mitrokhin, VENONA, Ames/Hanssen/Tolkachev; MSS 1983, casos DOJ Yanjun Xu/Su Bin/Shujun Wang; livros e documentários canônicos) — 44 fatos com URL
- [x] 3c.3 Pesquisar Brasil + ofício de CI (SNI Lei 4.341/1964→extinção 1990→ABIN Lei 9.883/1999, doutrina pública ABIN 2023 com CI preventiva/ativa, PCI EB70-MT-10.401, CCAI, ABIN 2.0/Última Milha; Dulles/Heuer/Pherson/ICD 203/KUBARK desclassificado; documentários verificados) — 44 fatos com URL
- [x] 3c.4 Consolidar data/military/intelligence_library.json (114 itens com URL; critério: nada classificado, nada vazado — só desclassificados oficiais, histórias autorizadas, editoras, processos públicos)
- [x] 3c.5 Escrever world/biblioteca_inteligencia.md (dossiê em 7 seções: EUA, Reino Unido, Israel, URSS/Rússia, China, Brasil, ofício de CI; regra da casa "o admitido é o piso"; KUBARK como artefato histórico com aviso)
- [x] 3c.6 Gerar scenarios/a_biblioteca_de_vidro.json (24 cards: 14 LORE, 6 NPC, 4 LOCATION; setup: player_function, era_focus, method, haunting_case)

## 3d. Avatar Mirror (espelhamento do jogador, todas as idades)

- [x] 3d.1 Spec openspec/specs/avatar-mirror/spec.md (7 requirements: níveis 0–3 com consentimento granular, Camada de Tradução Narrativa, deny-list absoluta, bandas de idade A/B/C, orçamento de contexto com zona volátil, fronteira de memória/esquecimento LGPD)
- [x] 3d.2 Schema data/mirror/mirror_profile.schema.json (mirror-profile/1.0; additionalProperties false em todos os objetos; _deny_list documentado) + exemplos adulto (nível 3 com recusa de eixo) e criança (banda A nível 1)
- [x] 3d.3 Documento de decisão world/avatar_mirror_decisao.md (orçamentos numerados: card 400t + cristal 600t, 0 tokens de dado real no request LLM; matriz de consentimento; LGPD arts. 7/14/18 como features; exemplo do que o narrador vê)

## 3e. Frente 1 — fechamento (EN, bandejas de idade, tabela regimento→mecânica)

- [x] 3e.1 Spec openspec/specs/age-banding/spec.md (6 requirements: compatibilidade por banda, injunções narrativas em zona volátil, preservação de mecânica, limite de espelhamento por banda, classificação auditável, "sem condescendência")
- [x] 3e.2 data/age_bands.json (5 cenários × 3 bandas: 6 full, 8 adapted, 1 blocked — Biblioteca de Vidro/A com substituto sugerido; injunções A/B prontas para injeção no prompt)
- [x] 3e.3 world/tabela_regimento_mecanica.md (30 arts. do regimento → mecânica verificável: TAAWS obrigatória, meter de credibilidade, SCAME como mini-jogo, linha vermelha MISO/PA como flag, MILDEC meta/objetivo em formulário, loop OPSEC de 5 passos, posse de frequência EMSO, FIMI ≥2 observáveis, Modo Inoculação, auditor como inspetor doutrinário)
- [x] 3e.4 Versões EN dos 5 cenários em scenarios/en/ (5 subagentes com regras de preservação: var_names, placeholders, números/URLs, designações militares, keywords PT+EN; validação programática por tradutor)

## 3f. Frente cibernética — certificações e corpos de conhecimento

- [x] 3f.1 Pesquisar certificações (CEH 312-50 blueprint v5/9 domínios, CEH Practical 6h/20 desafios, OSCP PEN-200 exame 24h + OSCP+ 2024, PenTest+ PT0-003 5 domínios, Cisco CEH programa 2024 sem exame 350-xxx, trilha complementar e legalidade) — 39 fatos/5 incertezas com URLs oficiais (data/military/certificacoes_ethical_hacking_fatos.json)
- [x] 3f.2 Pesquisar corpos de conhecimento (CyBOK v1.1: 21 KAs em 5 categorias; SEBoK/INCOSE-BKCASE; SWEBOK V4.0 ISO/IEC 19759; NICE Framework SP 800-181r1) — 35 fatos/4 incertezas com URLs oficiais (data/military/bok_facts.json)
- [x] 3f.3 Consolidar data/military/cyber_doctrine.json (74 fatos + trilha Recruta→Mestre espelhando certs reais + 5 ecos no universo)
- [x] 3f.4 Escrever world/doutrina_ciberdefesa.md (3 camadas do saber: o quê/como/porquê; regra de ouro da autorização escrita; Try Harder como doutrina da Reserva Sentinela)
- [x] 3f.5 Gerar scenarios/try_harder.json (19 cards: 10 LORE, 5 NPC, 4 LOCATION; setup: player_role, exercise_type, doctrinal_anchor, signature_tool) — Arena Try Harder, autorização primeiro, kill chain dupla vermelho/azul, mercado cinza como tentação narrativa
- [x] 3f.6 Registrar em data/age_bands.json (nativo banda B; A adaptado como aventura de segurança digital sem comandos reais; C full) e validar 11 cenários

## 3g. Frente 2 — antagonista jogável e Inoculação infantil

- [x] 3g.1 scenarios/o_mercado.json (17 cards: 8 LORE, 5 NPC, 4 LOCATION; setup: operator_role, target_market, moral_line) — a nação-mercado pelo lado de dentro: espectro cinza/ameaças híbridas, 5 pilares GEC, FIMI comportamental, bolsa de talento, Lei 12.737 como fronteira; regras do narrador: abaixo do limiar sempre, consequências humanas em close, recusa sempre jogável, nada de manual operacional literal; banda C nativa, B adaptado, A bloqueado (substituto: Inoculação)
- [x] 3g.2 scenarios/inoculacao.json (12 cards: 7 LORE, 3 NPC, 2 LOCATION; setup: player_age, module_day, virus_week) — Bad News-style para 9–14: fórmula da dose (germe rotulado + antes + antídoto), 6 gatilhos nomeados em voz alta, detector de comportamento com ≥2 indícios, vacina da turma mensurável; ancorado em McGuire 1961, Banas & Rains 2010, Roozenbeek & van der Linden 2019 (URLs no dataset psyops); banda A nativa, full em A/B/C
- [x] 3g.3 age_bands.json atualizado (8 cenários × 3 bandas: 12 full, 10 adapted, 2 blocked) e validação dos 14 cenários

## 4. Validação

- [x] 4.1 Validação estrutural dos specs (WHEN/THEN, sem duplicatas)
- [x] 4.2 Validação de formato scenario-authoring nos 3 cenários (var_names únicos, choice/text, tipos de card, interpolação)
- [ ] 4.3 A/B narrativo de 6+ turnos por cenário após import (nomes de unidades corretos; tom socioficção sem distopia cartoon nem propaganda utópica; procedimento doutrinário correto em guerra_das_mentes)

#### `changes/add-military-forces-catalog/specs/military-forces-catalog/spec.md`

## ADDED Requirements


### Requirement: Brazilian Armed Forces Catalog

The system SHALL maintain a structured catalog of the three Brazilian Singular Forces, covering the chain of command (Military Area Commands and equivalent Navy and Air Force commands), major units (brigades, divisions, flotillas, wings/groups), operational units (battalions, squadrons with a war name) and education/training establishments.

#### Scenario: Unit with a War Name

- **WHEN** an Air Force unit has a war name (e.g., a squadron)
- **THEN** the record SHALL carry the numerical designation, war name, base/headquarters, primary mission, and aircraft/asset employed when applicable

#### Scenario: Explicit Subordination

- **WHEN** a unit is registered in the catalog
- **THEN** the record SHALL indicate the command it is subordinate to and the headquarters city

### Requirement: Military Specializations and Courses

The catalog SHALL map military specializations (e.g., special forces, commandos, parachuting, diving, jungle warfare, search and rescue, fighter aviation, air traffic control) with the force that offers them, the responsible unit/training center, and the nature of the qualification.

#### Scenario: Specialization with an Associated Course

- **WHEN** a specialization requires a formal course (e.g., Estágio de Operações Especiais)
- **THEN** the record SHALL identify the training unit and the responsible force

### Requirement: World Elite Forces with Selection Standards

The catalog SHALL cover international elite units with country, name, typical mission and — when publicly documented — selection numbers (pass rates, duration, key tests), plus the physical, mental, cognitive, and psychological dimensions assessed.

#### Scenario: Selection Number with a Source

- **WHEN** a pass rate or selection duration is recorded
- **THEN** the record SHALL carry the source of the information
- **AND** numbers without a confirmed source SHALL be marked as unverified instead of being silently omitted or invented

### Requirement: Multidimensional Model of the Ideal Soldier

The system SHALL maintain a model of the "soldier closest to perfection" organized into the physical, mental, cognitive, and psychological dimensions, with measurable components per dimension and the documented trade-off that no single profile exists — optimal profiles differ by role.

#### Scenario: Profile by Role

- **WHEN** two distinct military roles are compared (e.g., special forces operator vs fighter pilot)
- **THEN** the model SHALL reflect differentiated requirements per dimension instead of a single "perfection" ranking

### Requirement: Provenance of All Data

Every catalog record SHALL carry a source field (URL or documentary reference) and a verification date; data drawn from general knowledge without verification SHALL be marked as unverified.

#### Scenario: Record Without a Source

- **WHEN** a fact could not be confirmed in an accessible source
- **THEN** the record SHALL be marked `verified: false` with an explanatory note

### Requirement: Export to Story Cards

The catalog SHALL be exportable as story cards of the types NPC, LOCATION, FACTION, ITEM, and LORE per scenario, ready for import in the scenario interchange format, with keywords extracted from unit names for RAG selection.

#### Scenario: Squadron as a LORE Card

- **WHEN** the author exports a force's units to a scenario
- **THEN** each unit SHALL become a card with name, type, descriptive text, and keywords including acronym and war name

### Requirement: Ready-Made Military Training Scenarios

The system SHALL offer complete, importable scenarios built on the catalog: (a) training in the Brazilian Armed Forces with career progression and real specializations, and (b) international elite selection toward the ideal soldier, both bilingual (en + pt-br) and with interpolatable setup questions.

#### Scenario: Importable Brazilian Scenario

- **WHEN** the author imports the Brazilian Armed Forces scenario
- **THEN** the scenario SHALL arrive with lore, setup questions, and complete story cards, ready to create a campaign

#### Scenario: Specialization Progression in the Fiction

- **WHEN** the player chooses force and specialization in the training scenario setup
- **THEN** the answers SHALL interpolate into the lore and tone to steer the training narrative

<!-- source: changes/add-worldbuilding-research/ -->

### add-worldbuilding-research

#### `proposal.md`

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

#### `tasks.md`

# Tasks

- [x] Create `data/worldbuilding/lessons.json` with schema (source game, mechanic, evidence/source, candidate translation, status, target spec) — created with 6 seed cards (2 per source game), all status=proposta
- [ ] Research and document Albion Online mechanics (economy, territories, risk bands, seasons) in public sources
- [ ] Research and document GTA San Andreas mechanics (stats, gangs, NPC routines, wanted level, gating) in public sources
- [ ] Research and document MUD mechanics (offline persistent world, room+look network, social channels, RPI, OLC/MOO) in public sources
- [ ] Build a Doom 3-style WebGL prototype (first-person, dynamic lighting, lore terminals, triggers) with original/free assets
- [ ] Map each prototype element to lesson cards
- [ ] Triage cards (proposta → aceita/descartada) and open changes in the target specs for the accepted ones
- [ ] Document provenance/license of all prototype assets

#### `changes/add-worldbuilding-research/specs/worldbuilding-research/spec.md`

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

<!-- source: changes/fix-auditor-agency-false-positive/ -->

### fix-auditor-agency-false-positive

#### `proposal.md`

# Proposal: Reduce agency false positive when NPC proposes a plan

## Why

The PHASE 3b A/B validation (docs/fase3b_ab.md, section "Fix — context-aware Auditor") measured a residue: when an NPC proposes a plan to the player, the Auditor still treats the NPC's speech as player agency in ~1/3 of cases (agency false positive). The prompt reinforcement already applied reduced it to ~2/3 clean, but the residue is model-dependent (DeepSeek; Opus judges 4/4 correct). Today there is no continuous telemetry to detect regression of this behavior in production — the only yardstick has been the one-shot A/B harness.

## What Changes

**Auditor — agency rule**
- From: the agency ceiling is "player input + established scene", but NPC speech proposing plans keeps being confused with player agency in ~1/3 of cases on the DeepSeek auxiliary model.
- To: the agency rule explicitly distinguishes speech authorship: speech initiated by an NPC that proposes/suggests/Offers something to the player does not count as player agency, even when the narrator writes it in an imperative tone.
- Reason: architectural false positive confirmed by adversarial validation (idx13, 4/4 judges); rewriting makes the prose worse instead of better.
- Impact: non-breaking; affects only the auditor prompt (backend/app/engines/auditor_engine.py) and the `_PRE_EMIT_KEYS` key list.

**False positive telemetry**
- From: no continuous metric for agency false positives; auditor quality measured only by a manual A/B harness.
- To: counter of rejected/kept rewrites by reason (parse_failed, item_tag_violation, agency_false_positive_flag) logged per turn and exposed in devtools.
- Reason: without telemetry, regressions in the model-dependent residue are invisible until the next manual A/B.
- Impact: non-breaking; adds a field to the already persisted trace payload.

## Impact

- Affected specs: narrative-audit (requirement "Rewrite scoped to agency and continuity" and "Telemetry").
- No data migrated, no contract broken. The flags LUNAR_FEATURE_NARRATOR_AUDIT / LUNAR_AUDIT_TIMEOUT_S / LUNAR_AUDIT_REASONING_HEADROOM remain valid.

#### `design.md`

# Design: fix-auditor-agency-false-positive

## Context

The AuditorEngine (backend/app/engines/auditor_engine.py, 395 loc) runs post-hoc over the narrator's prose. The PHASE 3b A/B validation documented: rewrite ≈ 5.6%, inert in aggregate, but with an agency false positive when an NPC proposes a plan (idx13). The context-aware fix (recent_scene + world_context) already landed; the residue comes from the agency prompt.

## Decisions

1. **Agency prompt reinforcement via "speech authorship"** — instead of listing more exceptions (pink-elephant pattern: exceptions become examples), the rule now asks "who initiated the speech?". NPC-initiated speech is never player agency, regardless of tone. Attack the principle, not the cases.
2. **Telemetry as a field on the existing trace** — no new channel: the TraceStore already persists entries per turn; add `decision`/`reason` to the auditor entry's payload. Zero migration (entries are JSON).
3. **Derived, not stored counters** — the devtools aggregates from traces on demand; no aggregation table, keeping the append-only event sourcing clean.

## Alternatives considered

- **Layer 1 (source gate via tool-call)**: deferred by the team (PLANO.md: "Camada 1 diferida"); A/B showed PHASE 3a already fixed tics at the source.
- **Separate judge model for agency**: rejected — doubles per-turn cost for a ~5% incidence problem.

## Risks

- Prompt changes may alter PT behavior (the rule exists in both EN and PT). Mitigation: minimum A/B of 12 passages (6 EN, 6 PT) with the backend/scripts/ab_auditor.py harness before merge.
- `_PRE_EMIT_KEYS` +2 keys already applied in the previous iteration — check for duplication when extending.

#### `tasks.md`

# Tasks: fix-auditor-agency-false-positive

## 1. Agency rule by speech authorship

- [x] 1.1 Extend the auditor's agency prompt with the speech authorship principle: speech initiated by an NPC that proposes/suggests/offers is not player agency (as-built: single PT-BR prompt in `backend/app/advanced.py::run_audit`)
- [x] 1.2 Replicate the change in the auditor's PT-BR prompt (as-built: the auditor has a single PT-BR prompt; the rule was applied to it)
- [x] 1.3 Unit test: prose with an NPC proposing a plan → auditor returns clean (`backend/tests/test_auditor.py`, check 2)
- [x] 1.4 Unit test: imperative NPC speech consistent with personality → kept (`backend/tests/test_auditor.py`, check 3)

## 2. Decision telemetry

- [x] 2.1 decision/reason fields in the return of `run_audit()` (decision ∈ clean/rewritten/rejected/parse_failed/timeout; discard now reports reason=item_tag_violation)
- [x] 2.2 Propagate the decision to the turn: `[AUDIT]` tag emitted on every turn in the SSE and decision persisted in the NARRATOR_RESPONSE event payload (`backend/app/main.py`)
- [x] 2.3 Accumulated count per decision in devtools (as-built: "auditor (devtools)" block in the Inspector of `frontend/src/App.tsx`)
- [x] 2.4 Integration test: persisted event contains the auditor decision (`backend/tests/test_auditor.py`, check 9 — [AUDIT] in the SSE + event payload)

## 3. Validation

- [ ] 3.1 Run the minimal A/B harness (12 EN+PT passages); compare the agency false positive rate before/after — **blocked**: requires a real LLM provider (`backend/.env`) and the referenced `ab_auditor.py` harness does not exist as-built
- [ ] 3.2 openspec validate --strict with no errors (25/25 ok on 2026-08-21); **missing** `openspec archive` upon completing 3.1

#### `changes/fix-auditor-agency-false-positive/specs/narrative-audit/spec.md`

## MODIFIED Requirements

### Requirement: Rewrite scoped to agency and continuity

The auditor SHALL rewrite only what the narrator invented beyond the player input plus the established scene; NPC initiative is not agency, and world contradictions have a high bar.

#### Scenario: NPC proposes a plan

- **WHEN** an NPC proposes, suggests or offers a plan/action to the player in speech initiated by the NPC itself
- **THEN** the auditor SHALL NOT treat the NPC's speech as player agency
- **AND** the auditor SHALL consider the speech authorship (who initiated it) before classifying agency

#### Scenario: Imperative NPC speech

- **WHEN** the narrator writes NPC speech in an imperative tone directed at the player
- **THEN** the auditor SHALL keep the speech when it is consistent with the NPC's goal and personality
- **AND** SHALL NOT rewrite it merely for sounding like a command

## ADDED Requirements

### Requirement: Auditor decision telemetry

The system SHALL record per turn the auditor decision (clean, rewritten, rejected, parse_failed, timeout) with its reason, and expose the accumulated count in the devtools panel.

#### Scenario: Rewrite rejected by item tag

- **WHEN** a rewrite is discarded for `[ITEM_*]` tag violation
- **THEN** the turn's trace SHALL record decision rejected with reason item_tag_violation

#### Scenario: Accumulated count visible

- **WHEN** devtools queries the campaign traces
- **THEN** the per-decision counts SHALL be derivable from the persisted traces

