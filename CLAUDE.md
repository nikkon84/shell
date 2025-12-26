# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Caelestia Shell is a desktop shell for Hyprland built with Quickshell (QML-based shell framework). It provides a complete desktop experience including bar, launcher, dashboard, notifications, lock screen, and more.

## Build Commands

```bash
# Standard build
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/
cmake --build build

# Install (system-wide)
sudo cmake --install build

# Local development install (to ~/.config/quickshell/caelestia)
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/ \
  -DINSTALL_QSCONFDIR=~/.config/quickshell/caelestia
cmake --build build
sudo cmake --install build
```

### Nix Development

```bash
# Enter dev shell with all dependencies
nix develop

# Run directly without installing
nix run github:caelestia-dots/shell

# Build with debug symbols
nix build .#debug
```

### Running the Shell

```bash
# Via caelestia-cli
caelestia shell -d

# Via quickshell directly
qs -c caelestia
```

## Architecture

### Directory Structure

- `shell.qml` - Entry point, creates ShellRoot with all modules
- `modules/` - Major feature modules (bar, launcher, dashboard, notifications, lock, etc.)
- `services/` - QML singletons for global state (Audio, Brightness, Colours, Network, Notifs, etc.)
- `components/` - Reusable styled UI components (containers, controls, effects, widgets)
- `config/` - Configuration system with JSON schema and typed accessors
- `plugin/` - C++ Qt plugin for performance-critical features (audio processing, beat detection, image analysis)
- `utils/` - Utility QML modules and shell scripts

### Key Patterns

**Singleton Services**: Services like `Audio.qml`, `Colours.qml`, `Network.qml` use `pragma Singleton` for global state management. Access via `import "root:/services"`.

**Configuration**: `Config.qml` singleton loads `~/.config/caelestia/shell.json` with hot reload support. Each feature has its own typed config (BarConfig, LauncherConfig, etc.).

**Module Composition**: Each major feature is self-contained in `modules/`. Modules compose smaller components from `components/`.

**C++ Plugin**: Located in `plugin/src/Caelestia/`, provides three QML modules:
- `Caelestia` - Core utilities (calculator, app database, HTTP requests)
- `Caelestia.Internal` - Device management, image caching, logind integration
- `Caelestia.Services` - Audio processing via PipeWire, beat tracking, cava visualizer

### IPC Communication

Shell exposes IPC targets via `modules/Shortcuts.qml`:
- `drawers` - toggle/list drawers
- `notifs` - clear notifications
- `lock` - lock/unlock session
- `mpris` - media player control
- `picker` - screen area picker
- `wallpaper` - wallpaper management

Access via: `caelestia shell <target> <function> [args...]`

## Build System Details

CMake options:
- `ENABLE_MODULES` - Modules to build: extras, plugin, shell (default: all)
- `INSTALL_LIBDIR` - Library install path (default: usr/lib/caelestia)
- `INSTALL_QMLDIR` - QML plugin path (default: usr/lib/qt6/qml)
- `INSTALL_QSCONFDIR` - Shell config path (default: etc/xdg/quickshell/caelestia)

C++20 required. Strict warnings enabled (-Wall -Wextra -Wpedantic and more).

## Key Dependencies

- Quickshell (git version) - QML shell framework
- Qt6 (Core, Qml, Gui, Quick, Concurrent, Sql, Network, DBus)
- PipeWire - Audio
- Hyprland - Window manager (IPC integration)
- libqalculate - Calculator
- aubio - Beat detection
- libcava - Audio visualizer

## Development Tips

- QML changes hot-reload automatically when Quickshell is running
- Configuration changes in `~/.config/caelestia/shell.json` trigger immediate reload with toast notification
- Use `caelestia shell -s` to list all available IPC commands
- For C++ plugin changes, rebuild required: `cmake --build build`
