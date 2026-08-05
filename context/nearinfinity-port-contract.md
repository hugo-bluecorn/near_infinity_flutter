# The NearInfinity port — project contract

**Status:** operative contract for `near_infinity_flutter` (2026-08-05). This document records the
decisions that govern the port; the sibling files in `context/` are the pinned canon it cites.
Where a plan, seed, or implementation disagrees with this contract, the disagreement is a finding.

## What this project is

A port of **Near Infinity** — the Java/Swing browser/editor for Infinity Engine game data — to a
**Flutter application for Linux desktop**, built as a full **MVVM** application per the target-side
canon in this folder, developed **model → UI**, with the porting-BFB program auditing the plan
before seeds are cut.

- **Source (read-only):** `/home/hugo/games/NearInfinity` — clone of Argent77/NearInfinity,
  `devel`. Never modified by this project.
- **Measured 2026-08-05:** 1230 Java files, ~297k LOC. `resource/` 863 files/167k · `gui/` 172/70k
  · `util/` 83/24k · `datatype/` 55/9k · `search/` 28/15k · `check/` 16/5k. **No test suite
  upstream.** 339 files import Swing; the core model (`AbstractStruct`) *is* a Swing `TableModel`.
  It is an **editor** — real save paths; a wrong writer corrupts a user's game installation.

## The target-side canon (this `context/` package)

| File | What it governs | Pin |
|---|---|---|
| `flutter-ai-rules.md` | style, quality, state-mgmt defaults, theming, lints | flutter/flutter `docs/rules/rules_10k.md` @ store `36b35dd79ff` (2026-07-16) |
| `effective-dart-style.md` | Dart naming/ordering/formatting | dart-lang/site-www @ store `fccd24a0` (2026-06-20) |
| `mvvm-architecture-record.md` | the layering: View/ViewModel/Repository/Service rules | docs.flutter.dev/app-architecture/guide, fetched 2026-08-05 |
| `java-semantics-notes.md` | the Java-side instrument — the platform-contract citation ledger the faithful half hinges on | seeded 2026-08-05; entries verified at first touch (JLS/Javadoc cites) |
| this file | project decisions + deviations | — |

Oracle-leg source pins (outside this folder, in the ai-context store): `rrousselGit/riverpod` @
`2fa6ad2a` (**3.3.2**, snapshot 2026-06-13).

## The per-layer stance (ruled by Hugo, 2026-08-05)

The faithful-vs-rewrite question is **per-layer, not app-global**:

- **Data/format layer — FAITHFUL.** The `resource/` codecs, `resource/key` access
  (chitin.key/BIFF/override resolution), the struct/datatype model, BCS compile/decompile.
  Behavioural divergence is a bug. **Acceptance instrument: round-trip byte-identity** — read a
  real game file, write it back, compare bytes. Adversarial fixtures over the contract's domain,
  not today's samples.
- **UI layer — REWRITE.** Swing→Flutter forces structural redesign (EDT vs isolates; the
  `AbstractStruct`-is-the-`TableModel` fusion is severed). Parity here means **feature and
  workflow parity**, tracked by the gap register — never structural fidelity.

**The layer mapping** (measured structure → MVVM slots):

| NearInfinity (Java) | MVVM slot | Stance |
|---|---|---|
| `resource/` codecs + `resource/key` access | **Services** | faithful |
| `Profile`, `ResourceFactory`, the caches | **Repositories** | faithful semantics, restructured shape |
| per-resource editing state (ex-`TableModel`) | **ViewModels** | rewritten |
| Swing frames, viewers, converters UI | **Views** | rewritten |

## Declared deviations from the canon

1. **State management: Riverpod 3.x** (Hugo, explicit request, 2026-08-05). This exercises the
   rules file's own clause — *"do NOT use Riverpod, Bloc, or GetX **unless explicitly
   requested**"* — and supersedes its native-first default and its manual-constructor-DI bullet
   (the provider graph is the DI). **Sub-decision ruled (Hugo, 2026-08-05): NO code generation** —
   manual notifier/provider declarations only; the annotation/`riverpod_generator`/`build_runner`
   toolchain is excluded. The oracle leg establishes the mapping against the manual style.

No other deviations are declared. A deviation discovered in code that is not recorded here is a
defect.

## Licensing (settled 2026-08-05)

Upstream Near Infinity is **LGPL-2.1**. A port is a translation and therefore a derivative work,
so **this project is LGPL-2.1** (`LICENSE`, verbatim FSF text). Consequences that bind the work:

- **Copyright attribution for this project's work: `hugo-bluecorn`** (ruled 2026-08-05).
- Ported source files carry **both** copyrights — LGPL-2.1 §1 requires a derivative work to keep
  the original notices intact, and the port adds its own. Use this header verbatim on every
  ported file:

  ```dart
  // near_infinity_flutter — a Flutter port of Near Infinity
  // Copyright (C) 2001 Jon Olav Hauglid and contributors (upstream, LGPL-2.1)
  // Copyright (C) 2026 hugo-bluecorn (port)
  // See LICENSE for license information
  ```

  Files that are **original** to this project — no upstream derivation — carry only the second
  copyright line. Judging which is which is a per-file call the implementer makes and the
  reviewer checks; when in doubt, include both.
- Any dependency added to the app must be license-compatible with LGPL-2.1. **Apache-2.0 is not**
  — its patent-termination clause conflicts with GPLv2-era licenses — so an Apache-2.0 package
  cannot be linked into the shipped application. Check a package's license before adding it.
- The pinned third-party documentation under `context/` stays under its own upstream terms and is
  not relicensed by this file.

## Scope

**Deliberately not stated here.** The first-pass conversion plan (plan mode) must state scope —
whole-app vs subset, which resource types, which tools — and the porting-BFB audit attacks that
statement (assertions *and* omissions). Recording scope in this contract before the plan exists
would pre-empt exactly the check the program exists to run.

## The pipeline

```
plan mode → first-pass conversion plan (names each Java subsystem's MVVM slot)
   → porting-BFB audits the plan (refutes assertions AND hunts omissions)
      → gaps + corrections
         → seeds, one at a time (seed-writer skill; premise re-verified at HEAD)
            → /tdd-plan → /tdd-implement
```

Constraint: no multi-role/mailbox machinery — the program is self-contained skills invoked from the
main session, with tdd-workflow doing plan/implement.
