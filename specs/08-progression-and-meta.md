# Progression & Meta Systems Spec

---

## Core Principle

> **Skill is the great equalizer. Levels are rewards, not power.**

A level 1 player who understands the combat system — combo timing, stamina management, directional blocking, parry windows — will beat a level 100 player who button mashes. This is non-negotiable. Every progression decision is made through this lens.

Reference: For Honor. The gap between a beginner and an intermediate player is not "I have better gear." It's "I know when to dodge, how to read feints, when to parry."

---

## XP & Leveling

### XP Sources
| Action | XP |
|---|---|
| Win a match | 100 |
| Lose a match | 30 |
| Successful parry | 5 |
| Stamina break opponent | 10 |
| Deal damage | 1 per 10 damage |
| Complete a match (any result) | 20 |

XP rewards are intentionally weighted toward **participation and skill expression**, not just winning.

### Level Curve
- Levels 1–10: Fast (get to "character feels complete" quickly)
- Levels 10–50: Moderate
- Levels 50+: Slow (cosmetic prestige, no power impact)

### What Levels Unlock
| Level | Unlock |
|---|---|
| 1–5 | Base character, base combo set |
| 6 | Alternate combo variant (visual reskin, not new mechanics) |
| 10 | Minor stat tweak (e.g. +5 max stamina — negligible) |
| 15 | Second ability variant (different Q or E option, not strictly better) |
| 20 | Cosmetic: elemental aura effect |
| 25 | Alternate combo variant 2 |
| 30 | Cosmetic: color palette alt |
| 50 | Prestige badge |
| 100 | Prestige title |

**Stat increases from levels:** Absolute maximum of +10% to any stat by level 100. A level 1 vs level 100 fight is not a stat fight.

---

## Ability Choice System

As players level, they unlock **ability variant choices** for Q and E slots.

Example for Fire:
- Default Q: *Flame Dash*
- Q Variant (unlocked at level 15): *Phoenix Strike* — dash forward, dealing damage along the path (higher risk, higher reward)

Players choose which variant to equip. Neither is strictly better — it's a playstyle preference.

This lets players customize their character to suit how they want to play, without creating pay-to-win or stat-inflation paths.

---

## Combo Unlocks

At certain levels, players unlock **new combo sequences** for their element.

This is the primary progression mechanic that adds depth without changing power:
- New combo = new option in your toolkit
- You have to learn when to use it → skill expression
- Not replacing old combos, adding to your palette

Example unlock tree for Fire:
```
Level 1:  L, L, H (burst lance)
Level 6:  H, L (scorch pulse)
Level 12: L, H, H (eruption)
Level 20: H, H, H (inferno meteor — new, powerful but very telegraphed)
```

A high-level player has more tools. But if they use those tools poorly, a low-level player with fewer tools used well will win.

---

## Character Permanence

- Element is **locked at creation** — cannot respec
- This is intentional: part of the game's identity is that your element is your identity
- Players who want to try a different element create a new character (future: multi-character support per account)

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
