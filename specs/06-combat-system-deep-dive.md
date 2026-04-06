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
 ├──→ DODGING (space, not in dodge cooldown — direction: forward | side | back relative to aim)
 │     └──→ DODGE_ATTACKING_LIGHT (LMB during dodge i-frames)
 │     └──→ DODGE_ATTACKING_HEAVY (RMB during dodge i-frames)
 ├──→ ROLLING (double-tap space — high stamina cost escape, no attack available)
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

DODGING (direction relative to player's aim: forward | side_left | side_right | back)
 ├── [i-frame window at start: DODGE_INVINCIBILITY_MS]
 ├── [during i-frames: can input LMB → DODGE_ATTACKING_LIGHT]
 ├── [during i-frames: can input RMB → DODGE_ATTACKING_HEAVY]
 └──→ RECOVERY → IDLE (after LOCK_DODGE elapses)

DODGE_ATTACKING_LIGHT / DODGE_ATTACKING_HEAVY
 ├── [fires attack immediately — no extra windup, inherits dodge momentum]
 ├── [attack variant is determined by dodge direction: forward / side / back]
 │     e.g. Water side-dodge heavy: dodge sideways, send torrent toward opponent
 │     e.g. Fire forward-dodge light: dash in, release close-range burst
 ├── [counts as combo starter — can chain into L/H combos after recovery]
 └──→ RECOVERY → IDLE

ROLLING (double-tap space within DOUBLE_TAP_WINDOW_MS — stamina required)
 ├── [costs STAMINA_COST_ROLL on activation — emergency resource, use sparingly]
 ├── [longer i-frame window: ROLL_INVINCIBILITY_MS]
 ├── [longer travel distance: ROLL_DASH_DISTANCE]
 ├── [no attack input available during roll]
 └──→ RECOVERY → IDLE (after LOCK_ROLL elapses)

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

  // Roll escape (double-tap space)
  DOUBLE_TAP_WINDOW_MS: 200,        // Max gap between two space presses to trigger roll
  STAMINA_COST_ROLL: 35,            // High cost — emergency use only
  ROLL_INVINCIBILITY_MS: 350,       // Longer i-frames than regular dodge
  ROLL_DASH_DISTANCE: 220,          // ~1.7x regular dodge distance
  LOCK_ROLL: 650,                   // Recovery after roll (punishing if misused)

  // Combo
  COMBO_WINDOW_MS: 500,             // Max gap between inputs to continue a chain

  // Soft target tracking (cast-time aim correction toward nearest opponent to cursor)
  TARGET_LOCK_RADIUS: 80,           // Units — if opponent is within this of the cursor, attack tracks them

  // Projectile homing (in-flight, only for trajectoryType 'homing')
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

### Core Concept: Variant Chains

Each variant (L1, L2, L3, H1, H2, H3) is an **ordered chain of hits**. Each hit in the chain is its own `AttackDefinition` with its own trajectory, timing, and animation. The chain length is the number of hits in that variant — L1 might have 4, L3 might have 2, H2 might have 3.

Players equip **one light variant** and **one heavy variant** before each match. Pressing L advances the equipped light chain; pressing H advances the equipped heavy chain. The two chains can be freely interleaved. This is where the combo variety comes from — not from fixed sequences mapped to fixed attacks, but from two independent chains playing off each other.

```typescript
// shared/types/combat.ts

type DodgeDirection = 'forward' | 'side_left' | 'side_right' | 'back'
type AttackInput = 'L' | 'H' | `DL_${DodgeDirection}` | `DH_${DodgeDirection}`

interface VariantChain {
  variantId: string          // e.g. 'fire_L1', 'water_H2'
  element: Element
  type: 'light' | 'heavy' | 'dodge_light' | 'dodge_heavy'
  dodgeDirection?: DodgeDirection   // only for dodge variants
  hits: AttackDefinition[]          // ordered chain; hits[last] is the finisher
}

interface CharacterLoadout {
  lightVariantId: string     // e.g. 'fire_L1' (base, always available)
  heavyVariantId: string     // e.g. 'fire_H2' (unlocked via progression)
  executionId: string
}

interface AttackDefinition {
  id: string
  variantId: string          // which VariantChain this hit belongs to
  chainIndex: number         // 0-based position within the chain
  isFinisher: boolean        // true = last hit of a heavy chain

  name: string
  element: Element

  // Timing
  windupMs: number           // Delay before attack fires
  recoveryMs: number         // Lock after attack fires

  // Soft target tracking (applies to ALL trajectory types at cast time)
  // When an opponent is within TARGET_LOCK_RADIUS of the cursor, the attack
  // direction adjusts toward them automatically. This is NOT in-flight homing —
  // it only affects the initial direction at the moment of casting.
  // Applies to cones (aim cone at opponent), ground_line (line erupts toward opponent),
  // spawn_at_target (snaps to opponent position), arcs (arc curves toward opponent), etc.
  // homingStrength governs additional in-flight correction for 'homing' trajectoryType only.
  softTrackingEnabled: boolean  // true for all attacks unless explicitly disabled

  // Trajectory
  trajectoryType:
    | 'straight'        // Direct line from cast point toward (tracked) target
    | 'arc'             // Parabolic curve toward target — arrives from an angle
    | 'homing'          // In-flight tracking (homingStrength governs turn rate)
    | 'cone'            // Fan spread toward target — wedge hitbox, no single head
    | 'ground_line'     // Sequential ground eruptions in a line toward target
    | 'spawn_at_target' // Spawns at opponent's position when attack fires — no travel.
                        // Opponent must reposition BEFORE windup ends; dodging after is too late.

  // Speed profile
  speedProfile: 'constant' | 'accelerating' | 'decelerating'
  // constant:     uniform speed throughout travel
  // accelerating: starts slow, builds to full speed — baits early dodges
  // decelerating: starts fast, slows — opponents who dodge early may walk back into it
  projectileSpeedInitial: number
  projectileSpeedFinal: number

  // Projectile
  homingStrength: number     // 0–1 turn rate; only meaningful for trajectoryType 'homing'
  hitboxRadius: number
  range: number              // Max travel distance or effect radius

  // Effect
  damage: number
  staminaCost: number
  staggerDuration: number
  knockback: number

  // Trail / stream hitbox
  // I-frames do NOT protect against trail damage — moving into the trail deals damage.
  trail: boolean
  trailDamage: number
  trailWidth: number
  trailDurationMs: number

  // Special
  aoe: boolean
  aoeRadius?: number
  specialEffect?: 'pull' | 'slow' | 'dot' | 'launch'

  // Visuals (client-side only)
  particleEffect: string
  projectileSprite: string
  castAnimation: string
  hitAnimation: string
}
```

### How Chaining Works

```
Player has L1 (4-hit chain) + H2 (3-hit chain) equipped.

L chain index starts at 0. H chain index starts at 0.
Each L press fires L_chain[L_index] and increments L_index.
Each H press fires H_chain[H_index] and increments H_index.

Example combo: L, L, H, H, H
  → L_chain[0], L_chain[1], H_chain[0], H_chain[1], H_chain[2] (isFinisher=true)
  → Heavy finisher fires → recovery begins
  → Grace window: one final L input allowed → fires L_chain[2] before full lock

Example combo: H, L, L, L
  → H_chain[0], L_chain[0], L_chain[1], L_chain[2]
  → No finisher hit yet; combo window still open for more inputs

Example combo: L, L (with L3, 2-hit chain)
  → L_chain[0], L_chain[1] — light chain exhausted, resets to 0
  → Combo continues; can now press L again (starts L chain over) or press H

Key rules:
  - Heavy chain's last hit (isFinisher=true) always triggers recovery and ends the combo
  - After a heavy finisher, one grace L input is allowed before full recovery locks in
  - Light chains do not have a hard finisher — reaching the end simply resets the L index
  - If COMBO_WINDOW_MS elapses without a new input, both chain indices reset
  - Recovery after any individual hit (non-finisher) is short — this is what creates
    the dodge/parry window; recovery after the heavy finisher is the full recovery lock
```

### Dodge Attacks as Chain Starters

Dodge attacks (DL/DH with direction) fire immediately during dodge i-frames and count as the combo's first hit. After the dodge attack recovers, the L/H chains start from index 0. The dodge attack itself is its own `VariantChain` (type `dodge_light` or `dodge_heavy`, direction-specific).

```
Player side-dodges and presses H → fires DH_side (its own AttackDefinition)
  → after recovery: H chain index = 0, L chain index = 0
  → player can continue with L, L, H, H... as a normal combo from here
```

### Variant Depth Per Element

Each element has **3 light variants (L1–L3)** and **3 heavy variants (H1–H3)**, plus directional dodge variants (DL and DH, each in 3 directions). Chain lengths and individual hit properties differ between variants, creating meaningfully different playstyles within the same element.

Variants are unlocked through progression. L1 and H1 are always available at character creation. Specific variant designs are defined per element in `02-elements-and-characters.md` — not here.

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

### Sustain Hold for Heavy Attacks

Heavy attacks have a long windup (`LOCK_HEAVY_ATTACK_WINDUP`). To block one successfully, the player must **hold block through the entire windup animation** — not just tap at the end. If the player releases block before the attack fires, the block fails and the hit lands.

This creates meaningful mind games: a patient attacker can delay the fire timing slightly (per-attack `windupMs` varies), baiting the defender into releasing block too early. The defender must read the animation, not just react to the impact.

Light attacks have shorter windups — the hold window is brief, and the parry timing is tighter. Heavy windups are longer, giving more time to prepare, but also more time to be baited.

### Parry Timing
- Parry is triggered by pressing block at the **moment of impact** (within `PARRY_WINDOW_MS`)
- The window begins when the attack visually arrives (client shows impact frame)
- Server validates: was block input received within window of hit timestamp?
- This is the main area where latency handling matters — we'll need generous server-side timing

---

## Hit Resolution (Server)

Every tick, server:
1. Advances all projectile positions
2. Checks each projectile head against all player hitboxes
3. For each projectile collision:
   a. Is target DODGING or ROLLING (in i-frames)? → miss
   b. Is target BLOCKING and facing correct direction?
      - Is it a successful parry? → apply PARRY result
      - Normal block: apply `DAMAGE_BLOCKED_MULTIPLIER` damage, drain target stamina
   c. Otherwise: full hit → apply damage, stagger target, apply knockback

4. For each active trail zone (stream attacks):
   a. Is target inside the trail hitbox this tick?
   b. **I-frames do NOT protect against trail damage.** A player who dodges or rolls
      INTO the trail zone still takes `trailDamage` per tick. This is intentional —
      the opponent must dodge perpendicular to or away from the stream's direction,
      not through it.
   c. Is target BLOCKING and facing? → apply `DAMAGE_BLOCKED_MULTIPLIER` reduction
   d. Otherwise: trail damage per tick, no stagger (trail hits are chip, not stagger)

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
