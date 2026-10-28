# dsh-mc-companion-preset

把 [dsh-mc-companion](https://github.com/banbixieyang/dsh-mc-companion) 注册为 DeepSeek Harness 的一个 **可选择的 Agent 预设**：
`MC 陪伴模式`（preset id `mc-companion`，Loader 行 `preset-mc-companion`）。

配套仓库：

- 主插件：https://github.com/banbixieyang/dsh-mc-companion
- 实时面板：https://github.com/banbixieyang/dsh-mc-companion-ui

它本身不包含 Bot 逻辑，只是把已安装的 `dsh-mc-companion` 包挂载到预设的 Cordis scope 中：
选择该预设后，`mc_companion_mode` 与 12 个游戏工具只对这个预设的 Agent 可见；其他预设（standard、游戏制作等）看不到这些工具。

## 安装与切换

0. 克隆三个仓库（主插件、预设、UI）到本地：
   ```bash
   git clone https://github.com/banbixieyang/dsh-mc-companion.git
   git clone https://github.com/banbixieyang/dsh-mc-companion-preset.git
   git clone https://github.com/banbixieyang/dsh-mc-companion-ui.git
   ```
1. 保证主插件包仍安装（profile 的 `node_modules/dsh-mc-companion` 存在）；
2. 关闭原来的 profile 级 bundle，避免出现两个实例：
   ```text
   plugin_manager set_bundle target=dsh-mc-companion enabled=false
   ```
3. 安装本预设 bundle：
   ```text
   plugin_manager install_bundle target=<你的路径>\dsh-mc-companion-preset
   ```
4. 安装独立的 UI 面板 bundle（浏览器半边必须由有自己的 Host 行的包提供）：
   ```text
   plugin_manager install_bundle target=<你的路径>\dsh-mc-companion-ui
   ```
5. 在 DSH 的 Agent 预设选择器中选择 **MC 陪伴模式**；已有会话保留启动时的插件版本，
   建议**新建会话**验证（或使用界面提供的预设切换入口）。

## 使用

选择预设只是让工具可用；**Bot 仍然默认离线**（主插件的模式门控不变）。

```text
用户：启用 MC 陪伴模式
→ mc_companion_mode { action: "enable" }
→ Bot 上线，mc_status / mc_follow_player / mc_build_schematic 可用
```

关闭：

```text
mc_companion_mode { action: "disable" }
```

## 实时面板与工作状态（Client UI）

面板由独立 bundle **`dsh-mc-companion-ui`** 提供（有自己的 Host 行，client bundle 才会被加载）；本预设只负责 Bot 工具与模式门控。

- **左侧栏「MC 陪伴」面板**：状态 HUD（生命/饥饿/坐标/维度）、启用/连接/断开、
  跟随输入、工作状态开关、建造进度、实时游戏聊天；
- **输入框下方的状态条**：当前会话里直接看到 Bot 在线状态、血量、坐标、正在执行的行为和最后一条聊天；
- **工作状态**：`探索 / 伐木 / 采矿 / 打怪`，每个都有开关 + 可输入的数量、半径、时长、目标列表；
  开关只在 DSH 界面里由用户操作，游戏内聊天无法打开它们；
- **实时 AI 聊天**：默认关闭；打开后使用当前默认模型，对游戏内普通聊天做短回复。
  像命令的消息会被跳过，**任何游戏内玩家（包括 owner）都不能通过聊天框命令 Bot**；
  手动聊天输入也只发送文本。

数据接口由主插件在宿主 webServer 上提供：

```text
GET  /api/mc-companion/state
POST /api/mc-companion/action
```

面板是 1 秒轮询；聊天/行为/建造状态会在下一次轮询刷新。

> **安装/更新预设或 UI 面板后需要重启 DSH**：client bundle 与预设内插件不在普通插件列表里，
> `install_bundle` 会返回 `restart-required`。重启后刷新页面，左侧栏才会出现「MC 陪伴」，
> 输入框下方才会出现状态条。

## 注意

- 该预设的插件列表已经包含 persona、常用工具（fs/shell/web/jobs/todo）与压缩组；
- 主插件的配置写在 [cordis.patch.yml](cordis.patch.yml) 的 `mc-companion-mode` 行里，
  修改后需要对预设重新执行 `remove_bundle` → `install_bundle`；
- 想改的是网络参数（host/port/username/owner）时，也可以直接在这个 YAML 里改；不需要动主插件源码；
- 不要重新启用 profile 级的 `dsh-mc-companion` bundle，否则会出现第二个全局插件实例（工具重复/模式状态分叉）；
- 26.3 兼容性与限制见主插件 README 第 8 节。

## 故障排查

- **预设不出现 / 模式选择器里没有 MC 陪伴模式**：检查预设是否 `broken`。
  本预设曾因 `tool-fs-search` 缺少必填配置 `sampleOverCapGlobResults` 而整体 broken，
  现已配置为 `false`。修改后需 `remove_bundle` → `install_bundle`，并刷新页面/新建会话；
- 预设出现在设置页但下拉框没有：客户端缓存了旧 roster，刷新页面或新建会话；
- 选择预设后仍看不到 `mc_companion_mode`：确认选择的是**新会话**（已有会话保留启动时的插件版本），
  并确认 `dsh-mc-companion` 包仍在 profile 依赖中。
