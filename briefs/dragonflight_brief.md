# Dragonflight Project Brief

## Objective

Deliver the MVP of Dragonflight — a single-player, turn-based fantasy strategy game where the player controls a dragon raiding human settlements from a central citadel — with clean, modular Python + Pygame architecture that supports incremental feature delivery and future expansion. See `Documentation/Dragonflight Specification and MVP.md` in the Dragonflight repo for the authoritative spec; this brief summarises scope and working mode for the agent flow.

## In-Scope

- Core gameplay loop and turn cycle (player turn, citadel phase, settlement phase, army phase, next day) per Section 2 of the spec.
- World map systems:
  - Hex grid using **axial coordinates** (Section 4).
  - Terrain types and their effects (grassland, woodland, mountains, rivers, bridges, settlement tiles) — terrain affects **armies only**; the dragon's flight ignores terrain (Section 5).
  - Handcrafted MVP map authored as data, with validation; procedural generation is a later phase.
- Settlement system:
  - Types (village, city, fort), attributes, growth, destruction rules, and aggression with configurable nearby-radius (default 15% of map width) per Section 6.
- Dragon system:
  - Single dragon (Red Fire Dragon for MVP), stats (HP, ATK, DFN, flight range, speed in hexes/hour), gold-funded permanent stat upgrades per Section 7.
- Combat system:
  - Automatic stat-based resolution using **30-minute damage rounds**.
  - Damage formula `attacker ATK − defender DFN` with floors: humans/armies floor at 0, dragon-dealt damage floors at 1 (Section 8).
- Army system:
  - Aggression-driven spawning, A* pathfinding to citadel via legal terrain and bridges.
  - End-of-turn movement: closest-to-citadel moves first; same-tile armies merge with **summed** stats (Sections 2 phase 4 and 9).
- Citadel system:
  - HP, gold-funded repairs (static high cost), 50%-of-max-HP daily heal at citadel (Sections 2 and 10).
- Economy:
  - Single resource (gold), raiding rewards and upgrade costs scale with settlement/skill tier (Section 11).
- Game-over conditions:
  - Dragon dies or citadel HP reaches 0 (Section 3).
- Scoring:
  - Primary metric is **turns survived** (Section 3).
- Project setup and tooling:
  - `src/dragonflight/` package layout with `pyproject.toml`, `assets/` for art, `tests/` for incremental pytest coverage, dev helper scripts for venv setup.
  - Python 3.11-3.13 (Pygame wheel coverage); Python 3.14+ explicitly avoided until Pygame supports it.
- Documentation upkeep for spec, README, and code-level docstrings as behavior changes.

## Out-of-Scope (MVP)

- Multiplayer, networking, save/load, fog of war.
- Procedural map generation (planned post-MVP; the Game Map Designer role still **owns the constraints** for it when the time comes).
- Multiple dragon types or species, elemental systems, skill trees, dragon abilities.
- Citadel upgrades, dynamic factions, diplomacy, weather/seasons.
- Tactical combat screens, terrain destruction, minions/allies.
- Naval systems, flying enemies, hero enemies.
- Formal input-binding spec and advanced input UX (worked out alongside GUI implementation when appropriate).
- Automated test suite as a shipping gate (tests added progressively once systems exist; see Section 21 of the spec).

## Acceptance Criteria

- [ ] Requested change conforms to the relevant section(s) of the Dragonflight specification.
- [ ] Hex-coordinate math, pathfinding, and movement remain consistent with the **axial coordinate** convention.
- [ ] Combat code respects damage-round timing (30 minutes) and damage floors (humans/armies = 0, dragon-dealt = 1).
- [ ] Dragon time-budget invariant holds: any action that would prevent returning to the citadel before turn end is rejected at the simulation boundary.
- [ ] Army-phase ordering is closest-to-citadel-first, and same-tile collisions resolve to a single army with **summed** stats.
- [ ] Aggression nearby-radius defaults to **15% of map width** in hexes and is configurable at game start.
- [ ] Map data lives in plain data files validated by reachability/spawn/bridge checks before being considered done.
- [ ] Single-source-of-truth simulation state (e.g. `MapState`) is preserved; no rule logic leaks into rendering or UI code.
- [ ] Pygame loop runs on Python 3.11-3.13 in a clean venv created via the repo dev scripts.
- [ ] Documentation (spec, README, docstrings) is updated where behavior or structure changed.
- [ ] QA report records regressions checked, defects found, and go/no-go status.
- [ ] Handoff notes capture decisions, assumptions, and follow-up work.

## Constraints

- Language and framework: **Python + Pygame**, object-oriented design (Section 12).
- Use the `src/dragonflight/` package layout; do not introduce competing layouts.
- Prefer pure functions for rule helpers; isolate side effects.
- Keep gameplay numbers in named constants or data tables (no magic numbers in rule code).
- Reject invalid actions at the simulation API boundary, not in render or UI code.
- Maintain modularity, loose coupling, and clear interfaces per Section 19 of the spec.
- Pygame compatibility: Python 3.11-3.13 only until Pygame ships wheels for newer versions.

## Risks/Assumptions

- Hex math errors (axial vs offset confusion) can silently break pathfinding and aggression radius calculations; treat these as high-risk surfaces.
- Rule edge cases not yet covered by automated tests can regress as the simulation grows; QA risk increases over time until the test suite catches up (Section 21 plan).
- Balance numbers are placeholders during development and will need iteration; do not encode them as immutable invariants.
- Map-data schema decisions made early can be expensive to revise; the Game Map Designer should document assumptions and version the schema.
- Pygame wheel availability constrains the supported Python version; users on bleeding-edge Python will be blocked until a compatible runtime is installed.

## Working Mode

- The main agent coordinates workflow using `agents/main-agent.md`.
- The corresponding project brief for this game is `briefs/dragonflight_brief.md`.
- Use the **`playbooks/game-feature.md`** playbook for normal Dragonflight work; it wires in the **Coder (Game Systems)** and **Game Map Designer** roles in addition to the standard set.
- Role-specific work should be delegated to real subagents where possible.
- The active Dragonflight project slug is **`dragonflight`** (see `learning/project-registry.md`); learning files use `<agentType>_dragonflight_learning.md`.
