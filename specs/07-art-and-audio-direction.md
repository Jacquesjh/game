# Art & Audio Direction

---

## Visual Style

### Reference: Genshin Impact
- Warm, cel-shaded look
- Clean outlines on characters
- Lush, saturated environments
- Elemental effects that feel magical but grounded
- Anime-adjacent proportions (not hyper-realistic, not chibi)

### For a Browser Top-Down Game (Adaptation)
Since we're working top-down and in a browser, the Genshin aesthetic adapts to:
- **Characters:** 2D sprite sheets, top-down perspective, warm outlines
- **Environments:** Tiled maps with layered sprites, warm lighting baked into tiles
- **Particle effects:** The main expressiveness tool — heavy use of particles for all magic

---

## Character Art

### Style Guidelines
- Top-down angled perspective (about 45°–60° — isometric-ish but not strict isometric)
- Characters readable from above: clear silhouette, distinct elemental color coding
- Outline art (dark stroke around character)
- Elemental identity in costume color, not just attack colors

### Animation States (per character)
- `idle` — subtle breathing loop
- `run` — 8 directional movement (fluid, analog)
- `attack_light_[variant]` — each light variant has its own cast animation (e.g. `attack_light_1`, `attack_light_2`)
- `attack_heavy_windup_[variant]` — telegraphed windup per variant
- `attack_heavy_fire_[variant]` — release animation per variant
- `combo_[sequence]` — specific animations per combo (e.g. `combo_L_L_H`, `combo_DL_H`) — every distinct combo sequence has a distinct animation
- `dodge` — dash animation
- `dodge_attack_light` / `dodge_attack_heavy` — attack during dodge, distinct from standing attack
- `block` — guard stance
- `stagger` — recoil
- `stamina_break` — stumble animation
- `downed` — collapse, lying animation (looped)
- `execution_[id]` — execution animation (one per unlocked execution, long form)
- `being_executed` — receiving execution animation (counterpart)

### Asset Source Strategy (Hobby Project)

**The approach:** Use Genshin Impact as a direct visual reference. For any asset (tree, character, building, effect), find a screenshot or official art of that thing in Genshin, then use an AI image generator (Midjourney, DALL-E 3, Stable Diffusion) to regenerate it in the correct perspective for our game (top-down 45°, sprite sheet format).

Workflow for an asset:
1. Find Genshin Impact reference image (Google, wiki, official art)
2. Prompt AI: *"top-down 45 degree isometric sprite, [description from reference], Genshin Impact art style, transparent background, game asset"*
3. Clean up in Photoshop/GIMP if needed
4. Import as sprite sheet

This is a private hobby project. We're not publishing this commercially. Use references freely.

For v1: **colored placeholder sprites** to get combat working. Real art pass comes after mechanics are solid.

---

## Elemental Visual Palettes

### Fire
- Primary: `#FF6B35` (orange), `#FF3366` (crimson)
- Accent: `#FFD700` (molten gold)
- Glow: warm orange-red
- Particles: ember sparks, smoke wisps, heat shimmer

### Water
- Primary: `#00B4D8` (cyan), `#0077B6` (deep blue)
- Accent: `#90E0EF` (ice), `#FFFFFF` (foam)
- Glow: cool aqua
- Particles: water droplets, ripples, mist, ice crystals

### Earth
- Primary: `#6B4226` (brown), `#8B7355` (sandstone)
- Accent: `#4CAF50` (moss green), `#D4A017` (amber)
- Glow: warm amber
- Particles: rock chunks, dirt puffs, vine pieces, crystal shards

### Air
- Primary: `#B0E0E6` (powder blue), `#E0FFFF` (pale cyan)
- Accent: `#9B59B6` (violet edge), `#FFFFFF`
- Glow: pale blue-white
- Particles: wind trails, feather wisps, cloud puffs, lightning arc

---

## UI Style

- **HUD:** Minimal, semi-transparent. Health bar and stamina bar only, near the edges.
- **Font:** Clean, slightly stylized. Not serif. Something that fits the fantasy aesthetic.
- **Menus:** Warm background (parchment, wood panel, or blurred game world). Large readable buttons.
- **Color:** Match the elemental palette of the player's character where relevant.

### HUD Layout

**In-world (visible to all players):**
- **Health bar:** Floats above the character sprite. Short, readable bar. This is visible to opponents — you can read their HP.
- **Stamina indicator:** On the side of the character (left or right), like Genshin Impact / Breath of the Wild. Vertical or arc format. Also visible to opponents — they can read when you're low.

**Screen HUD (local player only):**
- Clean minimal overlay — just round indicator, maybe a timer
- No ability cooldown bars (Q/E removed)
- Stamina and health are in-world, not duplicated on screen HUD
- For team modes: ally health bars shown as a compact list on one corner (small, not distracting)

```
                    [ROUND 1]

[HP]                                    [HP]
 ↑ (above player sprite)               ↑ (above enemy sprite)
[ST]                                   [ST]
 ↑ (side of player sprite)             ↑ (side of enemy sprite)
```

Reference: Genshin Impact's stamina circle, Zelda BotW stamina wheel — contextual, in-world, expressive.

---

## Audio Direction

### Music
- **Style:** Atmospheric, orchestral-lite. Not aggressive. Fits the "cozy but intense" vibe.
- **Arenas:** Each arena has its own ambient track. Subtle, sets the mood.
- **Combat:** Music doesn't dramatically change during combat (not Doom-style). Maybe a subtle tension layer added.
- **Source:** AI-generated (Suno.ai, Udio, etc.) or royalty-free. No licensed music.

### Environment Audio
Each arena has **layered ambient audio** that plays continuously:
- Base ambient layer: wind, distant birds, water, crowd murmur (low)
- Weather layer: rain hits, thunder rumbles, lightning crack+thunder sequence
- Dynamic: slight volume shift when combat intensifies (optional, subtle)

Weather SFX events:
- **Rain start/stop:** fade in/out rain loop
- **Thunder:** random rumble at low volume, every 15–60 seconds
- **Lightning strike:** sharp crack + roll, coincides with skybox flash

### Combat SFX
Each action has distinct, satisfying sound design:

| Action | Sound |
|---|---|
| Light attack (Fire) | Sharp whoosh + crackle |
| Light attack (Water) | Fluid swoosh |
| Light attack (Earth) | Stone scrape + thud |
| Light attack (Air) | Airy slice + wind |
| Dodge attack | More aggressive version of above + momentum whoosh |
| Heavy impact | Deep bass thud, satisfying |
| Block | Metallic/magical clang |
| Parry | Sharp ring + brief silence (hitstop feel) + counter projectile launch sound |
| Dodge | Whoosh, directional |
| Stamina break | Brittle crack, character gasp |
| Stagger | Short grunt |
| Hit | Distinct per element |
| Downed | Collapse sound, brief groan |
| Execution start | Charged dramatic sound (element-flavored) |
| Execution complete | Satisfying finish sound + elemental burst |
| Menu click | Soft magical chime |

### SFX Source
- **Generated:** sfxr / jsfxr for quick placeholder SFX
- **Recorded:** Record and mix in Audacity (rustling, whooshes)
- **Free packs:** freesound.org, zapsplat.com (attribution required or paid tier)
- Same order of operations: placeholder SFX first, polish later

---

## Particle System Design

Particles are the main visual wow factor for magic attacks. Phaser 3 has a built-in particle emitter — use it heavily.

### Key Particle Configs Per Element

**Fire — Fireball trail:**
- Emitter follows projectile position
- Short-lived (0.3s) bright orange/red particles
- Slight upward gravity (embers rise)
- Scale down as they age

**Water — Arc trail:**
- Small droplets along arc path
- Blue-white, brief lifetime
- Subtle splash at impact

**Earth — Boulder:**
- Dirt/pebble particles kicked up behind boulder
- Impact: stone chunks fly outward

**Air — Crescent:**
- White vapor trail
- Tight, fast, disappears quickly
- Hit: burst of air lines (like speed lines)

### Impact Particles
Every hit has an impact particle burst at the collision point. Elemental color. Satisfying.

---

## Animation Priority Order for v1

1. **Placeholder art** — get combat mechanics working (colored sprites)
2. **Character sprites** — idle + run + basic attack
3. **HUD** — health/stamina bars
4. **Particle effects** — these are high-impact and relatively easy with Phaser
5. **Full animation set** — all states polished
6. **Environment** — detailed tilemap art
7. **Audio** — SFX then music
