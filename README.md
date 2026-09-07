# Bethesda Plugin Manager

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)
![Static Badge](https://img.shields.io/badge/built_with-claude-blue?logo=claude&labelColor=%23e8e6e3&color=%23C15F3C)
![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/hierocles/modorganizer-bsplugins/build.yml)
![GitHub Release](https://img.shields.io/github/v/release/hierocles/modorganizer-bsplugins)


A fork of [Exit-9B/modorganizer-bsplugins](https://github.com/Exit-9B/modorganizer-bsplugins), the
plugin-load-order panel for Mod Organizer 2, with [Alaxouche](https://github.com/Alaxouche)'s
stability and group-management work, updated to build and run against current MO2 2.5.3.

## New features

Alaxouche's extensions:

- Improved stability and startup reliability for the plugin panel
- Group management: `Reset Groups`, `Merge Group`, `Clean Groups`, a "Remove Group..." prompt
- Optional confirmation prompts for risky actions (removing a separator, etc.)
- Improved drag-and-drop for full group moves, and performance on large load orders
- Load order snapshot restore after external tool runs, and protection against unwanted external
  load order changes
- An option to fully disable the group system
- Restored "Lock Load Order Position" for plugins, which upstream had removed

Since then:

- Plugin panel parity with upstream MO2's plugin list: Form Version / Header Version / Author /
  Description columns and tooltips, highlighting the masters of a selected plugin, and gating
  LOOT's sorted load order behind an explicit Apply
- Added [mo2-superbuild](https://github.com/mo2-modern/mo2-superbuild) support, alongside the
  existing `mob`/`modorganizer_super` standalone build
- Various build fixes for current `mo2-uibase`/Qt

## Compatibility

Requires Qt 6.11+. `QSortFilterProxyModel::beginFilterChange()`/`endFilterChange()`, used for the
masters-highlight feature, don't exist before Qt 6.11 — a build of this plugin won't load on MO2
2.5.2 or earlier (bundled Qt 6.7.1).

No public MO2 release ships Qt 6.11+ yet, though beta releases are available in the MO2 Discord.


## Building

Two ways: standalone against a `mob` tree, or nested inside
**mo2-superbuild**. `src/CMakeLists.txt` is the same either way — it resolves everything through
`find_package(mo2-* CONFIG REQUIRED)`.

### Nested in mo2-superbuild

Add this repo as a submodule under `repos/bsplugins` and add `mo2_add_repo(bsplugins)` to
mo2-superbuild's `CMakeLists.txt`.

### Standalone (mob / modorganizer_super)

Needs:

- A built `mob` tree, with at least `cmake_common`, `uibase`, `bsatk`,
  `lootcli-header` and `usvfs` installed
- A matching Qt 6.11+ install
- [vcpkg](https://github.com/microsoft/vcpkg), with `VCPKG_ROOT` set

```powershell
$env:DEPENDENCIES_DIR = "<path to your mob tree root>"
$env:QT_ROOT = "<path to your Qt install, e.g. ...\Qt\6.11.1\msvc2022_64>"
cmake --preset vs2022-windows
cmake --build vsbuild --config RelWithDebInfo --target bsplugins
```

`DEPENDENCIES_DIR` is the tree's root: `cmake_common` is expected under `.../build/cmake_common` or
`.../cmake_common`, and built packages under `<DEPENDENCIES_DIR>/install`.

`cmake --install vsbuild --config RelWithDebInfo --prefix <path>` places `bsplugins.dll` under
`bin/plugins` and its translation under `bin/translations`.

