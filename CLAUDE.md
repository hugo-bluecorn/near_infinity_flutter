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
`reference/README.md` documents local read-only reference snapshots (IESDP format docs — content
uncommitted, **informative only**: where IESDP and the Java source disagree, the Java source wins).

## Hard rules

- **The Java source is read-only:** `/home/hugo/games/NearInfinity` (Argent77/NearInfinity,
  `devel`). Never modify it from this project.
- **Dart/Flutter are NOT on PATH** — use `fvm flutter ...` / `fvm dart ...` for every command.
- **Architecture:** full MVVM per `context/mvvm-architecture-record.md`; state management is
  **Riverpod 3.x** (a declared deviation — see the contract). Development proceeds **model → UI**.
- **Faithful half = round-trip byte-identity.** Any change to format read/write code carries a
  round-trip test against real game data fixtures; byte-exact, over the format's domain.
- **Every source file carries the LGPL header** — the exact template is in the contract's
  Licensing section; ported files keep upstream's copyright alongside the port's. Check a new
  dependency's license before adding it: LGPL-2.1 compatibility is required, and Apache-2.0 is
  not compatible.
- **No multi-role machinery here.** This project runs from the main session with the tdd-workflow
  plugin (`/tdd-plan`, `/tdd-implement`); the porting-BFB and seed-writer are skills in this repo.

## Current stage

Pre-plan. The context package exists and **`/porting-bfb` is live** (build order: method
parameterisation ✅ → porting instance ✅ → seed-writer, built last). The working sequence:

1. Plan mode → the first-pass conversion plan → save it to
   `planning/first-pass-conversion-plan.md` (the BFB audits a committed artifact, not a chat).
2. `/porting-bfb` — the audit; output lands in `audit/` as a homed register of gaps + corrections.
3. Apply corrections via plan revision; seeds are then cut **one at a time** (seed-writer skill).

**The app scaffold exists** (`flutter create -e`, empty app, SDK pinned to 3.44.8 in `.fvmrc`) —
platform plumbing only. **`lib/` is still the plan's to decide:** the MVVM folder layout, the
Riverpod provider graph, and every feature boundary come from the first-pass plan and its audit,
not from improvisation on top of the scaffold. `lib/main.dart` is Flutter's empty-app stub and is
expected to be replaced.

## Build & run

```bash
fvm flutter pub get
fvm flutter run -d linux      # the target platform
fvm flutter analyze
fvm flutter test              # no tests yet — the plan builds the harness
```
