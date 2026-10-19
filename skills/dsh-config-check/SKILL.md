---
name: dsh-config-check
description: >-
  改完 DSH desktop 配置（权限预设、cordis.patch.yml、插件）后，核对它是不是真的生效，而不是以为生效。
  当用户说"我重启了 / 为什么没生效 / 这个设置生效了吗 / 装上了吗"，或任何改配置、装插件后的验证场景时调用。
---

# DSH 配置生效核对

DSH 的配置有"写了"和"真生效"两件事，经常被 UI 预设覆盖。本技能给一套核对步骤。

## 何时用
- 改了 `cordis.patch.yml`（sandbox-policy / approval / 任何插件挂载）之后。
- 在桌面 Plugin Hub 装 / 卸 / 升了插件之后。
- 用户说"重启了但没变化"。

## 核对步骤
1. 读 `C:\Users\bumah\.dsh\profiles\desktop\cordis.patch.yml`，确认目标 top-level id / insert 在、YAML 能解析。
2. 读同目录 `package.json`，确认 `dsh.profile.bundles` 里含目标包；读 `node_modules/<包>/package.json` 的 version 确认装的是想要的版本。
3. 读 `hub.log`，看末尾是 `install.done`（待重启生效）还是 `install.fail`；fail 时先怀疑插件本身（包坏 / 不兼容 / 缺依赖），不是网络。
4. 跑 `D:\AIdeepseek\resources\runtime\cli\bin\dsh.cmd --profile web --dump-config-schema` 看字段是否真实存在于 schema（desktop profile 被 Electron 独占，不能 dump）。
5. **关键坑**：生效的 file policy 由桌面端"设置 → 权限"选中的预设决定，运行时写入 volatile 的 `defaultPreset`，**会覆盖** `cordis.patch.yml` 里的 `sandbox-policy` / `approval` 底层值。所以切硬约束必须去 UI 改预设，不是只改 yml。

## 验证通过标准
- cordis.patch.yml 解析通过、目标 id 在。
- bundles 含目标包、`node_modules` 版本正确、hub.log 末尾 `install.done`。
- 重启 desktop 后行为符合预期（无热重载，必须完全退出含托盘再启动）。

## 坑
- `dsh config reload` / `dsh doctor` 在 DSH CLI 中不存在。
- CLI `dsh plugin --profile desktop add ...` 要写 `profiles/desktop/package.json.lock`，在 workspace-write 沙箱下会被 EPERM 拒；装插件走桌面 Hub。
- 没有 `permissions.allow/deny/ask` 这种路径 / 命令 glob 配置位（schema 里只有 sandbox / sandbox-policy / approval / permission 四个原语）。
