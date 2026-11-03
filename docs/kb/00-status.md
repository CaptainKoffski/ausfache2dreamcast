# Project status

**Updated:** 2026-11-03 (kit shipped; Phase 1 Foundation — DONE; Phase 2 may start)

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
- Sub-project order (charter §Decomposition): **Phase 1 → port kit
  (new repo `../naomi-dc-port-kit`, extracted from senkosp, byte-identical
  senkosp rebuild as its gate) → Phases 2–6 on the kit.** The kit shipped
  2026-11-03 (tag `v0.1.0` on kit `extraction` → `main` after the final
  review; contract in its `README.md`): Phase 2 may start.
- Emulator always headless (`FLYCAST_HEADLESS=1`, `CLAUDE.md`).
- Tester-run hardware legs: burned CD-R on a stock console and the
  arcade-stick matrix are executed by the operator's testers (rig has
  neither); verdicts recorded with date, rig, build hash, tester wording.
- Asset cutting/compression is a last resort (REQUIREMENTS.md).

## Phases

1. **Foundation — DONE 2026-11-02** (KB, tooling records, `.dat`, boot verification)
2. Instrumented analysis (streaming/input/memory truth; the M4 questions) — next; the kit has shipped
3. Reverse engineering (touchpoint addresses + patch plan)
4. Conversion (loader + shim + patch table → bootable GDI, on the kit: `../naomi-dc-port-kit` as a submodule at `kit/`, `v0.1.0`)
5. Real-hardware testing & fit (operator + tester legs)
6. Safety tripwires & release

## Phase 1 checklist

- [x] KB skeleton (00-status, game, tooling, img/)
- [x] tooling.md — Flycast build provenance, dat-extract, BIOS, reuse pointers
- [x] ausfache.dat extracted (cart2dat, M4 assemble + m4dec), header verified vs assessment
- [x] Boot verification: untouched romset reaches title + demo, headless, screenshots in KB (THE GATE)
- [x] Flat-image control test (`ausfache.dat` boots as a plain cartridge?) — result recorded either way
- [x] Exit audit + fresh-agent test

## Key facts so far

- `ausfache`: only member of its family; Naomi M4 cart 841-0058C, rom_board
  `5504`, 2× 64 MiB flash, PIC `317-05130-jpn.ic3`, ROT0. Never ported to
  any platform. Assessment 84.4 (S) — `../naomi2dreamcast/assessments/ausfache.md`
  (capture v9, 2026-08-09, fork `f014a410c`).
- Boots and reaches live demo gameplay under the fork (assessment §3:
  handoff 20 s, trigger `pio`, demo by 609 s).
- Boot verification 2026-11-02: untouched romset reaches title (~60 s) and live demo (~340-400 s, again ~620 s and ~800-840 s) headless in fork build `5a67c8d87` — `docs/kb/img/ausfache-{title,demo}.png`; leg `captures/phase1/boot.log` (18502793 lines, 457 CARTDMA, 5 CARTPIO; ARAMHANDOFF + VRAMHANDOFF `trigger=pio` at log line 10951-10952, seen by t+40 s). Title frame is from that leg (t+62 s); its t+602 s grab landed on a mid-attract stage intro, not a fight, so the demo frame is from a second headless leg `captures/phase1/boot.regrab.log` (26985634 lines, 690 CARTDMA, 5 CARTPIO; 31 grabs t+300..900 s, live fights at t+340/360/380/400/620/800/820/840 s; the committed frame is t+360 s). Attract loop phase differs between legs, so the demo is a loop, not a fixed-time event.
- Flat-image control test 2026-11-02: `ausfache.dat` boots identically: NO. Leg `captures/phase1/datboot.log` (150795 lines, 1 CARTDMA, 7 CARTPIO, 2 handoff), build `5a67c8d87`; screenshot `docs/kb/img/ausfache-datboot-sysmenu.png` (BIOS SYSTEM MENU, JAPAN VERSION, `[AKATSUKI BK AUSF. ACHSE]`), grabbed at ~t+60 s of the 120 s leg. Stdout: `HEADLESS: audio backend forced to null`, no `Failed to load BIN/DAT file`; the fork loads the `.dat` as a plain `DecryptedCartridge` (fork `naomi_cart.cpp:610`, `naomi_cart.h:112-116`) with no M4 register behaviour.
  - Observed (the game is entered on both legs; on the flat image it returns to the BIOS): the one CARTDMA tuple is identical to the romset leg's first (`src=00800000 dest=0c0d0920 len=800`; `IDENTICAL-PREFIX`) and `ARAMHANDOFF`/`VRAMHANDOFF` (`trigger=pio`) are at log line 10950-10951 (romset 10951-10952). On both legs that CARTDMA is issued from `CARTDMAPC pc=8c028898` (datboot.log 13938 / boot.log 13939), after `MMUCRWR val=00040005 pc=8c02dfb8` (12344 / 12345): all inside the game's 1 MiB load range at `0x8c020000`, so the BIOS handed off and the game ran. Only the flat leg then writes `MMUCRWR val=00000000 pc=8c02e09a` (datboot.log 13980; absent from the first 150,795 lines of boot.log), the MIE init sequence `MIERESP sub=31/01/03 addr=0c296220` follows (15393-15397; the BIOS boot-time buffer, line 1421, where the game had used `0c128a00` at 12570/12571) and the header is re-read (`CARTPIO` 15399-15400) for the menu: the SYSTEM MENU appears with TEST never pressed (all 5,964 post-entry `JVSREPORT` lines are `buttons=0000`). No further CARTDMA in 120 s (romset: 46 by the same log-line count, 457 over its full leg).
  - BIOS-side PIO volume is not equal at matched checkpoints: the flat leg is 0x500 bytes short. Post-handoff `CARTPIOCNT` `0xede64` (datboot.log 10964) vs `0xee364` (boot.log 10965); after the first DMA `0x100540` (13950) vs `0x100a40` (13951). The flat leg's `0x100a80` only appears at 18092, after the BIOS re-entry re-read the header. The game has not run by line 10964, so the BIOS itself PIO-reads 1,280 bytes more from the M4 cart than from the plain cart, consistent with the first divergence, the early cart probe at 1423-1425: 3 x `CARTPIO offset=00000001` on the romset vs 2 x `offset=00000000` on the flat image. Phase 2 lead: what the BIOS reads in those 0x500 bytes.
  - EEPROM confound, then closed: the Task 5 leg ran with a blank game EEPROM area (Flycast keys the file on the ROM filename, `core/hw/maple/maple_jvs.cpp:1459-1461`, loaded at `:1627-1643`; `ausfache.dat.eeprom` did not exist, so `naomi_default_eeprom` was used; first difference between legs: `MIERESP sub=03` at line 1421, game area `3c911c1c…` on the romset vs zeros on the flat image). Re-run 2026-11-02 with the romset's `ausfache.zip.eeprom` copied to `ausfache.dat.eeprom` (md5 `04a50597c3a2d78de080c2739546008b`, both): leg `captures/phase1/datboot.eeprom.log` (150597 lines, 1 CARTDMA, 7 CARTPIO, 2 handoff at 10950-10951), `MIERESP sub=03` now carries the romset's `3c911c1c…` record (line 1421 and 12570), but the outcome is unchanged: same CARTDMA, `MMUCRWR val=00000000 pc=8c02e09a` at line 13980, `CARTPIOCNT` `0x100540` at 13950 and `0x100a80` at 18092 (the post-handoff checkpoint at 10964 reads `0xede5c`, 8 bytes below the first leg's `0xede64`: run-to-run jitter, still 0x508 short of the romset), 5,955 post-entry `JVSREPORT` all `buttons=0000`, SYSTEM MENU at t+100 s (screenshot not committed). The blank EEPROM is not the cause; the verdict above stays NO.
  - Cause not established; the game, not the BIOS, exits to the menu (the M4 streaming-semantics probes below are candidates, not a finding). Next measurements (Phase 2): (a) a cart-register read log with PC (hook `NaomiCartridge::ReadMem` / `M4Cartridge::ReadMem`): the cart-ID `0x34` and PIC-present `0x2000` on `NAOMI_ROM_OFFSETH` probes are register reads (`m4cartridge.h:31-38`), which no write log can see; (b) an unmasked offset log (the first DMA's payload differs between an M4 raw read and the plaintext `.dat` if bit 30 was clear; unobservable from existing evidence, the `.ram.bin` dumps precede the first DMA); (c) the EEPROM-equalized re-run (done, above); (d) the game code preceding `0x8c02e09a`.
  - What the cartlog cannot show: bit 30 of the PIO offset. The fork logs `RomPioOffset & 0x1fffffff` (`naomi_cart.cpp:1020`), so `0x4xxxxxxx` never appears in either leg.
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

Sub-project 2, the port kit: DONE 2026-11-03. `../naomi-dc-port-kit`
holds senkosp's game-agnostic core; gate green: senkosp rebuilt against the
kit is byte-identical to its `afab584` baseline, also from a fresh clone from
GitHub, and both `make test` pass (kit `docs/kb/00-status.md`). Tag `v0.1.0`
follows the final review. Phase 2 (instrumented analysis: the M4 questions,
real-play legs, top-of-RAM headroom) may start.
