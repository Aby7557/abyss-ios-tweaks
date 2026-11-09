# 换包名/重命名迁移清单

已发布插件换包名（如 `com.abyss.old` → `com.abyss.new`）或改类前缀时，按此清单成组迁移，逐项打勾：

## 需要同步改的运行时字符串

- [ ] `control` 的 `Package`
- [ ] 偏好域（NSUserDefaults suite）与配置文件路径
- [ ] Darwin 通知名（`*.preferences-changed`）
- [ ] PreferenceLoader 入口 plist 文件名及其 `bundle`/`detail`
- [ ] 面板 bundle 名、Info.plist、控制器类名
- [ ] tweak 过滤器 plist 文件名（`*.plist`）
- [ ] Makefile / `*Build.mk` 里的目标名、文件清单、import
- [ ] 代码内所有字符串：`NSClassFromString`、`objc_getClass`、通知名、defaults 键
- [ ] `tweak.json` 记录

## 用户配置迁移

- [ ] 确认权威旧配置副本（单一文件，不扫描猜测）
- [ ] 新版本首启动做一次性、带版本号的迁移：读旧路径 → 原子写新路径 → 迁移标记
- [ ] 迁移失败不覆盖、报可观察错误；旧文件保留不删
- [ ] 换域后旧域不再读取

## 验收

- [ ] 覆盖安装（旧→新）后用户配置仍在
- [ ] 面板读写、目标进程收到通知
- [ ] 三环境各装一次验证
- [ ] 不为省事复用旧包名冒充新插件，也不双包并存改同一配置

改已发布包的 `Package` 是破坏性变更，默认不做；确需时单独设计并真机验证后再发。
