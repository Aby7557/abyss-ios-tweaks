# 最小 Hook 示例与 %orig 约定

以插件 `TouchTrail`（前缀 `AbyssTouchTrail`）为例，展示推荐的起步写法。

## 完整示例

```objc
// Hooks/AbyssTouchTrailMain.xm
#import "AbyssTouchTrailConfig.h"

%hook SBIconListView

- (void)layoutSubviews {
    %orig;  // 布局类方法：先让系统完成，再做自己的事
    if (!AbyssTouchTrailConfig.shared.enabled) return;  // 开关关闭 = 原行为
    // ...插件逻辑，只读 Config 的快照，不在 hook 里读文件
}

%end
```

```objc
// 有返回值的方法：先取 orig 再决定
- (BOOL)supportsIconReorder {
    BOOL orig = %orig;
    AbyssTouchTrailConfig *cfg = AbyssTouchTrailConfig.shared;
    if (!cfg.enabled) return orig;
    return cfg.allowReorder ?: orig;   // 缺键回退到系统行为
}
```

## %orig 约定

- 按被 hook 方法的语义决定调用时机，不统一规定先调或后调：
  - 布局/绘制类：先 `%orig` 再叠加效果；
  - 返回布尔/值类：先取 `%orig` 作回退值，再按配置决定；
  - 完全替换类：明确注释为什么不调 `%orig`。
- `enabled` 关闭时路径必须等价于原行为（通常是直接 `%orig` 返回）。
- UI 更新派发到主线程；高频 hook（每帧/触摸流）只读 Config 快照，不读文件、不创建对象池外的对象。

## 过滤器

`AbyssTouchTrail.plist` 只列实际注入的 bundle；SpringBoard 插件写 `com.apple.springboard`。加进程前先确认目标类存在于该进程。

## 构造函数

```objc
%ctor {
    [AbyssTouchTrailConfig.shared warmUp];   // 首次读配置、注册 Darwin 通知
    %init;
}
```
不在 `%ctor` 里写默认配置、弹 UI 或同步读大文件。
