# Game Design Document — Arcane Rift (Working Title)

> Top-down browser PvP mage game. Cozy aesthetics, deep strategic combat.

---

## 1. Vision Statement

A browser-based PvP arena game where players fight as elemental mages. The experience is:
- **Visually:** Warm, lush, Genshin Impact-inspired watercolor aesthetic
- **Mechanically:** Deep, skill-expressive combat inspired by For Honor — a good player with a weak character beats a bad player with a strong character, always
- **Socially:** Fun to play with friends, easy to set up matches, satisfying to spectate

Inspirations:
| Aspect | Inspiration |
|---|---|
| Art direction | Genshin Impact |
| Combat depth & skill focus | For Honor |
| Spell variety & combos | Wizard of Legend |
| Elemental identity | Avatar: The Last Airbender |

---

## 2. Core Pillars

1. **Skill over stats** — Progression exists but never trivializes skill gaps
2. **Readable combat** — Every attack has clear telegraph; timing and positioning matter
3. **Elemental identity** — Each element plays and feels radically different
4. **Strategic stamina** — Resource management creates tension without being tedious
5. **Expressive combos** — Input sequences unlock different attack forms, not just damage multipliers

---

## 3. Game Modes

| Mode | Players | Notes |
|---|---|---|
| 1v1 | 2 | Core competitive mode |
| 1v1v1 | 3 | Free-for-all, chaotic |
| 2v2 | 4 | Team coordination required |
| 4v4 | 8 | Larger team brawl |

All modes support:
- **Party system** — invite friends by party code or link
- **Coliseum spectator** — players not fighting can walk around the arena perimeter and watch in real time
- **Quick match** — skip party, just play

---

## 4. Character Creation

At character creation, players choose one **element** (locked, cannot change):
- Fire
- Water
- Earth
- Air

Each element has:
- Unique visual identity (particle effects, color palette, attack shapes)
- Unique ability kit (Q and E abilities, with distinct feel for L/H attacks)
- Unique playstyle archetype (see Element Specs)

Players name their character. Future: cosmetic customization (skins, effects).

---

## 5. Combat System Overview

### 5.1 Health & Stamina

- **Health (HP):** Takes damage from attacks. Death = round loss.
- **Stamina:** Consumed by attacks and blocking. Regenerates over time.
  - **Normal stamina:** All actions available.
  - **Low stamina (< 20%):** Block is weakened (can be broken), attacks are slower, recovery windows are longer.
  - **Stamina break (0):** Briefly staggered. Slow regen. Very vulnerable.
- Stamina regen is steady and automatic. No mechanic to speed it up (keep it simple).

### 5.2 Actions

| Action | Input | Stamina Cost | Notes |
|---|---|---|---|
| Light Attack | LMB | Low | Fast, low damage, low recovery |
| Heavy Attack | RMB (hold?) | Medium | Slower startup, higher damage, knockback |
| Ability 1 | Q | Variable | Element-specific |
| Ability 2 | E | Variable | Element-specific |
| Block | Hold Shift (or RMB?) | Medium/tick | Directional — must face incoming attack |
| Parry | Shift + timing | Low | Precise timing, triggers counter |
| Dodge | Space | None | Has cooldown or animation lock, not stamina |

> **TBD:** Exact key bindings are placeholder. Will finalize during UI spec.

### 5.3 Aiming & Directionality

- Mouse cursor determines aim direction at all times
- Attacks fire/arc toward mouse direction, with **homing tracking** on targets
- Opponent can **dodge** to break tracking (timing-based, not guaranteed)
- **Block and parry are directional:** player must be facing the general direction of the incoming attack
- This creates mind games: fake one side, attack the other

### 5.4 Combo System

Attack inputs (L = Light, H = Heavy) create combo strings. The sequence determines:
- Attack shape/trajectory
- Element-specific visual effect
- Functional behavior (projectile, AoE, trap, etc.)

Example combos (element-agnostic framework):
| Sequence | Attack Type | Notes |
|---|---|---|
| L | Standard light | Fast, small |
| L, L | Followup light | Slightly faster |
| L, L, H | Fast projectile | Ranged burst |
| L, H | Power shot | Medium range, knockback |
| L, H, H | Ground slam / attack beneath | AoE or trap |
| H | Slow heavy | High damage, telegraphed |
| H, H | Charged heavy | Maximum damage, long cast |
| H, L | Quick reset | Cancel heavy into a fast poke |

Combos are element-flavored — Fire's L,L,H might be a fireball burst; Water's L,L,H might be a water lance.

### 5.5 Stagger & Hitstop

- Landing an attack briefly staggers the opponent (short freeze/animation lock)
- Stagger window allows follow-up attacks
- During stagger, opponent can still block/parry/dodge **if they input it at the right time** (precise, not automatic)
- Creates the "pressure" dynamic: attacker tries to keep combo going, defender tries to find the escape window

### 5.6 Attack Timing & Mind Games

- Attacks can have **variable startup delays** (element-specific or combo-specific)
- Opponent who dodges/blocks too early gets caught
- Example: Fire heavy has a 0.8s delay before the projectile fires — bait the dodge, then punish
- Parry timing window is tight. Successful parry = brief counter opportunity

### 5.7 Parry

- Timed block input at the moment of impact
- Success: opponent is briefly stunned, parrying player gets a guaranteed counter window
- Fail (too early): normal block, stamina cost
- Fail (no block): take full damage

### 5.8 Dodge

- Instant dash in any direction (8-directional)
- No stamina cost
- Has an **animation recovery** time after dodge — cannot chain dodges spam
- Can be used to break homing on projectiles (requires timing)
- Dodge has an **invincibility frame** window at the start

### 5.9 Out-of-Stamina State

- Block becomes **penetrable** — heavy attacks break through
- Attack animations are visibly slower (telegraphed)
- After attacks, recovery windows are longer
- Creates a natural "comeback mechanic" — skilled opponent can bait stamina drain, then punish

---

## 6. Win Conditions

- **1v1:** First to 0 HP loses the round. Best of N rounds.
- **1v1v1:** Last player standing wins.
- **2v2:** Eliminate both opponents. Teams respawn? (TBD)
- **4v4:** Team elimination or score-based (TBD)

---

## 7. Progression System

- Characters gain XP and levels from matches
- Levels unlock: cosmetic variants, new combo possibilities, expanded ability choices
- **Levels do NOT significantly inflate base stats** — damage, HP, stamina values stay in a tight range
- Progression reward: more expressive toolkit, not a stat advantage
- A level 1 player with skill beats a level 100 player without skill

---

## 8. Map / Arena Design

See `specs/04-maps-and-arenas.md` for detailed breakdown.

Key principles:
- **1v1 arenas:** Tight, focused, minimal clutter. Coliseum-style ring with spectator walkway around perimeter.
- **2v2/4v4 arenas:** More space, some cover/terrain, potential for flanking
- Environmental hazards: possible but keep them minimal to preserve focus on player skill
- **Spectator zone:** Raised or perimeter area where non-fighting party members walk freely and watch

---

## 9. Audio Direction

- Ambient music: calm, atmospheric, matches elemental theme of the map
- Combat SFX: satisfying hit sounds, elemental flavor (fire crackle, water splash, stone thud, wind rush)
- UI SFX: minimal, clean
- Everything generated (no licensed music)

---

## 10. Out of Scope (v1)

- Melee attacks (future)
- Social hub / chat
- Character cosmetic gear system
- Large maps / 16v16 modes
- MMO elements / persistent world
- Ranked system / ELO (may add later)
