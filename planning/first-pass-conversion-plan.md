# First-pass conversion plan — NearInfinity → Flutter/Linux

**Status:** first-pass plan, 2026-08-05 — the artifact the porting-BFB audits. Where this plan
disagrees with `context/nearinfinity-port-contract.md`, the contract wins and the disagreement is a
finding.

**Source pin:** `/home/hugo/games/NearInfinity` (Argent77/NearInfinity, `devel`) @
`b5a3b4b2e4432af847e78560d999d77af391e136` (2026-07-25, "Cosmetic"), app version `v2.4-20260616`
(`src/nearinfinity.properties`). Measured: 1230 Java files, ~297k LOC. Read-only.

**Target pin:** Flutter SDK 3.44.8 (`.fvmrc`), Dart `^3.12.2` (`pubspec.yaml`), Linux desktop only,
Riverpod 3.x manual (no codegen) per the contract's declared deviation; Riverpod oracle sources
pinned at 3.3.2.

**Reference (informative, not normative):** IESDP — the Infinity Engine Structures Description
Project — pinned locally at `reference/iesdp-gh-pages/` (gh-pages @ `f820d4e5`, tree 2026-07-03;
provenance + hash in `reference/README.md`). Per-format/per-version structure pages
(`file_formats/ie_formats/`), effect-opcode docs (`opcodes/`), scripting actions/triggers
(`scripting/`). **Precedence rule:** the fidelity target is NearInfinity's measured behavior at
the pinned SHA — where IESDP and the Java source disagree, the Java source wins and the
disagreement is recorded (an IESDP gap or an upstream-bug candidate). IESDP's roles: independent
cross-check when reading Java codecs, authoring source for synthetic fixtures (§7.6), and input
to the audit's comprehension legs.

**How this document is addressed** (for the audit): inventory rows carry `INV-*` keys, phases
`P0…P16`, assumptions `A1…A12`, comparator classes `C1…C4`, gap-register seeds `G1…G14`, open
questions `Q1…Q9`. Findings should home to these keys.

---

## 1. Scope — the whole application

The contract fixes the denominator: **every resource type, every tool, every workflow**. This
section enumerates what that denominator actually contains, measured against the source. Everything
listed here appears again in the inventory (§2), the mapping (§3), and a phase (§5). Nothing is
silently dropped; deliberate replacements are declared in the gap-register seed (§8).

### 1.1 Resource-type denominator

`ResourceFactory.getResourceType()` (ResourceFactory.java:192–286) dispatches **52 extensions**:

> BAM, TIS, BMP, PNG, MOS, WAV, ACM, MUS, IDS, 2DA, BIO, RES, TXT, LOG, SRC (IWD2→text), LUA,
> SQL, GUI, MENU, GLSL, INI, MVE, WBM, PLT, BCS, BS, ITM, EFF, VEF, VVC, SRC (PST→struct), DLG,
> SPL, STO, WMP, CHU, CRE, CHR, ARE, WFX, PRO, WED, GAM, SAV, VAR, BAF, TOH, TOT, PVRZ, FNT,
> TTF, MAZE, VMAP

Plus, outside that chain and still in scope:

- **KEY / BIF / BIFC / BIFF** — the archive layer (`resource/key/`), including *writers* for both
  KEY (`Keyfile.write`, Keyfile.java:415) and BIFF (`BIFFWriter`, all three variants).
- **TLK** — `dialog.tlk` / `dialogf.tlk`, handled in `util/StringTable` (read **and write**), plus
  WeiDU `.tra` import/export.
- **SET** — appears only in the export-time decrypt list (ResourceFactory.java:1190–1194).
- **BIK** — listed by `Profile.getAvailableResourceTypes()` but has **no handler** upstream; falls
  to `UnknownResource`. The port reproduces exactly that (no BIK decoder).
- **Unknown** — any unrecognized file: hex view + raw save (`other/UnknownResource`, `Writeable`).
- Signature sniffing for extension-less/mislabelled files (`detectResourceType`, ~30 signatures).

### 1.2 Game denominator

`Profile.Game` — **19 games** over **7 engines** (`Unknown, BG1, BG2, PST, IWD, IWD2, EE`):
Unknown, BG1, BG1TotSC, BG2SoA, BG2ToB, Tutu, BGT, PST, IWD, IWDHoW, IWDHowTotLM, IWD2, IWD2EE,
BG1EE, BG1SoD, BG2EE, IWDEE, PSTEE, EET — plus mod-presence probes (TobEx, EEEx) that alter
behavior. 91 `Profile.Key` properties; ~80 `IS_SUPPORTED_*` capability keys form the per-game
format-version matrix.

### 1.3 Tool denominator

From the menu census (gui/menu/*) and window census (gui/*): 15 checkers, 8 search entry points
(+ text search over 6 types), 6 converters, mass exporter, string editor, BIFF editor, SAV
manager, area viewer, DLG tree viewer, worldmap viewer, creature animation browser, hex editor,
sound player + InfinityAmp music player, MVE player, IDS browser, script drop zone, SPLPROT
converter, random spell generator (`spl/generator`), WeiDU changelog, version-check updater,
bookmarks + game launcher, quick search, preferences (~90 options), clipboard viewer, debug
console, game properties, "new resource" for 21 types.

### 1.4 Workflow denominator

Open/detect game (auto-detect + forced type + CLI path) · browse tree (override display modes,
back/forward history) · view / edit / save with backup-on-save and BIFF→override redirection ·
export raw and converted (per-viewer menus) · mass export with conversion matrix · create new
resource (21 types) · add-copy / rename / delete / restore-backup on override files · search
(references ×6 target kinds, attribute, text, advanced with saved XML filters, extended-legacy) ·
check (15 checkers incl. TLK repair) · convert (BAM/BMP/MOS/PVRZ/TIS/TIS-batch) · edit TLK +
TRA export/import · BIFF repack · SAV pack/unpack · zip a save folder · WeiDU changelog · launch
game binary · bookmarks/recents · drag-drop open · struct clipboard cut/copy/paste ·
open-in-new-window · preferences · update check. All in scope; single-window consequences in G1.

### 1.5 Explicitly replaced or dropped (each homed in §8)

Swing look-and-feel machinery (52 FlatLaf themes) → Material theming (G2) · Windows-only
`InputKeyHelper` and macOS `com.apple.eawt` glue → dropped, Linux-only target (G3) · JVM
memory-usage status widget → process-RSS equivalent (G4) · Java `Preferences` store → new settings
store, no migration (G5) · bundled third-party UI libs (FlatLaf, RSyntaxTextArea, JHexView,
JFontChooser) → Flutter-native rewrites (G6) · MonteMedia AVI export → original RIFF/AVI muxer,
no MonteMedia derivation (G7) · in-app WBM playback — upstream has none either; parity is
raw-export (not a gap).

### 1.6 Independent cross-check — IESDP vs this denominator (run 2026-08-05)

The pinned IESDP snapshot (header, `reference/README.md`) was diffed against §1.1–§1.2 and the
§2.6 version matrix. Headline: **no engine format documented by IESDP is missing from this plan,
and no plan format is unknown to IESDP** beyond one-line stubs (PNG, TTF, WBM, GUI, MENU, GLSL,
SQL, LUA, BIO, RES, BAF are unlinked `general.htm` rows) and five site-wide absences (VMAP, BIK,
SET, LOG, TXT) — all of which are NI-side realities this plan already carries. The deltas, with
positions (precedence rule: NI wins, deltas recorded):

| Finding | Evidence | Position |
|---|---|---|
| NI accepts CRE version strings IESDP doesn't attest: `"V1.1"` (V1.2 layout) and `"V9.1"` (V9.0 layout) | `CreResource.java:568,572`; no `cre_v1.1` page; `cre_v9.htm` says `'V9.0'` only | Keep both (NI-attested); fixture matrix gains both labels |
| GAM V2.1: IESDP's index links `gam_v2.1.htm` but the page does not exist (dead link) | `file_formats/index.htm:78` | Keep V2.1 (Profile `GAM_V21` + engine branch); IESDP gap |
| TIS: IESDP documents palette *and* PVRZ variants under one `'V1  '` version string (PVRZ signalled by tile-size field = 12); no headerless-in-BIFF mention | `tis_v1.htm` | Keep NI handling incl. header synthesis (`AbstractBIFFReader.getTisHeader`); note "TIS V1/V2" is Profile-key vocabulary, not on-disk strings — fixture names follow the on-disk facts |
| SRC: IESDP scopes SRC to PST/PSTEE only; NI additionally routes IWD2 SRC to text | `src.htm` vs `ResourceFactory.java:210` | Keep (behavioral port) |
| PVRZ: IESDP names DXT1/DXT5 only; NI ships ETC2/PVRTC decode paths too | `pvrz.htm` vs `graphics/decoder/` | Keep the decoders |
| BIF naming collision: IESDP calls whole-file-zlib "BIFC (IWD1)" (sig `'BIF '` V1.0) and block-zlib "BIFC 1.0 (BG2)"; also names `.cbf` as the on-disk extension | `bif_v1.htm`, `general.htm` | Plan keeps **signature-based** vocabulary (`BIFF`/`BIF `/`BIFC`); fixtures named by signature; `.cbf` noted |
| BMP: IESDP splits classic vs **v5** (BITMAPV5HEADER, EE 32-bit+alpha) | `bmp.htm`, `bmp_v5.htm` | Matches NI `BMP_PAL`/`BMP_ALPHA`; fixture matrix gains a v5-header case |
| EE `.wav`/`.acm` are Ogg Vorbis containers; WAVC V1.0 = ACM + header | `general.htm`, `wavc_v1.htm` | Already handled by content sniffing (`OggS`) + `WavcBuffer`; confirms design |
| INI dialects: per-animation-slot (`ini_anim`) and per-area spawn (`ini_spawn`) INIs | `ini_anim.htm`, `ini_spawn.htm` | Covered by IniMap + sprite tables + INI-actor layer; both dialects named in the fixture matrix |
| Opcode ceilings per family: BG1 190, BG2 318, BGEE 367, IWD 296, IWD2 457 (holes 178–179, 195–196, 299–399), PST 212 (8 ids undocumented), PSTEE 383; ids 359/364 exist as allocated "Unused"/"Empty" slots; 384–399 documented nowhere; no TobEx/EEEx lists | `opcodes/*.htm`, `opcodes/index.htm` | NI's table (000–457; gaps 359/364/384–399; per-engine availability incl. TobEx/EEEx) stands; P5's per-engine differential adds an IESDP tertiary diff and records every discrepancy |
| Third-party tool formats: TBG v1–v4, TBGN, IAP, KFU, CFB, DAT; WeiDU `.D` language | `file_formats/misc_formats/` | **Out of the port's denominator** — not part of NI's surface, except `.D` as an export target (§2.15, already in scope). Deliberate. |
| Stray extensions `.bah` (undocumented even by IESDP), `.wac` (wav alias) | `general.htm` | No NI handler → Unknown fallback; parity preserved |
| `encryption.htm` documents a second cipher: executable cheat-code strings (`(c+8)<<1`), beyond the XOR data scheme | `encryption.htm` | Not a game-data format; NI does not implement it → out of the port's denominator |
| IESDP-internal contradiction: `general.htm` calls `.eff` a replacement for a "30-byte" effect struct; `eff_v1.htm` documents 48 bytes | both pages | Logged as a live example of why the NI-wins precedence rule exists |
| `files/2da/`, `files/ids/` per-game content catalogues; `scripting/` = 7 per-family action + trigger lists (IDS-dependent; sub-`0x4000` trigger caveat) | those trees | Reference inputs for P9/P12 seeds and the audit's comparison leg |

---

## 2. Subsystem inventory

Evidence base: direct reads of the registries plus five package sweeps of the tree (2026-08-05 at
the pinned SHA). File counts from `find`-based measurement.

### 2.1 INV-SHELL — application shell & global state

`org/infinity/NearInfinity.java` (1787 L, `JFrame` + `ViewableContainer`) and `AppOption.java`
(933 L). Responsibilities: startup key-file resolution (`./chitin.key` → last game → chooser),
CLI args (`-v`, `-h`, `-no-update`, `-no-launch-game`, `-t <Game>`, positional path), frame layout
(tree | viewer split, status bar, toolbar with tree expand/collapse + quick search + launch-game
button/popup), game lifecycle (`openGame`, `refreshGame`, `reloadFactory`), console capture into
an in-app Debug Console, window-state persistence, drag-drop open, `clearCache` (25-step
game-scoped cache reset — the authoritative list of global caches), shutdown hooks. `AppOption`:
~90 typed options over `java.util.prefs` across 5 nodes (legacy node
`org.infinity.gui.BrowserMenuBar` kept deliberately). Also `icon/Icons.java` (75 icon constants),
`exceptions/` (4 classes: Abort, Resource, ResourceNotFound, UnsupportedFormat).

### 2.2 INV-PROFILE — game identity & properties

`resource/Profile.java`: static holder for the open game. 19-game/7-engine enums (§1.2), 91
`Key` properties (paths, capability flags, name tables), engine gating consulted throughout
codecs, datatypes, and `ResourceFactory`. Mod probes `IS_GAME_TOBEX` / `IS_GAME_EEEX`.
`getAvailableResourceTypes()` (Profile.java:1046) is the authoritative per-game extension list.

### 2.3 INV-KEY — resource access layer

`resource/key/` (13 classes): `Keyfile` (chitin.key parse + **full KEY writer**, DLC keyfile
aggregation), `AbstractBIFFReader` (`Type {BIFF, BIF, BIFC}`, cache + lock) with `BIFFReader`
(plain), `BIFReader` (whole-file zlib), `BIFCReader` (block zlib), **`BIFFWriter`** (writes all
three variants; TIS special-cased with header synthesis), `BIFFEntry` (+`Writeable`),
`ResourceEntry` (abstract: buffer/stream/info/search-string/`matchSearchOptions`) with
`BIFFResourceEntry` (+override shadow logic), `FileResourceEntry`, `BufferedResourceEntry`
(in-memory), `ResourceTreeModel`/`ResourceTreeFolder` (tree data + sorting).
`util/io/` (9): `FileManager` (mandatory path hub), `CaseAwarePathResolver` +
`CaseAwareSingleDirectoryPathResolver` (case-insensitive resolution over case-sensitive FS),
`FileEx`, `StreamUtils` (the binary read/write primitive set), `DlcManager`,
`ByteBufferInput/OutputStream`, `FileNameFilter`. `util/io/zip/` (16): a read-only
`java.nio.FileSystem` provider for DLC zips — **store-method-only** entries, CP437 names, derived
from Oracle BSD sample code. Override resolution (loose file shadows BIFF) lives across
`ResourceFactory` + entries.

### 2.4 INV-TLK — string table & talk override

`util/StringTable.java` (1894 L): TLK V1 read **and in-place write**, male/female tables, entry
add/insert/remove + `syncTables`, flags/sound/volume/pitch per entry, TRA export/import, lazy
full-load, modified tracking. `util/CharsetDetector.java`: per-language default charsets
(cs/hu/pl→1250, ru/uk→1251, tr→1254, ja→windows-31j, ko→ibm-949, zh→gbk, else 1252; **note the
live `"window-125x"` typo strings** — behavior to verify at seed time, Q6), byte-sample
fingerprinting, Polish-BG1 `CharLookup` translation table. `resource/to/` (6): TOH/TOT — V1
classic pair (strings in TOT, chained segments) vs V2 EE (self-contained TOH); **upstream disables
saving** (`TohResource.close()` discards; SAVE buttons disabled) though `AbstractStruct.write()`
machinery exists.

### 2.5 INV-STRUCT — struct/datatype core (the heart of the faithful half)

`resource/` root interfaces: `StructEntry` (Comparable-by-offset + Readable + Writeable),
`Resource`, `Viewable`/`ViewableContainer`, `AddRemovable` (no-arg-ctor prototype convention),
`HasChildStructs` (28 implementors), `HasViewerTabs`, `TextResource`, `Closeable`,
`Referenceable`. **`AbstractStruct`** (extends Swing `AbstractTableModel` — the fusion the port
severs): read = `addField` in declaration order + `fixHoles` (injects "Unused bytes?" `Unknown`s)
+ `initAddStructMaps`; **write = sort fields by offset, write each** (no separate serializer);
`offsetmap`/`countmap` (`SectionOffset`/`SectionCount` → section class) drive
`adjustEntryOffsets`/`adjustSectionOffsets` on add/remove (incl. a hardcoded ITM/SPL
Effect-after-Ability ordering rule); `addDatatype`/`removeDatatype` insertion logic (partly
selection-dependent — see coupling note); `realignStructOffsets`, `removeFromList` (serializes a
field run back to bytes — powers opcode re-layout), `getFlatFields` (hex view), deep `clone`
with re-offsetting. Change notification today = 4 channels: TableModelEvents (incl. custom
`WILL_BE_DELETE = -2` consumed by DlgTreeModel), datatype `UpdateListener` (cross-row re-layout),
JavaBeans `PropertyChange` bubbling via `setParent`, and the recursive `structChanged` dirty flag.
**Measured view-couplings to sever:** `getDatatypeIndex` reads `viewer.getSelectedRow()`;
`Datatype.fireValueUpdated` stores/restores viewer selection; `getColumnCount`/`getValueAt` read
global `BrowserMenuBar` options.

`datatype/` — **55 files**: numeric (`DecNumber` signed 1/2/3/4B with 3-byte sign-extension,
`UnsignDecNumber`, `HexNumber`, `UnsignHexNumber`, `RemovableDecNumber`, `FloatNumber` 4/8B,
`MultiNumber` packed bit-groups), structure links (`SectionOffset`, `SectionCount`), bitmaps
(`AbstractBitmap<T>` + `Bitmap`, `HashBitmap`, `IdsBitmap`, `IdsFlag`, `KitIdsBitmap` word-swap,
`AnimateBitmap`, `PortraitIconBitmap`, `TableBitmap`, `WmpLinkBitmap`, `EffectBitmap`,
`ItemTypeBitmap`, `PriTypeBitmap`, `SecTypeBitmap`, `Summon2daBitmap`, `Song2daBitmap`,
`SpellProtType`, `IdsTargetType` with paired-field factories, `TextBitmap`), flags (`Flag` 1/2/3/4B
bitfields), references (`ResourceRef` 8B resref + probing, `AreResourceRef`, `SpawnResourceRef`,
`ResourceBitmap`/`ProRef`/`IwdRef`, `StringRef` 4B strref), text (`TextString`, `TextEdit` with
EOL/NUL semantics), raw (`Unknown`, `UnknownBinary`, `UnknownDecimal`), special (`EffectType` —
the opcode-driven field factory + `UpdateListener`, `ColorPicker`, `ColorValue`, PST `Bestiary`
260B), editing-surface interfaces (`Editable`, `InlineEditable`), marker interfaces (`IsNumeric`,
`IsTextual`, `IsReference`, `IsBinary`), events (`UpdateEvent`/`UpdateListener`), base
(`Datatype`, `Readable`). Plus `util/ResourceStructure` + `resource/StructureFactory` — blank
resources from scratch, `ResType` enum of **21 creatable types** (§1.3).

### 2.6 INV-FORMATS — struct-format codecs (versions measured in `read()`)

| Package | Format, versions (evidence) | Top-level + substructures | Write |
|---|---|---|---|
| `are/` (23) | ARE V1.0 / V9.1 (IWD2 branch), PST/PSTEE/EE field branches | `AreResource` + 17 subs (Actor, Ambient, Animation, AutomapNote, AutomapNotePST, Container, Door, Entrance, Explored, ITEPoint, Item, ProEffect, ProTrap, RestSpawn, Song, SpawnPoint, TiledObject, Variable) | inherited |
| `chu/` (4) | CHU single version | `ChuResource` + `Window`, `Control` (explicit `write`) | explicit |
| `cre/` (15) | CRE V1.0/V1.1/V1.2(PST)/V2.2(IWD2)/V9.0-9.1(IWD); CHR wrappers V1.0/V2.0/V2.1/V2.2 | `CreResource` + 10 subs (Item, Known/MemorizedSpells, SpellMemorization, Iwd2Struct+Ability/Shape/Song/Spell, InventorySlotIndex) | inherited |
| `dlg/` (25) | DLG V1.0 only (throws otherwise) | `DlgResource` + State, Transition, StateTrigger, ResponseTrigger, Action (`AbstractCode` raw script text) — rest is tree-viewer UI | inherited |
| `effects/` (442) | see INV-EFFECTS | — | via owner |
| `fnt/` (6) | FNT (EE, headerless) | `FntResource` + 5 subs | inherited |
| `gam/` (14) | GAM V1.1/V2.0/V2.1/V2.2 selected **by engine** | `GamResource` + 10 subs (PartyNPC, NonPartyNPC, Variable, KillVariable, JournalEntry, Familiar, StoredLocation, ModronMaze(+Entry), UnknownSection2/3) | inherited |
| `itm/` (4) | ITM V1.0/V1.1(PST)/V2.0(IWD2), KIT.IDS-gated fields | `ItmResource` + `Ability` (+root `Effect`/`Effect2`) | inherited |
| `maze/` (2) | MAZE (PSTEE), fixed 64 rooms | `MazeResource` + `MazeEntry` | inherited |
| `mus/` (4) | MUS text playlist | `MusResource` (+`Entry`, playback handler) | explicit |
| `other/` (5) | EFF V2.0 standalone (dual signature), VVC, WFX, TTF (read-only preview), Unknown | `EffResource`, `VvcResource`, `WfxResource`, `TtfResource`, `UnknownResource` | EFF/VVC/WFX yes; TTF no; Unknown raw |
| `pro/` (3) | PRO typed (No-BAM / single / AoE) + EE extensions | `ProResource` + `ProSingleType`, `ProAreaType` | inherited |
| `sav/` (3) | SAV container, per-member zlib (BEST_COMPRESSION) | `SavResource`, `IOHandler`, `SavResourceEntry` | explicit |
| `spl/` (4) | SPL V1 / V2.0 (IWD2 tail), PST anim table | `SplResource` + `Ability` | inherited |
| `src/` (2) | SRC PST struct (IWD2 SRC routes to text) | `SrcResource` + `Entry` | inherited |
| `sto/` (7) | STO V1.0 / V1.1 (PST, ItemSale11) / V9.0 (IWD capacity quirk) | `StoResource` + 5 subs (ItemSale, ItemSale11, Purchases, Drink, Cure) | inherited |
| `text/` (3+7) | 2DA, IDS, BIO, RES, TXT, LOG, INI, SET + EE LUA/SQL/GUI/MENU/GLSL + IWD2 SRC; XOR-encrypted variants; PST `quests.ini` (`QuestsResource`) | `PlainTextResource`, `QuestsResource`; `modes/` = 7 syntax token makers (BCS, GLSL, INI, MENU, TLK, WeiDU-log + BCS fold parser) | explicit (writes decrypted) |
| `var/` (2) | VAR headerless 44B records | `VarResource` + `Entry` | inherited |
| `vef/` (5) | VEF | `VefResource` + components | inherited |
| `vertex/` (5) | shared 8B vertices for ARE/WED polygons | `Vertex` + 4 semantic subclasses | via parent |
| `wed/` (10) | WED V1.3 (no branch), dual header | `WedResource` + 9 subs (Overlay, Door, Tilemap, Wallgroup, IndexNumber, Polygon + Open/Closed/Wall) | inherited |
| `wmp/` (9) | WMP | `WmpResource` (explicit `write` via flat fields) + MapEntry, AreaEntry, AreaLink×4 | explicit |

Root-shared: `Effect` (EFF V1 48B), `Effect2` (V2 264B), `AbstractAbility`, `AbstractVariable`,
`BoundingBox`.

### 2.7 INV-EFFECTS — the opcode framework

`resource/effects/`: **442 files** = `BaseOpcode` (registry + template method `makeEffectStruct`)
+ `DefaultOpcode` (unknown-id fallback) + **440 `OpcodeNNN` classes** covering ids 000–457 (gaps:
359, 364, 384–399). Registry is **reflective by name** (`Class.forName("...Opcode%03d")`,
scan bound `NUM_OPCODES = 460`), instances filtered by `isAvailable()` = per-game
`getOpcodeName() != null`. Engine variation via six overridable hooks
(`makeEffectParamsBG1/BG2/EE/PST/IWD/IWD2`; override counts measured: IWD2 148, EE 141, IWD 103,
PST 38, BG2 29, BG1 15) + TobEx/EEEx probes; **10 opcodes** override `update()` for dynamic
re-layout when parameters change. Field addressing via `EffectEntry` enum (paired IDX/OFS for V1
and V2). Feeds `EffectType`/`EffectBitmap` datatypes, `getEffectNames()`,
`getPortraitIconNames()`. Reset on game switch.

### 2.8 INV-GFX — graphics codecs

`resource/graphics/` (27) + `graphics/decoder/` (6). Decode: `BamDecoder` → `BamV1Decoder`
(palette+RLE), `BamV2Decoder` (PVRZ-backed), `PseudoBamDecoder` (synthetic); `MosDecoder` →
V1/V2; `TisDecoder` → palette/PVRZ (64px default, 512 max, headerless-in-BIFF handling);
`PvrDecoder` + `DxtDecoder`/`Etc2Decoder`/`PvrtcDecoder`/`DummyDecoder`; `BmpDecoder`;
`GifSequenceReader`; `Palette`; `ColorConvert` (quantization, color spaces). Encode/write:
`Compressor` (BAMC/MOSC zlib), **`DxtEncoder`** (DXT1/3/5, 2941 L), `BmpEncoder`,
`PseudoBamDecoder.exportBamV1/V2` (the BAM writer), `TisConvert` (palette↔PVRZ both directions +
PNG export + overlay conversion modes), `GifSequenceWriter`. Resources: `BamResource`
(`Writeable` — writes buffer back), `MosResource`, `TisResource`, `PvrzResource`, `PltResource`
(`Writeable`), `GraphicsResource` (BMP/PNG). Bundled `apng-writer` (BSD-3) serves animated-PNG
export. Profile keys: BAM_V1/V1_ALPHA/V2/BAMC_V1, MOS_V1/V2/MOSC_V1, TIS_V1/V2, BMP_PAL/ALPHA.

### 2.9 INV-SPRITE — creature-animation sprite system

`resource/cre/decoder/` (23) + `decoder/util/` (14) + `decoder/tables/` (2 + 2DA/IDS data):
`SpriteDecoder extends PseudoBamDecoder`, 19 concrete decoder types + placeholder, selected via
`SpriteUtils` `EnumMap` + per-game `NumberRange` slots (`AnimationInfo.Type`, 19 values) with
Infinity-Animations support; 12 bundled `avatars-*.2da` tables + 2 IA `.ids`. Consumed by the
creature browser and the area viewer's actor layer. Decode-only.

### 2.10 INV-AUDIO — audio codecs & playback

`resource/sound/` (16): `AudioFactory` dispatch (`FMT_WAV/ACM/WAVC/OGG`); `AcmBuffer` (833 L,
Interplay ACM→PCM), `WavcBuffer` (WAVC wrapper), `WavBuffer` (RIFF), `OggBuffer` (JOrbis-based).
**Decode-only — no ACM/WAVC/OGG encoders exist upstream.** Players: buffered + streaming stacks
(`AbstractAudioPlayer`, `BufferedAudioPlayer`, `StreamingAudioPlayer`, state events).
`SoundResource` (not `Writeable`; export raw or "as WAV"). MUS sequencing via `mus/` (Entry
resolution to ACM under music dirs, loop/end handling). UI: `gui/SoundPanel` (reusable player),
`gui/InfinityAmpPlus` (2816 L playlist player).

### 2.11 INV-VIDEO — video

`resource/video/` (10): MVE decoder stack (`MveDecoder`, `MveVideoDecoder`, `MveAudioDecoder`,
`MvePlayer`, `AudioQueue`, `BasicVideoBuffer`, `ImageRenderer`) — decode+play, export "as AVI"
via MonteMedia (`AVIWriter`). `WbmResource` — **no decoder**; raw export only. Neither writes the
format back.

### 2.12 INV-BCS — scripting

`resource/bcs/` (12): `BcsResource` (compiled BCS container; `Writeable`), `BafResource` (script
source; `Writeable`), `Compiler` / `Decompiler` (with `getErrors()`/`getWarnings()`,
`getStringRefsUsed()`, `getIdsErrors()`, comment generation), `Signatures` (trigger/action
signatures from IDS), `ScriptInfo` (per-engine quirks), `ScriptType`, `BcsAction`/`BcsObject`/
`BcsTrigger`/`BcsStructureBase`, `ScriptMessage`. `parser/` (14): JavaCC/JJTree-generated BAF
parser (`BafParser.jjt` grammar; `BafNode*` hand-maintained). Supporting: `util/IdsMap*` +
`StaticSimpleXorDecryptor` (encrypted BCS/IDS/2DA), `gui/ScriptTextArea` (editor), IDS browser,
Script Drop Zone (batch compile/decompile).

### 2.13 INV-SEARCH — search subsystem

`search/` (20): `AbstractSearcher` (threaded driver + progress), `AbstractReferenceSearcher` (+
`FileTypeSelector` with persisted per-key selections), concrete reference searchers —
`ReferenceSearcher` (generic per-class dispatch incl. PVRZ-index and script-name/symbolic-name
matching), `StringReferenceSearcher` (strref across 20 types incl. decompile paths),
`ScriptReferenceSearcher`, `WavReferenceSearcher` (incl. TLK sound-resref matching),
`SongReferenceSearcher` (MUS via SONGLIST + script regexes), `DialogStateReferenceSearcher`,
`DialogItemRefSearcher` (intra-DLG reverse links); query tools — `AttributeSearcher` (same-type
field search with numeric ops), `DialogSearcher`, `TextResourceSearcher`, `SearchMaster` (+
`SearchClient` incremental find), `SearchFrame` (name search with symbol support + "insert
reference"), `SearchResource` ("Extended search (deprecated)", 8 per-type option panels),
`SearchOptions` (the string-key query schema; `matchSearchOptions` implemented by
ARE/CRE/EFF/ITM/PRO/SPL/STO/VVC); results — `ReferenceHitFrame`, `TextHitFrame` (+ save TXT/CSV).
`search/advanced/` (8): `AdvancedSearch` (`ChildFrame`, MATCH_ALL/ANY/ONE), `AdvancedSearchWorker`
(structure-path matching, group reconciliation), advanced `SearchOptions` (BY_NAME /
BY_RELATIVE_OFFSET / BY_ABSOLUTE_OFFSET; TEXT/RESOURCE/NUMBER/BITFIELD with EXACT/AND/OR/XOR),
`FilterInput` (filter editor), `XmlConfig` (DTD-validated filter import/export), `FlagsPanel`,
`NumberFormatterEx`, `ToggleButtonDataModel`.

### 2.14 INV-CHECK — check subsystem

`check/` (16): `AbstractChecker` base + 15 checkers — `BCSIDSChecker`, `CreInvChecker`,
`DialogChecker` (all/override-only), `EffectsIndexChecker`, `EffectValidationChecker`,
`IDSRefChecker`, `ResRefChecker`, `ResourceUseChecker` (unused files, inverted corpus),
`ScriptChecker` (decompile→recompile), `StringDuplicatesChecker`, `StringSoundsChecker`,
`StringUseChecker` (unused strings incl. PST bestiary pre-marks + 2DA blacklist),
`StringValidationChecker` (raw TLK scan with charset error reporting **and a Repair action**),
`StrrefIndexChecker`, `StructChecker` (signature/version table, overlap/truncation/offset
validation + WED/STO special corruption rules). Result frames on `SortableTable` with TXT/CSV
export.

### 2.15 INV-EXPORT — export & batch operations

`gui/MassExporter` (type multi-select, regex filter, preview; conversion matrix: decompile
BCS/DLG→BAF/D, decrypt text, sounds→WAV, CHR→CRE, BAM/MOS decompress, MOS/PVRZ/TIS→PNG, TIS
version conversion, BAM frame extraction, MVE→AVI, trim/align text, overwrite policy).
`ResourceFactory.exportResource` (raw + XOR-decrypt only) / `saveResource` (override
redirection, backup-on-save, post-save cache invalidation) / `saveResourceAs` / zip archiver
(`ResourceTree.createZipInteractive` for save folders). Per-viewer export menus (BAM frames,
MOS/PVRZ/TIS/PLT→PNG, CHR→CRE, sound→WAV, MVE→AVI, DLG→WeiDU-D, TLK→TRA/TXT, worldmap→PNG).
`BcsDropFrame` batch compile/decompile. SAV pack/unpack (`IOHandler`). `util/Weidu` (external
binary discovery ≥ 24600, `--change-log` runner + results table).

### 2.16 INV-GUI — GUI shell, editors, viewers

(Complete census; rewrite side.) Shell: menus (Game/File/Edit/Search/Tools/Help per §1.3-1.4
census), `ResourceTree` (+context ops), `QuickSearch`, `StatusBar`, `ChildFrame` registry,
`WindowBlocker`, `ViewFrame`, bookmarks + `BookmarkEditor` (per-OS executable lists) + recents +
launcher. Editors: `StructViewer` (1966 L; Edit/View/Raw tabs, add/remove, context commands incl.
"edit as", apply-to-substructures, go-to-offset, advanced-search hookup), `StructCellEditor`,
`ButtonPanel` kit, `ViewerUtil` (View-tab panel kit), per-format viewer tabs (are/cre/chu/dlg/gam/
itm/spl/sto/wmp `Viewer*` classes), `StringEditor` (TLK), `BIFFEditor` + `ChooseBIFFrame`,
`OpenFileFrame`/`OpenResourceDialog`, `NewRes/NewChr/NewPro` settings dialogs,
`ItemCategoryOrderDialog`, `GameProperties`, `ClipboardViewer`, `DebugConsole`, `IdsBrowser`,
`SplProtFrame`, `StopWatchTester` (dev). Hex: `gui/hexview` (9) — JHexView-based editor with
struct-colored ranges (13 legend categories), find dialog, undo/redo, three data providers.
Rich viewers: `are/viewer` (57; 16 layer types + stacking extras, day/night, overlays, mini-maps
search/light/height/explored, zoom, edit-mode add/remove, settings dialog, PNG export),
`dlg` tree viewer (DlgTreeModel with cycle-breaking + options), `wmp/viewer` (8; icons/labels/
distances, PNG export, virtual entries), `cre/browser` (14; animation/equipment/color controls,
sequence playback, export), `gui/converter` (BAM converter 4751 L with 4 tabs + ~20 filter
classes in 3 families + session save/load + palette dialog; ConvertToBmp/Mos/Pvrz/Tis + batch
TIS), media UI (`SoundPanel`, `InfinityAmpPlus`, MVE player), text editing (`InfinityTextArea` /
`ScriptTextArea` on RSyntaxTextArea + 11 theme XMLs), `PreferencesDialog` (2217 L; page tree per
§ census), `SpriteAnimationPanel` (idle easter egg), assorted widgets (TextListPanel,
ResourceChooser, ColorChooser/Grid, FontChooser, popup/menu/list helpers, `resource/ui` renderers).

### 2.17 INV-INFRA — infrastructure

`util/` (45): caches (`IdsMapCache`, `Table2daCache`, `IniMapCache`, `CreMapCache`, `IconCache`,
`PortraitIconCache`) + parsers (`IdsMap(Entry)`, `Table2da`, `IniMap*` — `//` comments,
`LuaParser`/`LuaEntry`), `StringTable`/`CharsetDetector` (§2.4), `StaticSimpleXorDecryptor`,
`ResourceStructure`, byte helpers (`DynamicArray`, `DynamicByteArray`, `ArrayUtil`,
`StringBufferStream`, `BOMStringReader`, `FastMath`, `BinPack2D` MaxRects — feeds TIS/BAM/MOS
packing), collections (`DataString`, `MapEntry`, `MapTree`, `IntegerHashMap`, `Variables`,
`TriState`, `SimpleListModel`, `FilteredListModel`), `StructClipboard` (struct cut/copy/paste
singleton), `Threading` (pooled executor + EDT marshalling; 9 pool sites measured),
`Operation`, `Logger` (tinylog façade + `APP_LOG_LEVEL` gate), `Platform`, `LauncherUtils`
(xdg-open), `Weidu`, `InputKeyHelper` (Windows-only), `FileDeletionHook`, `Misc`
(CHARSET_DEFAULT=windows-1252, prefs plumbing, numeric/string helpers), `StopWatch`,
`DebugTimer`; `util/tuples/` (13; arity 1–6). `updater/` (6): version-check against two
`update.xml` URLs, proxy support, **manual-download-only** (opens browser; no self-install).
Swing-concurrency shape to translate: EDT + `SwingWorker` (27 sites) + `Threading` pools; all
global caches game-scoped and reset by `clearCache` (25 steps) — two stragglers cleared elsewhere
(`SharedResourceCache`, `mus/Entry`).

---

## 3. The mapping — every subsystem to its MVVM slot

Slots per the contract and the architecture record: **Service** (stateless, wraps data sources),
**Repository** (source of truth, caching, game-session state), **ViewModel** (Riverpod notifiers,
presentation state + commands), **View** (widgets). Stance: *faithful* = behavioral divergence is
a bug (round-trip instrument applies); *faithful semantics, restructured shape* = same decisions,
new statics-free shape; *rewrite* = feature/workflow parity via gap register.

| # | Subsystem (inventory key) | MVVM slot(s) | Stance |
|---|---|---|---|
| M1 | KEY/BIF/BIFC/BIFF read+write, XOR decryptor, zlib, byte primitives (INV-KEY) | Service (`KeyfileService`, `BiffService`, `CryptoService`, byte-io lib) | faithful |
| M2 | Case-insensitive path resolution, DLC zip reading, FileManager semantics (INV-KEY) | Service (`GamePathService`, `DlcArchiveService` — store-only zip reader, CP437) | faithful (zip-mount shape redesigned; semantics identical) |
| M3 | Override resolution + resource enumeration + tree data + signature sniffing (INV-KEY, ResourceFactory registry half) | Service (`ResourceCatalogService`: type registry, `detectResourceType`) feeding Repository R1 | faithful |
| M4 | `Profile` (INV-PROFILE) | Repository (`GameProfileRepository`) exposing an **immutable `GameProfile` snapshot**; codec Services receive it **as an argument** (severs the global static) | faithful semantics, restructured shape |
| M5 | `ResourceFactory` orchestration half: load/save/export/close, caches wiring | Repository (`ResourceRepository`: tree state, open-game lifecycle, save/backup/override redirection policy) | faithful semantics, restructured shape |
| M6 | TLK read/write + charsets + TRA (INV-TLK) | Service (`TlkCodec`, `CharsetService`) + Repository (`StringTableRepository`: loaded tables, modified state, male/female) | faithful |
| M7 | TOH/TOT (INV-TLK) | Service (`TalkOverrideCodec`) + surfaced through R-strings | faithful (incl. write path, though upstream UI disables it) |
| M8 | Struct core: StructEntry tree, AbstractStruct read/write/offset machinery, 55 datatypes' byte semantics, fixHoles, clone, removeFromList (INV-STRUCT) | Model layer library (`domain/models/struct/`) used by codec Services; **no UI types in it** | faithful |
| M9 | Datatype *editing surfaces* (Editable/InlineEditable panels, update flows, 4 notification channels) | ViewModel (`StructEditorViewModel`: rows projection, selection, commands add/remove/edit/paste; emits state instead of table events) + View (field editors) | rewrite |
| M10 | Section-offset/count auto-adjust + insertion-position policy | Model (faithful) **except** selection-dependent insertion → explicit `insertionHint` parameter supplied by the ViewModel | faithful, coupling severed |
| M11 | All struct-format codecs (INV-FORMATS table rows) | Services (`codecs/<fmt>`), one per format family; parse(bytes, GameProfile) → model; write(model) → bytes | faithful |
| M12 | Effects framework (INV-EFFECTS) | Service (`OpcodeTable`: static data+functions replacing reflection scan; per-engine hooks preserved; 10 dynamic-update rules) | faithful semantics, restructured shape (registry mechanism only) |
| M13 | StructureFactory create-new (21 types) | Service (`NewResourceService`, templates) + Repository save path | faithful |
| M14 | Graphics codecs (INV-GFX) incl. DXT encode, TisConvert, BAM writer, quantization, BinPack2D | Services (`media/graphics/*`) | faithful (pixel/byte behavior; C2/C3 where zlib nondeterminism applies) |
| M15 | Sprite system (INV-SPRITE) | Service (`SpriteDecoderService` + data tables as assets) | faithful |
| M16 | Audio codecs ACM/WAVC/WAV (INV-AUDIO) | Service (`media/audio/*`) | faithful (PCM-equal) |
| M17 | OGG decode | Service via **FFI to system libvorbis/libogg (BSD)**; fallback: faithful JOrbis port (LGPL-compatible) | faithful at PCM level |
| M18 | Audio playback + MUS sequencing | Service (`PlaybackService`, Linux audio out) + `MusPlaylistCodec` (faithful) | playback = rewrite; MUS parse/sequence rules = faithful |
| M19 | MVE decode/play, AVI export; WBM raw (INV-VIDEO) | Service (`MveDecoder` faithful port; `AviMuxer` **original**, spec-based — G7); WBM raw passthrough | faithful decode; export container original |
| M20 | BCS compiler/decompiler/signatures/ScriptInfo; BAF parser (INV-BCS) | Service (`bcs/*`); BAF parser rewritten as hand-written recursive-descent from the `.jjt` grammar (G8) | faithful (byte-level recompile parity) |
| M21 | IDS/2DA/INI/Lua parsers + their caches (INV-INFRA) | Services (parsers) + Repositories (`IdsRepository`, `TableRepository`, `IniRepository` — game-scoped caches) | faithful |
| M22 | Search engines (INV-SEARCH) incl. advanced-search matching + XML filter format | Service (`search/*` engines, isolate-parallel) + ViewModels (query forms, progress, results) + Views | engine logic faithful-semantics; concurrency + UI rewrite |
| M23 | Checkers ×15 (INV-CHECK) | Service (`check/*`) + VM/View for setup/progress/results; TLK Repair = command on StringTableRepository | same as M22 |
| M24 | Mass export + per-viewer exports + zip + SAV pack/unpack + batch script compile (INV-EXPORT) | Service (`ExportService`, `SavContainerService`) + VM/View | conversion behavior faithful; UI rewrite |
| M25 | WeiDU integration | Service (`WeiduService`, external process) + VM/View | faithful semantics |
| M26 | Settings (~90 options, 5 nodes) (INV-SHELL) | Repository (`SettingsRepository`, JSON file under XDG config) + Views (preferences) | rewrite; option *semantics* preserved; no prefs migration (G5) |
| M27 | Bookmarks/recents/launcher | Repository (`BookmarksRepository`) + Service (`GameLaunchService`) + VM/View | rewrite, feature parity |
| M28 | Window/tab management (ChildFrame registry), quick search, status bar, menus, tree UI | ViewModels + Views (`app/` shell + `ui/*`) — single-window workspace (G1) | rewrite |
| M29 | StructViewer / hex editor / text editor / rich viewers (area, DLG tree, WMP, CRE browser) / converters UI / StringEditor / BIFF editor / SAV manager / dialogs (INV-GUI) | View + ViewModel pairs per feature; decode via M11–M19 services | rewrite |
| M30 | Syntax highlighting definitions (7 token makers, 11 themes) | View-layer highlight rules (ported as data-driven span rules) | rewrite, rule-content faithful |
| M31 | Threading/EDT model (INV-INFRA) | Isolate pool utility + per-service `compute` offload; UI isolate holds repositories | restructured (java-semantics #4) |
| M32 | Logger/tinylog | `package:logging` (BSD) façade honoring the log-level option | rewrite |
| M33 | Updater (version check, manual download) | Service (`UpdateCheckService` reusing upstream `update.xml` endpoints) + VM/View | faithful semantics |
| M34 | StructClipboard | Repository (`StructClipboardRepository`, app-scoped) + VM commands | faithful semantics, restructured |
| M35 | Icons (75), themes | Asset set + Material icons | rewrite |
| M36 | Exceptions; tuples; misc helpers | Dart exceptions; records; per-need utils (`DynamicArray` → `ByteData` idioms) | mostly obviated |
| M37 | `FileDeletionHook`, temp management | Service (`TempFileService`) | rewrite |
| M38 | Easter egg `SpriteAnimationPanel` | View (late phase) on M15 | rewrite, parity item |

Cardinality note (architecture record): repositories never reference each other — cross-cutting
flows (open game = profile detect + catalog load + caches reset; save = write + backup + cache
invalidation) live in `domain/use_cases/` (the record's optional domain layer), which this plan
adopts explicitly.

---

## 4. Target-side architecture positions

### 4.1 Package layout (owned by this plan)

```
lib/
  main.dart                 # ProviderScope + app bootstrap
  app/                      # shell view + shell view-model, theming, workspace tabs
  ui/<feature>/             # one folder per view–viewmodel pair (tree, struct_editor,
                            #   hex_view, text_editor, are_viewer, dlg_tree, wmp_viewer,
                            #   cre_browser, converters/*, search/*, check/*, settings, ...)
  domain/
    models/                 # GameProfile, struct core, per-format models, results, options
    use_cases/              # open_game, save_resource, mass_export, ... (cross-repo flows)
  data/
    repositories/           # game_profile, resources, string_table, ids, tables_2da, ini,
                            #   settings, bookmarks, struct_clipboard
    services/
      io/                   # bytes, key, biff, dlc_zip, paths(case-aware), crypto(xor, zlib)
      codecs/<format>/      # one per format family (are, cre, itm, ... , tlk, toh_tot)
      effects/              # opcode table
      bcs/                  # compiler, decompiler, signatures, script_info, baf_parser
      media/                # graphics/, audio/, video/, sprites/
      tools/                # search engines, checkers, export, weidu, update_check, launch
  util/                     # charsets, logging, isolate pool
tool/                       # roundtrip_sweep.dart, fixture_manifest.dart, java_oracle/ (Java driver)
test/                       # unit + harness library
fixtures/synthetic/         # committed, hand-authored per format×version (+adversarial)
```

Every ported file carries the dual-copyright LGPL header from the contract; original files carry
the port line only.

### 4.2 Riverpod (bounded by the record — binding idioms are oracle-leg work)

Fixed now: manual declarations only; providers are the DI graph; Services = plain classes behind
`Provider`s; Repositories = `Notifier`-backed where they hold session state (open game, tree,
dirty TLK), plain providers otherwise; per-resource editor ViewModels = auto-dispose family
notifiers keyed by resource locator; commands = notifier methods. Deferred to the oracle leg
(mvvm record §Riverpod): the exact notifier types per VM shape, view binding idiom for the 1:1
rule, and provider-override patterns for tests.

### 4.3 Concurrency

Parsing sweeps, search, check, conversion, and export run in a worker-isolate pool
(`util/isolates`); messages are immutable (bytes in, models/results out); `GameProfile` snapshot
travels with the request. The UI isolate owns repositories (mirrors "EDT owns state" without
shared-memory mutation). Single-resource parse for viewing may run inline when small; the
harness measures where the isolate boundary pays (A9).

### 4.4 Byte-level platform contracts (java-semantics ledger)

Little-endian per call site (`Endian.little` — ledger #1); unsigned masking widths (#2);
charsets: windows-1252/1250/1251/1254 + windows-31J/IBM-949/GBK/Big5-HKSCS as in-repo codec
tables **generated from Unicode Consortium mapping data** (original work; no Apache-2.0 dep)
(#3); prefs → settings file (#5); zip-mount → store-only direct reader (#6); case-aware paths
(#7). **Proposed new ledger entries** (to verify at first touch): zlib output nondeterminism
across implementations (affects C1-vs-C2 assignment for BIF/BIFC/SAV/BAMC/MOSC writers);
`Charset.forName` behavior on the `"window-125x"` typo strings (Q6); `String.getBytes` lossiness
on malformed TLK bytes (StringValidationChecker's reason to exist); JavaCC token/lookahead
semantics relied on by `BafParser.jjt`.

### 4.5 Dependency policy

LGPL-2.1-compatible only; **no Apache-2.0** (contract). Planned surface is deliberately small:
`flutter_riverpod` (MIT, pinned line 3.x), `package:logging` (BSD-3), possibly a code-editor or
highlighting package for the text editor (license-gated at seed time; fallback = custom
span-based highlighter — the token rules are ported anyway, M30), FFI to system `libvorbis`
(BSD) and an audio-out path (candidate: `dart:ffi` → libasound/libpulse directly, original code;
any pub package here is license-gated). Codecs, compression wrappers (dart:io `ZLibCodec`
covers raw+zlib), DXT, ACM, MVE, AVI muxing, APNG: implemented in-repo. Every `pubspec.yaml`
addition states its license in the commit message.

---

## 5. Phasing — model outward to UI

Rules: development is model → UI (contract); each phase lists contents, deliverable, and
acceptance; seeds are cut from phases one at a time (pipeline). Format phases fan out after P3 —
P4–P12 items may proceed in parallel tracks; UI for a thing lands only after its model. Sizes:
S/M/L/XL are relative effort.

| Phase | Contents | Delivers / acceptance |
|---|---|---|
| **P0 — Harness & primitives** (M) | `tool/roundtrip_sweep.dart` skeleton + fixture manifest tooling; `tool/java_oracle/` Java driver against the upstream jar (A2); byte-io library (LE readers/writers, unsigned masks); charset tables (1252 first, rest per §4.4); XOR decryptor; zlib wrappers; LGPL-header lint; CI (analyze + test + synthetic sweep); logging. | Harness runs end-to-end on raw bytes (C0 = copy-through). **Oracle baseline sweep over BG:EE** produces the per-format upstream identity report that fixes each format's comparator class (§7.3). |
| **P1 — Resource access** (L) | KEY parse+write; BIFF/BIF/BIFC readers; BIFFWriter; override + extra-dir scan; DLC zip reader (store-only, CP437); case-aware path resolution; `ResourceEntry` family + tree model (pure data); signature sniffing; `GameProfile` detection for all 19 games; catalog service + repository + open-game use-case. | Enumeration of BG:EE == Java NI's resource count (differential); KEY round-trip C1; BIFF write validated C4 + C2-with-zlib-caveat; profile detection over synthetic roots for all 19 games. |
| **P2 — TLK & talk override** (M) | TLK V1 codec (read/write, flags/sound/volume/pitch), female table, charsets + detection semantics, TRA export/import, TOH/TOT codec (V1+V2), StringTable repository. | Round-trip `dialog.tlk` ×15 BG:EE languages (C1); CJK/ru decode differential vs oracle (C3); TOH/TOT C1. |
| **P3 — Struct core** (L) | StructEntry tree + all 55 datatype byte semantics; AbstractStruct read/write/fixHoles/offset-count maps/add-remove/realign/removeFromList/clone (view couplings severed per M10); Unknown fallback resource; ResourceStructure + 21 create-new templates. | Unit suite incl. adversarial synthetics (holes, overlaps, zero counts, 3-byte fields, schedule/orientation tables); UnknownResource sweep C1 over all of BG:EE. |
| **P4 — Text & table formats** (M) | PlainTextResource semantics (decrypt-on-read; trim/align/sort as explicit ops), 2DA (`Table2da.assemble`), IDS, INI (`//` dialect), Lua parser, QuestsResource model, MUS playlist codec, BIO/RES/TXT/LOG/SET/GUI/SQL/GLSL/MENU/LUA routing per profile gates; caches → repositories. | Sweep C1 for unencrypted text; encrypted files C2 (upstream writes decrypted — measured in P0 baseline); cache-reset parity with `clearCache`. |
| **P5 — Effects engine** (L) | Opcode table for ids 000–457 (440 entries + default), per-engine name/availability/params (6 hooks + TobEx/EEEx), 10 dynamic-update rules, EffectEntry addressing, Effect/Effect2 read/write. IESDP `opcodes/` as independent cross-check (NI wins on conflict; conflicts recorded — per-family ceilings and the IWD2 holes 178–179/195–196, §1.6). | All BG:EE effects parse + C1 round-trip inside owners; opcode-name tables differential vs oracle for **each** of the 7 engines (synthetic profiles). |
| **P6 — Struct formats I** (M) | ITM, SPL (+Ability/AbstractAbility), EFF standalone, VVC, WFX, PRO (+type-driven sections), VEF. | C1 sweep over BG:EE for all seven; synthetic V1.1/V2.0 (PST/IWD2) fixtures round-trip. |
| **P7 — Struct formats II** (M) | CRE all versions + CHR wrappers, STO all versions, SRC(PST), VAR, MAZE, FNT, CHU, vertex records. | C1 sweep (BG:EE: CRE/CHR/STO/CHU/FNT); synthetics for PST/IWD/IWD2 variants incl. STO V9.0 capacity quirk. |
| **P8 — Struct formats III** (L) | WED, ARE (V1.0/V9.1 + engine branches), WMP, GAM (4 versions), SAV container (member-level zlib + nested round-trip), DLG (+AbstractCode raw text), BCS/BS/BAF as containers (no compiler yet), TOH/TOT integration with strref display. | C1 sweep over BG:EE incl. local save games (GAM/SAV under `~/.local/share/Baldur's Gate - Enhanced Edition/save/`); SAV member bytes C2 (zlib) with decoded-identity C3. |
| **P9 — BCS compiler/decompiler** (L) | Signatures from IDS, ScriptInfo per engine, Decompiler (+strref/IDS-error collection, comment generation), Compiler (+errors/warnings), hand-written BAF parser from the `.jjt` grammar, IDS symbol resolution rules. IESDP `scripting/` + `file_formats/ie_formats/bcs.htm` as independent cross-check. | **Decompile→recompile byte-identity over every BCS/BS in BG:EE** (C4-strong); error/warning-list differential vs oracle on seeded-broken scripts; BcsResource C1. |
| **P10 — Graphics & sprites** (XL) | Compressor, palettes + ColorConvert quantization, BAM v1/BAMC/v2 decode + PseudoBam + BAM writer, MOS v1/v2, TIS both + TisConvert both directions + PNG export, PVR decode (DXT/ETC2/PVRTC), DxtEncoder, BMP dec+enc, GIF read/write, APNG writer, PLT; sprite decoder system (19 types + tables). | Decode C3 (RGBA hashes == oracle) across BG:EE BAM/MOS/TIS/PVRZ/BMP/PLT corpus; BamResource C1; conversion outputs C2/C3 with zlib caveat; sprite frame hashes C3 for a sampled animation set. |
| **P11 — Audio & video** (L) | ACM/WAVC decoders (faithful port), WAV, OGG via FFI, playback service, MUS sequencing player, MVE decoder + player, AVI muxer (original), WBM raw handling. | PCM C3 (hash == oracle) over sound corpus; MVE frame/audio C3 on movie corpus; AVI export plays in mpv/ffprobe-validates (C4). |
| **P12 — Tool services** (L) | Search engines (all §2.13 incl. advanced matching + XML filter round-trip), 15 checkers, mass-export pipeline, SAV pack/unpack ops, zip archiver, WeiDU service, update-check service, launch service, struct clipboard, temp-file service. | Result-set differential vs oracle on BG:EE (sorted, normalized) per searcher/checker (C4); advanced-search XML files interchangeable Java↔port; mass-export tree byte-compare per option matrix cell (C2 where conversion involved). |
| **P13 — App shell UI** (L) | First runnable app: shell + theming (M3 light/dark + follow-system), resource tree (display modes, bold-override, icons, sort, history), workspace tabs (G1 policy), menus, quick search, status bar, open/refresh/bookmarks/recents/launcher, preferences dialog over SettingsRepository (~90 options mapped), drag-drop, about/licenses, debug console, game properties. | Manual workflow pass: open BG:EE, browse, view raw hex of anything, switch games, prefs persist. Riverpod graph established per oracle-leg mapping (A6). |
| **P14 — Structure editing UI** (XL) | StructViewer parity (table + all Editable/InlineEditable editors, add/remove, context commands, clipboard), Raw tab (hex editor + struct coloring + find + undo/redo), text editor (highlight rules M30, folding for BCS), View tabs (ViewerUtil kit + per-format simple viewers), save/backup flows + dirty tracking, StringEditor, create-new dialogs, TOH/TOT viewer, SAV manager, BIFF editor. | Edit-and-save round-trips on golden cases match oracle byte-for-byte after identical edit scripts (§7.5); checker `StructChecker` clean on saved output. |
| **P15 — Rich viewers** (XL) | Area viewer (16 layers + stacking, tileset render with overlays/day-night, mini-maps, zoom, edit mode, settings, PNG export), DLG tree viewer (+options incl. break-cycles), worldmap viewer, creature browser, sound panel + InfinityAmp, MVE player UI, IDS browser, hex-view polish. | Feature-checklist parity against §2.16 census; PNG exports pixel-compare vs oracle where deterministic (C3). |
| **P16 — Converters & long tail** (L) | BAM converter (4 tabs, ~20 filters, session save/load, palette dialog), BMP/MOS/PVRZ/TIS (+batch) converters, SPLPROT converter, spell generator, script drop zone, mass exporter UI, WeiDU changelog UI, update-check UI, SpriteAnimationPanel easter egg; gap-register closure sweep. | Converter outputs C2/C3 vs oracle on shared inputs; full workflow census (§1.4) checked off or homed in the gap register. |

Critical path: P0→P1→P2→P3, then model fan-out; P13 requires P1–P4 minimum; P14 requires P3+P5+
formats; P15 requires P8+P10(+P11 for sound). The app is deliberately non-runnable before P13
(contract: model → UI; the harness is the feedback loop until then).

---

## 6. Assumptions (stated as assumptions)

- **A1 — Fixtures.** BG:EE (Steam) is locally present (verified: `chitin.key`, 83 BIFFs, 15
  languages, real save games). The other 18 games are **not** locally available; full-domain
  acceptance for their variants rests on synthetic fixtures until real installs are supplied
  (register entries per game). Position: proceed; per-game sweeps activate as fixtures arrive.
- **A2 — Java oracle drivability.** NearInfinity's load/save paths can run headless from a small
  driver jar in *our* repo against the unmodified upstream jar (reflection-constructed resources
  don't require Swing until `makeViewer`). Risk: hidden UI touchpoints (e.g. options reads) on
  some paths; fallback is GUI-driven export for affected formats. Verified at P0.
- **A3 — Upstream round-trip identity.** Struct formats are byte-identical on untouched
  read→write upstream (write = offset-sorted original fields). Known exceptions: encrypted text
  written decrypted; converted exports; zlib-recompressed containers. **Not assumed further than
  this** — P0's oracle baseline measures the actual per-format identity classes before any codec
  seed is cut.
- **A4 — Platform capability.** `dart:io` ZLib (raw+zlib) covers BIF/BIFC/SAV/BAMC/MOSC/PVRZ;
  FFI to system libs is available on Linux desktop; no zip-mount is needed (DLC zips are
  store-only, measured).
- **A5 — License surface.** The needed third-party surface stays as small as §4.5; anything
  larger triggers the contract's per-dependency license gate. No Apache-2.0 anywhere in the app.
- **A6 — Oracle-leg timing.** The Riverpod binding mapping (record §Riverpod) lands before the
  first UI seed (P13). Model phases do not block on it.
- **A7 — Single window.** Flutter/Linux multi-window is not mature enough to carry ChildFrame
  semantics; workspace tabs deliver workflow parity (G1). Revisited if the platform changes.
- **A8 — Settings.** No migration from `java.util.prefs`; fresh settings store (G5). Both apps
  can coexist against the same game data (NI's backup-on-save conventions are preserved).
- **A9 — Performance.** Dart parse throughput in isolates suffices for full-game sweeps and
  interactive browsing at BG:EE scale (~30k resources); measured at P1 via the enumeration
  benchmark before committing to the isolate-pool shape.
- **A10 — Upstream drift.** The port pins `b5a3b4b2`; upstream changes are adopted by conscious
  re-pin + plan delta, not tracked continuously.
- **A11 — Text write semantics.** `PlainTextResource`/`MusResource` write the edited text buffer
  without hidden normalization (EOL/trailing-space transforms are explicit user commands only).
  Verified in the P0 baseline; if false, affected formats move to C2 with the measured rule.
- **A12 — TLK cleanliness.** BG:EE TLKs decode cleanly under their declared charsets, so C1
  holds there; malformed-byte behavior (the reason `StringValidationChecker` exists) is matched
  to measured Java behavior (C2) on synthetic dirty TLKs.

---

## 7. Verifying the faithful half

### 7.1 The instrument

Round-trip byte identity (contract): read a real game file, write it back, compare bytes —
adversarial fixtures over the format's domain, not just today's samples.

### 7.2 Harness

`tool/roundtrip_sweep.dart` — pure-Dart CLI (no Flutter dependency): takes game roots
(`NI_FIXTURE_ROOTS` env or config), enumerates through the port's own KEY/override layer, and for
every resource with a codec: parse → serialize → compare; emits per-format × per-game JSON +
markdown (pass/fail/skip, first-diff offset + hex context); nonzero exit for CI. Synthetic corpus
(`fixtures/synthetic/`, committed, authored from the IESDP structure descriptions cross-checked
against the NI codecs — original work, **no game bytes committed**) runs everywhere including CI;
real-game sweeps run locally. A committed
hash-manifest of the BG:EE fixture set makes local runs reproducible-verifiable.

### 7.3 Comparator classes (assigned per format row; assignments are audit surface)

- **C1 — byte-identity vs the original file.** Default for every struct format, KEY, TLK, TOH/
  TOT, MUS, unencrypted text, BcsResource, BamResource-save. Anything less is a defect unless the
  P0 baseline proves upstream itself diverges.
- **C2 — byte-identity vs Java NI's output on the same input** (the oracle). For measured
  upstream normalizations: encrypted text (written decrypted), zlib-recompressed containers
  (BIF/BIFC/SAV members, BAMC/MOSC — compressed bytes may legitimately differ across zlib
  implementations; then C3 applies to the decompressed payload), conversion outputs. Every C2
  assignment carries a recorded rationale.
- **C3 — decoded-equality vs oracle goldens.** Graphics (RGBA frame hashes), audio (PCM hashes),
  video (frame + audio hashes), charset decoding. Golden hashes produced by the Java oracle
  driver.
- **C4 — semantic equality.** Search/check result sets (sorted, normalized), opcode name tables,
  BIFF-write re-read equivalence, decompiled scripts recompiling to identical bytes, AVI
  validation. Used only where C1–C3 cannot apply, with rationale.

### 7.4 The Java oracle

`tool/java_oracle/` — a small Java driver (lives in this repo; upstream stays untouched) linking
the upstream-built `NearInfinity.jar`: batch load→save→emit bytes + SHA-256 manifest; decode
dumps (RGBA/PCM/decompiled text) for C3/C4 goldens; scripted-edit mode for §7.5. Its first run
(P0) produces the **baseline identity report**: which formats upstream round-trips byte-identically
— fixing every format's comparator class from measurement rather than belief.

### 7.5 Edit-path verification

Round-trip-after-edit: scripted mutations (add/remove an `AddRemovable`, change an effect opcode
and re-layout, edit a strref, TLK entry add) applied identically in the port and the oracle;
outputs byte-compared. This is what protects the offset/count auto-adjust machinery — the part
whose failure corrupts installations.

### 7.6 Fixture plan

1. **Real (not committed):** BG:EE install + its save folder (verified present); other games as
   acquired (A1) — acquisition list: BG2EE, IWDEE, PSTEE, SoD, classic BG1/BG2/IWD/PST, IWD2, plus
   mod setups (Tutu/BGT/EET, TobEx/EEEx) for profile coverage.
2. **Synthetic (committed):** per format × version minimal files + adversarial cases, authored
   from IESDP layouts (header provenance; NI-wins precedence) and keyed to the
   struct machinery: holes ("Unused bytes?"), overlapping regions, zero-count/zero-offset
   sections, max/negative values, 3-byte fields, 260-byte PST Bestiary, dirty TLK bytes,
   encrypted 2DA/IDS/BCS, deep DLG cycles, headerless TIS, store-only DLC zip, DOS-cased paths.
   StructChecker's corruption taxonomy doubles as the adversarial checklist. IESDP-flagged
   variants from §1.6 join the matrix: CRE labelled `"V1.1"`/`"V9.1"`, BMP v5 header,
   `ini_anim`/`ini_spawn` dialects, `.cbf`-named compressed BIF, Ogg-content `.wav`/`.acm`.
3. **CI:** analyze + unit + synthetic sweep on every push; real-game sweep is a local/nightly
   target.

---

## 8. Gap-register seed and open questions

Initial register entries (deliberate divergences; the audit attacks both the list and the
positions):

| Id | Area | Position |
|---|---|---|
| G1 | ChildFrame multi-window → single-window workspace tabs; "open in new window" → new tab | A7; revisit on platform maturity |
| G2 | 52 FlatLaf themes → Material light/dark + follow-system | theme count is not parity-relevant; option semantics kept |
| G3 | Windows raw-keycode helper, macOS Quit glue | out of platform scope (contract §Platform) |
| G4 | JVM heap widget in status bar | replace with process-RSS readout |
| G5 | `java.util.prefs` migration | none; fresh settings (A8) |
| G6 | RSyntaxTextArea/JHexView/JFontChooser | Flutter-native rewrites; token rules ported (M30); hex undo/redo parity required (P14) |
| G7 | MonteMedia AVI export (LGPL-3.0 — cannot derive) | original RIFF/AVI muxer from spec |
| G8 | JavaCC-generated BAF parser | hand-written recursive descent from `BafParser.jjt`; C4 recompile-identity is the safety net |
| G9 | tinylog | `package:logging`; level mapping preserved incl. OFF |
| G10 | Reflective registries (resources, opcodes, BAM filters) | static tables/factories — Dart has no classpath reflection; behavior identical |
| G11 | `SwingWorker`/EDT | isolate pool + UI-isolate ownership (M31) |
| G12 | In-app WBM playback | none upstream either — parity is raw export; register documents the *shared* absence |
| G13 | BIK | unknown-resource fallback, as upstream |
| G14 | Update self-install | none upstream either (manual download); port keeps check+notify+open-browser |

Open questions the audit should rule on:

- **Q1** Comparator assignments (§7.3) — especially TLK C1 (A12) and text-format A11.
- **Q2** Effects table representation: pure data tables vs data+functions for the 10 dynamic
  `update()` opcodes and the param-hook variance — this plan says data+functions (M12).
- **Q3** Sprite system phase placement (P10 with graphics) vs deferral to P15 — plan says P10,
  because the area viewer's actor layer (P15) and the browser both need it.
- **Q4** Advanced-search XML compatibility as a hard requirement (plan: yes — user assets).
- **Q5** `SearchResource` ("Extended search (deprecated)") — port it (plan: yes, it is shipped
  behavior) or drop-with-register-entry; the audit may overrule toward dropping.
- **Q6** The `CharsetDetector` `"window-125x"` typo — determine actual runtime behavior at seed
  time and match *behavior*, not the string; ledger entry proposed (§4.4).
- **Q7** Whether `Profile`'s 91 keys port as a typed model (plan) or a keyed map (upstream shape).
- **Q8** BIFF-write acceptance: is C4 re-read equivalence + C2-decompressed sufficient, or must
  uncompressed BIFF writes be C1? (Plan: uncompressed BIFF C1; compressed variants C2/C3.)
- **Q9** The idle-panel easter egg (M38): parity item (plan) or drop.

---

*End of first-pass plan. Next pipeline step per the contract: commit this artifact, then
`/porting-bfb` audits it; corrections land as plan revisions before any seed is cut.*
