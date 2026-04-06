# Combat System Deep Dive

---

## State Machine: Player Combat States

Every player is always in one of these states. States are enforced server-side.

```
IDLE
 ├──→ MOVING (WASD held — movement is always free unless in specific locks)
 ├──→ ATTACKING_LIGHT (LMB, not in recovery)
 ├──→ ATTACKING_HEAVY (RMB, not in recovery)
 ├──→ BLOCKING (block held + stamina > 0)
 ├──→ PARRY_ATTEMPT (block input at precise moment of incoming hit)
 ├──→ DODGING (space input, not in dodge cooldown)
 │     └──→ DODGE_ATTACKING_LIGHT (LMB during dodge i-frames)
 │     └──→ DODGE_ATTACKING_HEAVY (RMB during dodge i-frames)
 ├──→ STAGGERED (hit by unblocked attack)
 ├──→ STUNNED (after being parried — attacker is stunned)
 ├──→ STAMINA_BROKEN (stamina hits 0)
 ├──→ DOWNED (HP hits 0)
 └──→ EXECUTING (pressing execute near downed enemy)

ATTACKING_LIGHT / ATTACKING_HEAVY
 ├── [during cast] → cannot dodge, cannot attack again
 ├── [stamina does NOT regenerate during attack]
 └──→ RECOVERY (after attack fires)
       └──→ IDLE (after recovery window ends)

DODGING
 ├── [i-frame window at start: DODGE_INVINCIBILITY_MS]
 ├── [during i-frames: can input LMB → DODGE_ATTACKING_LIGHT]
 ├── [during i-frames: can input RMB → DODGE_ATTACKING_HEAVY]
 └──→ RECOVERY → IDLE (after LOCK_DODGE elapses)

DODGE_ATTACKING_LIGHT / DODGE_ATTACKING_HEAVY
 ├── [fires attack immediately — no extra windup, inherits dodge momentum]
 ├── [counts as combo starter — can chain into L/H combos after recovery]
 └──→ RECOVERY → IDLE

PARRY_ATTEMPT (success)
 ├── [parried attack is nullified]
 ├── [auto-counter projectile fires instantly toward attacker]
 ├── [INSTANT_CAST_WINDOW opens: next attack has 0 windup for PARRY_COUNTER_WINDOW_MS]
 └──→ IDLE

STAGGERED
 ├── [brief window: can input dodge or block for escape if timed correctly]
 └──→ IDLE (after stagger duration)

STAMINA_BROKEN (stamina hits 0 — only two states: normal and broken)
 ├── [brief stumble animation: LOCK_STAMINA_BREAK]
 ├── [block is completely unavailable during break recovery]
 ├── [regen rate drops to STAMINA_REGEN_BROKEN_PER_SEC during recovery period]
 └──→ IDLE (after break recovery)

DOWNED (HP hits 0)
 ├── [player collapses, cannot act]
 ├── [downed timer starts: DOWNED_DURATION_MS]
 ├── [any opponent nearby can press execute to enter BEING_EXECUTED]
 └──→ DEAD (after timer expires OR after execution completes)

EXECUTING (executor's state)
 ├── [long animation: EXECUTION_DURATION_MS — varies per execution]
 ├── [executor cannot move, block, or dodge]
 ├── [if executor takes any hit: execution is cancelled, executor is staggered]
 └──→ IDLE (if interrupted) or IDLE + health_gain (if completed)

BEING_EXECUTED (downed player's state)
 └──→ DEAD (permanently, no timer resume)
```

---

## Timing Constants (All Tunable)

These are starting values for playtesting. Everything in `shared/constants/combat.ts`.

```typescript
export const COMBAT_CONSTANTS = {
  // Stamina
  STAMINA_MAX: 100,
  STAMINA_REGEN_IDLE_PER_SEC: 18,       // Regen while idle or moving
  STAMINA_REGEN_BLOCKING_PER_SEC: 8,    // Regen while blocking (reduced)
  STAMINA_REGEN_ATTACKING_PER_SEC: 0,   // NO regen while attacking
  STAMINA_REGEN_BROKEN_PER_SEC: 6,      // Regen during stamina break recovery (slow)
  STAMINA_BREAK_RECOVERY_MS: 1500,      // How long the "broken" recovery period lasts

  // Attack stamina costs
  STAMINA_COST_LIGHT: 10,
  STAMINA_COST_HEAVY: 22,
  STAMINA_COST_DODGE_LIGHT: 8,
  STAMINA_COST_DODGE_HEAVY: 18,
  STAMINA_COST_BLOCK_HIT_LIGHT: 8,
  STAMINA_COST_BLOCK_HIT_HEAVY: 20,

  // Animation locks (milliseconds)
  LOCK_LIGHT_ATTACK: 300,           // Recovery after light attack fires
  LOCK_HEAVY_ATTACK_WINDUP: 500,    // Windup before heavy fires (press, not hold)
  LOCK_HEAVY_ATTACK_RECOVERY: 600,  // Recovery after heavy fires
  LOCK_DODGE_ATTACK_RECOVERY: 350,  // Recovery after a dodge attack
  LOCK_DODGE: 400,                  // Recovery after dodge (if no attack triggered)
  LOCK_PARRY_ATTEMPT: 200,          // Block window + recovery
  LOCK_STAGGER: 450,                // How long stagger lasts (from light)
  LOCK_STAGGER_HEAVY: 750,          // Stagger from heavy attack hit
  LOCK_STAMINA_BREAK: 500,          // Initial stumble when stamina hits 0
  LOCK_STUNNED_PARRIED: 600,        // How long attacker is stunned after being parried

  // Parry
  PARRY_WINDOW_MS: 150,             // Block input received within 150ms of impact = parry
  PARRY_COUNTER_WINDOW_MS: 500,     // Window where parrying player's next attack has 0 windup
  PARRY_COUNTER_PROJECTILE_SPEED: 400,  // Speed of auto-fired counter projectile
  PARRY_COUNTER_DAMAGE: 18,             // Counter projectile damage

  // Dodge
  DODGE_INVINCIBILITY_MS: 150,      // I-frames at start of dodge (attack window also here)
  DODGE_DASH_DISTANCE: 130,         // Units traveled
  DODGE_COOLDOWN_MS: 750,           // Cannot dodge again for 750ms

  // Projectile homing
  HOMING_ANGLE_PER_SEC: 90,         // Degrees/sec the projectile can turn toward target
  HOMING_MAX_RANGE: 400,            // Beyond this range, no homing

  // Damage (ALL fixed — not affected by level. Element variants may differ slightly.)
  DAMAGE_LIGHT: 12,
  DAMAGE_HEAVY: 30,
  DAMAGE_DODGE_LIGHT: 10,
  DAMAGE_DODGE_HEAVY: 22,
  DAMAGE_BLOCKED_MULTIPLIER: 0.1,   // 10% damage bleeds through a normal block

  // Health (fixed per element — slight variance allowed for identity, not balance)
  HP_DEFAULT: 150,                  // Base. Elements may tune ±15.

  // Execution
  DOWNED_DURATION_MS: 6000,                // How long downed state lasts before natural death
  EXECUTION_HEALTH_GAIN_PER_SEC: 12,       // HP regained per second of execution animation
  EXECUTION_INTERRUPT_STAGGER_MS: 500,     // Executor stagger when interrupted
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
type AttackInput = 'L' | 'H' | 'DL' | 'DH'  // L=light, H=heavy, DL=dodge-light, DH=dodge-heavy
type ComboSequence = AttackInput[]  // e.g. ['L', 'L', 'H'] or ['DL', 'L', 'H']

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

Player dodges, then presses L during i-frames
  → combo starts as ['DL']
  → continues: ['DL', 'L'], ['DL', 'L', 'H'], etc.
```

Window resets after each attack's recovery completes. Can't chain into next attack mid-recovery (this is what creates the stagger opportunity).

### Attack Variant System

Players equip **one light variant** and **one heavy variant** before each match. Variants are unlocked through progression.

```typescript
interface CharacterLoadout {
  lightVariantId: string   // e.g. 'fire_light_2' (unlocked at level 15)
  heavyVariantId: string   // e.g. 'fire_heavy_1' (base)
  executionId: string      // which execution animation to use
}
```

The combo resolver uses the equipped variant to determine which `AttackDefinition` to apply. Two players with the same element but different equipped variants have different combo outcomes for the same input sequence.

```
Example: Fire player A (L1 + H1) vs Fire player B (L2 + H1)
Both press L, L, H:
  Player A: fires 'fire_l1_l1_h1' attack (burst lance)
  Player B: fires 'fire_l2_l1_h1' attack (different trajectory, same general threat)
```

Each `AttackDefinition` references its own unique animation key — every combo sequence has a distinct visual.

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

## Stamina States (Two Only)

### Normal Stamina (> 0)
All actions available. No penalties.

### Stamina Break (= 0)
- Character visually stumbles (`LOCK_STAMINA_BREAK` animation)
- **Block is completely unavailable** during the break recovery period (`STAMINA_BREAK_RECOVERY_MS`)
- Stamina regenerates at `STAMINA_REGEN_BROKEN_PER_SEC` (slow) during recovery
- After recovery period ends, normal regen resumes and block becomes available again

No "low stamina" intermediate state — the binary nature makes the moment of stamina break more decisive and readable.

Core rhythm: **bait aggressive attacking (no regen) or sustained blocking (slow regen) → opponent's stamina drains → stamina break → punish window**.

---

## Execution System (Server-Side)

### Downed Resolution

When server resolves `hp <= 0` for a player:
1. Transition player state to `DOWNED`
2. Start `downed_timer` countdown (`DOWNED_DURATION_MS`)
3. Broadcast `player_downed` event to all clients in room
4. Player becomes non-collidable for combat (projectiles pass through)

### Execute Attempt

When an opponent within `EXECUTE_RANGE` units presses execute:
1. Server validates: is target `DOWNED`? Is executor not currently in any lock state?
2. Server transitions executor to `EXECUTING`, target to `BEING_EXECUTED`
3. `execution_start` event broadcast with `executionId` (determines animation + duration)
4. Server starts tracking elapsed execution time

### Execution Tick

Every tick while executor is `EXECUTING`:
- If executor takes any damage: cancel execution, executor → `STAGGERED`, target → `DOWNED` (timer resumes from where it was)
- If `elapsed >= execution.durationMs`: complete execution
  - Target → `DEAD` (permanent)
  - Executor health += `elapsed_seconds * EXECUTION_HEALTH_GAIN_PER_SEC`
  - `execution_complete` event broadcast

### Downed Timer Expiry

If `downed_timer` reaches 0 with no execution in progress:
- Player → `DEAD` (same outcome, no health gain for anyone)
