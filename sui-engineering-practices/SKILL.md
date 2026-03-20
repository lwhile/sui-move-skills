---
name: sui-engineering-practices
description: Use when reviewing or hardening Sui Move code for upgradeability, gas limits, error handling, testing, and code quality.
---

# Sui Engineering Practices

Best practices for building production-quality Sui Move code. Use this skill for upgrade safety, gas and protocol limits, error handling, testing, and code review.

This skill is intentionally about production rigor, not object modeling or exact framework APIs. For data-model and visibility design, switch to `$sui-move-patterns`. For module signatures and framework calls, switch to `$sui-framework-modules`.

Use this for prompts like:
- "Review this Sui Move module for upgrade risks."
- "Will this hit protocol limits or cost too much gas?"
- "Are the error codes, tests, and public APIs production-safe?"

> **Source**: Distilled from [The Move Book — Guides](https://move-book.com/guides) and Sui ecosystem conventions.

## Quick Decision Matrix

### "I'm making a publish or upgrade decision"

| Situation | Recommended Move | Why |
|-----------|------------------|-----|
| A struct's fields may need to evolve later | Keep the public surface minimal or anchor changing data in dynamic fields | Published public fields are locked |
| A shared object may outlive several package versions | Add a version guard | Prevent old objects from silently drifting out of contract |
| A config layout may need to change | Use a stable anchor object plus extensible fields | Preserve upgrade compatibility |

### "I'm hitting a limit"

| Limit | Value | Workaround |
|-------|-------|-----------|
| Transaction size | 128 KB | Batch operations across multiple PTB commands |
| Object size | 256 KB | Use dynamic fields (Bag/Table) for overflow |
| Pure argument size | 16 KB | Join vectors dynamically in Move or PTB |
| Objects created per tx | 2,048 | Batch across transactions |
| Dynamic fields created per tx | 1,000 | Batch across transactions |
| Dynamic fields accessed per tx | 1,000 | Minimize reads per operation |
| Events emitted per tx | 1,024 | Aggregate events, emit summaries |

### "I need to handle an error"

| Pattern | When to Use |
|---------|-------------|
| `assert!(condition, EErrorName)` | Guard at function entry |
| `if (!check) return/abort` | Non-fatal early exit |
| Return `bool` from checks | Let callers handle errors their way |
| Unique error constant per check | Always — never reuse codes |

### "I'm reviewing code quality"

| Need | Reference |
|------|-----------|
| Naming, imports, and style consistency | `code_quality.md` |
| Test layout and expected-failure conventions | `code_quality.md` |
| Limit-aware data layout or batching decisions | `gas_and_limits.md` |
| Upgrade compatibility and versioning checks | `upgradeability.md` |

## Reference Files

| File | What It Covers |
|------|---------------|
| [upgradeability.md](./references/upgradeability.md) | Open when designing for future package upgrades and shared-state compatibility |
| [gas_and_limits.md](./references/gas_and_limits.md) | Open when sizing transactions, objects, dynamic fields, and gas-heavy flows |
| [error_handling.md](./references/error_handling.md) | Open when defining abort codes, guards, and caller-visible checks |
| [code_quality.md](./references/code_quality.md) | Open when reviewing naming, imports, APIs, macros, and tests |
