# Project status

**Updated:** 2026-11-02 (Phase 1 Foundation — in progress)

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
- [x] Boot verification: untouched romset reaches title + demo, headless, screenshots in KB (THE GATE)
- [x] Flat-image control test (`ausfache.dat` boots as a plain cartridge?) — result recorded either way
- [ ] Exit audit + fresh-agent test

## Key facts so far

- `ausfache`: only member of its family; Naomi M4 cart 841-0058C, rom_board
  `5504`, 2× 64 MiB flash, PIC `317-05130-jpn.ic3`, ROT0. Never ported to
  any platform. Assessment 84.4 (S) — `../naomi2dreamcast/assessments/ausfache.md`
  (capture v9, 2026-08-09, fork `f014a410c`).
- Boots and reaches live demo gameplay under the fork (assessment §3:
  handoff 20 s, trigger `pio`, demo by 609 s).
- Boot verification 2026-11-02: untouched romset reaches title (~60 s) and live demo (~340-400 s, again ~620 s and ~800-840 s) headless in fork build `5a67c8d87` — `docs/kb/img/ausfache-{title,demo}.png`; leg `captures/phase1/boot.log` (18502793 lines, 457 CARTDMA, 5 CARTPIO; ARAMHANDOFF + VRAMHANDOFF `trigger=pio` at log line 10951-10952, seen by t+40 s). Title frame is from that leg (t+62 s); its t+602 s grab landed on a mid-attract stage intro, not a fight, so the demo frame is from a second headless leg `captures/phase1/boot.regrab.log` (26985634 lines, 690 CARTDMA, 5 CARTPIO; 31 grabs t+300..900 s, live fights at t+340/360/380/400/620/800/820/840 s; the committed frame is t+360 s). Attract loop phase differs between legs, so the demo is a loop, not a fixed-time event.
- Flat-image control test 2026-11-02: `ausfache.dat` boots identically: NO — the 1 CARTDMA tuple logged is identical to the romset leg's first (`src=00800000 dest=0c0d0920 len=800`; `IDENTICAL-PREFIX`) and `ARAMHANDOFF`/`VRAMHANDOFF` (`trigger=pio`) appear at log line 10950-10951 (romset: 10951-10952), but the flat image never gets past the BIOS: no further CARTDMA in 120 s (romset: 46 CARTDMA by the same log line count, 457 over its full leg), and the screenshot shows the BIOS SYSTEM MENU (JAPAN VERSION, `[AKATSUKI BK AUSF. ACHSE]` read from the header) instead of the game, with no cart reads after log line 15400 (the log settles into a MIE/JVS poll + TA frame loop). Stdout: `HEADLESS: audio backend forced to null`, no `Failed to load BIN/DAT file`; the fork loads the `.dat` as a plain `DecryptedCartridge` (fork `naomi_cart.cpp:610`, `naomi_cart.h:112-116`) with no M4 register behaviour. First behavioural divergence: the early BIOS cart probe at log line 1423-1425 is 3 x `CARTPIO offset=00000001` on the romset and 2 x `offset=00000000` on the flat image. Bit 30 is not observable in these logs: the fork logs `RomPioOffset & 0x1fffffff` (`naomi_cart.cpp:1020`), so `0x4xxxxxxx` never appears in either leg. Phase 2 must capture which M4 probe the BIOS needs (cart-ID `0x34` / PIC-present on ROM_OFFSETH) to leave the system menu. Leg `captures/phase1/datboot.log` (150795 lines, 1 CARTDMA, 7 CARTPIO, 2 handoff); build `5a67c8d87`.
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
