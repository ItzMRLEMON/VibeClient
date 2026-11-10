# Vibe

An all-in-one, client-side Minecraft utility client built as a Fabric mod for **Minecraft 26.3 (Java Edition)**.
Dark UI, violet accent, modular architecture, everything configurable and saved.

> **Status: early development (v0.2.3).** The foundation, menu, HUD editor, keybind system, Zoom and Fullbright
> exist. Much of the original feature list is still to come; see [Roadmap](#roadmap). Nothing in the UI is a
> placeholder: every button does what it says.

## Requirements

| Thing | Version |
|---|---|
| Minecraft | 26.3 (Java Edition) |
| Java | 25 |
| Fabric Loader | 0.19.5+ (tested on 0.20.1) |
| Fabric API | 0.160.5+26.3 or newer for 26.3 (tested on 0.166.0) |
| Gradle (build only) | 9.5.1 via the wrapper |

Vibe is client-side only. Do not install it on a server.

## Building

```powershell
# one-time: create the Gradle wrapper (your global Gradle may be too old for Loom 1.17)
mkdir $env:TEMP\wrapgen; cd $env:TEMP\wrapgen
New-Item settings.gradle -ItemType File
gradle wrapper --gradle-version 9.5.1
Copy-Item gradlew, gradlew.bat <project>\
Copy-Item gradle\wrapper\* <project>\gradle\wrapper\ -Force

# build
cd <project>
.\gradlew build
```

The jar is written to `build/libs/vibe-<version>.jar`. Drop it into your `mods` folder next to Fabric API.

26.x is unobfuscated, so the build has no `mappings` line and uses `implementation` instead of `modImplementation`.
Versions are pinned in `gradle.properties`.

## Using Vibe

| Action | Default |
|---|---|
| Open the Vibe menu | **Right Shift**, or the **Vibe** button on the Game Menu (pause screen) |
| Zoom | Hold **C** |

### Menu pages

- **Home**: dashboard with username, Minecraft version, FPS, ping, server IP, enabled modules and recently used modules.
- **Modules**: paged list of modules with ON/OFF buttons. The `...` button opens that module's settings (toggles and sliders).
- **HUD Editor**: drag HUD elements around with snapping, resize, recolour, change opacity, enable/disable, and manage layout profiles. See below.
- **Keybinds**: searchable list of every Vibe keybind with rebinding, modifiers, Toggle/Hold mode, reset and conflict detection.

### Modules

| Module | Category | What it does |
|---|---|---|
| Zoom | Camera | OptiFine-style smooth zoom. Settings: zoom level (1.5-30x), smooth speed, lower sensitivity while zoomed. On by default. |
| Fullbright | Camera | Maximum light everywhere, client-side only. Setting: strength. |
| FPS | HUD | Frames per second. |
| Coordinates | HUD | Block coordinates and facing direction. |
| CPS | HUD | Clicks per second (left \| right). |
| Toggle Sprint | Player | Sprints automatically while you hold forward. |

### HUD editor

- Drag an element to move it. It snaps to a 10px grid, screen edges, the screen centre and other elements (pink guide lines show the snap).
- Select an element to change size, background opacity, text colour (8 presets), or reset its position.
- The bottom row enables/disables every HUD element.
- **Profiles**: click the profile button to switch, **New** to copy the current layout into a new profile, **Delete** to remove the active one. The active profile is autosaved when you switch or leave the editor.

### Keybinds

Each bind has a key, an exact modifier set (Ctrl / Shift / Alt) and a mode:

- **Toggle**: each press flips it (for module binds, on/off).
- **Hold**: active only while the key is held.

Every module gets an unbound toggle key. Click a key button, then press the new key (hold modifiers first for combos).
`Esc` cancels, `Backspace`/`Delete` unbinds, `R` resets that bind. Conflicts (same key + modifiers) are shown in red.
Binds are ignored while any screen is open (chat, inventory, menus).

## Files Vibe writes

All under your Minecraft `config/vibe/` folder:

| Path | Contents |
|---|---|
| `config.json` | Module enabled state, settings, keybinds, recently used list |
| `backups/` | Last 10 automatic backups of `config.json`, one per launch |
| `profiles/*.json` | HUD layout profiles |
| `profiles/_active.txt` | Name of the active HUD profile |
| `config.corrupt-<time>.json` | Created if `config.json` could not be parsed; Vibe then starts from defaults |

Key codes are SDL scancodes, so keybinds in `config.json` are specific to Minecraft 26.x.

## Project layout

```
src/main/java/dev/vibe/
  VibeClient.java        entrypoint: registers modules, config, profiles, tick/shutdown hooks
  Vibe.java              mod id + logger
  Compat.java            every Minecraft/Fabric API call that has been renamed between versions
  core/                  Module, ModuleManager, Setting, Keybind, Category, Recent
  config/                ConfigManager (atomic saves, backups, corrupt-file handling)
  hud/                   HudModule (base class), HudManager (renders elements), HudProfiles
  modules/               the actual features
  gui/                   Home, Modules, HUD editor and Keybind screens, setting slider
  mixin/                 CameraMixin (zoom), LightmapMixin (fullbright), PauseScreenMixin (menu button)
src/main/resources/
  fabric.mod.json, vibe.mixins.json, assets/vibe/lang/en_us.json
```

### Design rules

- **Modules are independent.** Each extends `Module`, owns its settings and keybinds, and is isolated: an exception in a module's tick is logged, and three in a row disable just that module.
- **Everything version-fragile goes in `Compat.java`** (drawing, screens, key constants, identifiers). If a Minecraft update renames something, fix it there first.
- **Mixins are minimal** and only call into module code. A mixin that fails to apply should fail loudly at launch (`defaultRequire: 1`), not silently do nothing.
- **Never leave game state changed.** Modules that touch game options (Zoom's mouse sensitivity) restore them on disable, on leaving a world and on shutdown.

### Adding a module

1. Create a class in `dev.vibe.modules` extending `Module` (or `HudModule` for a one-line HUD element).
2. Add settings with `add(new Setting.Bool(...))` / `add(new Setting.Num(...))`. They appear in the Modules page and are saved automatically.
3. Add extra keybinds with `addBind(new Keybind(id, label, defaultKey, defaultMods, mode))`.
4. Register it in `VibeClient.onInitializeClient()` with `modules.register(new YourModule())`.

## Known limitations

- Keybinds are polled 20 times a second, so an extremely quick tap can be missed.
- Conflict detection covers Vibe's own binds only, not vanilla Minecraft keys.
- Mouse buttons cannot be bound yet.
- HUD fonts cannot be changed yet.
- Fullbright overrides the lightmap state fields; how complete it looks in caves/the Nether has not been fully verified.
- The startup log line prints a hardcoded version string.

## Roadmap

Planned from the original spec, roughly in this order:

1. **Simple modules**: dynamic FOV, toggle perspective, freelook, custom crosshair + editor, armor / potion / keystrokes / reach / durability / hunger HUDs, stopwatch, timer, compass, chat timestamps, auto GG, overlay toggles, clear water.
2. **World**: waypoints, death coordinates, minimap and world map, light level overlay, chunk borders, slime chunks, block/entity info.
3. **Performance**: FPS/RAM/CPU/GPU monitor and graphs, particle limiter, presets (Potato to Maximum FPS).
4. **Account manager**: official Microsoft/Minecraft authentication only, tokens stored securely, never passwords.
5. **Skin editor**: PNG import, username lookup, 3D preview, pixel tools, layers, templates, export.
6. **Cosmetics**: capes, wings, hats, back items, particles, emotes, poses, with clear local-only vs shared indicators.
7. **Extras**: screenshot manager/editor, resource pack and shader managers, chat translator and filters, notepad, calculator, custom menu backgrounds/music, update checker.
8. **Social** (optional online backend, never required): friends, parties, messages, server list, profiles. This needs a backend that does not exist yet.

## Licence

All rights reserved for now.
