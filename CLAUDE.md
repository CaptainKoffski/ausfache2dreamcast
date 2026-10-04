# Akatsuki Blitzkampf Ausf. Achse — Naomi → Dreamcast port

Port of the Sega Naomi **M4 cartridge** game *Akatsuki Blitzkampf Ausf. Achse*
(`ausfache`, 841-0058C, Subtle Style 2008) to Sega Dreamcast by static binary
conversion (no source code), following the method proven by the Cleopatra
Fortune Plus and Senko no Ronde Special ports.

- **Start here:** `REQUIREMENTS.md` (project brief); once Phase 1 creates it,
  `docs/kb/00-status.md` — project state, strategy, next step.
- **Method:** `../naomi2dreamcast/docs/kb/port-playbook.md` — the port
  playbook, maintained in the umbrella repo (gates enforced, spec + plan per
  phase, Definition of done); `docs/kb/port-playbook.md` here is just the
  pointer.
- **Knowledge base:** `docs/kb/` — game notes, tooling records, findings.
- **Specs & plans:** `docs/superpowers/specs/`, `docs/superpowers/plans/`.
- **ROM:** `roms/` (gitignored — never commit, never upload): `ausfache.zip`
  (cart romset; library copy at `../naomi2dreamcast/naomi/ausfache.zip`).
  Flat decrypted image `ausfache.dat` at repo root (gitignored, regenerable
  with `../naomi2dreamcast/tools/dat-extract/cart2dat.py ausfache`).
- **BIOS:** `bios/` (gitignored).
- **Sibling repos:**
  - `../naomi2dreamcast` — Naomi library assessment; this game's report:
    `assessments/ausfache.md`. Also the `.dat` toolset (`tools/dat-extract/`).
  - `../flycast4naomi2dreamcast` — the instrumented Flycast fork.
  - `../senkosp2dreamcast` — the most recent, complete port: the
    implementation to copy (dual-backend GD driver, controls layer,
    pre-game settings menu, GDI + CDI mastering).
  - `../cleopatra` — the first port (cfp2dreamcast): KB and method lineage.

Rules: every hardware claim in the KB carries a citation (primary sources
outrank wikis); record every tool install in `docs/kb/tooling.md`; never
commit copyrighted bytes (ROMs, BIOS, disc images, extracted assets);
verify on real hardware — the emulator is a dev tool, never the verdict.
