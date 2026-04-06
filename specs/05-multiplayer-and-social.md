# Multiplayer & Social Systems Spec

---

## Party System

The party system is the core social loop for playing with friends.

### Creating a Party
1. Player logs in → lands on main menu
2. "Create Party" → generates a short party code (e.g., `FIRE-4829`) and a shareable link
3. Party leader picks game mode and map
4. Friends join by entering code or clicking link

### Joining a Party
1. Enter party code or click invite link → routed to lobby
2. Lobby shows all party members, their element icons, ready status
3. Party leader can kick, transfer leadership, start game

### Party Flow
```
Main Menu
  → Create/Join Party
  → Party Lobby
       ├── Select mode (1v1, 2v2, 4v4, 1v1v1)
       ├── Select map
       ├── Assign teams (for team modes)
       ├── All players ready
       └── Start → Game Room
                    ├── Match plays out
                    └── Post-match screen → Return to Party Lobby
```

### Party Capacity
- Max 8 players in a party (for 4v4)
- Extra players beyond the match size go to spectator mode automatically

---

## Spectator Mode

When a party has more players than a mode requires (e.g., 6 people in a 1v1 lobby), non-fighting players are spectators.

### Spectator Experience
- Spectators load into the arena map as **separate entities** (distinct character appearance, no collision with fighters)
- They walk around the **spectator zone** (perimeter walkway, stands, etc.) freely
- They can see the entire fight playing out in real time
- Spectators can use a simple **emote system** (cheer, boo, wow) that shows above their head
- Optional: spectator camera mode (lock to a fighter, zoom out overview — nice to have, not v1)

### Spectator Constraints
- Cannot enter the fight zone (hard boundary)
- Cannot interact with the match in any way (no casting spells, no interfering)
- Spectators see a slightly transparent HUD version (no stamina bars for fighters unless we add a broadcast view)

---

## Matchmaking (Future / Optional)

v1 is friends-only via party system. No matchmaking needed.

Future: if we want to open it up, we'd add:
- Quick match pool per game mode
- Colyseus matchmaking (`matchMaker.joinOrCreate`)
- ELO or MMR system

Not in scope for initial build.

---

## Rooms Architecture (Colyseus)

### LobbyRoom
- Type: `lobby`
- Handles: party creation, player joining, ready checks, settings selection
- State: party members, ready states, selected mode/map, teams
- Capacity: 8 players max
- Lifecycle: exists until match starts, then transitions players to GameRoom

### GameRoom
- Type: `game`
- Handles: authoritative game simulation, combat, win condition
- State: all player positions, health, stamina, active projectiles, game phase
- Capacity: 8 players (fighters) + N spectators (separate spectator slot type)
- Lifecycle: match duration, then post-match → return clients to lobby

### SpectatorRoom (merged into GameRoom)
- Spectators join the same GameRoom but with a `spectator` role
- They receive full game state but their inputs only control their own position in the spectator zone
- Server ignores their combat inputs entirely

---

## Server Messages Reference

### LobbyRoom Messages

**Client → Server:**
```typescript
'ready'         // toggle ready state
'set_mode'      // { mode: '1v1' | '2v2' | '4v4' | '1v1v1' } (leader only)
'set_map'       // { mapId: string } (leader only)
'set_team'      // { playerId: string, team: 0 | 1 } (leader only)
'start'         // begin match (leader only, all must be ready)
```

**Server → Client (via state sync):**
```typescript
// LobbyState schema
players: MapSchema<{
  uid: string
  displayName: string
  element: string
  ready: boolean
  team: number
  isLeader: boolean
  isSpectator: boolean
}>
mode: string
mapId: string
phase: 'waiting' | 'starting' | 'ingame'
```

### GameRoom Messages

**Client → Server:**
```typescript
'input'  // InputMessage (see technical spec)
'emote'  // { emoteId: string } (spectators only)
```

**Server → Client:**
```typescript
// State sync via Colyseus schema (automatic)
// Plus:
'round_start'   // { countdown: number }
'round_end'     // { winner: uid | uid[], reason: string }
'match_end'     // { winner: uid | uid[], stats: MatchStats }
'player_stagger' // { uid: string, duration: number }
'player_death'   // { uid: string }
```

---

## Authentication Flow

1. Player opens game in browser
2. If no Firebase session: redirected to login (Google OAuth or email)
3. On login: Firebase JWT retrieved
4. JWT passed to Colyseus server on room join as `auth` metadata
5. Colyseus `onAuth` hook verifies JWT, returns user data
6. Character data loaded from Firestore for that user
7. If no character exists: character creation flow before lobby

---

## Post-Match

After match ends:
- **Post-match screen:** shows winner, round breakdown, basic stats (damage dealt, stamina breaks, parries, etc.)
- Server writes match record to Firestore
- Server updates character XP/wins/losses in Firestore
- "Back to Lobby" button returns all players to the party lobby (party is preserved)

---

## Error Handling

- Player disconnects mid-match: they are removed, if 1v1 opponent wins by forfeit
- Connection issues: client attempts reconnect for 10 seconds before forfeit
- Room crashes: graceful error, players returned to main menu with error message
