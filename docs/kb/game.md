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
