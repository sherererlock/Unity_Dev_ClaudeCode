---
name: unity-editor-toolkit
description: |
  Unity 编辑器控制与自动化，基于 WebSocket 的实时通信。
  Unity Editor control and automation, WebSocket-based real-time communication.

  Features/功能: 游戏对象控制 (GameObject control), 变换操作 (Transform manipulation), 组件管理 (Component management), 场景管理 (Scene management), SQLite 数据库集成 (SQLite database integration), 基于 GUID 的持久识别 (GUID-based persistence), 多场景同步 (Multi-scene synchronization), 命令模式撤销/重做 (Command Pattern with Undo/Redo), 菜单执行 (Menu execution), ScriptableObject 管理 (ScriptableObject management), 数组/列表操作 (Array/List manipulation), 所有字段类型支持 (All field types support), 材质/渲染 (Material/Rendering), 预制件系统 (Prefab system), 资产数据库 (Asset Database), 动画 (Animation), 物理 (Physics), 控制台日志 (Console logging), EditorPrefs 管理 (EditorPrefs management), 编辑器自动化 (Editor automation), 构建管道 (Build pipeline), 光照 (Lighting), 相机 (Camera), 音频 (Audio), 导航 (Navigation), 粒子 (Particles), 时间轴 (Timeline), UI Toolkit, 分析器 (Profiler), 测试运行器 (Test Runner).

  Protocol/协议: WebSocket 上的 JSON-RPC 2.0 (端口 9500-9600). 500+ 指令, 25 个类别. 实时双向通信.

  Security/安全: 深度防御 (路径遍历保护, 命令注入防御, JSON 注入防御, SQL 注入防御, 事务安全). 仅限本地连接. 跨平台 (Windows, macOS, Linux).
---

## 目的 (Purpose)

Unity Editor Toolkit 实现了从 Claude Code 对 Unity 编辑器的全面自动化和控制。它提供：

- **广泛的指令覆盖**：涵盖 25 个 Unity 编辑器类别的 500+ 指令
- **实时通信**：即时双向 WebSocket 连接 (JSON-RPC 2.0)
- **SQLite 数据库集成**：具有基于 GUID 的持久性的实时 GameObject 同步
  - **基于 GUID 的识别**：跨 Unity 会话的持久 GameObject 跟踪
  - **多场景支持**：同时同步所有加载的场景（1秒间隔）
  - **命令模式**：数据库操作的撤销/重做支持
  - **自动迁移**：自动架构迁移系统
  - **批量操作**：高效的批量插入、更新和删除（500 个对象/批次）
- **菜单执行**：以编程方式运行 Unity 编辑器菜单项（Window, Assets, Edit, GameObject 菜单）
- **ScriptableObject 管理**：完整的 CRUD 操作，支持数组/列表和所有字段类型
  - **数组/列表操作**：添加、移除、获取、清除元素，支持嵌套访问 (`items[0].name`)
  - **所有字段类型**：整数、浮点数、字符串、布尔值、Vector*、Color、Quaternion、Bounds、AnimationCurve、ObjectReference 等
  - **嵌套属性遍历**：使用点符号和数组索引访问深层嵌套字段
- **深度编辑器集成**：GameObject/层级、变换、组件、场景、材质、预制件、动画、物理、光照、构建管道等
- **安全优先**：针对注入攻击（SQL、命令、JSON、路径遍历）和未经授权访问的多层防御
- **生产就绪**：跨平台支持，具有强大的错误处理和日志记录

**务必先使用 `--help` 运行脚本**以查看用法。在尝试运行脚本并发现绝对需要自定义解决方案之前，**不要**阅读源代码。这些脚本可能非常大，从而污染您的上下文窗口。它们的存在是为了作为黑盒脚本直接调用，而不是摄入到您的上下文窗口中。

---

## 📚 文档优先原则 (必须)

**⚠️ CRITICAL**: 使用 Unity Editor Toolkit skill 时，**必须遵循以下顺序：**

### 1️⃣ 确认 Reference 文档 (必须)
在使用指令之前，**务必**阅读 `skills/references/` 文件夹中的相应文档：
- **[COMMANDS.md](./references/COMMANDS.md)** - 所有指令的类别及概览
- **Category-specific docs** - 使用指令的类别文档：
  - [Component Commands](./references/COMMANDS_COMPONENT.md) - comp list/add/remove/enable/disable/get/set/inspect/move-up/move-down/copy
  - [GameObject Commands](./references/COMMANDS_GAMEOBJECT_HIERARCHY.md) - go find/create/destroy/set-active/set-parent/get-parent/get-children
  - [Transform Commands](./references/COMMANDS_TRANSFORM.md) - tf get/set-position/set-rotation/set-scale
  - [Scene Commands](./references/COMMANDS_SCENE.md) - scene current/list/load/new/save/unload/set-active
  - [Console Commands](./references/COMMANDS_CONSOLE.md) - console logs/clear
  - [EditorPrefs Commands](./references/COMMANDS_PREFS.md) - prefs get/set/delete/list/clear/import
  - [Other Categories](./references/COMMANDS.md) - 其他指令类别

### 2️⃣ 运行 `--help`
```bash
# 确认所有指令
cd <unity-project-root> && node .unity-websocket/uw --help

# 确认特定指令的选项
cd <unity-project-root> && node .unity-websocket/uw <command> --help
```

### 3️⃣ 运行示例
参考 reference 文档的 **Examples 部分**来运行指令。

### 4️⃣ 阅读源代码 (最后手段)
- 仅在 reference 文档和 --help 无法解决问题时才阅读源代码
- 源代码会占用大量上下文窗口，因此请尽可能避免

**如果忽略此顺序：**
- ❌ 可能误解指令用法
- ❌ 可能错过选项导致非预期结果
- ❌ 可能浪费上下文窗口

---

## 何时使用 (When to Use)

当您需要以下操作时使用 Unity Editor Toolkit：

1. **自动化 Unity 编辑器任务**
   - 创建和操作 GameObject、组件和层级
   - 配置场景、材质和渲染设置
   - 控制动画、物理和粒子系统
   - 管理资产、预制件和构建管道

2. **实时 Unity 测试**
   - 在开发期间监控控制台日志和错误
   - 查询 GameObject 状态和组件属性
   - 测试场景配置和游戏玩法逻辑
   - 调试渲染、物理或动画问题

3. **批量操作**
   - 创建具有特定配置的多个 GameObject
   - 跨多个对象应用材质/着色器更改
   - 根据规范设置场景层级
   - 自动化重复的编辑器任务

4. **菜单和编辑器自动化**
   - 以编程方式执行 Unity 编辑器菜单项 (`menu run "Window/General/Console"`)
   - 通过命令行打开编辑器窗口和工具
   - 自动化资产刷新、重新导入和构建操作
   - 使用通配符过滤查询可用的菜单项

5. **ScriptableObject 管理**
   - 以编程方式创建和配置 ScriptableObject 资产
   - 读取和修改所有字段类型（Vector、Color、Quaternion、AnimationCurve 等）
   - 使用完整的 CRUD 操作操作数组/列表
   - 使用数组索引符号访问嵌套属性 (`items[0].stats.health`)
   - 查询 ScriptableObject 类型并检查资产元数据

6. **数据库驱动的工作流**
   - 使用基于 GUID 的标识跨 Unity 会话进行持久 GameObject 跟踪
   - 将所有加载的场景实时同步到 SQLite 数据库
   - GameObject 层级和属性的分析与查询
   - 通过命令模式支持数据库操作的撤销/重做
   - 用于大型场景管理的高效批量操作（500 个对象/批次）

7. **CI/CD 集成**
   - 具有特定于平台设置的自动构建
   - 用于单元/集成测试的 Test Runner 集成
   - 资产验证和完整性检查
   - 构建管道自动化

## 先决条件 (Prerequisites)

### Unity 项目设置 (Unity Project Setup)

1. **安装 Unity Editor Toolkit Server 包**
   - 通过 Unity Package Manager（Git URL 或本地路径）
   - 需要 Unity 2020.3 或更高版本
   - 包位置：`skills/assets/unity-package`

2. **配置 WebSocket 服务器**
   - 打开 Unity 菜单：`Tools > Unity Editor Toolkit > Server Window`
   - 从 `~/.claude/plugins/...` 自动检测插件脚本路径
   - 点击 "Install CLI" 构建 WebSocket 服务器（一次性设置）
   - Unity 编辑器打开时服务器自动启动

3. **数据库设置** (可选)
   - 在 Server 窗口中，切换到 "Database" 选项卡
   - 点击 "Connect" 初始化 SQLite 数据库
   - 数据库文件位置：`{ProjectRoot}/.unity-websocket/unity-editor.db`
   - 点击 "Start Sync" 启用实时 GameObject 同步（1秒间隔）
   - **GUID 组件**：GameObject 自动标记有持久 GUID
   - **多场景**：自动同步所有加载的场景
   - **分析**：查看同步统计、数据库健康状况和撤销/重做历史记录

4. **服务器状态**
   - 端口：从 9500-9600 范围自动分配
   - 状态文件：`{ProjectRoot}/.unity-websocket/server-status.json`
   - CLI 从此文件自动检测正确的端口

5. **依赖项**
   - websocket-sharp（通过包安装脚本安装）
   - Newtonsoft.Json（Unity 内置版本）
   - Cysharp.UniTask（用于异步/等待数据库操作）
   - SQLite-net（嵌入式 SQLite 数据库）

### Claude Code 插件 (Claude Code Plugin)

Unity Editor Toolkit 插件提供用于 Unity 编辑器控制的 CLI 指令。

## 核心工作流 (Core Workflow)

### 1. 连接 (Connection)

Unity Editor Toolkit CLI 自动：

- 通过 `.unity-websocket/server-status.json` 检测 Unity 项目
- 从状态文件读取端口信息（9500-9600 范围）
- 如果 Unity 编辑器正在运行，则连接到 WebSocket 服务器

### 2. 执行指令 (Execute Commands)

⚠️ **在执行任何指令之前，请检查您的指令类别的参考文档**（见上方的 "📚 文档优先原则" 章节）。

Unity Editor Toolkit 在 15 个类别中提供 86+ 个指令。所有指令均从 Unity 项目根目录运行：

```bash
cd <unity-project-root> && node .unity-websocket/uw <command> [options]
```

**可用类别** (已实现):

| # | 类别 (Category) | 指令数 | 参考文档 (Reference) |
|---|----------|----------|-----------|
| 1 | Connection & Status | 1 | [COMMANDS_CONNECTION_STATUS.md](./references/COMMANDS_CONNECTION_STATUS.md) |
| 2 | GameObject & Hierarchy | 8 | [COMMANDS_GAMEOBJECT_HIERARCHY.md](./references/COMMANDS_GAMEOBJECT_HIERARCHY.md) |
| 3 | Transform | 4 | [COMMANDS_TRANSFORM.md](./references/COMMANDS_TRANSFORM.md) |
| 4 | **Component** ✨ | 10 | [COMMANDS_COMPONENT.md](./references/COMMANDS_COMPONENT.md) |
| 5 | Scene Management | 7 | [COMMANDS_SCENE.md](./references/COMMANDS_SCENE.md) |
| 6 | Asset Database & Editor | 3 | [COMMANDS_EDITOR.md](./references/COMMANDS_EDITOR.md) |
| 7 | Console & Logging | 2 | [COMMANDS_CONSOLE.md](./references/COMMANDS_CONSOLE.md) |
| 8 | EditorPrefs Management | 6 | [COMMANDS_PREFS.md](./references/COMMANDS_PREFS.md) |
| 9 | Wait Commands | 4 | [COMMANDS_WAIT.md](./references/COMMANDS_WAIT.md) |
| 10 | Chain Commands | 2 | [COMMANDS_CHAIN.md](./references/COMMANDS_CHAIN.md) |
| 11 | Menu Execution | 2 | [COMMANDS_MENU.md](./references/COMMANDS_MENU.md) |
| 12 | Asset Management | 9 | [COMMANDS_ASSET.md](./references/COMMANDS_ASSET.md) |
| 13 | Prefab | 12 | [COMMANDS_PREFAB.md](./references/COMMANDS_PREFAB.md) |
| 14 | Material | 9 | [COMMANDS_MATERIAL.md](./references/COMMANDS_MATERIAL.md) |
| 15 | Shader | 7 | [COMMANDS_SHADER.md](./references/COMMANDS_SHADER.md) |

**用法:**

```bash
cd <unity-project-root> && node .unity-websocket/uw <command> [options]
```

**必需：检查文档**

```bash
# 1. 首先阅读指令类别的 reference 文档
# 例: 使用 Component 指令 → 阅读 skills/references/COMMANDS_COMPONENT.md

# 2. 使用 --help 确认指令选项
cd <unity-project-root> && node .unity-websocket/uw --help
cd <unity-project-root> && node .unity-websocket/uw <command> --help

# 3. 参考 reference 文档的示例进行执行
```

**📖 按类别分类的完整文档**

**必读**：在使用任何指令之前，请阅读 **特定类别的参考文档**：
- 🔴 **必须先读** - [COMMANDS.md](./references/COMMANDS.md) - 概览和指令路线图
- 🔴 **必读** - 特定类别的文档（链接见上表）
  - [Component Commands](./references/COMMANDS_COMPONENT.md) - **新功能**: comp list/add/remove/enable/disable/get/set/inspect/move-up/move-down/copy
  - [GameObject Commands](./references/COMMANDS_GAMEOBJECT_HIERARCHY.md) - go find/create/destroy/set-active/set-parent/get-parent/get-children
  - [Transform Commands](./references/COMMANDS_TRANSFORM.md) - tf get/set-position/set-rotation/set-scale
  - [Scene Commands](./references/COMMANDS_SCENE.md) - scene current/list/load/new/save/unload/set-active
  - [Console Commands](./references/COMMANDS_CONSOLE.md) - console logs/clear
  - [EditorPrefs Commands](./references/COMMANDS_PREFS.md) - prefs get/set/delete/list/clear/import
  - [Other Categories](./references/COMMANDS.md) - 包含所有类别的完整列表

### 3. 检查连接状态 (Check Connection Status)

```bash
# 验证 WebSocket 连接
cd <unity-project-root> && node .unity-websocket/uw status

# 使用自定义端口
cd <unity-project-root> && node .unity-websocket/uw --port 9301 status
```

### 4. 复杂工作流 (Complex Workflows)

**创建并配置 GameObject：**
```bash
cd <unity-project-root> && node .unity-websocket/uw go create "Enemy" && \
cd <unity-project-root> && node .unity-websocket/uw tf set-position "Enemy" "10,0,5" && \
cd <unity-project-root> && node .unity-websocket/uw tf set-rotation "Enemy" "0,45,0"
```

**加载场景并激活 GameObject：**
```bash
cd <unity-project-root> && node .unity-websocket/uw scene load "Level1" && \
cd <unity-project-root> && node .unity-websocket/uw go set-active "Boss" true
```

**批量创建 GameObject：**
```bash
for i in {1..10}; do
  cd <unity-project-root> && node .unity-websocket/uw go create "Cube_$i" && \
  cd <unity-project-root> && node .unity-websocket/uw tf set-position "Cube_$i" "$i,0,0"
done
```

**等待编译然后执行：**
```bash
# 进行代码更改后，等待编译完成
cd <unity-project-root> && node .unity-websocket/uw wait compile && \
cd <unity-project-root> && node .unity-websocket/uw editor refresh
```

**按顺序链接多个指令：**
```bash
# 从 JSON 文件执行指令
cd <unity-project-root> && node .unity-websocket/uw chain execute commands.json

# 内联执行指令
cd <unity-project-root> && node .unity-websocket/uw chain exec \
  "GameObject.Create:name=Player" \
  "GameObject.SetActive:instanceId=123,active=true"

# 即使部分指令失败仍继续执行
cd <unity-project-root> && node .unity-websocket/uw chain exec \
  "Editor.Refresh" \
  "GameObject.Find:path=InvalidPath" \
  "Console.Clear" \
  --continue-on-error
```

**CI/CD 管道工作流：**
```bash
#!/bin/bash
cd /path/to/unity/project

# 清理
node .unity-websocket/uw.js chain exec "Console.Clear" "Editor.Refresh"

# 等待编译
node .unity-websocket/uw.js wait compile

# 运行测试（示例）
node .unity-websocket/uw.js chain exec \
  "Scene.Load:name=TestScene" \
  "GameObject.Find:path=TestRunner" \
  "Console.Clear"
```

## 最佳实践 (Best Practices)

1. **始终验证连接**
   - 在执行指令前运行 `cd <unity-project-root> && node .unity-websocket/uw status`
   - 确保 Unity 编辑器正在运行且服务器组件处于活动状态

2. **使用层级路径**
   - 首选嵌套 GameObject 的完整路径：`"Environment/Terrain/Trees"`
   - 避免当多个 GameObject 共享相同名称时的歧义

3. **监控控制台日志**
   - 使用 `cd <unity-project-root> && node .unity-websocket/uw console logs --errors-only` 捕获自动化期间的错误
   - 在运行自动化脚本之前清除控制台以获取干净的日志

4. **谨慎进行批量操作**
   - 如果创建许多 GameObject，请在指令之间添加延迟
   - 考虑 Unity 编辑器的性能限制

5. **连接管理**
   - Unity Editor Toolkit 使用仅限本地主机的连接 (127.0.0.1)
   - 端口范围限制在 9500-9600 以避免与其他工具冲突

6. **错误处理**
   - 指令针对无效操作返回 JSON-RPC 错误响应
   - 在自动化脚本中检查退出代码和错误消息

7. **端口管理**
   - 默认端口 9500 适用于大多数项目
   - 如果运行多个 Unity 编辑器实例，请使用 `--port` 标志
   - 插件避免与 Browser Pilot (9222-9322) 和 Blender Toolkit (9400-9500) 冲突

8. **等待指令用法**
   - 更改代码后使用 `wait compile` 以确保编译完成
   - 在自动化测试中使用 `wait playmode enter/exit` 进行播放模式同步
   - 需要时使用 `wait sleep` 在指令之间添加延迟
   - 注意：等待指令具有延迟响应（默认 5 分钟超时）
   - 域重新加载会自动取消所有挂起的等待请求

9. **链式指令最佳实践**
   - 使用 chain 进行具有自动错误处理的顺序指令执行
   - 默认行为：在第一个错误处停止（使用 `--continue-on-error` 覆盖）
   - 链中不支持等待指令（使用单独的等待指令）
   - 对复杂的多步工作流使用 JSON 文件
   - 对快速指令序列使用内联 exec

10. **开发路线图意识**
   - **第二阶段（当前）**：已在 15 个类别中实现 86 个指令
   - **第三阶段+**：动画、物理、光照、相机、音频、导航 - 计划 400+ 指令
   - 在 [COMMANDS.md](./references/COMMANDS.md) 中查看完整路线图

## 参考资料 (References)

`references/` 文件夹中提供了详细文档：

- **[QUICKSTART.md](../QUICKSTART.md)** - 快速设置和首个指令 (英语)
- **[QUICKSTART.ko.md](../QUICKSTART.ko.md)** - 快速设置指南 (韩语)
- **[COMMANDS.md](./references/COMMANDS.md)** - 完整的 500+ 指令路线图 (英语)
- **已实现的指令类别：**
  - [Connection & Status](./references/COMMANDS_CONNECTION_STATUS.md)
  - [GameObject & Hierarchy](./references/COMMANDS_GAMEOBJECT_HIERARCHY.md)
  - [Transform](./references/COMMANDS_TRANSFORM.md)
  - [Component](./references/COMMANDS_COMPONENT.md)
  - [Scene Management](./references/COMMANDS_SCENE.md)
  - [Asset Database & Editor](./references/COMMANDS_EDITOR.md)
  - [Console & Logging](./references/COMMANDS_CONSOLE.md)
  - [EditorPrefs Management](./references/COMMANDS_PREFS.md)
  - [Wait Commands](./references/COMMANDS_WAIT.md)
  - [Chain Commands](./references/COMMANDS_CHAIN.md)
  - [Menu Execution](./references/COMMANDS_MENU.md)
  - [Asset Management](./references/COMMANDS_ASSET.md)
  - [Prefab](./references/COMMANDS_PREFAB.md)
  - [Material](./references/COMMANDS_MATERIAL.md)
  - [Shader](./references/COMMANDS_SHADER.md)
- **[API_COMPATIBILITY.md](../API_COMPATIBILITY.md)** - Unity 版本兼容性 (2020.3 - Unity 6)
- **[TEST_GUIDE.md](../TEST_GUIDE.md)** - Unity C# 服务器测试指南 (英语)
- **[TEST_GUIDE.ko.md](../TEST_GUIDE.ko.md)** - Unity C# 服务器测试指南 (韩语)

Unity C# 服务器包位于 `assets/unity-package/` - 发布后通过 Unity Package Manager 安装。

---

**状态 (Status)**: 🧪 实验性 - 第二阶段 (已实现 86 个指令)
**Unity 版本支持 (Unity Version Support)**: 2020.3 - Unity 6
**协议 (Protocol)**: WebSocket 上的 JSON-RPC 2.0
**端口范围 (Port Range)**: 9500-9600 (自动分配)
