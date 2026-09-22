<div align="center">

[简体中文](README.md) · [繁體中文](README_TC.md) · [English](README_EN.md)

<img src="docs/logo_english.png" width="280" alt="Armello"/>

# Armello Tooltip Fix

<img src="docs/slogan.png" width="380" alt="By Armellians, for Armellians"/>

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-PC%20%2F%20Steam-green.svg)](#-installation)
[![Game](https://img.shields.io/badge/Game-Armello-orange.svg)](https://store.steampowered.com/app/290340/Armello/)
[![Release](https://img.shields.io/github/v/release/deserthouse/armello-tooltip-fix?include_prereleases&color=yellow&style=flat-square)](https://github.com/deserthouse/armello-tooltip-fix/releases)


</div>

---

> **Armello Tooltip Fix** is a [BepInEx](https://github.com/BepInEx/BepInEx) IL2CPP patch that fixes an Armello engine bug where tooltip link tags (`[/url]`) in card descriptions get split across line breaks and leak as visible garbled text.
>
> 🧩 Recommended companion to [Armello Chinese Relocalization](https://github.com/deserthouse/armello-chinese-localization) — independent installation, no dependency.

## 🐛 The Problem

Armello card descriptions contain tooltip links (e.g., **Explode Pool**, **Pact** in card effects — hoverable terms showing explanations). The engine converts `<tooltip_X>content</tooltip_X>` into `[url="tooltip://X"][u]content[/u][/url]` before passing to the text rendering pipeline.

**Bug**: The line-wrapping algorithm can break in the middle of the 6-character `[/url]` tag (e.g., `[/ur` at end of line, `l]` at start of next), causing the tag parser to fail to recognize the complete tag — rendering it as visible garbled text (like `[/ur l]`).

This bug exists in the game engine itself (Unity 2019.4 / NGUI / IL2CPP) and triggers in the official version too — it's just that official Chinese text lengths happen to mostly avoid the trigger zone. Any mod that changes text length in card descriptions can trigger it.

## ✨ The Fix

Hooks `NGUIText.WrapText` output via [Harmony](https://github.com/pardeike/Harmony) postfix, detecting `[/url]` tags split by `\n` and rejoining them so the parser consumes them correctly.

- ✅ Fixes visible `[/url]` leaking
- ✅ Does not affect tooltip functionality (hover and click both work)
- ✅ Does not affect other rich text tags (`[b]` `[i]` `[u]` `[c]` etc.)
- ✅ Does not modify any game files (pure runtime patch)
- ✅ Console disabled — invisible to players

## 🚀 Installation

**[📥 Download from Releases](https://github.com/deserthouse/armello-tooltip-fix/releases)**

1. Download `ArmelloTooltipFix-v1.0.zip` and extract
2. Copy all contents (`BepInEx/`, `dotnet/`, `winhttp.dll`, etc.) to the Armello game root directory (in Steam, right-click Armello → Manage → Browse Local Files)
3. Launch the game

> The patch does not modify any game files. Delete `winhttp.dll` to disable it; for a full cleanup, also remove `BepInEx/`, `dotnet/`, `doorstop_config.ini`, and `.doorstop_version`.

### Compatibility

Fully compatible with any Armello text/translation mod (including [armello-chinese-localization](https://github.com/deserthouse/armello-chinese-localization)). Independent installation, no dependency.

## 🔧 For Developers

| Layer | Detail |
|---|---|
| Engine | Unity 2019.4.11f1 / IL2CPP metadata v24 / x64 |
| Framework | BepInEx 6.0.0-pre.2 (Unity.IL2CPP.win-x64) |
| Hook | Harmony patch on `NGUIText.WrapText` (3 overloads) |
| Fix | Postfix regex repairs `[/url]` tags split by `\n` |

### Building from Source

```bash
git clone https://github.com/deserthouse/armello-tooltip-fix.git
cd armello-tooltip-fix/src
dotnet build -c Release
# Output: bin/Release/net8.0/ArmelloUrlFix.dll
```

DLL references in the project file point to your local BepInEx interop directory.

### Known Limitations

- Only fixes `[/url]` tag splitting (6-char tag, most likely to trigger)
- Theoretically `[/u]` (4-char) and other tags could also split, but occurrence rate is very low
- If other tag leaks appear, the regex can be extended

## 🤖 AI Usage Disclosure

The research, root cause analysis, source code reverse engineering, Harmony patch development, and debugging were performed by **AI (large language models)**. The human role was requirements, in-game testing, and final acceptance.

## 📄 License

- This is a non-commercial fan project for personal use by players who have purchased Armello.
- Armello is copyrighted by League of Geeks. This repository is not affiliated with the official team.
- Provided "AS IS", use at your own risk.

## ⚠️ Disclaimer

- This patch does not modify game files, but runtime hooks may be incompatible with game updates.
- Users assume all risks from using this patch.
- Use of this patch constitutes acceptance of these terms.
