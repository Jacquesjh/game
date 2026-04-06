# Progression & Meta Systems Spec

---

## Core Principle

> **Skill is the great equalizer. Levels are rewards, not power.**

A level 1 player who understands the combat system — combo timing, stamina management, directional blocking, parry windows — will beat a level 100 player who button mashes. This is non-negotiable. Every progression decision is made through this lens.

Reference: For Honor. The gap between a beginner and an intermediate player is not "I have better gear." It's "I know when to dodge, how to read feints, when to parry."

---

## XP & Leveling

### XP Sources (Simplified)
| Action | XP |
|---|---|
| Win a match | 100 |
| Lose a match | 25 |

Simple. No granular per-action tracking. Win a lot = level faster. The game itself is the engagement loop.

### Level Curve
- Levels 1–10: Fast (feel progression quickly, unlock first variants)
- Levels 10–50: Moderate
- Levels 50+: Slow (prestige, cosmetics)

### What Levels Unlock (No Stats — Ever)
| Level | Unlock |
|---|---|
| 1 | Base character: L1, H1, Execution 1 |
| 5 | L2 |
| 10 | H2 |
| 15 | Execution 2 |
| 20 | L3 |
| 25 | Cosmetic: elemental aura effect |
| 30 | H3 |
| 35 | Execution 3 |
| 50 | Cosmetic: color palette alt |
| 75 | Execution 4 |
| 100 | Prestige title |

Each element launches with **3 light variants (L1–L3)** and **3 heavy variants (H1–H3)**. More can be added later. Depth comes from how players mix their two equipped chains, not just from the number of variants.

**Zero stat increases at any level.** A level 100 vs level 1 fight has identical HP, damage, and stamina values. The level 100 player has more loadout options. That's it.

---

## Attack Variant System

This is the core progression mechanic. Players unlock **attack variants** — new versions of their light and heavy attack chains, each with distinct hit animations, projectile trajectories, and chain length.

### How Variants Work
- At level 1, player has **L1** and **H1** (base variants for their element)
- Before each match, player equips one L variant and one H variant
- Each variant is a **hit chain**: an ordered sequence of individual hits, each with its own attack properties
- Chain lengths differ per variant — L1 might be 4 hits, L3 might be 2
- Pressing L advances the light chain; pressing H advances the heavy chain. They interleave freely.
- The last hit of the heavy chain is the **finisher** — it ends the combo and triggers recovery
- After a heavy finisher, a brief grace window allows one more L input before full recovery

### Why This Works
- Variant choice changes your rhythm, not just your damage numbers
- A 4-hit L1 chain rewards sustained aggression; a 2-hit L3 chain rewards players who like to reset quickly
- Players mix their L and H chains in endlessly varied patterns — the same loadout plays differently depending on spacing, reads, and opponent reaction
- Players develop a "signature" — your L1+H2 playstyle becomes recognizable

### Variant Structure (per element)
```
Light variants:
  L1  (base, level 1)   — to be designed per element
  L2  (level 5)         — different chain length + hit feel
  L3  (level 20)        — different chain length + hit feel
  L4+ (future)          — can be added as the game expands

Heavy variants:
  H1  (base, level 1)   — to be designed per element
  H2  (level 10)        — different chain length + finisher type
  H3  (level 30)        — different chain length + finisher type
  H4+ (future)          — can be added as the game expands

Dodge variants (independent of L/H loadout):
  DL_forward, DL_side, DH_forward, DH_side, DH_back, etc.
  — each direction has its own attack; always available, no unlock needed
  — dodge attacks act as combo starters; L/H chains continue after dodge recovery
  — more dodge variants can be added per direction over time
```

Specific chain designs (hit trajectories, speeds, damage) are defined per element in `02-elements-and-characters.md`.

Players who understand their element deeply can build devastating combo patterns. A new player just has fewer building blocks to work with.

---

---

## Multiple Characters Per Account

- Players can create **multiple characters**, one per account slot (no hard limit in v1)
- Each character has its own element (locked at creation), name, level, and progression
- Players select which character to use when entering a party/lobby
- Characters share the account (Firebase Auth UID) but are independent records in Firestore
- This is a v1 feature, not future — it's part of the core UX from day one

## Character Permanence

- Element is **locked at creation** — cannot respec
- This is intentional: part of the game's identity is that your element is your identity
- Want to try a different element? Create a new character on the same account

---

## Match History & Stats

Tracked per character in Firestore:
- Win/loss record overall and per mode
- Win/loss per element matchup
- Favorite combo (most used)
- Parry success rate
- Average match duration
- Stamina break count

These are for player self-reflection and bragging with friends, not for matchmaking or ranking (v1).

---

## Future Meta (Not v1)

- **Ranked mode:** ELO/MMR per element or globally
- **Character customization:** cosmetic gear, color palettes, effect skins
- **Seasonal events:** limited cosmetics, special arenas
- **Social features:** friend list, match history vs specific opponents, replays
