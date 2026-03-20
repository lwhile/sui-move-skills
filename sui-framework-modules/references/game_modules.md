# Runtime Modules

> Covers `sui::clock`, `sui::random`, `sui::event`, `sui::math`

## Table of Contents

- [`sui::clock` — Timestamps](#suiclock--timestamps)
- [`sui::random` — On-Chain Randomness](#suirandom--on-chain-randomness)
- [`sui::event` — Off-Chain Events](#suievent--off-chain-events)
- [`sui::math` — Integer Utilities](#suimath--integer-utilities)

## `sui::clock` — Timestamps

Provides the current network timestamp. Useful for unlock times, TTLs, expirations, and timed workflows.

### The Clock Object

```move
/// Singleton shared object at address 0x6
/// Updated by validators each consensus round
public struct Clock has key {
    id: UID,
    timestamp_ms: u64,  // milliseconds since Unix epoch
}
```

### API

```move
use sui::clock::Clock;

// Get current timestamp in milliseconds
public fun timestamp_ms(clock: &Clock): u64
```

**That's the entire public API.** One function.

### Usage Pattern

```move
// Entry function receives Clock as immutable reference
entry fun claim_if_ready(
    vault: &mut Vault,
    clock: &Clock,
    ctx: &mut TxContext,
) {
    let now = clock.timestamp_ms();
    assert!(now >= vault.unlock_at, EVaultLocked);
    // ... proceed with the claim
}
```

### Important Rules

- Clock is **always passed as `&Clock`** (immutable reference)
- Entry functions accepting `&mut Clock` will be **rejected** by the runtime
- Clock address is always `0x6` — pass it from the client SDK
- Timestamp **resolution**: updated once per consensus commit (~2-3 seconds)
- **Don't use for sub-second timing** — use it for cooldowns (seconds/minutes/hours)

### Testing

```move
#[test]
fun test_cooldown() {
    let mut clock = clock::create_for_testing(ctx);
    clock.set_for_testing(1000); // set timestamp to 1000ms

    // ... test with clock at 1000ms

    clock.increment_for_testing(5000); // advance by 5 seconds

    // ... test with clock at 6000ms

    clock.destroy_for_testing();
}
```

---

## `sui::random` — On-Chain Randomness

Secure, unbiasable randomness from Sui's validator-produced random beacon.

### Architecture

```
Random (shared singleton at 0x8)
    └── RandomGenerator (created per-transaction)
            ├── generate_u64()
            ├── generate_u64_in_range(min, max)
            ├── generate_bool()
            └── shuffle(&mut vector)
```

### Step 1: Create a Generator

```move
use sui::random::{Random, RandomGenerator};

entry fun roll_dice(
    r: &Random,       // pass singleton 0x8
    ctx: &mut TxContext,
) {
    let mut gen = random::new_generator(r, ctx);
    // use gen to produce random values...
}
```

### Step 2: Generate Values

```move
// Unsigned integers
gen.generate_u8(): u8
gen.generate_u16(): u16
gen.generate_u32(): u32
gen.generate_u64(): u64
gen.generate_u128(): u128
gen.generate_u256(): u256

// Ranged integers (inclusive)
gen.generate_u8_in_range(min: u8, max: u8): u8
gen.generate_u16_in_range(min, max): u16
gen.generate_u32_in_range(min, max): u32
gen.generate_u64_in_range(min, max): u64
gen.generate_u128_in_range(min, max): u128

// Boolean
gen.generate_bool(): bool

// Shuffle a vector in place
gen.shuffle<T>(&mut vector<T>)

// Raw bytes
gen.generate_bytes(num_bytes: u16): vector<u8>
```

### Security Rules

> [!CAUTION]
> Randomness-dependent functions are vulnerable to gas manipulation attacks. Follow these rules:

1. **Randomness must be consumed in an `entry` function** — don't expose `RandomGenerator` through `public` functions
2. **Don't make randomness-dependent transfers conditional** — always transfer the result, even if it's a "loss"
3. **Keep the random decision and its side effects in the same `entry` function** where possible to reduce composability risk
4. **Test with `random::create_for_testing(ctx)`**

### Common Pattern: Randomized Outcome

```move
entry fun draw_reward(
    account: &mut Account,
    r: &Random,
    ctx: &mut TxContext,
) {
    let mut gen = random::new_generator(r, ctx);
    let reward_tier = gen.generate_u64_in_range(1, 100);
    let payout = if (reward_tier > 95) {
        1_000
    } else if (reward_tier > 70) {
        100
    } else {
        10
    };
    // apply payout...
    event::emit(RewardDrawn { reward_tier, payout });
}
```

### Testing

```move
#[test]
fun test_random() {
    let mut ctx = tx_context::dummy();
    // Create a test Random object
    random::create_for_testing(&mut ctx);
    // ... retrieve from test scenario, then:
    let mut gen = random::new_generator(&random_obj, &mut ctx);
    let val = gen.generate_u64_in_range(1, 6); // 1..6
    assert!(val >= 1 && val <= 6);
}
```

---

## `sui::event` — Off-Chain Events

Emits structured data for off-chain consumption (indexers, UIs, analytics).

### API

```move
/// Emit an event. The type T is used for indexing.
public native fun emit<T: copy + drop>(event: T)
```

**One function.** The event struct must have `copy + drop`.

### Common Pattern

```move
// Define event struct (copy + drop required)
public struct ValueAdjusted has copy, drop {
    object_id: ID,
    actor: address,
    delta: u64,
    timestamp: u64,
}

// Emit in business logic
public fun apply_update(
    target: &Target,
    actor: address,
    amount: u64,
    clock: &Clock,
) {
    // ... modify state ...
    event::emit(ValueAdjusted {
        object_id: object::id(target),
        actor,
        delta: amount,
        timestamp: clock.timestamp_ms(),
    });
}
```

### Event Best Practices

- Events are **write-only** — Move code cannot read past events
- Events are indexed by their **type** (module + struct name)
- Include enough data for the client to reconstruct what happened
- Include `ID` fields so clients can correlate events to on-chain objects
- Include timestamps when timing matters
- **Phantom type parameters** can be used for filtering: `emit(MyEvent<phantom GameType> { ... })`

---

## `sui::math` — Integer Utilities

Simple integer math helpers.

```move
use sui::math;

math::max(a: u64, b: u64): u64
math::min(a: u64, b: u64): u64
math::diff(a: u64, b: u64): u64     // |a - b| (no underflow)
math::pow(base: u64, exp: u8): u64  // base^exp
math::sqrt(x: u64): u64             // integer square root
math::divide_and_round_up(x: u64, y: u64): u64  // ceil division
```

### Common Usage

```move
// Clamp health to max
let new_health = math::min(current + heal_amount, max_health);

// Safe damage subtraction
let remaining = if (damage >= health) { 0 } else { health - damage };

// Or use math::diff for unsigned difference
let distance = math::diff(pos_a, pos_b);
```
