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
| 5 | L2 (alternate light attack variant) |
| 10 | H2 (alternate heavy attack variant) |
| 15 | Execution 2 |
| 20 | L3 |
| 25 | Cosmetic: elemental aura effect |
| 30 | H3 |
| 35 | Execution 3 |
| 40 | L4 |
| 50 | H4 + Cosmetic: color palette alt |
| 75 | Execution 4 |
| 100 | Prestige title |

**Zero stat increases at any level.** A level 100 vs level 1 fight has identical HP, damage, and stamina values. The level 100 player has more loadout options — more L/H variants to choose from, more executions. That's it.

---

## Attack Variant System

This is the core progression mechanic. Players unlock **attack variants** — new versions of their light and heavy attack, each with a distinct animation, projectile shape, and combo outcome.

### How Variants Work
- At level 1, player has **L1** and **H1** (base variants for their element)
- Before each match, player equips one L variant and one H variant as their loadout
- The equipped variants define what combos are available and what they look like
- Combo sequences (L, L, H) resolve differently depending on which L and H are equipped

### Why This Works
- New variants expand creative options without being power upgrades
- The same combo sequence (e.g. L, L, H) can produce completely different attacks depending on variant loadout
- Players develop preferences and "signatures" — your H2 + L1 combo setup becomes your style
- Watching a high-level player's animations is visually distinct from a new player's

### Example: Fire Variant Tree
```
L1: Standard fireball       (base)
L2: Crescent fire arc       (level 5)  — curves around obstacles
L3: Short-range burst       (level 20) — wider but less range
L4: Piercing bolt           (level 40) — thin, high velocity

H1: Slow heavy fireball     (base)
H2: Ground eruption         (level 10) — hits beneath instead of forward
H3: Spiraling fire ring     (level 30) — wide, slow, high damage
H4: Twin fire bolts         (level 50) — two simultaneous fast shots

// Dodge variants use the equipped L/H for their flavor
// DL = dodge + equipped light, DH = dodge + equipped heavy
```

### Combo Outcomes by Loadout (Fire examples)
| Loadout | Sequence | Result |
|---|---|---|
| L1 + H1 | L, L, H | Burst lance (classic) |
| L2 + H1 | L, L, H | Curving double arc into heavy burst |
| L1 + H2 | L, H | Light shot → ground eruption |
| L2 + H2 | DL, H | Dodge arc → ground eruption (aggressive opener) |

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
