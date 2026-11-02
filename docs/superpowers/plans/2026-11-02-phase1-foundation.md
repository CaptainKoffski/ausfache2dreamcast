# Phase 1 (Foundation) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stand up the ausfache2dreamcast repo: KB skeleton, recorded toolchain, the M4 cart decrypted to a flat `ausfache.dat`, and headless-emulator proof that the untouched game boots to attract, per the approved spec Part B.

**Architecture:** No game modifications. Repo scaffolding, tool verification with recorded steps, one ROM-format conversion (M4 cart romset → flat decrypted `.dat` via `cart2dat.py`), two evidence-backed emulator legs (romset boot = the gate; flat-image boot = the M4 control test). Sibling repos are referenced in place: `../naomi2dreamcast` (assessment, dat-extract), `../flycast4naomi2dreamcast` (fork source + built app), `../senkosp2dreamcast` (BIOS copy, tooling recipes), `../cleopatra` (`parse_header.py`).

**Tech Stack:** git, zsh, Python 3 (stdlib), clang (dat-extract auto-builds `m4dec`), the instrumented Flycast fork build (`../flycast4naomi2dreamcast/build/Flycast.app`), macOS `defaults`/`sips`.

**Spec:** `docs/superpowers/specs/2026-11-02-charter-phase1-foundation-design.md` (Part B; Part A is the charter the KB cites).

## Global Constraints

- **Never commit copyrighted bytes.** `roms/`, `bios/`, `*.dat`, `captures/`, `*.zip` are gitignored. Before `git add` of anything binary run `git check-ignore -v <file>`; game/BIOS data that is NOT ignored = stop and fix `.gitignore`. Screenshots in `docs/kb/img/` are fine (established practice).
- **Every hardware/behavioral claim in KB docs carries a citation**; primary sources (fork/MAME source, assessment sidecar) outrank wikis.
- **Every tool install or reuse is recorded** in `docs/kb/tooling.md` with version, exact steps, gotchas.
- **The emulator always runs headless:** `FLYCAST_HEADLESS=1` on every launch (`CLAUDE.md`; fork commit `a54f70116`). Never a visible window or sound.
- **No pushes** to any sibling repo. Commits in `../flycast4naomi2dreamcast` are local bookkeeping only (Task 2).
- **Do not rebuild Flycast** in this phase; record the existing binary's provenance.
- Commit trailer on every commit: `Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>`.
- All paths below are relative to `/Users/captainkoffski/AntigravityProjects/ausfache2dreamcast` unless absolute.

## Review Focus

1. **A stale Flycast instance or the macOS window-restore modal** makes a launch silently never boot; a reasonable person expects the leg to either boot or fail loudly. Pinned in Task 4 Step 1 (pkill by PID file + `defaults write` before every launch) and the stdout log check in Step 3.
2. **The SH4 vmem `Verify Failed` startup flake** (known host flake, `../senkosp2dreamcast/scripts/test_vmu_save.sh` header) kills a run in its first seconds; expected behavior: relaunch, not a FAIL verdict. Pinned in Task 4 Step 3 (stdout grep, up to 3 relaunches).
3. **A periodic screenshot sampling a black transition frame** must not be read as "no video". Pinned in Task 4 Step 4 (on-demand `kill -USR1` re-grab before any conclusion).
4. **The M4 header's load-table offsets may carry bit 30** (encrypted-read flag, assessment §6); a validator reading `ROM 0x40000000` must not call the image corrupt. Pinned in Task 3 Step 3 (explicit expected forms).
5. **The romset copy in `roms/` silently differs from the library copy** the assessment booted (truncated Finder copy, wrong revision); expected: byte-identical or stop. Pinned in Task 3 Step 1 (md5 both sides must match).

---

### Task 1: KB skeleton — 00-status.md, game.md

**Files:**
- Create: `docs/kb/00-status.md`
- Create: `docs/kb/game.md`
- Create: `docs/kb/img/.gitkeep`

**Interfaces:**
- Consumes: the committed scaffold (`CLAUDE.md`, `.gitignore`, `docs/kb/port-playbook.md` pointer) and the spec.
- Produces: `docs/kb/00-status.md`, the living status doc every later task appends to (Phase 1 checklist, Key facts, Next step sections by those exact headings). `docs/kb/tooling.md` does NOT exist yet (Task 2 creates it).

- [ ] **Step 1: Verify the ignore rules fire for every artifact this phase creates**

Run: `git check-ignore roms/ausfache.zip bios/naomi.zip ausfache.dat captures/phase1/boot.log ._x .DS_Store && git status --short`
Expected: all six paths print (= ignored); `git status --short` is empty (`.idea/` is ignored by the scaffold).

- [ ] **Step 2: Write docs/kb/00-status.md**

```markdown
# Project status

**Updated:** <today> (Phase 1 Foundation — in progress)

## What this is

Static binary conversion of *Akatsuki Blitzkampf Ausf. Achse* (Sega Naomi
M4 cartridge 841-0058C, `ausfache`, Subtle Style 2008) to Sega Dreamcast:
patch the Naomi-specific hardware touchpoints in the game binary (cart
reads → GD-ROM streaming, MIE/JVS input → Maple controllers,
EEPROM/settings → shims + pre-game menu, free play baked in) and boot it
from a disc via a custom loader. Charter + Phase 1 spec:
`docs/superpowers/specs/2026-11-02-charter-phase1-foundation-design.md`.

## Decisions

- Target: real Dreamcast hardware. The emulator is a dev tool, never the
  verdict. Definition of done = the playbook's four items + VMU tripwires
  (`../naomi2dreamcast/docs/kb/port-playbook.md`).
- Sub-project order (charter §Decomposition): **Phase 1 (now) → port kit
  (new repo `../naomi-dc-port-kit`, extracted from senkosp, byte-identical
  senkosp rebuild as its gate) → Phases 2–6 on the kit.** Phase 2 waits
  for the kit.
- Emulator always headless (`FLYCAST_HEADLESS=1`, `CLAUDE.md`).
- Tester-run hardware legs: burned CD-R on a stock console and the
  arcade-stick matrix are executed by the operator's testers (rig has
  neither); verdicts recorded with date, rig, build hash, tester wording.
- Asset cutting/compression is a last resort (REQUIREMENTS.md).

## Phases

1. **Foundation — IN PROGRESS** (KB, tooling records, `.dat`, boot verification)
2. Instrumented analysis (streaming/input/memory truth; the M4 questions) — after the kit
3. Reverse engineering (touchpoint addresses + patch plan)
4. Conversion (loader + shim + patch table → bootable GDI, on the kit)
5. Real-hardware testing & fit (operator + tester legs)
6. Safety tripwires & release

## Phase 1 checklist

- [ ] KB skeleton (00-status, game, tooling, img/)
- [ ] tooling.md — Flycast build provenance, dat-extract, BIOS, reuse pointers
- [ ] ausfache.dat extracted (cart2dat, M4 assemble + m4dec), header verified vs assessment
- [ ] Boot verification: untouched romset reaches title + demo, headless, screenshots in KB (THE GATE)
- [ ] Flat-image control test (`ausfache.dat` boots as a plain cartridge?) — result recorded either way
- [ ] Exit audit + fresh-agent test

## Key facts so far

- `ausfache`: only member of its family; Naomi M4 cart 841-0058C, rom_board
  `5504`, 2× 64 MiB flash, PIC `317-05130-jpn.ic3`, ROT0. Never ported to
  any platform. Assessment 84.4 (S) — `../naomi2dreamcast/assessments/ausfache.md`
  (capture v9, 2026-08-09, fork `f014a410c`).
- Boots and reaches live demo gameplay under the fork (assessment §3:
  handoff 20 s, trigger `pio`, demo by 609 s).
- Memory (assessment §4): main write-truth 8.0 MB, address peak 15.6 MB
  (0.975× 16 MiB, under the line, ~420 KB headroom at the top); VRAM
  content 4.4 MB, address peak 7.9 MB (0.941×); ARAM content 1.56 MB.
- M4 streaming semantics (charter §Risks, fork `m4cartridge.cpp`): bit 30
  of the PIO offset register = encrypted-read mode flag; cart-ID register
  `0x34` and the PIC-present bit on ROM_OFFSETH reads are the two probes a
  plain cart does not answer. The disc will serve plaintext.
- Controls: 8-way stick + 3 buttons (Weak/Medium/Strong) + Start, 2P.
- Guts flags for Phase 3: `eeprom_bios`, `serial`, `rtc`; SDK Kunoichi2
  2.07 / syStartCwKn 2.08 / AIPKN 0.91.

## Next step

Finish Phase 1 per `docs/superpowers/plans/2026-11-02-phase1-foundation.md`.
```

- [ ] **Step 3: Write docs/kb/game.md**

```markdown
# ausfache — game dump notes

Identity facts sourced from `../naomi2dreamcast/assessments/ausfache.md`
(battery v9, 2026-08-09) unless noted.

| | |
|---|---|
| Title | Akatsuki Blitzkampf Ausf. Achse (Japan) — header title filled below |
| Set | `ausfache` — single member, no parent, no clones (MAME `src/mame/sega/naomi.cpp` @59e7c0b GAME line 11116, `ROM_START` lines 6808–6822) |
| Maker / year | Subtle Style, 2008 (arcade debut 2008-02-20) |
| Media | Naomi **M4 cartridge** 841-0058C, rom_board id `5504`, 2× 64 MiB flash (`ic8.bin`, `ic9.bin`), ROT0; fork `naomi_roms.cpp:4571-4585` (M4, key `0x5504`) |
| Security PIC | `317-05130-jpn.ic3` (CRC `eccdcd59`, clean dump) |
| Carve (assessment guts) | base `0x8c020000`, entry `0x8c021000`, code 1,048,576 B, 2,111 functions; header load offsets carry the M4 bit-30 flag (assessment §6, `carve_boot.py` mask) |
| SDK stack | Kunoichi2 Library for NAOMI 2.07, syStartCwKn 2.08, AIPKN 0.91 (Mar 2001 builds) |
| Controls | 8-way stick + 3 buttons (A/B/C = Weak/Medium/Strong), 2P — fork `naomi_roms_input.h:195` |

## Our dump

(Filled by Task 3: `roms/ausfache.zip` md5, identical to `../naomi2dreamcast/naomi/ausfache.zip`.)

## Parsed .dat header

(Filled by Task 3 from `parse_header.py` output.)
```

- [ ] **Step 4: Create the img dir and commit**

```bash
mkdir -p docs/kb/img && touch docs/kb/img/.gitkeep
git add docs/kb/00-status.md docs/kb/game.md docs/kb/img/.gitkeep
git commit -m "Phase 1: KB skeleton — 00-status, game notes

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 2: Toolchain record — tooling.md, fork provenance, BIOS copy

**Files:**
- Create: `docs/kb/tooling.md`
- Create: `bios/naomi.zip` (gitignored copy)
- Modify (sibling, local commit only): `../flycast4naomi2dreamcast` working tree

**Interfaces:**
- Consumes: KB skeleton from Task 1.
- Produces: `docs/kb/tooling.md` with the verified values Tasks 3–5 use: `FLYCAST_BIN=/Users/captainkoffski/AntigravityProjects/flycast4naomi2dreamcast/build/Flycast.app/Contents/MacOS/Flycast`, `NAOMI_ROMS=/Users/captainkoffski/AntigravityProjects/flycast4naomi2dreamcast/core/hw/naomi/naomi_roms.cpp`, the fork HEAD hash, the headless launch line.

- [ ] **Step 1: Capture the fork's state**

```bash
cd ../flycast4naomi2dreamcast
git rev-parse --short HEAD                      # spec-time value: 4a7e80240
git status --short                              # spec-time: M core/hw/mem/addrspace.cpp, M core/hw/sh4/sh4_interrupts.cpp, m core/deps/Syphon, ?? build-*/
git diff --stat
stat -f '%Sm %N' build/Flycast.app/Contents/MacOS/Flycast core/hw/mem/addrspace.cpp core/hw/sh4/sh4_interrupts.cpp
file build/Flycast.app/Contents/MacOS/Flycast   # expect: Mach-O 64-bit executable arm64
grep -c FLYCAST_HEADLESS build/Flycast.app/Contents/MacOS/Flycast   # expect: 1 (headless knob present in the binary)
```

Record every output. The binary mtime must be newer than both modified sources (spec-time: binary 2026-10-24 19:16:05, sources 19:16:03 and 18:17:58) — that is the evidence the build includes the pending diff.

- [ ] **Step 2: Commit the pending logging-only instrumentation in the fork (local, no push)**

Only if `git status --short` still shows the two modified `.cpp` files. Read `git diff core/hw/mem/addrspace.cpp core/hw/sh4/sh4_interrupts.cpp` first and confirm it is the `FLYCAST_WRWATCH` write-watch + `BLOCKEDEXC` ERROR_LOG diagnostic and nothing else (35 insertions, 0 deletions at spec time). If the diff differs from that description, STOP and record the actual diff in `tooling.md` instead of committing.

```bash
cd ../flycast4naomi2dreamcast
git add core/hw/mem/addrspace.cpp core/hw/sh4/sh4_interrupts.cpp   # NOT the Syphon submodule, NOT build-*/
git commit -m "WRWATCH write watch + BLOCKEDEXC exception diagnostic (logging only; ausfache phase 1 bookkeeping)

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
git rev-parse --short HEAD    # the hash tooling.md cites as the build this port runs
git status --short            # expect: only ' m core/deps/Syphon' and '?? build-*/'
```

- [ ] **Step 3: Copy the Naomi BIOS and verify Flycast's install**

```bash
mkdir -p bios && cp ../senkosp2dreamcast/bios/naomi.zip bios/naomi.zip
md5 ../senkosp2dreamcast/bios/naomi.zip bios/naomi.zip ~/Library/Application\ Support/Flycast/data/naomi.zip   # all three equal
git check-ignore -v bios/naomi.zip   # must match /bios/
ls -la ~/Library/Application\ Support/Flycast/data/ausfache.zip.eeprom   # exists: the assessment already booted ausfache here
```

- [ ] **Step 4: Verify dat-extract prerequisites**

```bash
ls -la ../naomi2dreamcast/tools/dat-extract/{cart2dat.py,m4dec,m4dec.c,extract_dat}
/opt/homebrew/bin/7zz | head -2          # 7-Zip banner
clang --version | head -1
sed -n '19,22p' ../naomi2dreamcast/tools/dat-extract/cart2dat.py   # ROMS default points at ../cleopatra/tools/flycast-src (retired)
ls ../cleopatra/scripts/parse_header.py
ls ../cleopatra/tools/ghidra_12.1.2_PUBLIC/support/analyzeHeadless ../senkosp2dreamcast/tools/kos/environ.sh ../senkosp2dreamcast/tools/mkdcdisc   # reuse pointers exist
```

- [ ] **Step 5: Write docs/kb/tooling.md**

```markdown
# Tooling

Every tool this project uses: exact install/reuse steps, version, usage.
The environment must be rebuildable from this file. Deep recipes that live
in `../senkosp2dreamcast/docs/kb/tooling.md` are referenced, not duplicated.

### Instrumented Flycast (reused build)

- **Binary:** `../flycast4naomi2dreamcast/build/Flycast.app/Contents/MacOS/Flycast`
  (Mach-O arm64, built <binary mtime>), fork HEAD `<hash from Task 2 Step 2>`.
  Provenance: at Phase 1 start the tree was `4a7e80240` + two uncommitted
  logging-only changes (`FLYCAST_WRWATCH`, `BLOCKEDEXC`) that the binary
  includes by mtime; committed locally <date> as `<hash>` (no push).
- **Source of truth:** `../flycast4naomi2dreamcast`. Build recipe (submodules,
  Syphon patch, cmake flags): `../senkosp2dreamcast/docs/kb/tooling.md`
  §Decoupling (2026-09-21). Not rebuilt in Phase 1.
- **Headless (always):** `FLYCAST_HEADLESS=1` — hidden SDL window, background
  app that never activates, null audio; `FLYCAST_SHOT`/`_RAWFB` and the
  cartlog still work (fork commit `a54f70116`; `INSTRUMENTATION.md`
  §Headless screenshots; sources `core/sdl/sdl.cpp:841-843,965-975`,
  `core/audio/audiostream.cpp:70-74`).
- **BIOS:** `~/Library/Application Support/Flycast/data/naomi.zip`, md5
  `<md5>`, identical to `bios/naomi.zip` (copied from
  `../senkosp2dreamcast/bios/`, verified <date>).
- **Launch line (every leg in this repo):**
  `defaults write com.flyinghead.Flycast ApplePersistenceIgnoreState -bool YES`
  once per session, then
  `FLYCAST_HEADLESS=1 FLYCAST_CARTLOG=<abs log> FLYCAST_SHOT=<abs png> FLYCAST_SHOT_EVERY=3600 "$FLYCAST_BIN" -config config:rend.vsync=no -config config:pvr.rend=0 <abs rom> > <abs stdout log> 2>&1 & echo $! > <pidfile>`.
  Absolute paths only. `pvr.rend=0` pins OpenGL (a leftover Vulkan setting
  segfaults this bundle — senkosp `test_vmu_save.sh`). Stop by PID, never by
  name (other projects run Flycast). Known startup flake: SH4 vmem
  `Verify Failed` in stdout within seconds → relaunch (up to 3×).
- **Screenshots:** `FLYCAST_SHOT` is rewritten every `FLYCAST_SHOT_EVERY`
  frames and on `kill -USR1 <pid>` (the reliable on-demand grab); copy before
  reading. `FLYCAST_CARTLOG` lines start with an uppercase tag;
  `ARAMHANDOFF`/`VRAMHANDOFF` mark the BIOS→game handoff, `CARTDMA`/`CARTPIO`
  the cart reads (`INSTRUMENTATION.md` §3).

### dat-extract (../naomi2dreamcast/tools/dat-extract)

- **Purpose:** romset → flat decrypted `.dat` (`NAOMI` header) for Ghidra and
  the port build. M4 carts: assemble the chips at their `Games[]` offsets,
  then `m4dec` (whole-ROM stream cipher, 32-byte independent blocks;
  subkeys from the PIC Key blob at `0x5e0`/`0x5e4`). Flycast runs the romset
  directly and does not need the `.dat`.
- **Prereqs (verified <date>):** `/opt/homebrew/bin/7zz`, clang; `m4dec` and
  `extract_dat` already built in the tool dir.
- **ROM table:** `cart2dat.py`'s `ROMS` default points at the retired
  `../cleopatra/tools/flycast-src` checkout; this repo always runs it with
  `NAOMI_ROMS=/Users/captainkoffski/AntigravityProjects/flycast4naomi2dreamcast/core/hw/naomi/naomi_roms.cpp`.
- **Invocation for this game:** §ausfache.dat below.

### Reuse pointers (not exercised in Phase 1)

- Ghidra 12.1.2: `../cleopatra/tools/ghidra_12.1.2_PUBLIC/` (senkosp symlinks it as `tools/ghidra_12.1.2_PUBLIC`); recipe `../senkosp2dreamcast/docs/kb/tooling.md` §Ghidra.
- KOS: `../senkosp2dreamcast/tools/kos` (commit `705c862957b2f…`); recipe §Decoupling.
- mkdcdisc / cdi4dc / dcload-serial: `../senkosp2dreamcast/tools/`; recipes §CDI mastering, §dcload-serial.
- `parse_header.py`: `../cleopatra/scripts/parse_header.py` (Naomi header parser, offsets cross-checked vs netboot `rom.py`).

### ausfache.dat

(Filled by Task 3.)
```

Replace every `<…>` with the values captured in Steps 1–4.

- [ ] **Step 6: Commit**

```bash
git status --short            # bios/ must NOT appear
git add docs/kb/tooling.md
git commit -m "Phase 1: tooling.md — Flycast build provenance, headless launch line, dat-extract, BIOS, reuse pointers

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 3: Extract ausfache.dat + header sanity check

**Files:**
- Create: `roms/ausfache.zip` (gitignored copy), `ausfache.dat` (gitignored)
- Modify: `docs/kb/tooling.md` (§ausfache.dat), `docs/kb/game.md` (§Our dump, §Parsed .dat header)

**Interfaces:**
- Consumes: `NAOMI_ROMS` from Task 2.
- Produces: `ausfache.dat` at repo root (Task 5's boot input; Phase 3's Ghidra input), `roms/ausfache.zip` (Task 4's boot input), parsed header facts in `game.md`.

- [ ] **Step 1: Copy the romset and prove it is the assessed copy**

```bash
mkdir -p roms && cp ../naomi2dreamcast/naomi/ausfache.zip roms/ausfache.zip
md5 ../naomi2dreamcast/naomi/ausfache.zip roms/ausfache.zip     # must be equal; record the value
git check-ignore -v roms/ausfache.zip ausfache.dat               # both ignored, or STOP
```

- [ ] **Step 2: Run the extraction**

```bash
cd ../naomi2dreamcast/tools/dat-extract
NAOMI_ROMS=/Users/captainkoffski/AntigravityProjects/flycast4naomi2dreamcast/core/hw/naomi/naomi_roms.cpp python3 cart2dat.py ausfache
```

Expected: one line of the form `OK  ausfache  M4 assembled+decrypted  134217728 bytes  hdr@0x0  title='<header title>' -> …/out/ausfache.dat` (134,217,728 = 2 × 0x4000000, the two flash blobs at `naomi_roms.cpp:4580-4581`). Non-zero exit or a diagnostic = failure; the two causes the tool names are a missing blob (wrong zip) or no `NAOMI` header after decrypt (wrong key). Do not "fix" by editing the tool.

- [ ] **Step 3: Copy into this repo and validate the header**

```bash
cd /Users/captainkoffski/AntigravityProjects/ausfache2dreamcast
cp ../naomi2dreamcast/tools/dat-extract/out/ausfache.dat ausfache.dat
head -c 5 ausfache.dat; echo            # NAOMI
python3 ../cleopatra/scripts/parse_header.py ausfache.dat
```

Expected: `Magic: NAOMI`; a real publisher and Japan title (not `SAMPLE GAME`); main load entries with `RAM 0x8c020000`; `Entrypoint (main): 0x8c021000`. The ROM offset of the first entry is `0x00000000` **or** `0x40000000` (bit 30 = the M4 encrypted-read flag the assessment's `carve_boot.py` masks, assessment §6) — both are correct; record which. Any other base or entry → STOP: wrong PIC or bad extraction; re-run Step 2 and compare its output.

- [ ] **Step 4: Record in the KB**

`docs/kb/tooling.md` §ausfache.dat: the exact Step 1–3 commands, the `OK` line, byte size, md5 of `ausfache.dat`. `docs/kb/game.md` §Our dump: `roms/ausfache.zip` md5 + "identical to the library copy the assessment booted". §Parsed .dat header: the verbatim `parse_header.py` output and the line `Matches assessment guts: base 0x8c020000, entry 0x8c021000 ✓` (plus the bit-30 note if the offsets carry it). Fill the `Title` row of the identity table from the header.

- [ ] **Step 5: Verify nothing copyrighted is staged, then commit**

```bash
git status --short            # ausfache.dat and roms/ must NOT appear
git add docs/kb/tooling.md docs/kb/game.md
git commit -m "Phase 1: ausfache.dat extracted (cart2dat M4 assemble+m4dec), header verified vs assessment

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 4: Boot verification — the Phase 1 gate (headless)

**Files:**
- Create: `captures/phase1/boot.log`, `captures/phase1/boot.stdout.log` (gitignored primary data)
- Create: `docs/kb/img/ausfache-title.png`, `docs/kb/img/ausfache-demo.png`
- Modify: `docs/kb/00-status.md` (checklist + Key facts)

**Interfaces:**
- Consumes: `FLYCAST_BIN` and the launch line (Task 2), `roms/ausfache.zip` (Task 3).
- Produces: the gate evidence Phase 2's spec cites; `captures/phase1/boot.log`, which Task 5 diffs against.

- [ ] **Step 1: Launch headless with cartlog + screenshot armed**

```bash
R=/Users/captainkoffski/AntigravityProjects/ausfache2dreamcast
S=/private/tmp/claude-501/-Users-captainkoffski-AntigravityProjects-ausfache2dreamcast/65995ffe-4cc5-44a7-9c7e-9c2406faa3d9/scratchpad; mkdir -p "$S" "$R/captures/phase1"
FLYCAST_BIN=/Users/captainkoffski/AntigravityProjects/flycast4naomi2dreamcast/build/Flycast.app/Contents/MacOS/Flycast
[ -e "$R/captures/phase1/boot.log" ] && echo "STOP: leg exists — rename it by hand, legs are primary data"
defaults write com.flyinghead.Flycast ApplePersistenceIgnoreState -bool YES
FLYCAST_HEADLESS=1 FLYCAST_CARTLOG="$R/captures/phase1/boot.log" FLYCAST_SHOT="$S/shot.png" FLYCAST_SHOT_EVERY=3600 \
  "$FLYCAST_BIN" -config config:rend.vsync=no -config config:pvr.rend=0 "$R/roms/ausfache.zip" \
  > "$R/captures/phase1/boot.stdout.log" 2>&1 & echo $! > "$S/flycast.pid"
```

Expected within ~5 s: `captures/phase1/boot.stdout.log` contains `HEADLESS: audio backend forced to null`; no window appears; `pgrep -F "$S/flycast.pid"` shows the process.

- [ ] **Step 2: Confirm the BIOS→game handoff from the log**

Run after ~40 s: `grep -cE '^(ARAMHANDOFF|VRAMHANDOFF)' captures/phase1/boot.log; grep -cE '^CARTDMA ' captures/phase1/boot.log`
Expected: handoff count ≥ 1 (assessment: handoff at 20 s), CARTDMA count > 0 and rising on a second run of the command.

- [ ] **Step 3: Flake and failure triage (only if Step 2 shows nothing by 60 s)**

`grep -E 'Verify Failed|Fatal|FlycastException' captures/phase1/boot.stdout.log`. A `Verify Failed` within the first seconds = the known startup flake: kill by PID, move `boot.log`/`boot.stdout.log` to `boot.flake1.*`, relaunch (Step 1), up to 3 tries. Anything else, or three flakes: control-test senkosp's romset through the same command (`"$R/../senkosp2dreamcast/roms/senkosp.zip"`, leg name `ctl-senkosp`, handoff expected by ~20 s). senkosp boots and ausfache does not → ausfache-side (romset path, BIOS region); neither boots → environment. Record the verdict in `00-status.md` before touching anything else.

- [ ] **Step 4: Grab the title card and a demo frame**

```bash
P=$(cat "$S/flycast.pid")
sleep 60;  kill -USR1 "$P"; sleep 2; cp "$S/shot.png" "$S/title.png"
sleep 540; kill -USR1 "$P"; sleep 2; cp "$S/shot.png" "$S/demo.png"
```

View both PNGs (Read tool). Expected: `title.png` ≈ the assessment's `evidence/ausfache/shot-060s.png` (red kanji story card, "PRESS START BUTTON", "FREE PLAY"); `demo.png` ≈ `shot-609s.png` (two characters fighting, "FREE PLAY" at the bottom; the logo may be overlaid). A black or mid-transition frame = re-grab with `kill -USR1` 10 s later before concluding anything; a demo frame may also appear earlier, any live-fight frame qualifies.

- [ ] **Step 5: Stop by PID, record the leg's health, commit the evidence**

```bash
kill "$P"; sleep 2; pgrep -F "$S/flycast.pid" || echo stopped
wc -l captures/phase1/boot.log; grep -c '^CARTDMA ' captures/phase1/boot.log; grep -c '^CARTPIO ' captures/phase1/boot.log
cp "$S/title.png" docs/kb/img/ausfache-title.png; cp "$S/demo.png" docs/kb/img/ausfache-demo.png
```

In `00-status.md`: tick the boot-verification item; under Key facts add "Boot verification <date>: untouched romset reaches title (~60 s) and live demo (~600 s) headless in fork build `<hash>` — `docs/kb/img/ausfache-{title,demo}.png`; leg `captures/phase1/boot.log` (<N> lines, <n> CARTDMA, <m> CARTPIO)."

```bash
git add docs/kb/img/ausfache-title.png docs/kb/img/ausfache-demo.png docs/kb/00-status.md
git commit -m "Phase 1: boot verification — untouched ausfache reaches title + demo, headless Naomi profile

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 5: Flat-image control test (stretch; result recorded either way)

**Files:**
- Create: `captures/phase1/datboot.log`, `captures/phase1/datboot.stdout.log`
- Modify: `docs/kb/00-status.md` (checklist + Key facts)

**Interfaces:**
- Consumes: `ausfache.dat` (Task 3), `captures/phase1/boot.log` (Task 4), the launch line.
- Produces: the first M4 signal for Phase 2: whether the BIOS + game boot with no cart-ID/PIC-present answers (`DecryptedCartridge`, fork `naomi_cart.cpp:610`, `naomi_cart.h:112-116`).

- [ ] **Step 1: Launch the flat image headless (same line as Task 4 Step 1 with `boot` → `datboot` in both log names and rom = `$R/ausfache.dat`)**

Shell state does not persist between steps: re-define `R`, `S`, `FLYCAST_BIN` exactly as Task 4 Step 1 does, and use a separate pidfile `$S/flycast-dat.pid`. Expected within ~5 s: `datboot.stdout.log` contains `HEADLESS: audio backend forced to null` and does NOT contain `Failed to load BIN/DAT file` (the fork's flat-image loader error, `naomi_cart.cpp` legacy path). Whether `ARAMHANDOFF` appears in `datboot.log` by ~40 s is the finding, not a launch check. The flake triage of Task 4 Step 3 applies.

- [ ] **Step 2: Compare the cart-read sequence against the romset boot (senkosp precedent: `../senkosp2dreamcast/docs/kb/tooling.md` §Phase 3: flat-.dat boot control test)**

After 120 s, kill by PID (`kill "$(cat "$S/flycast-dat.pid")"`), then:

```bash
grep -c '^CARTDMA ' captures/phase1/datboot.log
N=$(grep -c '^CARTDMA ' captures/phase1/datboot.log)
diff <(grep '^CARTDMA ' captures/phase1/datboot.log) <(grep '^CARTDMA ' captures/phase1/boot.log | head -"$N") && echo IDENTICAL-PREFIX
grep -cE '^(ARAMHANDOFF|VRAMHANDOFF)' captures/phase1/datboot.log
```

Verdict rule: handoff seen **and** `IDENTICAL-PREFIX` (same `src=/dest=/len=` tuples, nonzero count) = **boots identically: YES** → no M4 register emulation needed for boot. No handoff, or zero CARTDMA, or a diverging prefix = **NO** → record the last 30 cartlog lines before the stall; Phase 2 captures which probe is missed. Note: `CARTPIO offset=` values in `boot.log` may carry bit 30 (`0x4xxxxxxx`) while `datboot.log`'s do not; that is the flag, not a mismatch, and is itself a finding to record.

- [ ] **Step 3: Record and commit**

`00-status.md`: tick the control-test item; Key facts: "Flat-image control test <date>: `ausfache.dat` boots identically: YES/NO — <N> CARTDMA tuples identical to the romset leg / <stall description>; `captures/phase1/datboot.log`." Also note the bit-30 observation.

```bash
git add docs/kb/00-status.md
git commit -m "Phase 1: flat-image control test — ausfache.dat boots identically: <YES|NO>

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 6: Status finalization, exit audit, fresh-agent test

**Files:**
- Modify: `docs/kb/00-status.md`, `CLAUDE.md:8-9`

**Interfaces:**
- Consumes: all prior task outputs.
- Produces: Phase 1 marked DONE; the gate record the kit brainstorm and Phase 2's spec cite.

- [ ] **Step 1: Finalize 00-status.md and CLAUDE.md**

Tick all six checklist items. Header: `**Updated:** <date> (Phase 1 Foundation — DONE; gate green)`. Phases line 1: `1. **Foundation — DONE <date>**`. Replace "Next step" with:

```markdown
## Next step

Sub-project 2: the port kit. Brainstorm + spec in the new repo
`../naomi-dc-port-kit` (charter §Decomposition: extract senkosp's
game-agnostic core; gate = senkosp rebuilt against the kit byte-identical
+ `make test` green). Phase 2 (instrumented analysis: the M4 questions,
real-play legs, top-of-RAM headroom) starts when the kit has shipped.
```

In `CLAUDE.md` change the Start-here line to: `- **Start here:** `docs/kb/00-status.md` — project state, strategy, next step; `REQUIREMENTS.md` is the origin brief.`

- [ ] **Step 2: Exit audit — repo hygiene**

```bash
git status --short                                   # empty
git ls-files | grep -iE '\.(dat|chd|gdi|cdi|iso|zip|bin|7z)$' || echo CLEAN   # expect CLEAN
find . \( -name '._*' -o -name '.DS_Store' \) -not -path './.git/*' | while read f; do git check-ignore -q "$f" || echo "NOT IGNORED: $f"; done
git log --oneline                                    # scaffold + spec commits + 6 Phase-1 commits
```

- [ ] **Step 3: Fresh-agent test**

Dispatch a clean-context subagent (Agent tool, `general-purpose`) with EXACTLY this prompt:

> Read /Users/captainkoffski/AntigravityProjects/ausfache2dreamcast/CLAUDE.md and then /Users/captainkoffski/AntigravityProjects/ausfache2dreamcast/docs/kb/00-status.md. From ONLY those two files, answer: (1) What is this project? (2) What state is it in? (3) What is the next step? (4) What rule applies to running the emulator? Answer in four short paragraphs.

PASS = (1) static binary conversion of Akatsuki Blitzkampf Ausf. Achse / ausfache, Naomi M4 cart, to Dreamcast; (2) Phase 1 Foundation done, gate green, boot verified in the emulator; (3) the port kit sub-project in `../naomi-dc-port-kit` before Phase 2; (4) always headless (`FLYCAST_HEADLESS=1`). Any miss → fix the under-communicating doc, re-dispatch, repeat until PASS.

- [ ] **Step 4: Commit**

```bash
git add docs/kb/00-status.md CLAUDE.md
git commit -m "Phase 1 complete: gate green — status finalized, exit audit + fresh-agent test passed

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

## Gate check (end of phase)

| Spec gate item | Evidence |
|---|---|
| Untouched game reaches attract in the Naomi profile, screenshots in KB | Task 4: `docs/kb/img/ausfache-{title,demo}.png`, `captures/phase1/boot.log` |
| Toolchain recorded | Task 2: `docs/kb/tooling.md` with fork hash, headless launch line, dat-extract, BIOS md5 |
| `.dat` header verified | Task 3: `game.md` §Parsed .dat header, base/entry match |
| Flat-image control test recorded either way | Task 5: `00-status.md` Key facts, `captures/phase1/datboot.log` |
| Fresh agent resumes from CLAUDE.md + 00-status.md | Task 6 Step 3 PASS |
| Port kit shipped (Phase 2 precondition) | Sub-project 2, its own repo — not this plan |
