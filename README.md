# Sui Move Skills

A multi-skill repository for Codex focused on **Sui Move development**.

This project is forked from [Prajeet-Shrestha/sui-move-skills](https://github.com/Prajeet-Shrestha/sui-move-skills), but this fork no longer follows the ECS Game Engine direction of the original repository. It is organized as a reusable Codex skill pack for Sui Move modeling, framework lookup, and engineering review.

## Included Skills

- `sui-move-patterns`
  Modeling guidance for ownership, objects, dynamic fields, collections, visibility, and reusable design patterns.
- `sui-framework-modules`
  Quick reference for exact Sui Framework APIs such as `object`, `transfer`, `dynamic_field`, `coin`, `clock`, `random`, and `package`.
- `sui-engineering-practices`
  Production guidance for upgradeability, gas and protocol limits, error handling, testing, and code review.

## Installation

Codex discovers skills from `${CODEX_HOME}/skills` or `~/.codex/skills`.

Install by copying or symlinking the individual skill folders into your Codex skills directory:

```bash
git clone <your-fork-url> /tmp/sui-move-skills

mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"

ln -s /tmp/sui-move-skills/sui-move-patterns "${CODEX_HOME:-$HOME/.codex}/skills/sui-move-patterns"
ln -s /tmp/sui-move-skills/sui-framework-modules "${CODEX_HOME:-$HOME/.codex}/skills/sui-framework-modules"
ln -s /tmp/sui-move-skills/sui-engineering-practices "${CODEX_HOME:-$HOME/.codex}/skills/sui-engineering-practices"
```

If you prefer copying instead of symlinking:

```bash
cp -R /tmp/sui-move-skills/sui-move-patterns "${CODEX_HOME:-$HOME/.codex}/skills/"
cp -R /tmp/sui-move-skills/sui-framework-modules "${CODEX_HOME:-$HOME/.codex}/skills/"
cp -R /tmp/sui-move-skills/sui-engineering-practices "${CODEX_HOME:-$HOME/.codex}/skills/"
```

## Usage

Example prompts:

- `Use $sui-move-patterns to help me choose between shared objects and owned objects.`
- `Use $sui-framework-modules to look up the exact API for dynamic_object_field.`
- `Use $sui-engineering-practices to review this Sui Move module for upgrade risks and gas issues.`

These skills are designed to complement each other:

- Start with `sui-move-patterns` for architecture and data-model decisions.
- Switch to `sui-framework-modules` when you need exact APIs.
- Use `sui-engineering-practices` for review, hardening, and production checks.

## Repository Structure

```text
sui-move-patterns/
sui-framework-modules/
sui-engineering-practices/
```

Each skill contains:

- `SKILL.md` for trigger metadata and core instructions
- `references/` for detailed docs loaded on demand
- `agents/openai.yaml` for UI-facing metadata
