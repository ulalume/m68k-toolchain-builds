# m68k-toolchain-builds

Native **m68k-elf GCC + binutils** (and optional **GDB**) cross toolchain for
[SGDK](https://github.com/Stephane-D/SGDK), built per platform in CI.

Targets: linux x86_64, linux arm64, macOS arm64, macOS x86_64.

## Versions

| Component | Version |
|---|---|
| GCC      | 13.2.0 (matches SGDK's bundled `bin/gcc.exe`) |
| binutils | 2.44 |
| GDB      | 16.2 |

## Based on sgdk-setup

The build flags are adapted from **[garrettjwilke/sgdk-setup](https://github.com/garrettjwilke/sgdk-setup)**
— `m68k-setup.sh` (gcc/binutils) and `gdb-setup.sh` (gdb).

## Workflows

| Workflow | Builds | Trigger |
|---|---|---|
| `toolchain.yml` | gcc + binutils | manual / `gcc*` tag |
| `gdb.yml` | gdb (`--without-python` + dylib bundling) | manual / `gdb*` tag |

Assets are `*-<platform>.tar.gz`; extract the toolchain tree whole (GCC resolves
`cc1`/`libgcc` relative to the binary).

## Licenses

| Item | License |
|---|---|
| GCC 13.2.0 | GPL-3.0-or-later (libgcc: + GCC Runtime Library Exception) |
| binutils 2.44 | GPL-3.0-or-later |
| GDB 16.2 | GPL-3.0-or-later |
| **This repository** (workflows / scripts) | **GPL-3.0-or-later** |

Source for the GNU components is the FSF mirror (`GNU_MIRROR` in the workflows) at
the versions above.
