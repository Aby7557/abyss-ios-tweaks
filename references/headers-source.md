# 私有声明头文件来源

`Headers/` 只放确认过的声明，按目标来源归档，每条注明出处类和核实方式。

## 本机 SDK（iPhoneOS 17.4）已包含的

编译时 `-isysroot` 可直接读到，不必抄进 Headers/：

| 框架 | 常用内容 |
|---|---|
| UIKit / Foundation | 公开 API，直接 import |
| SpringBoard | `SBIconListView`、`SBApplication` 等常用类 |
| BackBoardServices / FrontBoardServices | 触摸、前台应用 |
| Preferences.framework（Apelin 支持组件头） | `PSTableCell`、`PSListController`，面板用 |

## 需要自己声明进 Headers/ 的

- 上述类的未导出方法和 ivar：先在本机用 Apelin 的 disassemble/symbols 工具或 class-dump 结果核实签名，再写进 `*Private.h`，注明来源版本。
- 面板 bundle 用到的：放 `*Prefs/` 自带的头或面板工程 `-I../Headers`。

## 约定

- 只声明用到的成员，不整类照搬。
- 每个声明后注释 `// iOS xx 验证` 或 `// dump 自 iOS xx`。
- 系统版本升级后，hook 失效先核对声明是否过期，而不是先改逻辑。
