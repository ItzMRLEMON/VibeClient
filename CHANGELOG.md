# Changelog

All notable changes to Vibe. Versions follow the order the builds were produced.
Target: Minecraft 26.3 (Java Edition), Fabric.

## 0.2.3 - Pause-menu button and keybind fix
### Added
- **Vibe button** on the Game Menu (pause screen), top-right corner, opens the Vibe Home page (`PauseScreenMixin`).
### Fixed
- The Right Shift menu key never triggered: Right Shift counted itself as an extra modifier, so the "exact modifiers" check always failed. A modifier key bound on its own now matches.

## 0.2.2 - Fullbright
### Added
- **Fullbright** module with a strength slider (`LightmapMixin` on `LightmapRenderStateExtractor.extract`). Overrides night-vision intensity, brightness, darkness effect and boss darkening in the lightmap state; forces a refresh while on and once after turning off.
### Fixed
- `gradle.properties` had a corrupted Fabric API version (`0.2.2.5+26.3`) caused by a bad find-and-replace during the version bump. Restored to `0.160.5+26.3`.

## 0.2.1 - Keybind system and smooth zoom
### Added
- **Vibe keybind system**: key + exact modifiers (Ctrl/Shift/Alt) + Toggle or Hold mode, saved in config. Replaces the vanilla key registration from 0.1.0.
- **Keybinds page**: search, click-to-rebind, Hold/Toggle button, per-bind reset, Reset all, red conflict markers and a conflict status line.
- Every module now has an unbound toggle key; global "open menu" bind (Right Shift by default).
- **Smooth zoom** via `CameraMixin` on `Camera.calculateFov` (FOV moved from `GameRenderer` to `Camera` in 26.3): true float FOV, no 30-degree limit, frame-rate-independent smoothing.
- Zoom settings: zoom level (1.5-30x), smooth speed, lower mouse sensitivity while zoomed. Zoom is enabled by default.
- `vibe.mixins.json` and the `mixins` entry in `fabric.mod.json`.
### Changed
- The zoom key (C) is now an ordinary Vibe bind (Hold mode by default, can be switched to Toggle).
- Language file reduced; vanilla Controls no longer lists Vibe keys.
### Removed
- Zoom no longer drives the vanilla FOV option.

## 0.2.0 - Menu, HUD editor and profiles
### Added
- Menu navigation bar: Home, Modules, HUD Editor.
- **Home dashboard**: username, Minecraft version, FPS, ping, server IP, enabled modules, recently used modules (persisted).
- **Modules page**: paged list with ON/OFF and a settings panel (toggles and sliders) for the selected module.
- **HUD editor**: drag and drop with grid, edge, centre and element snapping plus guide lines; size, opacity and colour controls; per-element enable/disable; reset position.
- **HUD profiles**: switch, create from current layout, delete; stored in `config/vibe/profiles/`, active profile restored on startup.
- HUD elements gained saved scale, colour (8 presets) and background opacity settings.
### Changed
- The old single-page module list was replaced by the multi-page menu.

## 0.1.0 (fixes) - First successful build on 26.3
### Fixed
- Gradle: Loom 1.17.21 needs Gradle 9.5+; the wrapper must be generated outside the project when the global Gradle is older.
- 26.3 API changes found while compiling:
  - GLFW is gone (SDL3 input): key codes are SDL scancodes, `Type.KEYSYM` became `Type.KEYBOARD`.
  - `GuiGraphics` became `GuiGraphicsExtractor`, and `drawString` became `text`.
  - `Minecraft.screen` / `setScreen` moved to `Minecraft.gui` (`gui.screen()`, `gui.setScreen(...)`).
  - `Options.hideGui` no longer exists (check removed; vanilla already hides the HUD with F1).
- Introduced `Compat.java` to keep these renames in one place.

## 0.1.0 - Phase 1 foundation
### Added
- Fabric project setup for Minecraft 26.3: Loader 0.19.5, Fabric API 0.160.5+26.3, Loom 1.17, Gradle 9.5.1, Java 25, unobfuscated (no mappings).
- Module system with settings, per-module error isolation (three errors disable only that module).
- JSON config with atomic saves, 10 rolling backups and corrupt-file handling.
- HUD framework and the first modules: **Zoom** (vanilla FOV option based, replaced in 0.2.1), **FPS**, **Coordinates**, **CPS**, **Toggle Sprint**.
- Basic Right Shift menu with module on/off buttons.
