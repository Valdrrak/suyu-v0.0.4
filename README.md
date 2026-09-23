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
  <a href="#building">Building</a> |
  <a href="#license">License</a>
</p>

---

> **This fork provides the v0.0.4.1 startup hotfix for suyu v0.04.**
>
> The upstream codebase is preserved under GPL-3.0. This fork provides a versioned
> rebuild containing the debugger-pane initialization fix and a guard for builds
> without MicroProfile, which otherwise can crash after selecting or restoring
> an interface mode.
>
> **Note:** a small number of fixes were later pulled in from a community
> continuation of this codebase, after users reported that installed
> updates/DLC, service handler registration, and RomFS registration were all
> broken in the archived build. Those fixes are folded in here as bug fixes to
> suyu itself and are retained in this hotfix.

## About

suyu is a Nintendo Switch emulator and AArch64 native recompiler written in C++. It can run decrypted Switch titles using either:

- **HLE/emulation mode** — full hardware-level emulation via the suyu core (GPU, CPU, audio, services)
- **Recompiler mode** — ahead-of-time static recompilation of Switch AArch64 game code to native x86-64 executables, bundled with suyu's HLE backend

Based on [Eden](https://git.eden-emu.dev/eden-emu/eden), with suyu's own improvements to UI, recompiler, and platform support.

## Status

Hotfix version: **v0.0.4.1**. See the [release page](../../releases/tag/v0.0.4.1)
for the Windows portable download and its verification limits. Extract the whole
archive and launch `suyu.exe`; keep its DLLs, plugins, and `user` folder together.

GitHub Actions is configured to validate branch and pull-request builds. Its
tag-publication job requires successful builds and preserves an existing release
without replacing its assets.

Windows packages must pass startup smoke tests in Gamer, Technical, and Hacker
modes before publication. The workflow also builds Linux, macOS, Android, and
Libretro targets, with optional FreeBSD and Android Libretro jobs.

The Windows startup crash, source changes, and verified local portable build are
documented in [Windows Startup Hotfix](docs/WINDOWS_STARTUP_HOTFIX.md). The local
build was tested before publication; no personal settings, saves, keys, firmware,
or games are included in the release archive.

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
