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

Players can create **multiple characters per account**, each independent (own progression, element, name).

At character creation, players choose one **element** (locked to that character, cannot change):
- Fire
- Water
- Earth
- Air

Each element has:
- Unique visual identity (particle effects, color palette, attack shapes)
- Unique attack variants and combo outcomes (see Element Specs)
- Unique playstyle archetype (see Element Specs)

Before each match, players choose their **attack loadout**: which variant of their light and heavy attack to equip (variants are unlocked through progression). This is how players express their playstyle within their element.

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
| Heavy Attack | RMB | Medium | Slower startup, higher damage, knockback. Press, not hold. |
| Dodge Light | LMB during dodge | Low | Unique attack from dodge state; combo starter |
| Dodge Heavy | RMB during dodge | Medium | Unique attack from dodge state; combo starter |
| Block | Hold Shift | Medium/tick | Directional — must face incoming attack. Regen slowed while blocking. |
| Parry | Shift + timing | Low | Precise timing window. Success: auto-fires counter projectile + brief instant-cast window. |
| Dodge | Space | None | Has cooldown + animation recovery. No stamina cost. |
| Execute | F (near downed enemy) | None | Begins execution animation. Long, leaves you exposed. |

> **TBD:** Exact key bindings are placeholder. Will finalize during UI spec.
> **Removed:** Q/E ability slots. The combo system is the expression system.

### 5.2b Stamina Regeneration Rules

- **While attacking:** Stamina does **not** regenerate. This is critical — an aggressive player who keeps swinging will drain their stamina with no recovery.
- **While blocking:** Stamina regenerates at a **reduced rate** (roughly half). A patient opponent can bait an aggressive player into blocking for a long time, draining their stamina slowly.
- **While idle/moving:** Full stamina regen rate.
- **Stamina states:** Only two — **normal** and **stamina break** (at 0). No intermediate "low stamina" state.
  - Stamina break: character briefly stumbles, short vulnerability window, slower regen during recovery period.

### 5.3 Aiming & Directionality

- Mouse cursor determines aim direction at all times
- Attacks fire/arc toward mouse direction, with **homing tracking** on targets
- Opponent can **dodge** to break tracking (timing-based, not guaranteed)
- **Block and parry are directional:** player must be facing the general direction of the incoming attack
- This creates mind games: fake one side, attack the other

### 5.4 Movement

Movement is **free and fluid** — 8-directional with analog feel, like Wizard of Legends. No grid snapping, no locked movement directions. Players move with WASD while aiming with the mouse independently (twin-stick style). This is essential for the evasion and positioning layer of combat.

### 5.5 Combo System

**Attack variants:** Each element has multiple variants of light and heavy attacks (L1, L2, H1, H2, etc.). Players unlock more variants through progression. Before a match, the player equips one light variant and one heavy variant as their base attacks. These are mechanically and visually distinct — not strictly better, just different.

Attack inputs (L = Light variant equipped, H = Heavy variant equipped, DL = Dodge Light, DH = Dodge Heavy) create combo strings. The sequence determines:
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
| DL | Dodge light | Engage from movement, short-range burst |
| DL, L | Dodge into light chain | Momentum combo |
| DH | Dodge heavy | High-risk engage from movement |
| DH, H | Dodge into heavy chain | Commitment combo |

Combos are element-flavored — Fire's L,L,H might be a fireball burst; Water's L,L,H might be a water lance. Each combo sequence has its **own unique animation** — the visual for L,L,H is different from L,H,H is different from H,H. Dodge attack animations are also distinct.

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
- **Success:** 
  1. The parried attack is nullified (0 damage)
  2. An **auto-counter projectile** fires instantly toward the attacker — element-flavored, no windup, moderate speed + slight homing. Can be blocked (costs attacker stamina) but cannot be parried.
  3. Parrying player also gets a brief **instant-cast window** (~500ms) where their next attack has 0 windup, for a follow-up combo opportunity
  4. Attacker is briefly stunned
- **Fail (too early):** normal block, stamina cost
- **Fail (no block):** take full damage

This resolves the parry counter at range problem: you don't need to close distance — the counter projectile crosses the gap for you.

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

## 5b. Execution System

Inspired by For Honor. When a player reaches 0 HP, they enter a **downed state** instead of immediately dying.

### Downed State
- Player collapses into a downed animation (lying on ground, weakly trying to get up)
- Downed state lasts **N seconds** (tunable, starting value: 6 seconds)
- Downed player **cannot act** — no attacks, no block, no dodge
- At the end of the timer: player dies normally (round loss in 1v1, permanently out in team modes)

### Execution
- Any opponent can approach a downed player and press **[Execute]** to begin an execution
- Execution is a **long cinematic animation** (3–8 seconds depending on the execution type)
- During execution, the executing player is **fully exposed**: they cannot move, block, or dodge
- **Other players can interrupt the execution** by hitting the executor — this cancels the execution and the downed player's timer resumes
- If the execution **completes successfully**:
  - The downed player is **permanently dead** (cannot be revived in team modes)
  - The executor **regains health** proportional to the execution duration (longer, riskier executions = more health)
- If the execution is **interrupted**: executor is briefly staggered, no health gained

### Multiple Executions
- Each element has multiple unlockable executions (unlocked via progression)
- Executions are purely cosmetic variants — same function (permanent death + health regen), different animation
- Executions have distinct durations and thus distinct risk/reward profiles
- Short execution: fast, less health gained, safer
- Long execution: slow, more health gained, very risky in team modes

### Execution in Different Modes
- **1v1:** Execution is a stylistic choice. Win is secured at downed. Execute for health bonus going into next round.
- **1v1v1 / team modes:** Much higher stakes. Downed player's teammate can potentially act to interrupt the execution. Enemy executing a teammate is a priority threat to respond to.

---

## 5c. Environment & Ambiance

The arenas are alive. Each arena has dynamic environment effects that create a cozy, atmospheric backdrop — the combat happens *within* a world that feels real.

### Weather Effects
- **Rain:** Particles falling, puddle ripples on ground, ambient rain sound. Subtle on character sprites (wet look).
- **Wind:** Ambient particles (leaves, petals, embers) drift across the screen. Affects some lighter projectiles slightly.
- **Thunder:** Distant rumbles (audio). Occasional lightning flash on the skybox (not a gameplay hazard — pure ambiance).
- **Lightning strike:** Dramatic visual event — a bolt hits somewhere in/near the arena with a crack + flash. Rare, random, pure atmosphere. Does not affect gameplay in v1.

Each arena has a fixed **weather theme** (e.g. Coliseum = occasional wind and petals, Ember Peaks = ash drift, Tidecaller's Basin = rain, Sky Sanctuary = dramatic clouds and lightning).

### Elemental Environment Reactions (Future, Not v1)
- Fire attacks on grass tiles can ignite them (burning ground deals contact damage)
- Water attacks can create slippery zones
- Earth attacks can create terrain pieces (rocks that block movement)
- Air attacks can clear environmental hazards

This is the "elemental reactions" system. Designed to add in a future version without breaking v1 maps (maps are built to support it, but it's not implemented yet).

---

## 6. Win Conditions

- **1v1:** First to 0 HP loses the round. Best of N rounds.
- **1v1v1:** Last player standing wins.
- **2v2:** Eliminate both opponents. Teams respawn? (TBD)
- **4v4:** Team elimination or score-based (TBD)

---

## 7. Progression System

- Characters gain XP from **winning and losing matches** (wins give significantly more)
- Levels unlock: new **attack variants** (L2, H2, L3...), new **executions**, cosmetic effects
- **Health and damage are completely fixed** — not affected by level at all. A level 1 and level 100 character have identical base stats.
- Stat values may vary slightly by **element** (e.g. Earth has more HP, Air has less) — not by level
- Progression reward: more expressive toolkit (more loadout options), not a stat advantage
- A level 1 player with skill beats a level 100 player without skill — always

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
