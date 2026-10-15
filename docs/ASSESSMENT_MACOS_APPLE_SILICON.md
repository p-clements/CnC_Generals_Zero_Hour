# Assessment: Native macOS (Apple Silicon) for C&C Generals / Zero Hour

**Repo assessed:** `p-clements/CnC_Generals_Zero_Hour` (EA GPL v3 source dump)  
**Date:** 2026-10-15  
**Goal:** Can this game be assessed, “decompiled,” and run natively on macOS Apple Silicon?

## Executive verdict

| Question | Answer |
|----------|--------|
| Do we need to decompile? | **No.** Full engine source is already here (GPL v3 + EA additional terms). |
| Does *this* tree build/run on Mac today? | **No.** Win32-only VS6 / DirectX / Miles / Bink stack. |
| Is native Apple Silicon already solved elsewhere? | **Yes.** [GeneralsX](https://github.com/fbraz3/GeneralsX) ships macOS ARM64 builds (DXVK → Vulkan → MoltenVK → Metal). |
| Recommended next step for this fork | **Adopt or track GeneralsX / TheSuperHackers**, don’t rewrite from this archival dump. |

You still need **legally owned game data** (`.big` archives, INIs, maps, audio). Engine source ≠ game assets.

---

## What this repository actually is

This is Electronic Arts’ 2025 archival release of **Command & Conquer Generals** and **Zero Hour**, preserved as two parallel trees:

- `Generals/` — base game
- `GeneralsMD/` — Zero Hour (“Mission Disk”)

Architecture (both trees):

```
Code/
  Main/                 WinMain.cpp entry (HWND / Win32 message pump)
  GameEngine/           Platform-agnostic-ish game logic (~1.1k files ZH)
  GameEngineDevice/     Concrete backends:
                          Win32Device  — OS, FS, input, CD
                          W3DDevice    — Westwood 3D renderer (Direct3D)
                          MilesAudioDevice
                          VideoDevice/Bink
  Libraries/            WWVegas (WW3D2, WWMath, …) + vendor stubs
  Tools/                Editors / exporters (Max SDK, etc.)
```

**Scale (Zero Hour):** ~2.9k C/C++ headers+sources, ~534k lines. Generals vs GeneralsMD differ in thousands of paths.

Build system: legacy **MSVC 6** workspaces (`RTS.dsw` / `.dsp`). README explicitly says **Win32 only**.

---

## Decompilation: not applicable

“Decompile and run natively” usually applies when you only have binaries. Here:

1. **Engine source is present** — Westwood/EA C++ for logic, W3D, networking hooks, tools.
2. **Binaries are not in-repo** — `Run/` only has `BrowserEngine.dll` + a Steam-build placeholder.
3. **Assets are not licensed for redistribution** — Steam / EA App / retail data must be supplied by the player.
4. **Missing proprietary SDKs** are gitignored stubs (not something decompilation recovers cleanly):
   - DirectX 9 SDK
   - STLport 4.5.3
   - RAD Miles Sound System
   - RAD Bink
   - GameSpy SDK
   - SafeDisk API
   - 3DS Max 4 SDK, NVASM, BYTEmark, etc.

Decompiling a retail `.exe` would duplicate work EA already open-sourced and would still leave you without Miles/Bink/DX and without assets.

---

## Blockers for native macOS / arm64 from *this* tree

### 1. OS / windowing / I/O (`Win32Device` + `WinMain`)

Entry and OS glue are pure Win32 (`windows.h`, `HWND`, DirectInput keyboard/mouse, registry, CD manager, BIG file paths). There is no POSIX/SDL/Cocoa device layer in this dump.

### 2. Graphics (`W3DDevice` + WW3D2)

Renderer talks **Direct3D** (~38 translation units reference D3D types/APIs). Apple Silicon has no native D3D. Practical path used by community ports:

`D3D8/9 calls → DXVK → Vulkan → MoltenVK → Metal`

Rewriting WW3D to Metal/OpenGL from scratch is a much larger project than translating D3D.

### 3. Audio / video

- Miles → needs OpenAL / MiniAudio (or similar) replacement  
- Bink → needs FFmpeg (or skip cinematics)

### 4. Language / toolchain

- Written for MSVC 6-era C++ (non-standard habits, STLport)
- Inline x86 assumptions / `__declspec` / SEH appear in places
- Modern Clang (arm64-apple-darwin) needs the modernization work already done upstream in community forks

### 5. Multiplayer / DRM leftovers

GameSpy and SafeDisk paths exist; modern ports replace matchmaking (e.g. GeneralsOnline / NGMP) and strip CD checks.

### 6. License / branding

GPL v3 **plus** EA §7 terms: no EA trademark use in distribution branding, mark modified versions as modified, no warranty. Fine for a clearly named community engine binary; do not ship as “official EA Generals.”

---

## Feasibility matrix

| Approach | Effort | Risk | Result on M-series Mac |
|----------|--------|------|-------------------------|
| **A. Play via GeneralsX release** | Low (install + copy owned assets) | Low | Native arm64 today |
| **B. Build GeneralsX from source on Mac** | Medium (CMake, Vulkan SDK, MoltenVK) | Medium | Native arm64, hackable |
| **C. Rebase this fork onto TheSuperHackers → GeneralsX** | Medium–High (git surgery) | Medium | Same as B, under your org |
| **D. Port this dump from scratch** | Very high (hundreds of invasive changes across 500k+ LOC) | High | Reinvents GeneralsX |
| **E. Decompile retail EXE** | High, low value | High | Redundant with GPL source |

**Recommendation:** **A** for “get it running,” **B/C** if this repo should become your development home. Avoid **D/E**.

---

## Community lineage (already did the hard parts)

```
electronicarts/CnC_Generals_Zero_Hour   (this shape — archival)
        ↓
TheSuperHackers/GeneralsGameCode        (modernization, CMake, fixes)
        ↓
Fighter19 / feliwir work                (SDL3, DXVK, OpenAL prototypes)
        ↓
fbraz3/GeneralsX                        (macOS Apple Silicon + Linux + Win)
        ↓
ammaarreshi/Generals-Mac-iOS-iPad       (iOS/iPadOS on same stack)
```

GeneralsX macOS build model (summary):

- CMake preset `macos-vulkan` (arm64)
- DXVK (pinned fork) + MoltenVK / LunarG Vulkan SDK
- SDL3 for window/input; OpenAL/MiniAudio; FFmpeg for video
- Targets such as `z_generals` / `GeneralsXZH`
- Requires game files under e.g. `~/GeneralsX/GeneralsZH/`

Docs: [GeneralsX macOS build](https://github.com/fbraz3/GeneralsX/blob/main/docs/BUILD/MACOS.md), [installation](https://github.com/fbraz3/GeneralsX/blob/main/docs/HOWTO/INSTALLATION.md).

---

## What a from-scratch port of *this* repo would entail

If you insist on evolving this tree without importing GeneralsX history, minimum workstreams:

1. **Build** — Replace `.dsp`/`.dsw` with CMake; drop STLport for libc++; fix MSVC-isms for Clang.
2. **Host** — New `PosixDevice`/`SDLDevice` (files, threads, window, mouse/keyboard); replace `WinMain`.
3. **Gfx** — Keep WW3D API surface; inject DXVK or a Vulkan/Metal backend under W3D.
4. **Audio/Video** — OpenAL + FFmpeg shims behind existing `AudioManager` / `VideoPlayer` interfaces.
5. **Net** — Stub or replace GameSpy; optional GeneralsOnline client.
6. **Packaging** — `.app` bundle, dylib rpaths for MoltenVK, Gatekeeper notes, no EA trademarks in product name.
7. **QA** — Campaign, skirmish, replays, popular mods; CRC/sync for multiplayer.

That is engine-port work measured in large coordinated diffs, not a weekend spike. GeneralsX already encodes most of it.

---

## Constraints of this assessment environment

This Cloud Agent runs on **Linux**, not macOS. We cannot compile or smoke-test an Apple Silicon `.app` here. Validation of native Mac binaries requires a macOS 15+ ARM host (or CI macOS runners) plus owned game data.

---

## Suggested decision for this fork

1. **Immediate play:** Install a GeneralsX macOS release; point it at Steam/EA game files you own.  
2. **If this GitHub repo should be the Mac port home:** Add `fbraz3/GeneralsX` (or TheSuperHackers) as upstream remote and rebase/replace the archival tree, keeping LICENSE attribution and EA additional terms.  
3. **Do not** invest in binary decompilation of Generals/ZH for this goal.  
4. **Optional follow-up in-repo:** thin `docs/MACOS.md` playbook + CI notes once an Apple host is available.

---

## References

- In-repo `README.md` — Win32-only compile notes, dependency list  
- In-repo `LICENSE.md` — GPL v3 + EA additional trademark/disclaimer terms  
- https://github.com/electronicarts/CnC_Generals_Zero_Hour  
- https://github.com/TheSuperHackers/GeneralsGameCode  
- https://github.com/fbraz3/GeneralsX  
- https://github.com/ammaarreshi/Generals-Mac-iOS-iPad  
