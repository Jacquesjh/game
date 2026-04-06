# Combat System Deep Dive

---

## State Machine: Player Combat States

Every player is always in one of these states. States are enforced server-side.

```
IDLE
 ├──→ MOVING (WASD held)
 ├──→ ATTACKING_LIGHT (L input, not in recovery)
 ├──→ ATTACKING_HEAVY (H input, not in recovery)
 ├──→ ABILITY_1 (Q input)
 ├──→ ABILITY_2 (E input)
 ├──→ BLOCKING (block held + stamina > 0)
 ├──→ PARRY_ATTEMPT (block input at precise moment)
 ├──→ DODGING (space input, not in dodge cooldown)
 ├──→ STAGGERED (hit by unblocked attack)
 └──→ STUNNED (post-stagger, parried)

ATTACKING_LIGHT
 ├── [during cast] → cannot dodge, cannot attack again
 ├── [can block mid-cast if light attack allows it]
 └──→ RECOVERY (after attack fires)
       └──→ IDLE (after recovery window)

STAGGERED
 ├── [brief window: can input dodge or parry for escape]
 └──→ IDLE (after stagger duration)

STAMINA_BROKEN (stamina hits 0)
 ├── STAGGERED briefly
 ├── enters LOW_STAMINA substate
 └── all actions still available but slower + weakened
```

---

## Timing Constants (All Tunable)

These are starting values for playtesting. Everything in `shared/constants/combat.ts`.

```typescript
export const COMBAT_CONSTANTS = {
  // Stamina
  STAMINA_MAX: 100,
  STAMINA_REGEN_PER_SEC: 15,       // Normal regen
  STAMINA_REGEN_BROKEN_PER_SEC: 8, // While stamina-broken
  STAMINA_LOW_THRESHOLD: 20,        // Below this = "low stamina" state

  // Attack stamina costs
  STAMINA_COST_LIGHT: 10,
  STAMINA_COST_HEAVY: 20,
  STAMINA_COST_ABILITY: 15,        // Per ability (override per element)
  STAMINA_COST_BLOCK_HIT_LIGHT: 8,
  STAMINA_COST_BLOCK_HIT_HEAVY: 18,

  // Animation locks (milliseconds)
  LOCK_LIGHT_ATTACK: 300,           // Cannot act for 300ms after light attack fires
  LOCK_HEAVY_ATTACK_WINDUP: 600,    // Windup before heavy fires
  LOCK_HEAVY_ATTACK_RECOVERY: 500,  // Recovery after heavy fires
  LOCK_DODGE: 400,                  // Recovery after dodge
  LOCK_PARRY_ATTEMPT: 200,          // Window + recovery
  LOCK_STAGGER: 500,                // How long stagger lasts
  LOCK_STAGGER_HEAVY: 800,          // Stagger from heavy attack
  LOCK_STAMINA_BREAK: 600,          // Stagger when stamina hits 0

  // Parry
  PARRY_WINDOW_MS: 150,             // Must be in BLOCKING and receive hit within 150ms of block start
  PARRY_COUNTER_WINDOW_MS: 500,     // Window for attacker to follow up after parry

  // Dodge
  DODGE_INVINCIBILITY_MS: 150,      // I-frames at start of dodge
  DODGE_DASH_DISTANCE: 120,         // Units traveled
  DODGE_COOLDOWN_MS: 800,           // Cannot dodge again for 800ms

  // Projectile homing
  HOMING_ANGLE_PER_SEC: 90,         // Degrees/sec the projectile can turn toward target
  HOMING_MAX_RANGE: 400,            // Beyond this range, no homing

  // Damage (base values, element-specific overrides in element files)
  DAMAGE_LIGHT: 12,
  DAMAGE_HEAVY: 28,
  DAMAGE_BLOCKED_MULTIPLIER: 0.1,   // 10% damage bleeds through blocks
  DAMAGE_LOW_STAMINA_BLOCK_MULTIPLIER: 0.5, // 50% damage bleeds when low stamina blocking
}
```

---

## Combo Resolver

The combo system tracks the recent attack sequence and resolves it to a specific attack type.

### How It Works

1. Client records attack inputs with timestamps
2. Server has a **combo window** — if inputs come within the window, they chain
3. After the window expires (or recovery ends), combo resets
4. The resolved sequence maps to an `AttackDefinition`

```typescript
// shared/types/combat.ts
type AttackInput = 'L' | 'H'
type ComboSequence = AttackInput[]  // e.g. ['L', 'L', 'H']

interface AttackDefinition {
  id: string
  element: Element
  comboSequence: ComboSequence
  name: string

  // Timing
  windupMs: number         // Delay before attack fires (the "variable delay" for mind games)
  recoveryMs: number       // Lock after attack fires

  // Projectile
  projectileSpeed: number
  homingStrength: number   // 0 = no homing, 1 = full homing
  hitboxRadius: number
  range: number            // Max travel distance

  // Effect
  damage: number
  staminaCost: number
  staggerDuration: number  // How long target is staggered on hit
  knockback: number        // Force applied to target

  // Special
  aoe: boolean
  aoeRadius?: number
  specialEffect?: 'pull' | 'slow' | 'dot' | 'launch'

  // Visuals (client-side only, server doesn't care)
  particleEffect: string   // Key into particle system
  projectileSprite: string
  castAnimation: string
  hitAnimation: string
}
```

### Combo Window

```
Player presses L
  → starts combo, waits COMBO_WINDOW_MS (e.g. 500ms) for next input
  → if L again within window: combo is now ['L', 'L'], continue waiting
  → if H within window: combo is ['L', 'H'], resolve
  → if nothing within window: resolve ['L']
```

Window resets after each attack's recovery completes. Can't chain into next attack mid-recovery (this is what creates the stagger opportunity).

---

## Directional Block / Parry System

### Block Facing
- Player always has an `aimAngle` (from mouse)
- Incoming attack has a `sourceAngle` (from where it came)
- Block succeeds if: `|aimAngle - sourceAngle| < BLOCK_CONE_DEGREES` (e.g. 90°)
- Block fails (hit lands) if outside that cone

```
   Enemy
    ↓
  attack
    ↓
[Player] ← must be facing within 90° of this direction to block
```

This means you can be attacked from the side or behind to bypass block. In 1v1 it's mostly about aiming at your opponent, but creates depth in team modes.

### Parry Timing
- Parry is triggered by pressing block at the **moment of impact** (within `PARRY_WINDOW_MS`)
- The window begins when the attack visually arrives (client shows impact frame)
- Server validates: was block input received within window of hit timestamp?
- This is the main area where latency handling matters — we'll need generous server-side timing

---

## Hit Resolution (Server)

Every tick, server:
1. Advances all projectile positions
2. Checks each projectile against all player hitboxes
3. For each collision:
   a. Is target DODGING (in i-frames)? → miss
   b. Is target BLOCKING and facing correct direction?
      - Is it a successful parry? → apply PARRY result
      - Normal block: apply `DAMAGE_BLOCKED_MULTIPLIER` damage, drain target stamina
   c. Otherwise: full hit → apply damage, stagger target, apply knockback

---

## Visual Feedback (Client)

Client plays effects based on state received from server:
- **Hit landed:** hit flash on target, particle burst at impact point, screen shake (small)
- **Block:** block spark, shield flash
- **Parry:** distinct sound + flash, brief freeze frame (hitstop), then counter window visual indicator
- **Stagger:** target's sprite does a brief recoil animation
- **Stamina break:** dramatic flash, character stumbles
- **Dodge:** motion blur trail in dash direction, whoosh sound

All visual timing should feel tight and satisfying. Hitstop (brief freeze on impact) is a key feel element — see Wizard of Legend and Street Fighter for reference.

---

## Anti-Spam Design

Combat design decisions that discourage spamming:

1. **Animation locks** — you can't spam light attacks, there's always a recovery window
2. **Stamina costs** — spamming attacks drains stamina fast, leaving you vulnerable
3. **Combo commitment** — entering a heavy attack is a commitment; if it misses, you have a long recovery window
4. **Dodge cooldown** — dodge is free but has an 800ms cooldown, can't be spammed
5. **Variable delays** — mindless spamming is punishable because opponents can read the timing

---

## Low Stamina State

Entering LOW_STAMINA (< 20 stamina):
- Block visual changes (shield cracks, wobbles)
- Attack animations play at 70% speed (more readable to opponent)
- Recovery windows increase by 30%
- Blocking does NOT provide stagger immunity — opponent can break through with heavy

Stamina Break (0 stamina):
- Character visually stumbles briefly (LOCK_STAMINA_BREAK)
- Regen rate drops to slower value during recovery
- Short vulnerability window

This creates the core rhythm: **pressure the opponent's stamina → create an opening → punish**.
