# near_infinity_flutter

A work-in-progress port of [Near Infinity](https://github.com/Argent77/NearInfinity) — the
Java/Swing browser and editor for Infinity Engine game data (Baldur's Gate, Icewind Dale,
Planescape: Torment and their Enhanced Editions) — to **Flutter on Linux desktop**.

> **Status: pre-implementation.** There is no application code here yet, and nothing builds or
> runs. What exists is the groundwork: the pinned target-platform canon, the project contract that
> governs how the port is done, and the audit program that will check the conversion plan before
> any code is written. This is an independent port, not affiliated with or endorsed by the
> upstream project.

## What's in the repo

| Path | What it is |
|---|---|
| `context/nearinfinity-port-contract.md` | **Start here.** The operative contract: the per-layer stance, the layer mapping, and every declared deviation from the canon. |
| `context/flutter-ai-rules.md`, `context/effective-dart-style.md` | Pinned copies of the official Flutter AI rules and the Dart style guide, with provenance headers (source, upstream commit, pin date). |
| `context/mvvm-architecture-record.md` | The MVVM layering the app targets — View / ViewModel / Repository / Service — distilled from Flutter's official architecture guidance. |
| `context/java-semantics-notes.md` | A citation ledger of the Java platform contracts the port's correctness depends on (byte order, unsigned emulation, character encodings, threading model, and so on). |
| `.claude/skills/porting-bfb/` | The plan-audit program described below. |

## The approach

The port is **faithful in one half and a rewrite in the other**, and the split is deliberate:

- **The data and format layer is faithful.** Near Infinity is an *editor* — it writes files back
  into a real game installation, so a divergence in a writer silently corrupts someone's game.
  Correctness here is checked by **round-trip byte identity**: read a real game file, write it
  back, compare bytes.
- **The user interface is a rewrite.** Swing and Flutter differ structurally enough that a
  faithful translation is not meaningful; the goal is feature and workflow parity, not widget
  fidelity. In particular the upstream core model doubles as a Swing table model, and that fusion
  has to be severed for an MVVM target.

Development proceeds **from the model outward to the UI**. State management is Riverpod 3.x with
manually declared providers and notifiers — no code generation.

## How the work is sequenced

Rather than starting from a plan and trusting it, this project audits the plan first:

```
plan mode  →  a first-pass conversion plan (committed, not a chat)
   →  the porting-BFB audit of that plan
      →  gaps and corrections
         →  seeds, one at a time
            →  test-driven implementation
```

The audit (`/porting-bfb`) is a dimensional sweep: independent agents establish what the Java
source does and what the Flutter platform offers **separately** — so that no single reader maps
one onto the other while believing they are merely describing it — and a comparison pass produces
the register of things that have no counterpart on the target. Findings are then attacked in two
directions, because a plan can be wrong by asserting something false *and* by staying silent about
something real.

The methodology is not specific to this project; it is one instance of a general audit program.

## Upstream and licensing

The Java source is used **read-only**; nothing in this project modifies it. Upstream
[Near Infinity](https://github.com/Argent77/NearInfinity) — copyright (C) 2001 Jon Olav Hauglid
and contributors — is licensed under the **GNU Lesser General Public License, version 2.1**.

A port is a translation, and a translation is a derivative work, so **this project is licensed
under LGPL-2.1 as well**. The full text is in [`LICENSE`](LICENSE).

Two clarifications, because a blanket claim would be inaccurate:

- The pinned third-party documentation under `context/` is **not** covered by this project's
  license. Those files are reproductions of Flutter and Dart documentation, carry their upstream
  provenance in each file's header, and remain under their own terms.
- Everything else here — the contract, the semantics ledger, the audit skill, this README — is
  original work of this project, offered under the same LGPL-2.1 for simplicity and to keep the
  repository under a single license as ported code arrives.

## Development

Built with [Claude Code](https://claude.com/claude-code); `CLAUDE.md` and `.claude/skills/` carry
the working rules and the audit program. Dart and Flutter are invoked through
[fvm](https://fvm.app/).
