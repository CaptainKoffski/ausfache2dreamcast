# ausfache — game dump notes

Identity facts sourced from `../naomi2dreamcast/assessments/ausfache.md`
(battery v9, 2026-08-09) unless noted.

| | |
|---|---|
| Title | Akatsuki Blitzkampf Ausf. Achse (Japan) — header title `AKATSUKI BK AUSF. ACHSE` |
| Set | `ausfache` — single member, no parent, no clones (MAME `src/mame/sega/naomi.cpp` @59e7c0b GAME line 11116, `ROM_START` lines 6808–6822) |
| Maker / year | Subtle Style, 2008 (arcade debut 2008-02-20) |
| Media | Naomi **M4 cartridge** 841-0058C, rom_board id `5504`, 2× 64 MiB flash (`ic8.bin`, `ic9.bin`), ROT0; fork `naomi_roms.cpp:4571-4585` (M4, key `0x5504`) |
| Security PIC | `317-05130-jpn.ic3` (CRC `eccdcd59`, clean dump) |
| Carve (assessment guts) | base `0x8c020000`, entry `0x8c021000`, code 1,048,576 B, 2,111 functions; header load offsets carry the M4 bit-30 flag (assessment §6, `carve_boot.py` mask) |
| SDK stack | Kunoichi2 Library for NAOMI 2.07, syStartCwKn 2.08, AIPKN 0.91 (Mar 2001 builds) |
| Controls | 8-way stick + 3 buttons (A/B/C = Weak/Medium/Strong), 2P — fork `naomi_roms_input.h:195` |

## Our dump

`roms/ausfache.zip` (gitignored) md5 `b54bd62d3c5024da23cfef211a304f61` — identical to the library copy the assessment booted (`../naomi2dreamcast/naomi/ausfache.zip`, same md5, checked 2026-11-02).

## Parsed .dat header

`python3 ../cleopatra/scripts/parse_header.py ausfache.dat` (2026-11-02), verbatim:

```
- **File:** `ausfache.dat` (134,217,728 bytes)
- **Magic:** `NAOMI`
- **Publisher:** PIC CORPORATION
- **Title (Japan):** AKATSUKI BK AUSF. ACHSE
- **Title (USA):** AKATSUKI BK AUSF. ACHSE
- **Title (Export):** AKATSUKI BK AUSF. ACHSE
- **Title (Korea):** AKATSUKI BK AUSF. ACHSE
- **Title (Australia):** AKATSUKI BK AUSF. ACHSE
- **Title (Reserved1):** SAMPLE GAME RESERVED 1
- **Title (Reserved2):** SAMPLE GAME RESERVED 2
- **Title (Reserved3):** SAMPLE GAME RESERVED 3
- **Main load entries:**
  - ROM 0x40000000 -> RAM 0x8c020000, 0x100000 bytes
- **Test load entries:**
  - ROM 0x40400000 -> RAM 0x8c020000, 0x100000 bytes
- **Entrypoint (main):** 0x8c021000
- **Entrypoint (test):** 0x8c021000
```

Matches assessment guts: base 0x8c020000, entry 0x8c021000 ✓

The load-entry ROM offsets print as `0x40000000` / `0x40400000`: bit 30 is the M4 encrypted-read flag, which the assessment's `carve_boot.py` masks off (assessment §6). The real file offsets are `0x00000000` / `0x00400000`.
