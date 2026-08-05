# Java semantics notes — the citation ledger

**Status:** operative ledger (seeded 2026-08-05). This is the Java-side instrument — *what the code
means*. It exists because the faithful half's correctness hinges on Java **platform contracts**,
and such claims must rest on citations, not model memory (*a signature is not a mechanism; neither
is a remembered default*).

**Protocol:** entries grow evidence-driven — seeded here as known-critical, each **verified at
first touch** by whichever leg or seed reaches it first: fetch the official citation (JLS /
Javadoc URL + date), pin the NearInfinity sites (file:line), state the Dart-side counterpart, flip
the status. A SEEDED entry is a hypothesis; only VERIFIED entries may be cited by verdicts.
Add new entries the moment a mapping decision leans on an uncited platform contract.

| # | Contract | Bites in NearInfinity | Dart-side counterpart | Status |
|---|---|---|---|---|
| 1 | **Byte order** — `ByteBuffer` defaults to BIG_ENDIAN; Infinity Engine formats are little-endian, so codecs must (and do) set `ByteOrder.LITTLE_ENDIAN` | **32 measured `LITTLE_ENDIAN` sites** (2026-08-05 grep) — `resource/key/BIF{,F}Reader.java`, `Effect{,2}.java`, `wed/WedResource.java`, … | `ByteData` get/set take `Endian` **per call**, default `Endian.big`; typed-data *views* use host order — a per-call-site hazard, not a one-time setting | SEEDED |
| 2 | **No unsigned types** — Java emulates unsigned bytes/shorts/ints by masking (`& 0xFF`, …); sign-extension on widening is the trap | the codec family throughout `resource/` + `datatype/` (`DecNumber`, `Unknown`, section counts) | Dart `int` is 64-bit; masks still required but at different widths; `Uint8List`/`ByteData` getUint* solve the byte layer | SEEDED |
| 3 | **Charsets** — which encoding per string field is a per-format contract, not a global | **32 files** touch `Charset`/encodings (2026-08-05 grep); resource names, TLK strings | ⚠️ **Dart has no built-in cp1252 codec** — `latin1` ≠ cp1252 for 0x80–0x9F; if any format field is cp1252, the port needs an explicit table or package | SEEDED |
| 4 | **The EDT** — Swing is single-threaded; all UI mutation via the Event Dispatch Thread (`invokeLater`/`invokeAndWait`); background work via workers/pools | `util/Threading` pooled executor; `AbstractSearcher`/`AbstractChecker` batch ops; every viewer | Flutter main isolate + `compute()`/isolates; no shared-memory UI mutation at all — a *stronger* model, but blocking work must leave the main isolate explicitly | SEEDED |
| 5 | **`java.util.prefs`** — platform backing store (Linux: `~/.java/.userPrefs` XML), node paths are API | `AppOption` registry; the legacy node `org.infinity.gui.BrowserMenuBar` kept deliberately for settings compat | no direct equivalent; settings file or `shared_preferences`; **user-settings migration is a plan-level question** | SEEDED |
| 6 | **Zip as `FileSystem`** — NIO mounts zips and serves `Path`s through them transparently | `DlcManager` mounts DLC zips; `ResourceEntry` reads through | Dart has no zip-mount; `package:archive` reads entries — the transparent-Path abstraction must be rebuilt or designed around | SEEDED |
| 7 | **Case-insensitive path resolution over a case-sensitive FS** — game data is DOS-cased; Java code resolves via a custom layer, not the platform | `util/io/FileManager` + `CaseAwarePathResolver` — the mandatory path hub | same problem, same answer: a custom resolution layer; nothing in `dart:io` provides it | SEEDED |

## Entries verified so far

None — all seven are SEEDED. First verification belongs to whichever leg-J finder, comparison
item, or seed touches the contract first; the verifier updates the row in place (citation URL +
fetch date + measured sites) and moves it below with a dated line.
