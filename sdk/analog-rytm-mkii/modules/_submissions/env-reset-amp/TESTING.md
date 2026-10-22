# Env Reset Amp — test report

- Module: `env-reset-amp` 1.0.0, for Analog Rytm MKII OS 1.74 on core-ar2 0.3.
- Source: https://github.com/Iced-Cream/rytm2_mods at `afd461d4a8ef4306bb2a57d4c61ad35c362b3940` (tag `v1.2`), `env-reset-amp`.
  `src/env-reset-amp.s` (`9dc26b75…2803`, rytm2_mods' `env-reset-amp/os1.74/env-reset-amp.s`) is byte-identical to it.
  The file has not changed since rytm2_mods' OS 1.74 port (`13ad537`), so v1.1 and v1.2 carry the same source.
- Original: MrScruff's rytm-mods `0015-env-reset-amp`.
- Stock OS 1.74: `.syx` `cb0f99d6…e123`, main OS `34cd0c4d…1f3b`.
- Tested build R8T2 (OS 1.74, core-ar2 0.1 ported to 1.74 + eight ports): `.syx`
  `94c8b8c4f3f537132b7ee689ad1221591186f14dc605a420cdefe3df804e3c39`.
- Tested build C3A4 (OS 1.74, core-ar2 0.3 before #153 + all eleven rytm2_mods v1.2 ports): `.syx`
  `04d987ced36850eef5eb5cd1215d411edf621a5af94f980525ab561c9108f951`, main OS `fc871476…d529`.
- Build ER74 (OS 1.74, released core-ar2 0.3 + this module): `.syx` `48db5cf7368c517a0559028468cb05f48be699ff1efd338cc2ef821cf61fd809`,
  main OS `771e5951c7cd56e43e8848b6ea6479ead21eb17ff5a5475a62c1af88b40c0487`. Emulator only.

## Hardware

On an Analog Rytm MKII with OS 1.74, 2026-10-18, Iced-Cream flashed R8T2 with Elektron
Transfer, ran the build's test steps and restored stock 1.74. This module's step:

7. FUNC + MUTE > SOUND SETTINGS > ENV RESET, stepped to AMP. With AMP ATK about 40 and DEC
   long, hitting the pad quickly, the attack is heard on every hit.

It passed. On 2026-10-20, in C3A4 (core-ar2 0.3 and all eleven v1.2 ports, including
this module's release object `881bcfed…40d1`), each mod's own test passed again.
Relinking C3A4's core and v1.2's eleven release objects on stock 1.74 gives C3A4's main OS
byte for byte. Duration was not recorded. See [the hardware report](evidence/hardware-testing.json).

## Source build comparison

The module is a submission (`modules/_submissions/`) until the browser can build it. Modwerk's own validation and compiler path, run on this folder:

```sh
node scripts/machine-modules.mjs --drafts --base origin/main
MODWERK_CROSS=m68k-linux-gnu- node scripts/build-elemod-packages.mjs --output OUT --source-commit <commit> --drafts --module analog-rytm-mkii:env-reset-amp
node scripts/verify-elemod-source-parity.mjs --packages OUT --oracle ORACLE --out RESULT --firmware analog-rytm-mkii:Analog-Rytm_MKII_OS1.74.syx
```

`ORACLE` held elekloader's `core-ar2-0.3.elemod` release (`core-ar2-v0.3`, 769d9aa,
`6b97dfb3…3e58`) and rytm2_mods v1.2's release `env-reset-amp-1.0-os1.74.elemod` (`881bcfed…40d1`).
The materialised Modwerk package linked with that core gives main OS `771e5951…0487`,
byte-identical to the core linked with the release object, and to ER74
([source-parity.json](evidence/source-parity.json)). `.run` is 52 bytes, three sites, no
relocations.

Compiler: Ubuntu 24.04 `m68k-linux-gnu` GCC 13.3.0 and binutils 2.42. Modwerk's pinned Docker
toolchain was not run here, and CI's `elemod-source` job compiles published folders only. The
firmware and the linked builds stayed on the owner's computer.

## Emulator

[rytm-emu](https://github.com/trigcondition/rytm-emu) `c5732f0`, headless `dtboot`, the same
input on ER74 and on stock 1.74 ([capture plan](media/capture.json)): FUNC + MUTE >
SOUND SETTINGS on BD, ENV RESET selected at FILTER, then LEFT, LEFT, RIGHT, RIGHT, RIGHT, RIGHT.

| Key | Stock 1.74 | ER74 |
| --- | --- | --- |
| (start) | FILTER | FILTER |
| LEFT | OFF | AMP |
| LEFT | OFF | OFF |
| RIGHT | FILTER | AMP |
| RIGHT | FILTER | FILTER |
| RIGHT | FILTER | AMP+FILT |
| RIGHT | FILTER | AMP+FILT |

Every screen before the first LEFT is identical on both. In a second ER74 run with ENV
RESET at AMP, BD's ENV RESET byte in the engine (`0x8000ea30`) reads `01` (the next three
tracks' bytes stay at `02`, FILTER), and three BD pad hits each pass the stub's resume address
`0x40109b76`. The emulator's analog voice model is not used to judge the envelope; the
restart from silence was judged by ear on the unit.

## Load and memory

52 bytes in the shared DDR area, no `.bss`, no fast SRAM. The detour runs once per trig in
the voice-command routine: two table lookups, a bit test and, with the amp bit set, two
register loads. The two byte sites are in ENV RESET's setter and the menu's stepper, not in
the audio path. Static reading, not a measurement.

## Not tested

- A build with the released core-ar2 0.3 (with elekloader #153) on a unit, and the module alone on a unit.
- Other modules beside it, except rytm2_mods' own ports.
- AMP+FILT on its own was not recorded as a separate step; saving and reloading a sound with AMP set, and a power cycle with this build, were not recorded.
