# Arcane Rift — Agent Instructions

This file provides context and conventions for AI agents (Claude Code, Cursor) working on this project.

---

## Project Overview

Top-down browser PvP mage game. Players choose an element (Fire, Water, Earth, Air) and fight in real-time multiplayer arenas. Core inspirations: For Honor (depth, skill > stats), Genshin Impact (art), Wizard of Legend (combos), Avatar (elemental identity).

**Read the specs first.** Before working on any system, read the relevant spec in `/specs/`:
- `01-game-design-document.md` — overall vision and mechanics
- `02-elements-and-characters.md` — element archetypes and abilities
- `03-technical-architecture.md` — stack, folder structure, networking model
- `04-maps-and-arenas.md` — arena designs
- `05-multiplayer-and-social.md` — party system, Colyseus rooms
- `06-combat-system-deep-dive.md` — state machine, timing, combo resolver
- `07-art-and-audio-direction.md` — visual and audio guidelines
- `08-progression-and-meta.md` — XP, leveling, unlock system

---

## Tech Stack

| Layer | Tech |
|---|---|
| Game engine | Phaser 3 |
| Multiplayer server | Colyseus (Node.js) |
| Language | TypeScript everywhere |
| Build | Vite (client), tsc (server) |
| Auth | Firebase Auth |
| Database | Firestore |
| Hosting | Firebase Hosting (client), Cloud Run (server) |
| Package manager | pnpm workspaces |

---

## Repository Structure

```
/client/src/          Phaser 3 frontend
/server/src/          Colyseus game server
/shared/              Shared TypeScript types and constants
/specs/               Game design and technical specs
```

Key shared files:
- `shared/constants/combat.ts` — ALL tunable combat values (stamina costs, timing, damage). Never hardcode these in game logic.
- `shared/types/combat.ts` — AttackDefinition, ComboSequence, Element types
- `shared/types/rooms.ts` — Colyseus message types

---

## Code Conventions

### TypeScript
- Strict mode enabled everywhere
- No `any` types — use proper interfaces
- All Colyseus message types defined in `shared/types/rooms.ts`
- Export types from barrel files per module

### Combat Logic
- **All authoritative combat resolution happens server-side** (`server/src/game/CombatEngine.ts`)
- Client only does: input capture, visual effects, client-side prediction for movement
- Never compute damage, stagger, or parry on the client and trust it
- All timing constants imported from `shared/constants/combat.ts`

### Networking
- Inputs flow: client → Colyseus room → CombatEngine → state update → broadcast
- Do not use `room.send` for game state — use Colyseus schema state sync
- Use `room.send` only for discrete events (round start, round end, emotes)

### Element System
- Each element has a server-side combat file: `server/src/elements/{Element}Combat.ts`
- Each element implements the `IElementCombat` interface
- Each element defines its attack variants (L1, L2... H1, H2...) and all combo `AttackDefinition` entries
- Client-side visual: `client/src/elements/{Element}Element.ts`
- New elements follow existing pattern exactly

### Phaser
- Each major screen is a Phaser Scene: Boot, Menu, Lobby, Game, HUD
- HUD runs as a parallel scene on top of Game scene
- Use Phaser's event emitter for intra-scene communication
- Asset keys use SCREAMING_SNAKE_CASE: `FIRE_PROJECTILE_SPRITE`

### Colyseus
- Room state defined with `@colyseus/schema` decorators
- All game logic in `GameRoom.ts` delegates to `CombatEngine.ts`
- `LobbyRoom.ts` handles pre-game only, no combat logic

---

## Core Design Rules — Do Not Violate

1. **Skill > Stats always.** Levels unlock attack variants and executions — zero stat increases, ever. HP and damage are fixed values, not level-scaled.

2. **Server is authoritative.** Never trust client-computed damage, position, or hit detection. Client sends inputs, server sends results.

3. **Timing constants are tunable.** All timing values (stamina costs, recovery windows, damage) live in `shared/constants/combat.ts`. Do not hardcode them.

4. **Elements are distinct.** Do not homogenize elements for balance. A Fire player and Water player should feel like completely different games. Balance passes come later.

5. **Readable over flashy.** Attacks must be telegraphed. A player should always be able to see what's coming and have a chance to react. Never add attacks with no windup.

6. **No Q/E abilities.** The combo system (L/H variants + dodge attacks + combos) is the expression layer. There is no separate ability slot system.

7. **Two stamina states only.** Normal and Stamina Break. No "low stamina" intermediate state.

8. **Stamina doesn't regen while attacking.** This is load-bearing for the combat depth — don't change it without discussion.

---

## Development Order (Suggested)

1. **Scaffold project** (monorepo, Vite, Colyseus, Firebase)
2. **Single-player prototype** — one character, movement, placeholder sprites, basic attacks
3. **Combat mechanics** — stamina, blocks, dodge, stagger
4. **Combo system** — resolver, 2-3 combos working for Fire
5. **Multiplayer** — Colyseus room, two players fighting
6. **Auth + persistence** — Firebase login, save character data
7. **Party/lobby system**
8. **All 4 elements**
9. **Maps** — proper tilesets
10. **Art pass** — replace placeholders
11. **Audio**
12. **Polish** — hitstop, particles, screen shake

---

## Running the Project

```bash
# Install dependencies
pnpm install

# Dev: start Colyseus server
cd server && pnpm dev

# Dev: start Phaser client
cd client && pnpm dev

# Or with Docker (preferred for server)
docker-compose up
```

---

## Environment Variables

**Client (`client/.env.local`):**
```
VITE_COLYSEUS_URL=ws://localhost:2567
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
```

**Server (`server/.env`):**
```
FIREBASE_PROJECT_ID=...
FIREBASE_CLIENT_EMAIL=...
FIREBASE_PRIVATE_KEY=...
PORT=2567
```

---

## What to Ask About

If anything in the specs is ambiguous or contradicts itself, check with the user before implementing. The user is a senior developer and prefers direct technical discussion.

Do NOT add features beyond what's in the specs without asking. Do NOT add extra error handling, abstractions, or "future-proofing" that wasn't asked for. Build what's needed, not what might be needed.
