# macOS (Apple Silicon) — practical playbook

This archival tree does **not** build for macOS. Use the community native stack, or rebase onto it.

## Play (fastest)

1. Own Generals and/or Zero Hour (Steam, EA App, The First Decade, or retail).
2. Download a macOS build from [GeneralsX Releases](https://github.com/fbraz3/GeneralsX/releases).
3. Copy game data (`.big`, INIs, maps, etc.) to the layout GeneralsX expects, typically:
   - `~/GeneralsX/Generals` — base game  
   - `~/GeneralsX/GeneralsZH` — Zero Hour  
4. On first launch, allow the unsigned app under **System Settings → Privacy & Security**.
5. Prefer windowed mode (`-win`) while validating GPU/MoltenVK.

Steam on macOS often will not install this Windows-only title; copy files from a Windows install, another machine, or an extraction method documented by GeneralsX (`GETTING_THE_GAME_FILES.md` in their wiki).

## Build GeneralsX yourself (Apple Silicon)

On a Mac with **macOS 15+**, Xcode CLT, CMake, Ninja, Meson, and the **LunarG Vulkan SDK** (not Homebrew `vulkan-headers`):

```bash
git clone https://github.com/fbraz3/GeneralsX.git
cd GeneralsX
# see docs/BUILD/MACOS.md — typical flow:
./scripts/build/macos/build-macos-zh.sh
./scripts/build/macos/deploy-macos-zh.sh
```

Or CMake presets:

```bash
cmake --preset macos-vulkan
cmake --build build/macos-vulkan --target z_generals
```

Full detail: [GeneralsX docs/BUILD/MACOS.md](https://github.com/fbraz3/GeneralsX/blob/main/docs/BUILD/MACOS.md).

## Making *this* repository the Mac port

```bash
git remote add generalsx https://github.com/fbraz3/GeneralsX.git
git fetch generalsx
# Then either:
#  - reset/replace main onto generalsx/main (clean break from archival layout), or
#  - merge with a large conflict budget and keep LICENSE.md EA terms.
```

Prefer tracking [TheSuperHackers/GeneralsGameCode](https://github.com/TheSuperHackers/GeneralsGameCode) if you want the broader modernization project rather than the GeneralsX packaging focus.

## Legal reminders

- Engine: GPL v3 + EA additional terms in `LICENSE.md` (no EA trademarks in product naming).
- Assets: not included; player must supply a legitimate copy.
- Mark modified engines as modified; do not claim EA affiliation.
