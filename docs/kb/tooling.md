# Tooling

Every tool this project uses: exact install/reuse steps, version, usage.
The environment must be rebuildable from this file. Deep recipes that live
in `../senkosp2dreamcast/docs/kb/tooling.md` are referenced, not duplicated.

### Instrumented Flycast (reused build)

- **Binary:** `../flycast4naomi2dreamcast/build/Flycast.app/Contents/MacOS/Flycast`
  (Mach-O arm64, built 2026-10-24 19:16:05), fork HEAD `5a67c8d87`.
  Provenance: at Phase 1 start the tree was `4a7e80240` + two uncommitted
  logging-only changes (`FLYCAST_WRWATCH`, `BLOCKEDEXC`) that the binary
  includes by mtime (binary 19:16:05 is newer than `addrspace.cpp` 19:16:03
  and `sh4_interrupts.cpp` 18:17:58) and by content (`grep -a -c
  FLYCAST_WRWATCH` = 1, `grep -a -c BLOCKEDEXC` = 1 in the binary); committed locally 2026-11-02 as
  `5a67c8d87` (35 insertions, 0 deletions; no push).
- **Source of truth:** `../flycast4naomi2dreamcast`. Build recipe (submodules,
  Syphon patch, cmake flags): `../senkosp2dreamcast/docs/kb/tooling.md`
  §Decoupling (2026-09-21). Not rebuilt in Phase 1. Caveat before any
  rebuild: the fork's Syphon submodule is clean and `4a7e80240` moved the
  Syphon fix into the top-level CMake, so that recipe's step
  `git -C core/deps/Syphon apply …` may no longer apply to this tree —
  verify first.
- **Headless (always):** `FLYCAST_HEADLESS=1` — hidden SDL window, background
  app that never activates, null audio; `FLYCAST_SHOT`/`_RAWFB` and the
  cartlog still work (fork commit `a54f70116`; `INSTRUMENTATION.md`
  §9 "Seeing the screen without asking the OS: FLYCAST_SHOT"; sources `core/sdl/sdl.cpp:841-843,965-975`,
  `core/audio/audiostream.cpp:70-74`). The binary contains the knob
  (`grep -a -c FLYCAST_HEADLESS` = 1; plain `grep -c` prints nothing on this
  binary, use `-a`).
- **BIOS:** `~/Library/Application Support/Flycast/data/naomi.zip`, md5
  `9a50ea3c17d32b7ea450c9539fbc71cb`, identical to `bios/naomi.zip` (copied from
  `../senkosp2dreamcast/bios/`, verified 2026-11-02).
- **Launch line (every leg in this repo):**
  `defaults write com.flyinghead.Flycast ApplePersistenceIgnoreState -bool YES`
  once per session, then
  `FLYCAST_BIN=/Users/captainkoffski/AntigravityProjects/flycast4naomi2dreamcast/build/Flycast.app/Contents/MacOS/Flycast`
  and
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
- **Capture disk budget:** ~580 MB of cartlog per 10-min Naomi leg (845 MB
  for 15 min), plus a 32 MiB `<cartlog>.ram.bin` per leg (the RNDREG probe's
  one-shot RAM dump at render 1500, `INSTRUMENTATION.md` §14).

### dat-extract (../naomi2dreamcast/tools/dat-extract)

- **Purpose:** romset → flat decrypted `.dat` (`NAOMI` header) for Ghidra and
  the port build. M4 carts: assemble the chips at their `Games[]` offsets,
  then `m4dec` (whole-ROM stream cipher, 32-byte independent blocks;
  subkeys from the PIC Key blob at `0x5e0`/`0x5e4`). Flycast runs the romset
  directly and does not need the `.dat`.
- **Prereqs (verified 2026-11-02):** `/opt/homebrew/bin/7zz` (7-Zip 26.00
  arm64), clang (Apple clang 21.0.0); `m4dec` and `extract_dat` already built
  in the tool dir.
- **ROM table:** `cart2dat.py`'s `ROMS` default points at the retired
  `../cleopatra/tools/flycast-src` checkout; this repo always runs it with
  `NAOMI_ROMS=/Users/captainkoffski/AntigravityProjects/flycast4naomi2dreamcast/core/hw/naomi/naomi_roms.cpp`.
- **Invocation for this game:** §ausfache.dat below.

### Reuse pointers (not exercised in Phase 1)

- Ghidra 12.1.2: `../cleopatra/tools/ghidra_12.1.2_PUBLIC/` (senkosp symlinks it as `tools/ghidra_12.1.2_PUBLIC`); recipe `../senkosp2dreamcast/docs/kb/tooling.md` §Ghidra.
- KOS: `../senkosp2dreamcast/tools/kos` (commit `705c862957b2f…`); recipe §Decoupling.
- mkdcdisc / cdi4dc / dcload-serial: `../senkosp2dreamcast/tools/` (`mkdcdisc/`, `img4dc/cdi4dc`, `dcload-serial/`); recipes §CDI mastering, §dcload-serial.
- `parse_header.py`: `../cleopatra/scripts/parse_header.py` (Naomi header parser, offsets cross-checked vs netboot `rom.py`).

### ausfache.dat

Extracted 2026-11-02 (romset md5 `b54bd62d3c5024da23cfef211a304f61`, see `game.md` §Our dump):

```bash
mkdir -p roms && cp ../naomi2dreamcast/naomi/ausfache.zip roms/ausfache.zip
cd ../naomi2dreamcast/tools/dat-extract
NAOMI_ROMS=/Users/captainkoffski/AntigravityProjects/flycast4naomi2dreamcast/core/hw/naomi/naomi_roms.cpp python3 cart2dat.py ausfache
cd /Users/captainkoffski/AntigravityProjects/ausfache2dreamcast
cp ../naomi2dreamcast/tools/dat-extract/out/ausfache.dat ausfache.dat
python3 ../cleopatra/scripts/parse_header.py ausfache.dat
```

Tool output, verbatim:

```
OK  ausfache  M4 assembled+decrypted  134217728 bytes  hdr@0x0  title='AKATSUKI BK AUSF. ACHSE' -> /Users/captainkoffski/AntigravityProjects/naomi2dreamcast/tools/dat-extract/out/ausfache.dat
```

`ausfache.dat`: 134,217,728 bytes (2 x 0x4000000, the two flash blobs at `naomi_roms.cpp:4580-4581`), md5 `97467a0eb4d875da272743d797bcd8de`. Gitignored (`.gitignore:10 *.dat`), as is `roms/` (`.gitignore:2`).
