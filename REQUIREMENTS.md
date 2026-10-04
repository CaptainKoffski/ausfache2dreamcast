# The idea
Akatsuki Blitzkampf Ausf. Achse (`ausfache`, Subtle Style, 2008) is a 2D fighter on Sega Naomi. It has never been ported to any platform. We have already ported two Naomi games to Dreamcast this way — Cleopatra Fortune Plus (`../cleopatra`) and Senko no Ronde Special (`../senkosp2dreamcast`) — so this is the third run of a known method, not research from scratch.

# The goal
Port Ausf. Achse to Dreamcast with AI heavylifting, as a static binary conversion (no source code). I can help with routine things: installing apps, running builds on real hardware, burning discs, reporting what I see on the TV.

# The main guideline
`../naomi2dreamcast/docs/kb/port-playbook.md` — follow it. The six phases in order, gates enforced, one superpowers spec + plan per phase. Its **Definition of done** is part of the deliverable, not polish:
1. Every boot path PASS on the release candidate, verdicts recorded: GDEMU-class ODE, DreamShell serial-SD, burned CD-R; VGA and composite. Flycast is a dev tool, never the verdict.
2. Two disc formats: GDI and CDI.
3. Fully playable on both a standard DC pad and an arcade stick.
4. The game's arcade settings (test-mode GAME ASSIGNMENTS) reachable from a pre-game menu.

Plus the VMU-safety tripwires before release: the port must never write a VMU.

Senko no Ronde Special is the complete, hardware-verified implementation of all of this — reuse its code and scripts instead of re-inventing them.

# What is different this time
- **It's a cartridge game, not GD-ROM.** Both previous ports were GD-ROM (DIMM) games. Ausf. Achse is an **M4 cart** (841-0058C, 2× 64 MiB flash, the whole ROM encrypted). The cart data has to be served from the Dreamcast disc instead; figure out what this changes in the streaming path vs. the previous ports.
- The portability assessment is `../naomi2dreamcast/assessments/ausfache.md` (84.4 S). Everything measured fits Dreamcast budgets; main-RAM writes are address-sparse (address peak ~0.975× of 16 MB), so relocation may be needed. Re-measure — the assessment is a starting point, not ground truth.
- Controls: 8-way stick + 3 buttons (Weak/Medium/Strong), 2 players.

# Inputs
- Romset: `../naomi2dreamcast/naomi/ausfache.zip` → copy into `roms/`. The flat decrypted image comes from `../naomi2dreamcast/tools/dat-extract/cart2dat.py ausfache`.
- Naomi BIOS: `bios/` (I'll supply it; ask me).
- Instrumented emulator: `../flycast4naomi2dreamcast`.
- Never commit copyrighted bytes: ROMs, BIOS, disc images, extracted assets.

# Data collecting
Save everything to `docs/kb/` so future agents know how to do their job: tool installs (`tooling.md`), findings with citations (primary sources outrank wikis), hardware test verdicts. If a lesson generalizes beyond this game, propose an update to the playbook in `../naomi2dreamcast`.
