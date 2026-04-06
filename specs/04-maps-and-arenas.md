# Maps & Arenas Spec

---

## Design Philosophy

Maps are designed around the combat system, not the other way around.

- **1v1 arenas:** Small, tight, minimal obstacles. The fight is about the players, not the environment.
- **Team arenas:** More space for flanking, some terrain features, but still readable.
- **Spectator support:** Every arena has a designated spectator zone where non-fighting party members can walk and watch.
- **Visual identity:** Each arena has an elemental theme (but any element can fight in any arena).

---

## Arena 1: The Coliseum (1v1 / Flagship Map)

**Theme:** Ancient stone coliseum, Genshin-esque architecture — warm stone, flowering vines, glowing elemental runes on the floor.

**Layout:**
```
+------------------------------------------+
|  [SPECTATOR WALKWAY - raised stone path] |
|  +------------------------------------+  |
|  |                                    |  |
|  |         FIGHT ZONE                 |  |
|  |   (circular, ~20 tile radius)      |  |
|  |                                    |  |
|  |         [center marker]            |  |
|  |                                    |  |
|  +------------------------------------+  |
|  [SPECTATOR WALKWAY - raised stone path] |
+------------------------------------------+
```

**Details:**
- Fight zone is circular, open, flat
- Spectator walkway is raised ~1 tile higher, runs around the perimeter
- Spectators cannot enter the fight zone (invisible wall or hard boundary)
- Floor: decorative elemental rune that lights up when players are fighting
- Light sources: lanterns, glowing crystals, sky lighting
- Ambiance: crowd noise (subtle), wind, elemental ambient sounds based on active elements

**Size:** Fight zone ~20-unit radius (adjust during playtesting). Enough space to maneuver without feeling cramped or endless.

**Hazards:** None. Pure combat arena.

---

## Arena 2: Ember Peaks (1v1 / Elemental Theme: Fire)

**Theme:** Volcanic ridge. Molten rock channels on the sides, stone platforms, fiery sky.

**Layout:** Rectangular with two lava channels on the sides that serve as out-of-bounds zones (fall in = instant death or heavy damage).

**Hazard:** The lava edges are out-of-bounds. Knockback-heavy attacks (Earth boulder, Water undertow) become more dangerous here.

**Spectator:** Raised stone platforms on the short ends.

---

## Arena 3: Tidecaller's Basin (2v2)

**Theme:** Flooded ruins. Columns, partial walls, shallow water ground.

**Features:**
- Some waist-high stone columns that block movement but NOT projectiles (forces positioning)
- Water on the ground (cosmetic, slight audio effect)
- More space for team coordination

**Spectator:** Ruined archways at the ends, elevated.

---

## Arena 4: Sky Sanctuary (1v1 / 1v1v1)

**Theme:** Floating islands in the clouds. Genshin-aesthetic sky environment.

**Layout:** Main island for fighting, small adjacent islands for spectators reachable by short bridges.

**Features:** No hazards. Clean and airy. Good for showcasing Air mages.

---

## Arena 5: Stonehold Citadel (4v4)

**Theme:** Castle courtyard. Earth-flavored. Stone walls, battlements.

**Layout:** Larger rectangular space with some waist-high barriers for cover. Not a maze — still readable.

**Spectator:** Battlements around the perimeter.

---

## Technical Spec for Maps

### Tilemap
- Tiled (.tmx) format, loaded by Phaser 3
- Tile size: 32x32px
- Layers:
  - `ground` — walkable floor
  - `objects` — decorative foreground
  - `walls` — collision layer (impassable)
  - `spectator` — spectator zone tiles (different collision group)
  - `hazards` — damage/death zones (optional)
  - `spawn` — spawn point markers (as Tiled objects)

### Spawn Points
- Each arena defines spawn points per player count
- 1v1: 2 spawns, opposite ends
- 1v1v1: 3 spawns, equidistant
- 2v2: 4 spawns, teams on same half
- 4v4: 8 spawns, teams on same half

### Camera
- Camera follows local player
- Bounded to arena edges
- Slight zoom out when players are far apart (optional, nice to have)
- Spectator camera: free-roam within spectator zone, or could follow the action with a dedicated spectator camera mode (nice to have later)

### Boundaries
- Out-of-bounds: instant teleport back to nearest spawn with brief invincibility (keeps game moving)
- Or: hard walls everywhere (simpler, preferred for v1)

---

## Map Selection

- Auto-assigned per game mode (each mode has a pool of valid maps)
- Or: host selects map when creating party (preferred)
- Spectator map is always the same as the fight map
