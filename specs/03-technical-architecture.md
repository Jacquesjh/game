# Technical Architecture Spec

---

## Stack Overview

| Layer | Technology | Rationale |
|---|---|---|
| Rendering / Game Engine | **Phaser 3** | Most mature 2D browser game framework. Great docs, plugin ecosystem, top-down game support, scene management, input handling. |
| Multiplayer Server | **Colyseus** (Node.js) | Purpose-built multiplayer game server. Handles rooms, state sync, matchmaking. Scales well for small-medium player counts. |
| Auth | **Firebase Auth** | Already familiar, handles Google/email login cleanly. |
| Persistent Data | **Firestore** | Character data, match history, player profiles. |
| Real-time State | **Colyseus Room State** | Game state during matches lives in Colyseus, not Firestore. Firestore only for persistent/post-match data. |
| Frontend Hosting | **Firebase Hosting** | Fast CDN, easy CI deploy, already in ecosystem. |
| Game Server Hosting | **Google Cloud Run** | Colyseus server as a container. Scales to zero when not in use. Good for small player base. |
| Asset Pipeline | **Custom** | Sprites, animations created manually. Sound effects generated (e.g., sfxr, Audacity). Music generated (Suno, etc.) |
| Build Tool | **Vite** | Fast dev server + builds. Works great with Phaser 3. |
| Language | **TypeScript** | Shared types between client and server. |

---

## Architecture Diagram (Text)

```
[Browser Client - Phaser 3]
       |
       | WebSocket (Colyseus Client SDK)
       |
[Colyseus Server - Cloud Run]
       |
       |--- Room State (authoritative, in-memory during match)
       |--- Firebase Admin SDK
              |
              |--- Firestore (character data, match results)
              |--- Firebase Auth (token verification)

[Firebase Hosting] --> serves static Phaser bundle
[Firebase Auth] --> handles login, issues JWT
```

---

## Project Structure

```
arcane-rift/
├── client/                    # Phaser 3 frontend
│   ├── src/
│   │   ├── main.ts            # Phaser game init
│   │   ├── scenes/
│   │   │   ├── BootScene.ts   # Asset loading
│   │   │   ├── MenuScene.ts   # Main menu
│   │   │   ├── LobbyScene.ts  # Party / matchmaking
│   │   │   ├── GameScene.ts   # Main gameplay
│   │   │   └── HUDScene.ts    # Overlay HUD (runs parallel to GameScene)
│   │   ├── entities/
│   │   │   ├── Player.ts      # Local player entity
│   │   │   ├── RemotePlayer.ts # Interpolated remote player
│   │   │   └── Projectile.ts  # Projectile entity
│   │   ├── combat/
│   │   │   ├── ComboSystem.ts # Tracks input sequences, resolves combo
│   │   │   ├── StaminaSystem.ts
│   │   │   └── AttackFactory.ts # Creates attack instances by element+combo
│   │   ├── elements/
│   │   │   ├── FireElement.ts
│   │   │   ├── WaterElement.ts
│   │   │   ├── EarthElement.ts
│   │   │   └── AirElement.ts
│   │   ├── input/
│   │   │   └── InputHandler.ts
│   │   ├── network/
│   │   │   └── ColyseusClient.ts # Colyseus connection + room management
│   │   ├── ui/
│   │   │   ├── HUD.ts
│   │   │   └── Menus.ts
│   │   └── assets/
│   │       ├── sprites/
│   │       ├── tilesets/
│   │       └── audio/
│   ├── index.html
│   └── vite.config.ts
│
├── server/                    # Colyseus game server
│   ├── src/
│   │   ├── index.ts           # Colyseus + Express app init
│   │   ├── rooms/
│   │   │   ├── GameRoom.ts    # Main game room (authoritative sim)
│   │   │   └── LobbyRoom.ts   # Party / matchmaking room
│   │   ├── game/
│   │   │   ├── GameState.ts   # Colyseus schema state
│   │   │   ├── PlayerState.ts
│   │   │   ├── CombatEngine.ts # Server-side combat resolution
│   │   │   ├── ComboResolver.ts
│   │   │   └── PhysicsSimple.ts # Lightweight server-side position/collision
│   │   ├── elements/
│   │   │   ├── FireCombat.ts
│   │   │   ├── WaterCombat.ts
│   │   │   ├── EarthCombat.ts
│   │   │   └── AirCombat.ts
│   │   └── firebase/
│   │       └── admin.ts       # Firebase Admin SDK init + helpers
│   └── tsconfig.json
│
├── shared/                    # Shared types between client + server
│   ├── types/
│   │   ├── combat.ts          # Attack, combo, element types
│   │   ├── player.ts          # Player state interfaces
│   │   └── rooms.ts           # Room message types
│   └── constants/
│       ├── combat.ts          # Stamina costs, damage values, timing
│       └── elements.ts        # Element identifiers
│
├── specs/                     # This directory
├── CLAUDE.md                  # Agent instructions
├── package.json               # Root workspace config (pnpm workspaces)
└── docker-compose.yml         # Local dev (Colyseus server)
```

---

## Networking Model

### Authority Model: Server-Authoritative

- Client sends **inputs** (not positions). Server simulates, sends back **state**.
- Prevents cheating. Essential for PvP.
- Client does **client-side prediction** for local player movement (smooth feel).
- **Lag compensation:** Colyseus built-in interpolation for remote players.

### Message Types (Client → Server)

```typescript
// Player input message
interface InputMessage {
  type: 'input'
  tick: number
  movement: { x: number; y: number }  // -1 to 1 normalized
  aimAngle: number                     // radians, mouse direction
  actions: {
    lightAttack?: boolean
    heavyAttack?: boolean
    ability1?: boolean  // Q
    ability2?: boolean  // E
    block?: boolean
    dodge?: boolean
  }
}
```

### Message Types (Server → Client)

```typescript
// Colyseus state sync is automatic via @colyseus/schema
// Server sends delta-compressed state every tick
```

### Tick Rate

- Server: **20 ticks/sec** (50ms). Suitable for this game's pace.
- Client: Renders at 60fps, interpolates between server ticks.

---

## Combat Resolution (Server)

All damage, stagger, parry, and stamina calculations happen on the server.

Flow:
1. Client sends input (attack intent + aim angle)
2. Server validates (stamina available? animation lock? etc.)
3. Server spawns projectile/attack entity with server timestamp + trajectory
4. Server checks collision against all players each tick
5. Server resolves: hit → damage, stagger state applied
6. State broadcast to all clients

Client plays visual effects optimistically (fires animation immediately on input) but accepts server correction if desync occurs.

---

## Firebase Integration

### Auth Flow
1. Client logs in via Firebase Auth (Google or email)
2. Firebase issues JWT
3. Client sends JWT to Colyseus on room join (`auth` metadata)
4. Colyseus server verifies JWT via Firebase Admin SDK
5. Player identity confirmed, character data loaded from Firestore

### Firestore Collections

```
users/{uid}
  - displayName: string
  - createdAt: timestamp

characters/{uid}
  - element: 'fire' | 'water' | 'earth' | 'air'
  - name: string
  - level: number
  - xp: number
  - wins: number
  - losses: number
  - createdAt: timestamp

matchHistory/{matchId}
  - mode: '1v1' | '2v2' | '4v4' | '1v1v1'
  - players: uid[]
  - winner: uid | uid[]  (team)
  - duration: number     (seconds)
  - playedAt: timestamp
```

---

## Hosting & Deployment

### Local Dev
- `docker-compose up` → starts Colyseus server
- `vite dev` → starts client dev server with HMR
- Firebase emulator for Auth + Firestore

### Production
- **Client:** `firebase deploy --only hosting`
- **Server:** Docker image → Google Artifact Registry → Cloud Run service
- Environment variables via Cloud Run env config (Firebase service account, etc.)

---

## Performance Targets

- Client: 60fps on a mid-range laptop (2020+)
- Server: <50ms round-trip on same continent
- Room capacity: up to 8 players per match room (4v4 max)
- Concurrent rooms: Cloud Run auto-scales. Fine for small friend groups.

---

## Tech Decisions Log

| Decision | Chosen | Rejected | Why |
|---|---|---|---|
| Rendering | Phaser 3 | PixiJS, Three.js | Phaser has built-in game loop, input, physics, scene system. PixiJS is just a renderer — more work. Three.js overkill for 2D. |
| Multiplayer | Colyseus | Socket.io raw, Photon | Colyseus has schema-based state sync, room lifecycle, matchmaking built in. Socket.io requires building all that yourself. |
| Language | TypeScript | JavaScript | Shared types are critical for client/server consistency in combat logic. |
| Monorepo | pnpm workspaces | Separate repos | Shared types package between client/server. Easier to iterate. |
| Server hosting | Cloud Run | GKE, Cloud Functions | Cloud Run is simple Docker hosting, scales to zero, no k8s complexity needed for this scale. |
