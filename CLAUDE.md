# Akatsuki Blitzkampf Ausf. Achse — Naomi → Dreamcast port

Port of the Sega Naomi **M4 cartridge** game *Akatsuki Blitzkampf Ausf. Achse*
(`ausfache`, 841-0058C, Subtle Style 2008) to Sega Dreamcast by static binary
conversion (no source code), following the method proven by the Cleopatra
Fortune Plus and Senko no Ronde Special ports.

- **Start here:** `docs/kb/00-status.md` — project state, strategy, next step; `REQUIREMENTS.md` is the origin brief.
- **Method:** `../naomi2dreamcast/docs/kb/port-playbook.md` — the port
  playbook, maintained in the umbrella repo (gates enforced, spec + plan per
  phase, Definition of done); `docs/kb/port-playbook.md` here is just the
  pointer.
- **Knowledge base:** `docs/kb/` — game notes, tooling records, findings.
- **Specs & plans:** `docs/superpowers/specs/`, `docs/superpowers/plans/`.
  Charter + sub-project order (Phase 1 → port kit `../naomi-dc-port-kit` →
  Phases 2–6): `docs/superpowers/specs/2026-11-02-charter-phase1-foundation-design.md`.
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
    implementation the port kit (`../naomi-dc-port-kit`, sub-project 2)
    is extracted from (dual-backend GD driver, controls layer,
    pre-game settings menu, GDI + CDI mastering).
  - `../naomi-dc-port-kit` — the port kit, shipped 2026-11-03 (tag
    `v0.1.0`): consumed as a git submodule at
    `kit/`; the contract (what a port supplies) is its `README.md`.
  - `../cleopatra` — the first port (cfp2dreamcast): KB and method lineage.

Rules: every hardware claim in the KB carries a citation (primary sources
outrank wikis); record every tool install in `docs/kb/tooling.md`; never
commit copyrighted bytes (ROMs, BIOS, disc images, extracted assets);
verify on real hardware — the emulator is a dev tool, never the verdict.

# Flycast fork

Always launch `../flycast4naomi2dreamcast` builds with `FLYCAST_HEADLESS=1`
(hidden window, background app, null audio; `FLYCAST_SHOT` dumps and the
cartlog still work). Never run it with a visible window or sound unless the
user asks for one.
