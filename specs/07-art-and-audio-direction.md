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
- `run` — 4/8 directional movement
- `attack_light` — casting animation
- `attack_heavy_windup` — telegraphed windup
- `attack_heavy_fire` — release animation
- `block` — guard stance
- `stagger` — recoil
- `stamina_break` — stumble
- `dodge` — dash
- `death` — defeat animation

### Asset Source Strategy (Hobby Project)
Since this is a personal project:
- **Option A:** Screenshot/rip sprites from Genshin Impact or similar games (legally gray but fine for private use)
- **Option B:** Use AI image generation (Midjourney, DALL-E) with the aesthetic prompt
- **Option C:** Find CC0/free sprite packs from itch.io that match the vibe
- **Option D:** Placeholder art first (colored rectangles), polish later

Recommended approach: Start with **colored placeholder rectangles** to get the game running. Replace with proper art once mechanics are solid. This is the right order of operations for a hobby game.

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

### HUD Layout (1v1)
```
[Player 1 HP] ████████░░  [Player 2 HP]  ████████░░
[Player 1 ST] ████░░░░░░  [Player 2 ST]  ████░░░░░░

                    [ROUND 1]

[Q cooldown]  [E cooldown]          [Q cooldown]  [E cooldown]
```

---

## Audio Direction

### Music
- **Style:** Atmospheric, orchestral-lite. Not aggressive. Fits the "cozy but intense" vibe.
- **Arenas:** Each arena has its own ambient track. Subtle, sets the mood.
- **Combat:** Music doesn't dramatically change during combat (not Doom-style). Maybe a subtle tension layer added.
- **Source:** AI-generated (Suno.ai, Udio, etc.) or royalty-free. No licensed music.

### SFX Design
Each action has distinct, satisfying sound design:

| Action | Sound |
|---|---|
| Light attack (Fire) | Sharp whoosh + crackle |
| Light attack (Water) | Fluid swoosh |
| Light attack (Earth) | Stone scrape + thud |
| Light attack (Air) | Airy slice + wind |
| Heavy impact | Deep bass thud, satisfying |
| Block | Metallic/magical clang |
| Parry | Sharp ring + brief silence (hitstop feel) |
| Dodge | Whoosh, directional |
| Stamina break | Brittle crack, character gasp |
| Stagger | Short grunt |
| Hit | Distinct per element |
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
