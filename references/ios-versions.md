# 最低 iOS 版本约定

## 三处版本的关系

| 设置 | 来源 | 作用 |
|---|---|---|
| `sdk_version`（16.5） | `assets/defaults.json` | 编译 SDK，决定能看到的最高 API |
| `minimum_ios`（16.0） | `assets/defaults.json` | 业务上的最低支持版本 |
| TARGET 部署版本 | 共享 `*Build.mk` | 二进制里写死的最低运行版本 |

三者独立：部署版本不得高于 `minimum_ios`，也不得高于实际真机版本；用到 SDK 16.5 才有的 API 时，代码里必须用 `@available`/`respondsToSelector` 做运行时判断。

## 各打包环境的部署版本映射

| 环境 | 部署版本 |
|---|---|
| Rootful | `max(14.0, minimum_ios)` |
| Rootless | `max(15.0, minimum_ios)` |
| Roothide | `max(15.0, minimum_ios)` |

同一份源码在三个环境各编一次；当前 `minimum_ios = 16.0`，所以三环境实际都是 16.0。下调 `minimum_ios` 时按此映射逐环境检查，不要静默改低某一个。

## 检查点

- 生成后核对 `*Build.mk` 的 TARGET 第 4 段等于映射结果。
- 新增 API 时查其可用版本：低于部署版本可直接用，否则加可用性判断并写清回退行为。
- 不用 `latest`/`last` 做 SDK；升 SDK 走 `defaults.json` 的 `sdk_version` 并同步 CI。
