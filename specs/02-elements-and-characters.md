# Element & Character Specs

Each element is a distinct playstyle archetype. Same mechanical framework, radically different feel.

---

## Design Principles

- **Fire** = aggression, burst, pressure
- **Water** = control, flow, adaptation
- **Earth** = defense, punishment, immovability
- **Air** = evasion, speed, unpredictability

Each element has:
- Distinct **color palette** and **particle style**
- Unique L/H attack visual forms
- Unique Q and E abilities
- Unique combo outcomes (same L,L,H input = different attack)
- Unique passive trait (small always-on modifier)

---

## Fire Mage

**Archetype:** Aggressive duelist. High burst, punishes mistakes hard. Stamina-hungry.

**Visual Identity:**
- Palette: deep oranges, crimson, molten gold
- Particles: ember trails, flare bursts, heat distortion
- Attack shapes: lances, spiraling fireballs, explosive rings

**Passive:** *Ignite* — attacks that land on a blocked opponent apply a small DoT (damage over time). Rewards constant pressure.

**Light Attack:** Fast, short-range fire bolt. Low damage, fast recovery.

**Heavy Attack:** Charged fireball with visible windup glow. High damage, moderate knockback. Variable delay — hard to dodge reliably.

**Combo Outcomes:**
| Sequence | Attack |
|---|---|
| L, L | Dual bolt — two bolts in quick succession |
| L, L, H | Burst lance — fast-traveling long-range bolt |
| L, H | Concussive blast — shorter range, knockback heavy |
| L, H, H | Eruption — ground burst beneath target, brief AoE |
| H, H | Meteor — very slow, very high damage, large projectile |
| H, L | Scorch pulse — quick radial fire burst to interrupt |

**Q Ability:** *Flame Dash* — short-range dash that leaves a fire trail. Trail persists briefly and damages opponents who step on it.

**E Ability:** *Inferno Barrage* — rapid-fire sequence of small fireballs (3-5). Low individual damage, but combo-able and stamina-draining for blocker.

**Stamina Cost Profile:** Moderate to high. Fire is punishing to play sloppy with — you'll run out of stamina fast if you don't land hits.

---

## Water Mage

**Archetype:** Adaptive controller. Best at reading opponents and punishing patterns. Excels at parrying.

**Visual Identity:**
- Palette: seafoam, deep navy, aqua, silver-white crests
- Particles: flowing streams, splash ripples, ice crystal fragments
- Attack shapes: arcing streams, spherical bubbles, crescent waves

**Passive:** *Flow State* — successfully blocking an attack restores a small amount of stamina. Rewards smart defensive play.

**Light Attack:** Curved water arc. Moderate speed, slight homing. Can curve around obstacles.

**Heavy Attack:** Water lance — straight, high velocity. Less damage than Fire heavy but faster travel.

**Combo Outcomes:**
| Sequence | Attack |
|---|---|
| L, L | Double arc — two curved streams |
| L, L, H | Water lance — fast piercing projectile |
| L, H | Undertow — pulls opponent slightly toward caster |
| L, H, H | Tide crash — large slow wave, wide AoE, knockdown on hit |
| H, H | Blizzard bolt — slows opponent movement briefly |
| H, L | Current snap — quick mid-range water snap, interrupts dodge |

**Q Ability:** *Mirror Flow* — for 2 seconds, Water Mage can "absorb" one incoming projectile and redirect it. High skill ceiling, huge payoff.

**E Ability:** *Whirlpool* — places a vortex on the map. Opponents near it are pulled toward the center and slowed. Short duration.

**Stamina Cost Profile:** Low to moderate. Water rewards patience — you regenerate stamina through blocking, so you can sustain long fights.

---

## Earth Mage

**Archetype:** Immovable wall. Punishes aggression, sets up guaranteed hits with terrain, hardest to stagger.

**Visual Identity:**
- Palette: stone grays, moss greens, amber sandstone, deep brown
- Particles: rock chunks, dust clouds, vine fragments, crystal spires
- Attack shapes: boulders, stone shards, rising pillars

**Passive:** *Fortitude* — stamina drain from blocking is reduced by 30%. Earth can sustain a block stance significantly longer.

**Light Attack:** Stone shard — medium speed, heavy hit feel even on light attacks. Short range.

**Heavy Attack:** Boulder throw — slow travel, massive stagger on hit. Hard to dodge due to size.

**Combo Outcomes:**
| Sequence | Attack |
|---|---|
| L, L | Shard burst — two shards in spread |
| L, L, H | Stone javelin — fast, narrow, piercing shot |
| L, H | Ground spike — spike erupts beneath target at short range |
| L, H, H | Earthshatter — massive radial ground slam, huge AoE, slow cast |
| H, H | Monolith — extremely slow, if it hits: massive damage + long stagger |
| H, L | Rubble spray — quick low-damage spread shot to interrupt |

**Q Ability:** *Stone Wall* — places a temporary solid wall on the map. Blocks projectiles and movement. Can be used to force opponents into kill zones.

**E Ability:** *Granite Form* — brief defensive stance (1.5 sec). During it, incoming attacks deal reduced damage and Earth is immune to stagger. High cooldown.

**Stamina Cost Profile:** Low. Earth is designed to outlast opponents. Your attacks cost less, your blocks drain less. The threat is that your attacks are slow — you need to create openings.

---

## Air Mage

**Archetype:** Evasion specialist. Extremely mobile, hard to pin down. Lowest HP, highest dodge uptime.

**Visual Identity:**
- Palette: sky blue, pale white, violet-edged clouds, cyan lightning
- Particles: wind trails, feather wisps, cloud puffs, micro-lightning arcs
- Attack shapes: slicing crescents, cyclone spirals, compressed air bursts

**Passive:** *Gust Step* — dodge has shorter recovery time than other elements. Air can chain movement more freely.

**Light Attack:** Air crescent — fast-traveling slice, thin hitbox.

**Heavy Attack:** Cyclone bolt — spiraling projectile, wide hitbox, medium travel speed. Deceptively hard to dodge due to spin.

**Combo Outcomes:**
| Sequence | Attack |
|---|---|
| L, L | Twin slash — two rapid crescents |
| L, L, H | Wind burst — fast AoE radial burst around caster |
| L, H | Updraft — launches opponent briefly (reduces their dodge availability) |
| L, H, H | Tempest — large spiraling wave, homing but slow |
| H, H | Storm call — brief channeled storm (multiple small hits) |
| H, L | Pressure snap — close-range knockback |

**Q Ability:** *Tailwind* — brief self-speed boost. Moves significantly faster for 2 seconds, harder to track with homing projectiles.

**E Ability:** *Vacuum* — creates a localized void that pulls all nearby projectiles inward and destroys them. Counter-play against projectile-heavy opponents.

**Stamina Cost Profile:** Moderate. Air is less about stamina management and more about positioning. You're less tanky, so you need to avoid hits entirely rather than block them.

---

## Balance Notes

The goal is not numerical balance at v1 — it's **clear differentiation**. Each matchup should feel like a different puzzle:
- Fire vs Earth: Fire needs to create openings fast before stamina runs dry; Earth waits for punish
- Water vs Air: Water tries to parry and redirect; Air tries to never be where the attack lands
- Fire vs Air: Chaotic — Fire has burst, Air has mobility
- Water vs Earth: Slow, methodical, longest fights

Balance passes come later. Right now, make each element feel **right**.
