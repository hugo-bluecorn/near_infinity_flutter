# CLAUDE.md

This file guides Claude Code sessions in this repository.

## What this repo is

The **NearInfinity → Flutter/Linux-desktop port**: the porting program's artifacts now, the Flutter
application as it comes into existence. The port is governed by the **porting-BFB program** — plan
first, audit the plan, then seeds.

**Start here: `context/nearinfinity-port-contract.md`** — the operative contract (per-layer stance,
layer mapping, declared deviations, pipeline). The `context/` folder is the pinned target-side
canon: the Flutter AI rules, Effective Dart style, and the MVVM layering record. Where code or a
plan disagrees with `context/`, the disagreement is either a recorded deviation or a defect.

## Hard rules

- **The Java source is read-only:** `/home/hugo/games/NearInfinity` (Argent77/NearInfinity,
  `devel`). Never modify it from this project.
- **Dart/Flutter are NOT on PATH** — use `fvm flutter ...` / `fvm dart ...` for every command.
- **Architecture:** full MVVM per `context/mvvm-architecture-record.md`; state management is
  **Riverpod 3.x** (a declared deviation — see the contract). Development proceeds **model → UI**.
- **Faithful half = round-trip byte-identity.** Any change to format read/write code carries a
  round-trip test against real game data fixtures; byte-exact, over the format's domain.
- **No multi-role machinery here.** This project runs from the main session with the tdd-workflow
  plugin (`/tdd-plan`, `/tdd-implement`); the porting-BFB and seed-writer are skills in this repo.

## Current stage

Pre-plan. The context package exists; the porting-BFB skill is being built (build order: method
parameterisation ✅ → porting instance → seed-writer). The Flutter app is **not created yet** — the
first-pass plan decides its structure; do not `flutter create` ahead of it.
