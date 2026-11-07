# Animal Crossing: New Leaf VR — Development Plan

This repository is a game-specific development fork of AzaharXR for investigating an immersive Meta Quest 3 experience for **Animal Crossing: New Leaf**.

## Project goals

- Retain AzaharXR's Quest-native Android, OpenXR, and 3DS emulation foundation.
- Keep the original game's simulation, dialogue, villagers, items, shops, activities, progression, and save data intact wherever technically possible.
- Develop a title-specific immersive presentation and input layer.
- Support the user's own legally obtained game dump. Do not include Nintendo game images, firmware, keys, or copyrighted game assets in this repository or APK.

## Important technical distinction

AzaharXR already provides an OpenXR headset session, stereo views, tracked controller input, and a VR presentation layer. Those capabilities do **not** automatically turn a 3DS game's camera into a first-person, 6DoF world. New Leaf's game camera, scene geometry, UI, and interactions are controlled by the emulated title. A true first-person conversion will require title-specific investigation and likely game-aware camera/rendering/input work. Do not describe a stereoscopic screen or floating 3DS display as a completed immersive port.

## Milestones

### 0. Establish a reproducible baseline
- Build the unmodified fork for Android/Quest using its documented toolchain.
- Record required SDK/NDK, CMake, Gradle, Java, and native dependency versions from the repository.
- Verify APK packaging and installation before changing emulator behavior.
- Keep upstream functionality and the default branch unchanged.

### 1. New Leaf title detection and profile
- Identify New Leaf reliably from its title metadata.
- Add an opt-in per-title VR profile without changing behavior for other games.
- Provide safe fallback to the normal AzaharXR presentation.

### 2. Player avatar as the VR body

The intended first-person body is the player's own New Leaf villager avatar, not a generic humanoid or a replacement character.

- Reuse the avatar appearance, clothing, hair, skin/face textures, proportions, and animation style from the user's own New Leaf game data. Do not ship extracted Nintendo assets in source control or the APK.
- Preserve the in-game avatar as the source of truth for customization. Clothing/hair changes and saved appearance should remain reflected in the VR body.
- The VR body should be aligned to the player's tracked head and hands, with the original avatar's body proportions and compatible idle/walk/run/tool-use animations.
- Avoid showing a duplicate head or clipping the camera through the face in first person. Investigate a first-person body mesh/head-visibility treatment while retaining the full avatar in third-person or mirror views.
- Do not assume that the emulator's current stereo screen output exposes a reusable 3D character mesh. The current renderer presents the emulated top/bottom framebuffers; it does not provide a game-agnostic avatar model API. A title-specific asset/animation path or game-aware runtime hook is required.
- First prototype should validate extraction/identification of the avatar model and its skeleton from a user-supplied game dump, then prove one correctly scaled idle pose before adding tracked-hand retargeting.

### 3. Rendering and camera feasibility
- Trace the existing stereo render path from emulated frame output to OpenXR eye swapchains.
- Determine whether game camera matrices or scene data can be accessed through supported emulator interfaces.
- Prototype camera/head-pose mapping behind an experimental setting; preserve a recenter and comfort fallback.
- Validate depth, scale, clipping, UI readability, and motion comfort before enabling by default.

### 4. Input and interaction
- Map Quest controller input to existing game controls first.
- Prototype context-sensitive interaction only after confirming reliable game-state/input hooks.
- Keep menus, inventory, dialogue, touch-screen functions, and text entry accessible.
- Do not claim tracked-hand interaction until hand poses are actually consumed by the game-specific layer.

### 5. Compatibility and release
- Test boot, new game, save/load, town traversal, dialogue, shops, inventory, activities, and long sessions.
- Measure Quest 3 frame timing, thermal behavior, memory use, and crash recovery.
- Publish reproducible build instructions and release notes.
- Distribute no copyrighted game data; users supply their own backups.

## Current status

- Development branch: `feature/new-leaf-immersive-vr`
- Base: `master` (AzaharXR fork)
- Branch setup is complete.
- A clean baseline Quest APK build has succeeded and the artifact was uploaded by GitHub Actions (run `42410024866`).
- No New Leaf-specific avatar model, first-person camera, or immersive rendering has been implemented yet.
- Next avatar milestone: identify and validate the player model/skeleton and animation data from the user's own game dump; then prototype a correctly scaled VR body.

## Contribution rules

- Make changes on the feature branch, not `master`.
- Prefer small, reviewable commits.
- Keep experimental New Leaf behavior behind a title-specific setting or feature flag.
- Document test evidence; distinguish verified behavior from planned behavior.
- Respect the GPL-3.0 license and preserve applicable upstream notices.
