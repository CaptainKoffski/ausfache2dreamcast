# Akatsuki Blitzkampf Ausf. Achse → Dreamcast: project charter + Phase 1 Foundation spec

Approved in brainstorm 2026-11-02 (brainstorm opened 2026-10-07 from
`REQUIREMENTS.md`). Two parts, on the senkosp precedent
(`../senkosp2dreamcast/docs/superpowers/specs/2026-08-13-charter-phase1-foundation-design.md`):
a project-level charter (goal, decomposition, phases, risks; stable across
the port) and the full spec for Phase 1 (Foundation), the first sub-project.
Later sub-projects get their own specs when reached.

## Part A — Project charter

### Goal

Port *Akatsuki Blitzkampf Ausf. Achse* (`ausfache`, Naomi M4 cartridge
841-0058C, Subtle Style 2008) to Sega Dreamcast as a static binary
conversion: patch the Naomi-specific hardware touchpoints in the game
binary (cart reads → GD-ROM streaming, MIE/JVS input → Maple controllers,
EEPROM/settings → shims and the pre-game menu, free play baked in) and boot
it from a disc via a custom loader.

**Done means** the playbook's Definition of done in full
(`../naomi2dreamcast/docs/kb/port-playbook.md` §Definition of done):

1. Every boot path PASS on the release candidate, verdicts recorded: GDEMU-class
   ODE, DreamShell serial-SD, burned CD-R; VGA and composite.
2. Two disc formats, GDI and CDI.
3. Fully playable on a standard DC pad and on an arcade stick.
4. The game's arcade settings (test-mode GAME ASSIGNMENTS) reachable from
   the pre-game menu.

Plus the VMU-safety tripwires (the port never writes a VMU), a pipeline
reproducible from the KB, and findings documented so a fresh agent can
retrace the port. The emulator is a dev tool, never the verdict.

### Decomposition and order (decided 2026-10-07)

Four sub-projects, each with its own spec and plan (superpowers loop per
sub-project, as the playbook's working cadence prescribes):

1. **Charter + Phase 1 Foundation** — this spec, this repo. Runs now.
   Consumes nothing from the kit.
2. **Port kit** — a new repo, `naomi-dc-port-kit`, extracted from
   senkosp2dreamcast's game-agnostic core (loader, shims, mastering and
   capture scripts, test harnesses, menu/controls generators). This is the
   tooling-extraction handoff the 2026-07-26 reuse spec deferred
   (`../cleopatra/docs/superpowers/specs/2026-07-26-experience-reuse-design.md`
   §3 bucket 3). Its own brainstorm happens in that repo. Its gate, fixed
   here so the kit cannot drift from its only proven consumer: **senkosp
   rebuilt against the kit produces byte-identical `shim.bin`,
   `1ST_READ.BIN` and disc tracks, and senkosp's `make test` stays green**
   (the same control the 2026-09-21 KOS decoupling used,
   `../senkosp2dreamcast/docs/kb/tooling.md` §Decoupling). Extraction base
   = senkosp's `main` at the commit current when the kit brainstorm starts
   (senkosp is still moving: VMU persistence work landed 2026-10-24 to
   2026-11-01).
3. **Phases 2 to 6** of the playbook, each on the kit, each with its own
   spec and plan when its gate is reached. Phase 2 does not start until the
   kit has shipped.

Alternatives rejected in brainstorm: phase-by-phase copy of senkosp into
this repo (what senkosp did with cleopatra; a third copy of the same code
is the wrong direction), and forking senkosp wholesale (drags seven phases
of senkosp-only patch tables, relocation maps and menu definitions that
would have to be unpicked, and muddies the phase gates). Rejected at
charter level for cause during Cleopatra, still rejected: a trap-based
generic Naomi runtime (Naomi cart registers and DC GD-ROM ATA registers
share addresses) and asset extraction / engine reimplementation.

### Phase table with the ausfache deltas

| Phase | Gate | Ausfache-specific notes |
|---|---|---|
| 1. Foundation | Untouched romset boots to attract in the fork's Naomi profile | `.dat` via `cart2dat.py` (M4: assemble two 64 MiB chips, then `m4dec`). No byte-exact reference exists for carts, so validation = header parse + boot. Fork build recorded as a hash. Spec: Part B. |
| 2. Instrumented analysis | Streaming, input, memory truth from real traces | Settle the M4 questions (§Risks): does the game write bit 30 into ROM_OFFSETH, does it read cart-ID register `0x34` or the PIC-present bit. Confirm the assessment's 1 MB PIO volume is the BIOS boot load (code is exactly 1 MiB; handoff trigger was `pio`), using the entry-PC gate. Real-play legs (roster, 2P, test menu), not attract only. Measure top-of-RAM headroom for the isoldr carve. |
| 3. Reverse engineering | Every hardware touchpoint has an exact address and a patch plan | Base `0x8c020000`, entry `0x8c021000` (assessment §6). SDK: Kunoichi2 Library for NAOMI 2.07, syStartCwKn 2.08, AIPKN 0.91, the same era as senkosp's Kunoichi2 2.07, so expect the BIOS-thunk EEPROM literals again; guts flags `eeprom_bios`, `serial`, `rtc`. |
| 4. Conversion | Boots and plays in the fork's Dreamcast profile | On the kit: dual-backend GD driver, three-button controls (pad presets per port + fixed stick layout, device-classified per poll), pre-game menu after a GAME ASSIGNMENTS recon. Cart image 128 MiB, half senkosp's 245 MB; fits the track-4 layout and a CD-R with margin. |
| 5. Hardware test & fit | Every boot path PASS, controls matrix PASS | Operator rig: one console, GDEMU-class ODE, DreamShell serial-SD, VGA + composite. **Tester-run legs:** burned CD-R on a stock console, arcade-stick matrix. Schedule the tester legs early in Phase 5, not at release. |
| 6. Tripwires & release | Three VMU tripwires + two static checks PASS; both zips + DS preset | Kit harness (`test_maple_literals.py`, `test_vmu_untouched.sh`, serial-compiled-out and no-debug-sleep checks). |

### Risk watch list

Source for the numbers: `../naomi2dreamcast/assessments/ausfache.md` (84.4 S,
capture v9, 2026-08-09, fork `f014a410c`). The assessment is a starting
point; Phase 2 re-measures.

- **M4 cart semantics** (new vs both prior ports, which were GD-ROM/DIMM).
  Grounded in the fork's cart model, primary source:
  - The encrypted-read flag is **bit 30 of the PIO offset register**:
    `M4Cartridge::PioOffsetChanged` sets `encryption = pio_offset & 0x40000000`
    (`../flycast4naomi2dreamcast/core/hw/naomi/m4cartridge.cpp`), and that
    flag governs both PIO and DMA reads. `NAOMI_DMA_OFFSETH` keeps only
    `(data & 0x7fff) << 16` (`naomi_cart.cpp` register-write switch), so
    bit 30 can only arrive through `ROM_OFFSETH`. Reads mask the address
    with `0x1ffffffe`.
  - The cart answers two probes a plain cart does not: register `0x34`
    returns the cart ID (`m4id & 0xff80`, ausfache `0x5504` per
    `naomi_roms.cpp:4575`) and `NAOMI_ROM_OFFSETH` reads carry `0x2000`
    (security PIC present) (`m4cartridge.h` `ReadMem`).
  - The disc serves plaintext (`cart2dat.py` output), so the shim must
    treat bit 30 as a mode flag, never an address bit, and must answer any
    cart-ID or PIC-present probe the game or the BIOS thunks make. Which
    of these the game actually exercises is a Phase 2 measurement; the
    Phase 1 flat-image control test (Part B §5) gives the first signal.
- **Memory.** Main RAM: write-truth content 8.0 MB, touched address peak
  16,349,952 B (0.975× the DC's 16 MiB). Everything the game writes sits
  under the line, so relocation is probably unnecessary, unlike senkosp;
  but the top-of-RAM headroom is about 420 KB and the isoldr carve
  (senkosp: 64 KB at `0x8cff0000`) and loader staging must be fitted
  there. VRAM: address peak 7,892,608 B (0.941× 8 MiB) with 4.4 MB
  content; senkosp's VQ or relocation treatment may return. ARAM:
  content 1.56 MB of 2 MiB, address peak 16 B under the cap from a
  uniform boot fill; fits if that fill is really uniform.
- **Assessment coverage is attract-only** (`capture.coverage = "demo"`).
  Re-measure with real play (senkosp's 2P-stages leg found 2.1 MB of
  territory attract never touched).
- **Controls.** 8-way stick + 3 buttons (Weak/Medium/Strong) + Start, 2P.
  Simpler than senkosp's five, but both layouts (pad presets, fixed stick)
  are still required by Definition of done §3.
- **GAME ASSIGNMENTS unknown.** Needs the EEPROM byte-map recon
  (`eeprom_game_diff.py` method) before the menu can be defined, Phase 4.
- **Tester-run verdicts.** The CD-R and arcade-stick legs depend on people
  outside the rig. Each verdict is recorded with date, rig, build hash and
  the tester's wording, the way senkosp's phase files table them.

### Assumptions

- Operator rig: real Dreamcast + GDEMU-class ODE + DreamShell serial-SD,
  VGA and composite (same rig as senkosp). No arcade stick, no CD-R burner
  or optical-drive console on the rig; the operator's testers run those
  legs (user, 2026-10-07).
- Sibling repos stay in place and are referenced by relative path:
  `../naomi2dreamcast` (assessment, dat-extract, playbook),
  `../flycast4naomi2dreamcast` (instrumented emulator, source of truth and
  the built app), `../senkosp2dreamcast` (the implementation to extract the
  kit from), `../cleopatra` (lineage only). `../naomi-dc-port-kit` once
  sub-project 2 creates it.
- No netplay or online features. The one set that exists (Japan, PIC
  `317-05130-jpn`) is the set we ship for.
- Asset cutting or compression is a last resort; the assessment says
  everything fits.

### Rules

- Never commit copyrighted bytes: ROM, BIOS, disc images, `.dat`,
  extracted assets stay gitignored. Ship method, tools, patches.
- Every hardware/behavioral claim carries a citation; primary sources
  (emulator/kernel/library source) outrank wikis and forums.
- Every tool install or reuse is recorded in `docs/kb/tooling.md` with
  version, flags and exact steps.
- Verify on real hardware; the emulator is a dev tool.
- **The emulator always runs headless** (`FLYCAST_HEADLESS=1`, fork commit
  `a54f70116`: hidden SDL window, background app that never activates,
  null audio; `FLYCAST_SHOT`/`_RAWFB` dumps and the cartlog still work,
  `INSTRUMENTATION.md` §Headless screenshots). Never a visible window or
  sound unless the user asks for one. Recorded in `CLAUDE.md` (user
  instruction 2026-11-02).
- All findings land in `docs/kb/`; a lesson that generalizes is proposed
  as a playbook update in `../naomi2dreamcast`.

## Part B — Phase 1: Foundation spec

### Purpose

Stand up the repo so every later phase has its infrastructure: knowledge
base, verified and recorded toolchain, the ROM in workable form, and proof
the untouched game runs. Produces no game modifications.

### Deliverables

1. **Repo scaffolding.** The scaffold commit already carries `CLAUDE.md`,
   the hardened `.gitignore` (`roms/`, `bios/`, `*.dat`, `*.gdi`, `*.cdi`,
   `._*`, …) and the playbook pointer. Phase 1 adds the KB skeleton:
   `docs/kb/00-status.md` (living status index, senkosp-style: what this
   is, decisions, phases, Phase 1 checklist, key facts, next step),
   `docs/kb/game.md` (identity from the assessment + parsed header facts),
   `docs/kb/tooling.md` (install/reuse records), `docs/kb/img/`
   (evidence). `CLAUDE.md` gains the headless rule and points at this
   spec. `REQUIREMENTS.md` stays as the origin document.

2. **Toolchain verified and recorded** in `tooling.md`:
   - **Instrumented Flycast**: the built app
     `../flycast4naomi2dreamcast/build/Flycast.app/Contents/MacOS/Flycast`.
     At brainstorm time: fork HEAD `4a7e80240`, binary built 2026-10-24
     19:16, which by mtime includes two then-uncommitted logging-only
     changes (`FLYCAST_WRWATCH` write watch in `core/hw/mem/addrspace.cpp`,
     `BLOCKEDEXC` diagnostic in `core/hw/sh4/sh4_interrupts.cpp`). Phase 1
     records HEAD and, if those changes are still uncommitted when it runs,
     commits them in the fork (own GPL instrumentation, no push) so the
     build this port relies on is a hash, not a hash plus a diff. Build
     recipe: reference `../senkosp2dreamcast/docs/kb/tooling.md`
     §Decoupling; do not rebuild unless the binary is older than HEAD.
   - **dat-extract** (`../naomi2dreamcast/tools/dat-extract/`): record
     location and invocation. Its `ROMS` default (`cart2dat.py:19-20`)
     points at the retired `../cleopatra/tools/flycast-src` checkout;
     Phase 1 runs it with `NAOMI_ROMS=` pointing at the canonical fork's
     `core/hw/naomi/naomi_roms.cpp` and records that.
   - **Naomi BIOS**: copy `naomi.zip` from `../senkosp2dreamcast/bios/`
     into this repo's gitignored `bios/`, md5 recorded; verify Flycast's
     data directory (`~/Library/Application Support/Flycast/data/`) holds
     it (the assessment already booted ausfache there).
   - **Ghidra 12.1.2, KOS, mkdcdisc, cdi4dc, dcload**: recorded as reuse
     pointers to senkosp's `tooling.md` only; exercised by later phases.

3. **`.dat` extraction.** Copy `../naomi2dreamcast/naomi/ausfache.zip`
   into `roms/` (md5 both sides). Run `python3 cart2dat.py ausfache`,
   copy `out/ausfache.dat` to `ausfache.dat` at repo root (gitignored,
   `git check-ignore` verified first). Validate with
   `../cleopatra/scripts/parse_header.py`: `NAOMI` magic, a real title,
   sane load entries, base `0x8c020000` and entry `0x8c021000` against
   the assessment (§6, which notes the header's load-table offsets carry
   the bit-30 encrypted-read flag, masked by `carve_boot.py`). Record the
   tool's `OK` line, byte size and the parsed header in `game.md` and
   `tooling.md`. If title or entry disagree with the assessment, stop:
   wrong PIC or bad extraction.

4. **Boot verification, the gate.** The untouched romset (`roms/ausfache.zip`)
   in the fork's Naomi profile, headless, with `FLYCAST_CARTLOG` and
   `FLYCAST_SHOT`: reaches title and attract/demo gameplay. Timeline from
   the assessment: BIOS→game handoff at 20 s, live demo fight by ~10 min
   (`evidence/ausfache/shot-609s.png`). On-demand grabs via `kill -USR1`.
   Two screenshots into `docs/kb/img/` (title, demo). If it will not boot,
   control-test senkosp's romset through the same binary before
   theorizing about our setup (playbook gotcha #2).

5. **Flat-image control test, stretch.** Boot `ausfache.dat` directly in
   the same binary. The fork loads a flat file as a `DecryptedCartridge`
   (`naomi_cart.cpp:610`, `naomi_cart.h:112-116`): a plain cartridge with
   no M4 register behaviour. A boot to attract means neither the BIOS nor
   the game needs the cart-ID / PIC-present answers; a stall means Phase 2
   must capture which probe is missed. Either outcome, with its cartlog,
   is a Phase 2 input. Not part of the gate.

6. **Exit audit + fresh-agent test.** Copyright tripwire over tracked
   files (`git ls-files | grep -iE '\.(dat|chd|gdi|cdi|iso|zip|bin)$'`
   must be empty), AppleDouble/`.DS_Store` scan, `git status` clean of
   derived bytes. Then a clean-context agent must identify the project,
   its state and the next step from `CLAUDE.md` + `00-status.md` alone.

### Out of scope

Any patching, any reverse engineering beyond the header/carve sanity
check, disc mastering, Flycast instrumentation changes (committing the
fork's pending diff is bookkeeping, not instrumentation work), and the port
kit (sub-project 2, its own spec in its own repo).

### Gate to Phase 2

Untouched game reaches attract in the Naomi profile (screenshots in the KB)
+ toolchain recorded + `.dat` header verified + the port kit has shipped
(sub-project 2 gate green). The flat-image control test result is recorded
either way.
