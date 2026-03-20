---
name: sui-move-patterns
description: Use when modeling Sui Move objects, ownership, dynamic fields, collections, visibility, and reusable design patterns.
---

# Sui Move Patterns

Language-level patterns for building on Sui Move. Use this skill when deciding **how to model data and APIs**: ownership, object shape, dynamic storage, collection choice, visibility, and reusable design patterns.

If you need exact framework signatures, switch to `$sui-framework-modules`. For upgradeability, limits, testing, and review guidance, switch to `$sui-engineering-practices`.

Use this for prompts like:
- "How should I model this object on Sui?"
- "Should this be a dynamic field, table, or plain struct field?"
- "What pattern should I use for admin control or one-time init?"

> **Source**: All content is distilled from [The Move Book](https://move-book.com/) and the [Sui Framework reference](https://docs.sui.io/references/framework/sui_sui).

## Quick Decision Matrix

Use this when you need to choose a pattern:

### "How should I store this data?"

| Situation | Use | Why |
|-----------|-----|-----|
| Fixed fields known at compile time | Regular struct fields | Cheapest, fastest |
| Unknown/extensible fields at runtime | `dynamic_field` | Lazy loading, no schema migrations |
| Storing objects that need off-chain ID lookup | `dynamic_object_field` | Preserves object identity (2x cost) |
| Small set of unique items (<100) | `VecSet` | In-memory, object-local |
| Small key-value map (<100) | `VecMap` | In-memory, object-local |
| Large key-value map (100+) | `Table` | Dynamic field-backed, no size limit |
| Heterogeneous collection | `Bag` | Dynamic field-backed, mixed types |

### "How should I control access?"

| Situation | Use |
|-----------|-----|
| Admin-only operations | Capability pattern (`AdminCap` object) |
| Prove package identity | Witness pattern (struct from module) |
| Ensure multi-step completion | Hot Potato (no-ability struct) |
| A caller should exclusively own and mutate data | Single-owner objects (transfer to sender) |
| Shared state or a registry should be accessible by many callers | Shared objects (`share_object`) |

### "What visibility should this function have?"

| Visibility | Who Can Call | Use For |
|------------|------------|---------|
| (none) | Same module only | Internal helpers |
| `public(package)` | Same package | Package-internal coordination helpers |
| `public` | Any package | Package API, reusable utilities |
| `entry` | Transaction only (not other Move) | Top-level transaction endpoints |

## Reference Files

| File | What It Covers |
|------|---------------|
| [object_model.md](./references/object_model.md) | Open when choosing ownership, shared vs owned objects, or fast path vs consensus |
| [abilities_and_generics.md](./references/abilities_and_generics.md) | Open when deciding `key`/`store`/`copy`/`drop` and generic constraints |
| [dynamic_fields.md](./references/dynamic_fields.md) | Open when deciding between struct fields, `dynamic_field`, and `dynamic_object_field` |
| [collections.md](./references/collections.md) | Open when choosing between vector-style and dynamic-field-backed collections |
| [design_patterns.md](./references/design_patterns.md) | Open when choosing Capability, Witness, OTW, Hot Potato, or Wrapper patterns |
| [api_design.md](./references/api_design.md) | Open when designing visibility, receiver syntax, `init`, enums, and naming |
