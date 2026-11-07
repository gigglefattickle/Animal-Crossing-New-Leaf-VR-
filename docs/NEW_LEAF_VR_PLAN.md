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
- Initial format/layout research narrows the asset path: New Leaf's documented RomFS tree has a `Player` directory with separate `head_mdl`, `btms_mdl`, `legs_mdl`, `face_tex`, and related clothing/texture resources. The player is therefore an assembly of modular parts, not one monolithic model. The game uses CGFX/BCRES resources; public format tooling documents CGFX model parsing, but that is not yet an in-repo runtime importer.
- Treat the RomFS assets as the base model library and the user's own save data as a separate source of player-specific appearance/customization state. The exact mapping from save fields to head/hair/face/clothing resource IDs still needs to be established from a legally obtained local save and verified against in-game screenshots.
- Azahar already has a `RomFSReader` abstraction and NCCH loader methods for opening/dumping RomFS. This gives us a likely source for title assets without asking users to upload a full game image. It does not yet expose a convenient title-specific player-resource API to the VR renderer.
- Recommended extraction proof: use the emulator's existing RomFS/VFS path to enumerate/read only the relevant `Player/` resources locally; identify BCRES/CGFX headers; parse model, skeleton/bone hierarchy, materials, and animation references with a small offline inspection tool; export a neutral diagnostic format for validation. Keep all extracted assets local and out of Git/APKs.
- Do not hard-code an assumption that all appearance is stored in the base RomFS model. Custom designs, selected clothing, and save-specific appearance may require save-data parsing and runtime refresh hooks.
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
- Avatar research has narrowed the source assets to the modular `Player/` RomFS resources (head, lower body, legs, face textures, and clothing), using CGFX/BCRES model data. Save-data-to-resource mapping, skeleton/animation parsing, and a runtime importer remain unimplemented.
- Next avatar milestone: build a local-only RomFS resource inventory/parser proof, map save customization to the correct player parts, and validate a correctly scaled assembled idle pose.

## Contribution rules

- Make changes on the feature branch, not `master`.
- Prefer small, reviewable commits.
- Keep experimental New Leaf behavior behind a title-specific setting or feature flag.
- Document test evidence; distinguish verified behavior from planned behavior.
- Respect the GPL-3.0 license and preserve applicable upstream notices.
