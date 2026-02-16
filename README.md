# Procedural Grid Roguelike

## Overview

Tactical Roguelike Arena is a turn-based grid-based roguelike built with SwiftUI.

The game has two primary modes:

1. Single Player Roguelike Runs
2. Asynchronous Arena (build vs build PvP)

Both modes use the same deterministic combat engine.

The project prioritizes:
- Clean architecture
- Deterministic simulation
- Modular combat systems
- Extensible ability and status systems
- Testable game logic separate from UI


# Core Design Principles

1. All gameplay logic must be UI-independent.
2. Game engine must be deterministic.
3. SwiftUI is a rendering layer only.
4. Engine must be testable without UI.
5. Multiplayer uses build snapshots + local simulation.


# Architecture

## Layers

### 1. Engine Layer (Pure Swift)

No SwiftUI imports.

Responsible for:
- Grid state
- Turn resolution
- Combat resolution
- AI logic
- Pathfinding
- Status effects
- Ability system
- Procedural generation
- Simulation

### 2. Persistence Layer

Responsible for:
- Saving meta progression
- Saving arena builds
- Loading player upgrades

Use:
- SwiftData

### 3. UI Layer (SwiftUI)

Responsible for:
- Rendering grid
- Animations
- Player input
- Navigation
- HUD
- Feedback

Engine state is exposed via ObservableObject wrapper.


# Core Gameplay

## Grid

- Default size: 8x8
- Tile types:
  - empty
  - wall
  - entity reference

Represented as:
- 2D array of tiles


## Turn System

Phases:
- playerTurn
- enemyTurn
- resolving
- gameOver

Turn Order:
1. Player moves (optional)
2. Player uses 1 ability
3. End turn
4. All enemies act sequentially
5. Apply end-of-turn effects
6. Check win/loss
7. Next turn


# Entities

## Entity Model

Properties:
- id
- position
- maxHP
- currentHP
- attack
- movementRange
- abilities
- activeStatusEffects
- team (player or enemy)


# Combat System

## Damage Resolution

- Flat damage initially
- Optional crit later
- Status effects modify damage
- Death removes entity from grid

Combat must be deterministic.


# Status Effect System

Use protocol-based design.

Each status effect must define:
- onApply()
- onTurnStart()
- onTurnEnd()
- modifyIncomingDamage()
- modifyOutgoingDamage()
- duration

Initial status effects:
- Poison (damage over time)
- Shield (absorbs damage)
- Stun (skip turn)

System must support stacking.


# Ability System

Abilities must define:
- range
- cooldown
- target type
- effect logic
- area of effect (optional)

Initial abilities:
- Basic Attack
- Dash
- Poison Strike
- Fireball
- Shield Wall

Abilities must be modular and extendable.


# Enemy AI

AI Decision Logic:

If player in attack range:
    attack
Else:
    pathfind toward player
    move toward best tile

Pathfinding:
- Implement A* search
- Heuristic: Manhattan distance
- Grid-based
- Avoid walls and occupied tiles

AI must not depend on UI.


# Roguelike Mode

## Run Structure

Each run:
- 5 floors
- Each floor contains:
  - Procedurally generated map
  - 3–6 enemies
  - 1 reward choice after clearing

Floor 5:
- Boss fight


## Procedural Generation

Initial implementation:
- Random wall placement
- Random enemy placement

Later:
- Room + corridor generation


## In-Run Progression

After clearing a floor:
Present 3 random upgrades.

Upgrade types:
- +10 Max HP
- +2 Attack
- +1 Movement
- New Ability
- Status modifier enhancement

Upgrades stack.
All run-based upgrades are lost on death.


# Meta Progression

After run ends:
- Grant Arena Tokens based on performance

Permanent upgrades:
- +5 starting HP
- +1 starting Attack
- Unlock new abilities in reward pool
- Unlock new classes
- Unlock new enemy types

Saved via SwiftData.


# Arena Mode (Asynchronous Multiplayer)

## Concept

Players upload completed builds.
Other players fight those builds.
Simulation runs locally.
Results affect ranking.

No real-time multiplayer required.


## Build Snapshot

A build snapshot contains:
- Base stats
- Ability list
- Passive modifiers
- Status interactions
- Seed (if needed)

Must be serializable.


## Arena Match Flow

1. Download opponent build
2. Spawn both builds on arena map
3. AI controls both sides
4. Run deterministic simulation
5. Determine winner
6. Update ranking


# Determinism Requirements

- All randomness must use seeded RNG.
- No reliance on system random during simulation.
- Same seed + same build = same result.

This is required for fair PvP.


# MVP Scope

For first playable version:

- 8x8 grid
- 1 class
- 3 enemy types
- 1 boss
- 5 abilities
- 3 status effects
- 5 floors per run
- Meta upgrade: starting HP + attack
- Local arena simulation only (no networking)


# Stretch Goals

- Fog of war (line-of-sight)
- Multiple classes
- Seasonal ranking
- Replay system
- Leaderboards
- Widgets
- Achievements


# Development Order

1. Grid + movement
2. Turn system
3. Combat resolution
4. Enemy AI (basic)
5. A* pathfinding
6. Procedural map generation
7. Upgrade selection system
8. Status effect engine
9. Meta progression persistence
10. Arena simulation
11. Networking integration
12. Polish and animation


# Non-Goals (For Initial Versions)

- Real-time multiplayer
- 3D graphics
- Complex animation engine
- Live PvP
- Large-scale backend
