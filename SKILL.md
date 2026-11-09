---
name: abyss-ios-tweaks
description: 生成和维护 Abyss 约定的 Theos deb 插件工程：插件专属命名、可选 PreferenceLoader 面板与自定义滑条、Rootful/Rootless/Roothide 打包，以及 GitHub Actions 构建工作流。用于 Theos tweak 工程，不用于 Apelin 项目的构建，也不用于 IPA 签名和侧载。
---

# Abyss iOS Tweaks（Theos 工程）

生成可手动维护的 deb 插件工程。沿用 Abyss 个人约定，但所有自定义文件、类、协议、函数和新增 selector 都必须带插件命名空间，不复用裸 `AbyssConfig`、`AbyssSliderCell`、`abyss_reload` 等通用名称。

**注意：本技能面向 Theos 工程（外部仓库/CI 构建）。Apelin 项目内不使用本技能的 Theos/Logos 流程，Apelin 的 tweak 是纯 Objective-C + MSHookMessageEx。**

## 工作流程

1. 明确插件英文名、小写标识、显示名、目标进程、最低 iOS 版本和所需打包环境。已有工程先检查现有结构，不重新生成覆盖。
2. 阅读 [命名与个人约定](conventions.md)。英文名用于代码，显示名可用中文；不要把作者前缀当作插件命名空间。
3. 新工程通过 `scripts/create.py` 生成（依赖 Python 3.8+ 标准库），默认附带 GitHub Actions 构建/发布工作流。按需启用设置面板和滑条，不默认添加无关模式、参数、服务或全局注入。
4. 在插件专属 `Hooks/`、`Headers/`、`Sources/` 文件中实现需求。按模块拆分，新增私有声明应来自实际目标，不保留虚构类。
5. 有设置面板时阅读 [配置与面板](preferences.md)；有滑条时再读 [滑条组件](custom-slider-cell.md)。面板是配置文件的唯一写入方，Tweak 不持久化默认值。
6. 涉及资源路径或多环境时阅读 [越狱适配](jailbreak-adaptation.md)。配置与只读资源分开解析，不扫描目录决定配置写入位置。
7. 按 [构建与环境](building.md) 检查、编译并核对 deb。失败时查 [排错与验证记录](troubleshooting.md)。
8. 需要时参考：[最低 iOS 版本约定](ios-versions.md)、[最小 Hook 示例](hook-example.md)、[私有声明头文件来源](headers-source.md)、[换包名迁移清单](rename-migration.md)、[设置面板图标规范](panel-icon.md)（带面板的工程必读第 5 步）。

## 生成示例

在 skill 目录执行，输出目录必须不存在：

```sh
python3 scripts/create.py /work/touchtrail --name TouchTrail --id touchtrail --display-name "触摸轨迹" --preferences --slider
python3 scripts/doctor.py /work/touchtrail
```

只需要核心插件时省略 `--preferences --slider`。生成器仅生成文件，不安装依赖、不编译、不安装插件、不注销设备。`--bundle` 可指定注入 Bundle ID，默认仅 SpringBoard。

- 默认生成 `.github/workflows/build.yml`（macOS/Xcode、SDK 16.5、Rootless 矩阵）；仅本地构建时加 `--no-github-actions`。详见 [构建与环境](building.md)。
- 主工程、面板与 CI 的 SDK 统一为 `16.5`，由 `assets/defaults.json` 的 `sdk_version` 同步；TARGET 只在共享 `*Build.mk` 定义。
- Release 标题自动使用插件显示名；所有插件的 CI 包版本统一为 `0.0.1-${{ github.run_number }}`。
- 工作流在推送到 `main/master` 或手动触发时构建并发布 Release，上传前先检查名称、版本和触发条件。

## 实现边界

- 自定义标识使用生成的类前缀和函数前缀；系统 override、协议要求的 selector 和工具链固定文件名保持原样。
- 默认不启用带面板的插件；无面板工程默认启用，由实际需求决定是否增加开关。
- `enabled` 关闭时保持原行为；`%orig` 的调用时机和返回值按目标方法语义决定，不能统一规定先调用或永不调用。
- 过滤器只覆盖需要的进程。检查目标类、方法及 iOS 版本，避免不必要的全局 hook。
- UI 更新在主线程；高频 hook 只读配置快照，不扫目录、读文件或创建 defaults。对象复用、缓存和清理按实际负载落实。
- 自定义非交互浮层不接管触摸。复杂逻辑写简短原因注释，不为每次修改追加历史流水注释。
- 保留编译警告。生成器与 doctor 通过不代表 Theos 编译或真机测试通过；分别报告验证范围。

## 维护入口

| 要维护的内容 | 唯一入口 |
|---|---|
| 作者、包域、SDK 版本、包版本 | `assets/defaults.json` |
| 默认生成的代码、Makefile、plist | `assets/base/`、`assets/preferences/`、`assets/slider/` |
| GitHub Actions 构建与发布 | `assets/github-actions/.github/workflows/build.yml` |
| 参数替换、文件生成 | `scripts/create.py` |
| 工程与工具链只读检查 | `scripts/doctor.py` |
| 原理、适配边界、人工验收 | `references/` 对应专题 |

修改模板后可生成示例工程，用 `scripts/doctor.py` 检查结构与引用。默认值只影响之后生成的项目，不会自动修改已发布插件。原安装目录中的另一个 skill 副本也不会自动同步。

## 参考文档（随技能安装）

技能目录包含以下参考文档，用 apelin_skill_read 按需读取（传 file 参数）：
conventions.md、preferences.md、custom-slider-cell.md、jailbreak-adaptation.md、building.md、troubleshooting.md、boilerplate.md、ios-versions.md、hook-example.md、headers-source.md、rename-migration.md、panel-icon.md，以及 scripts/create.py、scripts/doctor.py 和 assets/ 下的完整实体模板。
