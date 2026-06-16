# m68k-toolchain-builds

Native **m68k-elf GCC + binutils** (and optional **GDB**) cross toolchain for
[SGDK](https://github.com/Stephane-D/SGDK), built per platform in CI. This is the
heavy, rarely-changing layer of the native SGDK distribution: built once per
GCC/binutils version and **reused across many SGDK releases** (do not rebuild on
every SGDK update).

Targets: linux x86_64, linux arm64, macOS arm64, macOS x86_64.
(No Windows — SGDK already ships a Windows toolchain; this exists to avoid Wine on
Linux/macOS.)

## Versions

| Component | Version | Source |
|---|---|---|
| GCC      | 13.2.0 | matches SGDK's bundled `bin/gcc.exe` |
| binutils | 2.44   | |
| GDB      | 16.2   | optional, experimental |

> **How the SGDK ↔ toolchain version mapping is determined:** read it out of the
> SGDK release you target — `strings $SGDK/bin/gcc.exe | grep -oE '[0-9]+\.[0-9]+\.[0-9]+'`.
> Pin this repo's `GCC_VERSION` to match what SGDK validated against.

## Workflows

### `toolchain.yml` — GCC + binutils (stable)
Mirrors the validated build flags from `sgdk-setup/m68k-setup.sh`:
`--target=m68k-elf --with-cpu=68000`, C only, freestanding. GCC's
`download_prerequisites` links gmp/mpfr/mpc **statically**, so `m68k-elf-gcc`
depends only on system libraries and is portable when the **whole tree** is
shipped (GCC resolves `cc1`/`libgcc` relative to the binary; the baked-in
`--prefix` is only a fallback).

- Trigger: **manual** (`workflow_dispatch`) or **tag `gcc*`** (e.g. `gcc13.2.0-1`).
- Asset: `m68k-elf-toolchain-gcc<ver>-<platform>.tar.gz` (extract the tree whole).

### `gdb.yml` — GDB (⚠️ experimental, needs a CI validation run)
A locally-built gdb links brew (gmp/mpfr/…) and pyenv `libpython` at absolute
paths and is not portable. This workflow makes it self-contained, the same way
`blastem-builds` handles SDL2:

1. `--without-python` — drops the worst dependency (pyenv libpython).
2. Bundle remaining libs next to the binary + rewrite paths:
   - macOS: `dylibbundler` → `@executable_path/../lib/bundled/`, then **re-sign**
     (`codesign --force -s -`, because rewriting a Mach-O invalidates the arm64
     ad-hoc signature).
   - Linux: copy non-system `.so` into `lib/bundled` + `patchelf --set-rpath '$ORIGIN/../lib/bundled'`.

- Trigger: **manual** or **tag `gdb*`** (e.g. `gdb16.2-1`).
- Asset: `m68k-elf-gdb-<ver>-<platform>.tar.gz`.
- The bundling step is the part to watch on the first CI run.

## Distribution model

These artifacts are the cached, rarely-rebuilt base. `sgdk-builds` downloads the
matching toolchain to build `libmd.a` per SGDK version; `sgdkx` downloads + composes
everything at install time (programmatic download → no macOS quarantine prompt;
binaries are linker-ad-hoc-signed so they run on Apple Silicon).

## Licensing

GCC, binutils and GDB are **GPL** (GPLv3 for current GCC/binutils/gdb). They are
standalone build tools — their GPL does **not** propagate to games built with them
(same as any compiler). Ship the license/notices and make corresponding source
available (the GNU mirror URLs + versions above suffice as the written offer).
