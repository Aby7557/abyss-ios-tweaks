# 设置面板图标规范

从建工程就放对，事后不救火。两类图标互相独立：

| 图标 | 决定文件 | 出现位置 |
|---|---|---|
| 设置面板图标（本文重点） | deb 内 `PreferenceBundles/*Prefs.bundle/icon*.png` | 设置 → 越狱插件 列表行左侧 |
| 源列表图标 | 源仓库 `icons/<Package>.png`，由源的 gen.yml 注入 `Icon:` 字段 | Irisin/源页 每个包的图 |

## 尺寸（29pt 基准，记死）

| 文件 | 像素 |
|---|---|
| `icon.png` | 29×29 |
| `icon@2x.png` | 58×58 |
| `icon@3x.png` | 87×87 |

- 禁止 60/120/180（App 图标规格，设置里会渲染成 60pt，大一圈）。
- PNG、透明背景、方形画布、glyph 居中留边距。三张都给。

## 位置

图标三件套放在工程的 `layout/Library/PreferenceBundles/<前缀>Prefs.bundle/`（与 Info.plist、Root.plist 同级），打包自动进 bundle，不用改 Makefile。roothide 的 `var/jb` 前缀由打包方案处理，layout 内按逻辑路径写。

本技能模板：`assets/preferences/__PREFIX__Prefs/Resources/` 已带三张正确尺寸的占位图标，生成后替换成自己的即可。

## 入口 plist 的 icon 键（最容易忘）

`layout/Library/PreferenceLoader/Preferences/<前缀>Prefs.plist` 的 entry 内必须写：

```xml
<key>icon</key>
<string>icon.png</string>
```

三条铁律：

1. `icon` 值只写 `icon.png` 纯文件名，不带 `<前缀>Prefs.bundle/` 前缀（带了设置里无图）。
2. `bundle` 值必须等于 bundle 目录名（`<前缀>Prefs`，不带 `.bundle`）。
3. 图标三件套和入口 plist 的 icon 键缺一不可，缺一个都没图。

## 源列表图标

插件图命名 `<Package>.png`（如 `com.abyss.touchtrail.png`）丢进源仓库 `icons/`，重建 Packages 时自动注入。规格 64×64 或 128×128 即可。

## 建完工程过一遍

- [ ] bundle 目录下有 icon.png / icon@2x.png / icon@3x.png
- [ ] 入口 plist 有 icon 键且值为 `icon.png`；bundle = `<前缀>Prefs`
- [ ] 尺寸是 29/58/87，不是 60/120/180
