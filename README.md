# suyu

<h1 align="center">
  <br>
  <img src="dist/suyu.svg" alt="suyu" height="128">
  <br>
  <b>suyu</b>
  <br>
</h1>

<h4 align="center">
Nintendo Switch emulator and native recompiler — based on <a href="https://git.eden-emu.dev/eden-emu/eden">Eden</a>, which itself descends from yuzu.
</h4>

<p align="center">
  <a href="#status">Status</a> |
  <a href="#changes-in-v005">Changes in v0.0.5</a> |
  <a href="#building">Building</a> |
  <a href="#license">License</a>
</p>

---

> **This is a continuation of suyu, which was archived upstream at v0.04.**
>
> [`suyu-emu/suyu-v0.0.4`](https://github.com/suyu-emu/suyu-v0.0.4) is a public
> archive and no further development was planned there. This fork picks it up
> from commit `d1d09321d7` and continues the numbering: **v0.0.5**.
>
> The name and version line are kept deliberately, so the lineage stays legible.
> `BUILD_FULLNAME` reads `suyu v0.0.5 (mk8-recomp)` — the suffix says *which*
> 0.0.5 a binary is, since the archived repository could in principle be picked
> up by others too. See [PROVENANCE.md](PROVENANCE.md).
>
> Work happens on the `mk8-recomp` branch, driven by
> [mk8-recomp](https://github.com/dougchansan/mk8-recomp) — a project statically
> recompiling Switch titles to native x86-64 using this recompiler. Fixes that
> are not recompiler-specific are listed below and are useful to anyone running
> suyu.

## About

suyu is a Nintendo Switch emulator and AArch64 native recompiler written in C++. It can run decrypted Switch titles using either:

- **HLE/emulation mode** — full hardware-level emulation via the suyu core (GPU, CPU, audio, services)
- **Recompiler mode** — ahead-of-time static recompilation of Switch AArch64 game code to native x86-64 executables, bundled with suyu's HLE backend

Based on [Eden](https://git.eden-emu.dev/eden-emu/eden), with suyu's own improvements to UI, recompiler, and platform support.

## Status

Current version: **v0.0.5**, continuing from the archived v0.04.

Upstream was inconsistent about its own version — the repository is named
`suyu-v0.0.4`, the tag reads `v0.04-latest`, and `BUILD_FULLNAME` was hardcoded
to `v0.04`. This fork normalises to the three-part form. Read literally, `v0.04`
means 0.4, which was evidently not the intent.

Platforms: Windows and Linux both build and run. Android is inherited from
upstream and untested since the fork; macOS/iOS are not included.

Linux needs five things Windows does not, all handled by
[`scripts/build-suyu.sh`][bld] in the consuming project:

- CMake 3.31 (`CMakeModules/CPMUtil.cmake` requires it; Ubuntu 24.04 ships 3.28)
- `-Dfmt_FORCE_BUNDLED=ON` — the system fmt 9 has no `format_string::get()`, and
  suyu only forces the bundled one inside a branch that does not apply here
- Qt6 Charts, which Ubuntu packages separately
- system Boost
- skipping the `externals/ownfoil` submodule, whose own nested submodule no
  longer resolves; nothing in suyu's CMake references it

Building on Linux found two defects that MSVC had silently accepted: literal
carriage returns inside string literals, and a boost forwarding header that
resolved only where CPM had fetched boost.

[bld]: https://github.com/dougchansan/mk8-recomp/blob/main/scripts/build-suyu.sh

## Changes in v0.0.5

Five of these are defects in suyu itself rather than recompiler work, and affect
ordinary emulation. Each is one commit.

### Fixes

- **Installed updates and DLC in NAND were never indexed.** `GetFileAtID` tried
  eight storage-layout variants but skipped every odd index except 7, so the
  `.cnmt.nca` suffix was only ever looked for at the cache root — never inside a
  `000000XX/` directory, which is exactly where meta NCAs are stored and what
  `InstallEntry` writes. Every meta NCA in NAND was therefore unreachable and no
  installed update or DLC ever entered the cache, silently: a miss is
  indistinguishable from nothing being installed, which is why the frontend's
  installed-title listing reported zero. A second defect behind it let an older
  update overwrite a newer one, because the metadata map is keyed by title id
  with no version comparison — now the higher `GetTitleVersion()` wins.

- **Service handler registration dropped most commands.** A
  `FunctionInfoTyped<T>` array was walked through a `FunctionInfoBase*` with a
  different member layout. `sizeof()` agrees, so a size assertion passes and
  tells you nothing, but every element after the first was read from the wrong
  offset. `IpcController` registered 2 of its 6 handlers;
  `QueryPointerBufferSize` was among the lost, and it is part of CMIF session
  setup — so titles stalled in early service initialisation.

- **RomFS registration was silently dropped.** `emplace` where
  `insert_or_assign` was meant, so re-registration kept the stale entry and the
  title panicked on boot.

- **AOT image dispatch resolved every PC to the wrong module.** Double base
  subtraction made every lookup underflow, and a four-entry module table
  mismapped any title with more than one subsdk.

- **The AOT exporter read the base ExeFS, not the update's.** `PatchManager`
  replaces the ExeFS wholesale when an update is present, so the exported image
  diverged from live execution on any updated title.

### Additions

- **AArch64 → C recompiler work.** Exclusives now route through
  `Core::ExclusiveMonitor` (previously a plain load/store with `STXR` always
  reporting success, which makes every compare-and-swap non-atomic under real
  threads); FPCR/FPSR are modelled; the counter and `CTR_EL0` are read from the
  emulator's own sources so the two engines cannot disagree across a transition.
  Plus EXTR/ROR, ADC/SBC, LDPSW, exclusive pair forms, PRFM, and the DC
  cache-maintenance family.

- **Static and runtime coverage instrumentation** — per-module JSON of
  emitted/unhandled counts, and runtime histograms of blocks executed,
  transitions by cause, unimplemented opcodes and SVCs.

- **`suyu-cmd --probe-isa-list`** reports each title's CPU architecture without
  booting it, reading the update's NPDM as well as the base's. An update can
  change the answer: the target title ships an ARM32 base whose 4.0.0 update
  is AArch64.

- **Diagnostics** — the NPDM log line carries a content hash, because size is not
  an identity (two the target title updates share a 1476-byte `main.npdm`, one ARM32 and one
  AArch64), and `PatchExeFS` names which provider slot answered for an update.

Full change set:

```
git diff d1d09321d7ab84252291e05b3efbc8a8dfa57481..mk8-recomp
```

## Legal Notice

suyu is a GPLv3 program, which allows fully free redistribution of its source code and releases liability of its authors for how this software is used as stated in Section 15 and 16.

The suyu Emulator program does not circumvent Nintendo's technological protection measures (TPMs) as the user is required to provide both the Nintendo Switch software & the encryption keys for these games, and the suyu Emulator uses a mode of the Advanced Encryption Standard (AES), an open encryption standard established by the US NIST, along with the encryption keys that the user themselves must lawfully acquire, to decrypt the software. As the standard is public and available to use by all, it does not constitute as the Digital Market Copyright Act's (DMCA) definition of "circumventing a technological measure" as defined in Section 1201(a)(3).

The suyu Emulator also falls under the exemptions stated in Section 1201(f) of the DMCA as this software was created for the purposes of reverse engineering the Nintendo Switch software (known as Horizon OS) to create interoperability with Nintendo Switch games and software with the Windows, macOS, and GNU/Linux operating systems.

Any aggressive DMCA claims or takedown notices against projects that explicitly disclaim piracy support, require user-provided keys, and limit functionality to interoperability (such as suyu) could constitute overreach or misuse of the DMCA.

As derived from §512(f), if Nintendo (or an affiliated entity) knowingly materially misrepresents that a project like suyu is infringing (or circumvents TPMs) when it does not, especially if they fail to consider fair use, interoperability exemptions under §1201(f), or the fact that the emulator requires user-provided keys and does not itself contain proprietary Nintendo code, they can be made liable for any Damages against suyu.

## Building

### Dependencies

- CMake 3.15+, Ninja
- Qt 6.4+ (without bundled Qt: `-DYUZU_USE_BUNDLED_QT=OFF`)
- Vulkan SDK, libusb, OpenSSL

### Windows

```bat
cmake -B build -DCMAKE_BUILD_TYPE=Release -DENABLE_QT=ON -DYUZU_USE_BUNDLED_QT=OFF -GNinja
cmake --build build --target suyu suyu-cmd
```

### Linux

```sh
sudo apt-get install ninja-build qt6-base-dev libqt6svg6-dev libusb-1.0-0-dev libssl-dev
cmake -B build -DCMAKE_BUILD_TYPE=Release -DENABLE_QT=ON -DYUZU_USE_BUNDLED_QT=OFF -GNinja
cmake --build build --target suyu suyu-cmd
```

### Android

```sh
cd src/android && ./gradlew assembleMainlineRelease
```

## License

GPL-3.0-or-later. See [LICENSE.txt](LICENSE.txt).
