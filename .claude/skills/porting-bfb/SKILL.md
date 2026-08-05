---
name: porting-bfb
description: Audit the first-pass NearInfinity→Flutter conversion plan with the porting-BFB program — split oracle (Java comprehension / Flutter-MVVM canon / comparison), eight port-failure dimensions, dual-mode adversarial verification, sealed controls; output is a homed register of gaps and corrections to the plan. EXPENSIVE (subagent fleets; confirm cost before fan-out). Trigger ONLY on explicit invocation — /porting-bfb, or a request naming BFB ("run the BFB on the plan", "do the porting BFB", "BFB audit"). NEVER trigger on generic "review the plan" / "check the plan". Optional argument: path to the plan file (default planning/first-pass-conversion-plan.md).
---

# Porting BFB — audit the conversion plan

You are running the **porting instance of the BFB method** (parameterised in
`zenoh_dart_dev/development/reference/bfb-method-and-rationale-20260804.md` §5 — background, not
required reading; everything operative is inline here). The object is a **plan about to be executed
on an unverified analysis** — the method's defining trigger. Your product is **gaps + corrections
to the plan**, never fixes and never seeds.

## Inputs — verify before anything else

1. **The plan:** the argument path, else `planning/first-pass-conversion-plan.md`. If it does not
   exist, STOP and tell the user to run plan mode first and save the plan there — this program
   audits a committed artifact, not a conversation.
2. **The contract:** `context/nearinfinity-port-contract.md` — binding. The per-layer stance
   (faithful data/format layer · rewritten UI), the layer mapping, and the declared deviations
   (Riverpod 3.x, manual, no codegen) govern every verdict. A plan/contract disagreement is itself
   a finding.
3. **The canon:** the rest of `context/` (AI rules, Effective Dart, the MVVM record), plus the
   pinned stores named in the contract (riverpod 3.3.2 et al.).
4. **The source:** `/home/hugo/games/NearInfinity` — read-only, always.

## Invariants — what makes this BFB and not "a careful review"

Sweep by **architectural concern**, not by module · ground the standard **outside** the artifact ·
**blind** the finders so they derive rather than grade · **plant controls** — a clean result from
an unproven instrument is worth nothing · **per-item verdicts**, never grouped · judge the
adversarial pass by **force exerted** · **home** every finding · state **what the pass could not
see**. These are not optional; drop blinding or the controls and the clean rows stop meaning
anything.

**Blinding is imperfect by construction — disclose, don't pretend.** The harness injects this
repo's CLAUDE.md and memory into every subagent's prompt; an agent cannot decline receipt. So
"blinded" means *opened no forbidden file and cited only permitted sources* — instruct every agent
to disclose the injection and ground each claim in its permitted sources alone.

## Cost gate

This program runs subagent fleets (the validation instance cost ~1.5–2M tokens per dimension).
Before Phase 1, present the planned fleet (per-phase agent counts) and get the user's explicit go.

## Phase 1 — the split oracle (three legs; commit each as it lands)

**One agent must never hold both languages** — a finder that reads Java while knowing Flutter
*designs while claiming to describe*, baking its favourite idiom into the map as if it were a fact
about the source. Hence three legs:

- **Leg J — Java-side comprehension.** Finders fan out over **disjoint slices** of the source
  (weight from the contract's measured table: `resource/` is over half the app). They read ONLY the
  Java tree; the NearInfinity repo's own CLAUDE.md is permitted grounding. **Their prompts must not
  name Flutter, Dart, or the port** — grep your drafted prompts for those tokens before firing.
  Product: `audit/oracle-java.md` — subsystem inventories: responsibilities, state, lifecycles,
  data formats read/written, threading, coupling points, with file:line citations.
- **Leg F — the Flutter/MVVM platform oracle.** Established from `context/` + the pinned Riverpod
  3.3.2 sources + official docs. **Never opens the Java tree.** Product: `audit/oracle-flutter.md`
  — what the target platform offers and requires: the MVVM slot rules, the manual-Riverpod idiom
  (which notifier types realize a ViewModel, commands, the provider graph as DI), isolates vs EDT,
  desktop-Linux specifics (file IO, windowing, big-table rendering).
- **Leg C — the comparison pass.** Consumes J + F artifacts only. Product: `audit/mapping.md` —
  the construct map (Java subsystem → MVVM slot, confirming or challenging the contract's table)
  and the **no-equivalent register**: platform couplings with no Flutter counterpart. No-equivalent
  items are identifiable *here and only here* — never by a single-language leg.

## Phase 2 — sealed controls, then the dimensional audit of the plan

**First, seal controls.** From J/C ground truth, the orchestrator privately establishes and
verifies at source: **≥2 omission controls** (subsystems/behaviours provably in the source that the
plan does not mention) and, if cheaply verifiable, **≥1 assertion control** (a plan claim provably
false against the source). Write them to `audit/controls-sealed.md` and **commit BEFORE any finder
fires** — the commit timestamp is what makes the pre-registration auditable. Finders are told
nothing.

**Then fan out one finder per dimension** — each gets the plan + the three oracle artifacts + its
dimension brief; none sees another's output. Each finder works in **dual mode**:

- **Mode A — attack assertions:** every plan claim touching its dimension gets a per-item verdict
  (HOLDS / FALSE / UNVERIFIABLE) with source citations.
- **Mode B — hunt omissions:** everything its dimension covers in the ground truth that the plan
  never mentions.

**The eight dimensions** (how ports fail — not the validation instance's seven):

1. **Domain & data** — formats, structs, encodings, the read/write paths. Verdicts judged against
   the FAITHFUL stance: round-trip byte-identity is the bar.
2. **Observable behaviour** — what the app does that a user or a saved file can see; edge
   semantics (override shadowing, per-game format branching on `Profile`).
3. **State & lifecycle** — global singletons, caches and their invalidation (`clearCache()`),
   session/game-switch lifecycles, who owns what when.
4. **Integration boundaries** — filesystem contracts (case-insensitive resolution, DLC zips as
   filesystems), external tools, the game installation itself.
5. **UI structure & navigation** — screens, windows, tools, workflows. Judged against the
   REWRITE stance: feature/workflow parity, never widget fidelity.
6. **Platform coupling with no Flutter equivalent** — seeded by leg C's no-equivalent register;
   the plan must have an answer for every entry.
7. **Build / config / release** — Java 8+ant+fat-jar → Flutter desktop packaging; settings
   migration (`java.util.prefs`); update mechanism.
8. **What the tests reveal about intent** — upstream has NO test suite (measured); the dimension
   audits what the manual verification culture implies, and whether the plan builds the missing
   verification (fixtures, round-trip harnesses) rather than assuming it exists.

## Phase 3 — adversarial verification (dual-mode here too)

Independent verifiers, blind to Phase-2 reasoning: **refute** each finding by re-tracing it at
source (per finding: CONFIRMED / REFUTED / IMPRECISE, with the corrected form) — and re-trace a
**sample of clean verdicts**, not only the findings. Separately, fresh **omission-hunters** attack
what both the plan AND the finders missed, from the oracle artifacts alone.

**Check the controls now:** a sweep that missed a sealed control is a broken instrument — re-fire
the affected finder with the gap named, and say so in the synthesis. **Judge the verification by
force exerted:** zero refutations/corrections across the board is a rubber stamp — re-fire it.

## Phase 4 — synthesis: the register

Product: `audit/register.md` — one item per row, no grouped dispositions:

`ID · dimension · type (GAP / CORRECTION / RISK / NO-EQUIVALENT) · the claim, one sentence ·
evidence (file:line + oracle cite) · verification verdict · HOME`

**HOME every item**: a named plan section to amend · a future seed candidate · a contract
amendment · or a user ruling (no-equivalent resolutions are ALWAYS the user's ruling, never the
register's). End with **"what this pass could not see"** — honestly, including the blinding
disclosure and any control that had to be re-fired. Commit, then report to the user: the register
headline, the items awaiting their ruling, and nothing else pre-digested — they read the artifact.

## What this skill never does

Fix code · edit the plan (it outputs corrections; the user applies them via plan revision) · cut
seeds (the seed-writer skill's job) · write into `/home/hugo/games/NearInfinity` · run without the
cost gate.
