<div align="center">

[簡體中文](README.md) · [繁體中文](README_TC.md) · [English](README_EN.md)

<p><img src="docs/logo_tc.png" width="280" alt="愛門羅"/></p>

<p><img src="docs/slogan.png" width="380" alt="By Armellians, for Armellians"/></p>

# Armello Tooltip Fix

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-PC%20%2F%20Steam-green.svg)](#-安裝)
[![Game](https://img.shields.io/badge/Game-Armello-orange.svg)](https://store.steampowered.com/app/290340/Armello/)
[![Release](https://img.shields.io/github/v/release/deserthouse/armello-tooltip-fix?include_prereleases&color=yellow)](https://github.com/deserthouse/armello-tooltip-fix/releases)

</div>

<br/>

> **Armello Tooltip Fix** 是一個 [BepInEx](https://github.com/BepInEx/BepInEx) IL2CPP 補丁，修復 Armello 遊戲引擎中卡牌描述的 tooltip 連結標籤（`[/url]`）在換行時被拆斷、洩露為可見亂碼文字的 bug。
>
> 🧩 [Armello 簡體中文重譯補丁](https://github.com/deserthouse/armello-chinese-localization) 的推薦搭配——兩者獨立安裝，互不依賴。

## 🐛 問題描述

Armello 的卡牌描述中包含 tooltip 連結（如卡牌效果中的**爆發池**、**契約**等可懸停查看解釋的詞）。遊戲引擎將 `<tooltip_X>內容</tooltip_X>` 轉換為 `[url="tooltip://X"][u]內容[/u][/url]` 後交給文字渲染管線。

**Bug**：換行演算法可能在 `[/url]` 六個字元的中間斷行（如 `[/ur` 在行尾、`l]` 在下一行頭），導致標籤解析器無法識別完整標籤，將其渲染為可見的亂碼文字（形如 `[/ur l]`）。

此 bug 存在於遊戲引擎本身（Unity 2019.4 / NGUI / IL2CPP）而非第三方修改。官方中文版本透過調整每一行的文字長度來規避此問題，而我不願意為此妥協，因此製作了這個補丁。需注意任何修改遊戲文字導致描述長度變化的 mod 都可能觸發此問題，而該補丁在理論上均能將其修復。

### 效果對比（以月亮鐮刀卡牌為例，需注意此問題是由遊戲引擎本身的 bug 導致的，與第三方修改無關）

**修復前**——`[/url]` 被換行拆斷，洩露為可見亂碼文字：

<img src="docs/screenshots/before.png" width="780" alt="修復前：卡牌描述中可見 [/url] 殘片"/>

**修復後**——文字乾淨，關鍵詞懸停提示（tooltip）正常運作：

<img src="docs/screenshots/after.png" width="780" alt="修復後：描述乾淨，懸停提示正常"/>

## ✨ 修復方案

透過 [Harmony](https://github.com/pardeike/Harmony) hook `NGUIText.WrapText` 的輸出端，在換行完成後檢測被 `\n` 拆斷的 `[/url]` 標籤，移除標籤內的換行符使標籤恢復完整，讓解析器正確消費。

- ✅ 修復 `[/url]` 可見洩露
- ✅ 不影響 tooltip 連結功能（懸停、點擊均正常）
- ✅ 不影響其他富文字標籤（`[b]` `[i]` `[u]` `[c]` 等）
- ✅ 不修改任何遊戲檔案（純執行時補丁）
- ✅ 控制台已關閉，玩家無感知

## 🚀 安裝

**[📥 前往 Releases 下載最新版本](https://github.com/deserthouse/armello-tooltip-fix/releases)**

1. 下載 `ArmelloTooltipFix-v1.0.zip` 並解壓
2. 將解壓出的全部檔案（`BepInEx/` `dotnet/` `winhttp.dll` 等）複製到 Armello 遊戲根目錄（Steam 庫中右鍵 Armello → 管理 → 瀏覽本地檔案）
3. 啟動遊戲，完成

> 補丁不修改任何遊戲檔案，刪除 `winhttp.dll` 即可停用補丁；如需徹底清理，一併刪除 `BepInEx/`、`dotnet/`、`doorstop_config.ini` 與 `.doorstop_version`。

### 與文字 mod 的相容性

本補丁與任何 Armello 文字/翻譯 mod 相容（包括 [armello-chinese-localization](https://github.com/deserthouse/armello-chinese-localization)）。兩者獨立安裝、互不依賴。

## 🔧 給開發者

| 層 | 說明 |
|---|---|
| 引擎 | Unity 2019.4.11f1 / IL2CPP metadata v24 / x64 |
| 框架 | BepInEx 6.0.0-pre.2 (Unity.IL2CPP.win-x64) |
| Hook | Harmony patch on `NGUIText.WrapText` (3 overloads) |
| 修復 | Postfix 正規表示式修復被 `\n` 拆斷的 `[/url]` 標籤 |

### 從原始碼構建

```bash
git clone https://github.com/deserthouse/armello-tooltip-fix.git
cd armello-tooltip-fix/src
dotnet build -c Release
# 輸出: bin/Release/net8.0/ArmelloUrlFix.dll
```

專案檔案中的 DLL 引用路徑需要指向你本機的 BepInEx interop 目錄。

### 已知侷限

- 僅修復 `[/url]` 標籤跨行（6 字元標籤，最容易觸發）
- 理論上 `[/u]`（4 字元）等其他標籤也可能跨行，但實際發生率極低
- 如果未來出現其他標籤洩露，可擴充套件正規表示式匹配

## 🤖 AI 使用宣告

本補丁不含任何人類成分，絕大多數工作都由 **AI** 完成。

## 📄 版權宣告

- 本補丁為非商業粉絲專案，僅供已購買 Armello 的玩家個人使用
- 《Armello》版權歸 League of Geeks 所有，本倉庫與官方無任何關聯
- 補丁按「現狀」提供，使用風險自負

## ⚠️ 免責聲明

- 本補丁不修改任何遊戲檔案，但執行時 hook 可能與遊戲更新不相容
- 因使用本補丁導致的任何直接或間接損失，維護者不承擔責任
- 使用本補丁即表示你已閱讀並同意上述條款
