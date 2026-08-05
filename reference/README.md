# reference/ — local read-only reference material

This directory holds third-party reference snapshots used by the port. The content is **not
committed** (no redistribution license; see per-entry notes) — only this README is tracked. Treat
everything under here as **read-only**: it is source material, never edited, never imported into
`lib/`.

## IESDP — Infinity Engine Structures Description Project

- **Path:** `reference/iesdp-gh-pages/` (extracted) + `reference/iesdp-gh-pages.zip` (the archive,
  kept so the recorded hash stays verifiable)
- **Source:** https://github.com/gibberlings3/iesdp/archive/gh-pages.zip
  (the built site of https://gibberlings3.github.io/iesdp/)
- **Pinned:** gh-pages commit `f820d4e519184044afb892de7d95a9560fab4674` (embedded in the zip
  comment; tree dated 2026-07-03), downloaded 2026-08-05.
  Zip SHA-256: `6ebe5461dc69d5d0a8cea1ea5ffb488a72dda6de80f68ca1849413c88551459b`.
- **License:** the repository carries no license file; content is used here as local reference
  only and must not be committed, vendored, or redistributed with this project.
- **What it is:** the community description of Infinity Engine data — per-format, per-version
  structure layouts (`file_formats/ie_formats/`, e.g. `cre_v1.2.htm`, `are_v9.1.htm`,
  `key_v1.htm`, `encryption.htm`), effect opcode documentation (`opcodes/`), and scripting
  actions/triggers (`scripting/`), plus appendices.
- **Role in this project (precedence rule):** IESDP is **informative, not normative**. The
  faithful half's fidelity target is NearInfinity's measured behavior at the pinned SHA — where
  IESDP and the Java source disagree, the Java source wins, and the disagreement is worth
  recording (it is either an IESDP gap or a NearInfinity bug candidate). IESDP's uses here:
  independent cross-check when reading Java codecs, authoring source for the committed synthetic
  fixture corpus (`fixtures/synthetic/`), and reference material for the porting-BFB's
  comprehension legs. See `planning/first-pass-conversion-plan.md` §7.6.

**Refetch:** `curl -sSL -o iesdp-gh-pages.zip https://github.com/gibberlings3/iesdp/archive/gh-pages.zip`
— note that refetching tracks the branch head; verify or update the pinned commit/hash above.
