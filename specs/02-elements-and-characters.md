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
- Unique L/H attack variants (L1, L2... H1, H2...) with distinct animations
- Unique combo outcomes (same L,L,H input = different attack per element)
- Unique dodge attack feel (DL, DH) — all three dodge directions (forward, side, back) have distinct attacks
- Unique execution animations
- Unique passive trait (small always-on modifier)

**Trajectory & Speed Variety (applies to all elements):**
Every element's kit must mix trajectory types and speed profiles. No element should have all straight constant-speed attacks. The variety forces opponents to learn each attack individually:
- *Straight* attacks reward aim; *arc* attacks reward misdirection
- *Accelerating* attacks bait early dodges; *decelerating* attacks punish early dodges by lingering
- *ground_line* and *spawn_at_target* attacks require repositioning before the attack fires, not dodging after

**Variant Structure (per element):**
- **L1, L2, L3...** — light chains of varying length. Longer chains reward sustained aggression; shorter chains suit players who mix L/H more. L1 is the base. More variants can be added over time.
- **H1, H2, H3...** — heavy chains of varying length. H chain's last hit is always the combo finisher (triggers recovery). H1 is the base. More variants can be added over time.
- **DL (forward / side / back)** — dodge light attacks, one per direction
- **DH (forward / side / back)** — dodge heavy attacks, one per direction

Specific chain lengths, hit properties, trajectories, and animations for each variant are **to be defined per element** — see the element sections below. This is where the bulk of design work lives and will be expanded over time.

All attacks across all elements apply **soft target tracking at cast time** — the attack direction adjusts toward the nearest opponent within `TARGET_LOCK_RADIUS` of the cursor. This applies to every trajectory type (cone orients toward opponent, ground_line erupts toward opponent, spawn_at_target snaps to opponent position, etc.).

---

## Fire Mage

**Archetype:** Aggressive duelist. High burst, punishes mistakes hard. Stamina-hungry.

**Visual Identity:**
- Palette: deep oranges, crimson, molten gold
- Particles: ember trails, flare bursts, heat distortion
- Attack shapes: lances, cones, spiraling fireballs, explosive rings

**Passive:** *Ignite* — attacks that land on a blocked opponent apply a small DoT (damage over time). Rewards constant pressure.

**Attack flavor:** Fast, direct, in-your-face. Fire attacks should feel like commitment — high damage when they land, punishing recovery when they miss. Mix of straight and accelerating trajectories for the core kit, with cone and ground eruption options in variants. Even Fire's arcs should feel aggressive, not passive.

**Variants (to be designed):**
- L1 (base): long chain, aggressive poke-focused
- L2, L3: to be defined — vary chain length and hit feel
- H1 (base): to be defined
- H2, H3: to be defined
- DL forward / side / back: forward is the core engage; side flick an arc; back punishes chasers
- DH forward / side / back: forward is high-commitment burst; side sweeps wide arc; back escape shot

---

## Water Mage

**Archetype:** Adaptive controller. Best at reading opponents and punishing patterns. Excels at parrying.

**Visual Identity:**
- Palette: seafoam, deep navy, aqua, silver-white crests
- Particles: flowing streams, splash ripples, ice crystal fragments
- Attack shapes: arcing streams, spherical bubbles, crescent waves, long torrents

**Passive:** *Flow State* — successfully blocking an attack restores a small amount of stamina. Rewards smart defensive play.

**Attack flavor:** Curved, flowing, deceptive. Water attacks should rarely go straight — arcs are the identity. The one time Water fires straight, it should feel like a deliberate surprise. Some attacks pull the opponent; some deny space. Key concept: side DH sends a long arcing stream with a trail hitbox — opponent must read the stream's direction to dodge correctly.

**Variants (to be designed):**
- L1 (base): arc-heavy poke chain
- L2, L3: to be defined
- H1 (base): to be defined
- H2, H3: to be defined
- DL forward / side / back: forward close-range surprise straight; side curved arc; back spreading arc
- DH forward / side / back: forward undertow pull; side long arcing stream (trail, signature); back slow wave area denial

---

## Earth Mage

**Archetype:** Immovable wall. Punishes aggression, sets up guaranteed hits with terrain, hardest to stagger.

**Visual Identity:**
- Palette: stone grays, moss greens, amber sandstone, deep brown
- Particles: rock chunks, dust clouds, vine fragments, crystal spires
- Attack shapes: boulders, stone shards, rising pillars, spike lines

**Passive:** *Fortitude* — stamina drain from blocking is reduced by 30%. Earth can sustain a block stance significantly longer.

**Attack flavor:** Heavy, deliberate, ground-based. Earth attacks should feel slow but inevitable — large hitboxes, massive stagger, hard to dodge if you misread them. Key trajectory concepts: ground_line spike eruptions (straight and arc variants) that must be dodged sideways; boulders that decelerate to punish early dodges. Earth is about forcing the opponent into bad positions, not chasing them.

**Variants (to be designed):**
- L1 (base): slow but hard-hitting shard chain
- L2, L3: to be defined
- H1 (base): boulder throw (straight, decelerating)
- H2: spike line (ground_line, straight) — must be dodged sideways
- H3: arc spike line (ground_line, curved path) — different read from H2; two spike variants teach opponent to distinguish windup
- DL forward / side / back: forward aggressive poke; side arc flick; back counter-flick (signature — retreats, punishes chasers)
- DH forward / side / back: forward boulder charge; side perpendicular spike line; back ground spike area denial

---

## Air Mage

**Archetype:** Evasion specialist. Extremely mobile, hard to pin down. Lowest HP, highest dodge uptime.

**Visual Identity:**
- Palette: sky blue, pale white, violet-edged clouds, cyan lightning
- Particles: wind trails, feather wisps, cloud puffs, micro-lightning arcs
- Attack shapes: slicing crescents, cyclone spirals, compressed air bursts, tornadoes

**Passive:** *Gust Step* — dodge has shorter recovery time than other elements. Air can chain movement more freely.

**Attack flavor:** Fast, unpredictable, positional. Air attacks mix straight fast hits with arcs and spawn_at_target to keep opponents guessing. No single attack is obviously threatening — the danger is that Air can chain them from any direction, any position. Key concept: forward DH spawns a tornado at the opponent's location (spawn_at_target) — the tell is the windup; the opponent must move before it fires. Side DH leaves a wind trail.

**Variants (to be designed):**
- L1 (base): fast crescent chain, straight
- L2, L3: to be defined
- H1 (base): cyclone bolt (arc, wide hitbox)
- H2, H3: to be defined
- DL forward / side / back: forward blink-in straight crescent; side blink arc; back blink-away straight poke
- DH forward / side / back: forward tornado spawn_at_target (signature); side spin trail + cyclone arc; back wide arc area denial

**Stamina Cost Profile:** Moderate. Air's Gust Step passive makes dodging more viable as a defense strategy, reducing reliance on blocking. Less tanky overall — position or die.

---

## Balance Notes

The goal is not numerical balance at v1 — it's **clear differentiation**. Each matchup should feel like a different puzzle:
- Fire vs Earth: Fire needs to create openings fast before stamina runs dry; Earth waits for punish
- Water vs Air: Water tries to parry and redirect; Air tries to never be where the attack lands
- Fire vs Air: Chaotic — Fire has burst, Air has mobility
- Water vs Earth: Slow, methodical, longest fights

Balance passes come later. Right now, make each element feel **right**.
