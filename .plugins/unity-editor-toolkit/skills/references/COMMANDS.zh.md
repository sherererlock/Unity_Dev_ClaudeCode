# Unity Editor Toolkit - 命令参考

用于控制 Unity Editor 的 500+ 命令路线图。

**当前状态**: 第 2 阶段 - 已实现 58 个命令

## 快速参考

```bash
# 基本用法
cd <unity-project-root> && node .unity-websocket/uw <command> [options]

# 显示所有可用命令
cd <unity-project-root> && node .unity-websocket/uw --help

# 显示特定命令的帮助
cd <unity-project-root> && node .unity-websocket/uw <command> --help
```

## 📖 按类别分类的文档

### ✅ 已实现 (第 1 阶段+)

| 类别 | 命令数 | 文档 |
|----------|----------|---------------|
| **连接与状态 (Connection & Status)** | 1 个命令 | [COMMANDS_CONNECTION_STATUS.md](./COMMANDS_CONNECTION_STATUS.md) |
| **游戏对象与层级 (GameObject & Hierarchy)** | 8 个命令 | [COMMANDS_GAMEOBJECT_HIERARCHY.md](./COMMANDS_GAMEOBJECT_HIERARCHY.md) |
| **变换 (Transform)** | 4 个命令 | [COMMANDS_TRANSFORM.md](./COMMANDS_TRANSFORM.md) |
| **组件 (Component)** | 10 个命令 | [COMMANDS_COMPONENT.md](./COMMANDS_COMPONENT.md) |
| **场景管理 (Scene Management)** | 7 个命令 | [COMMANDS_SCENE.md](./COMMANDS_SCENE.md) |
| **资产数据库与编辑器 (Asset Database & Editor)** | 3 个命令 | [COMMANDS_EDITOR.md](./COMMANDS_EDITOR.md) |
| **控制台与日志 (Console & Logging)** | 2 个命令 | [COMMANDS_CONSOLE.md](./COMMANDS_CONSOLE.md) |
| **EditorPrefs 管理** | 6 个命令 | [COMMANDS_PREFS.md](./COMMANDS_PREFS.md) |
| **等待命令 (Wait Commands)** | 4 个命令 | [COMMANDS_WAIT.md](./COMMANDS_WAIT.md) |
| **链式命令 (Chain Commands)** | 2 个命令 | [COMMANDS_CHAIN.md](./COMMANDS_CHAIN.md) |
| **菜单执行 (Menu Execution)** | 2 个命令 | [COMMANDS_MENU.md](./COMMANDS_MENU.md) |
| **资产管理 (ScriptableObject)** | 9 个命令 | [COMMANDS_ASSET.md](./COMMANDS_ASSET.md) |
| **预制件 (Prefab)** | 12 个命令 | [COMMANDS_PREFAB.md](./COMMANDS_PREFAB.md) |
| **材质 (Material)** | 9 个命令 | [COMMANDS_MATERIAL.md](./COMMANDS_MATERIAL.md) |
| **着色器 (Shader)** | 7 个命令 | [COMMANDS_SHADER.md](./COMMANDS_SHADER.md) |

### 🔄 即将推出 (第 2 阶段+)

| 类别 | 状态 |
|----------|--------|
| **材质与渲染 (Material & Rendering)** | 🔄 计划 25+ 个命令 |
| **动画 (Animation)** | 🔄 计划 20+ 个命令 |
| **物理 (Physics)** | 🔄 计划 20+ 个命令 |
| **光照 (Lighting)** | 🔄 计划 15+ 个命令 |
| **相机 (Camera)** | 🔄 计划 15+ 个命令 |
| **音频 (Audio)** | 🔄 计划 15+ 个命令 |
| **导航与 AI (Navigation & AI)** | 🔄 计划 15+ 个命令 |
| **粒子系统 (Particle System)** | 🔄 计划 15+ 个命令 |
| **时间轴 (Timeline)** | 🔄 计划 10+ 个命令 |
| **构建与播放器 (Build & Player)** | 🔄 计划 15+ 个命令 |
| **项目设置 (Project Settings)** | 🔄 计划 20+ 个命令 |
| **包管理器 (Package Manager)** | 🔄 计划 10+ 个命令 |
| **版本控制 (Version Control)** | 🔄 计划 10+ 个命令 |
| **分析器与性能 (Profiler & Performance)** | 🔄 计划 15+ 个命令 |
| **测试运行器 (Test Runner)** | 🔄 计划 10+ 个命令 |
| **输入系统 (Input System)** | 🔄 计划 10+ 个命令 |
| **UI 工具包 (UI Toolkit)** | 🔄 计划 10+ 个命令 |
| **编辑器窗口与 UI (Editor Window & UI)** | 🔄 计划 10+ 个命令 |
| **实用工具命令 (Utility Commands)** | 🔄 计划 20+ 个命令 |

---

## 快速命令示例

### 连接与状态 (Connection & Status)
```bash
cd <unity-project-root> && node .unity-websocket/uw status [--port <port>] [--json]
```

### 游戏对象与层级 (GameObject & Hierarchy)
```bash
# 按名称或路径查找 GameObject
cd <unity-project-root> && node .unity-websocket/uw go find <name> [--json]

# 创建 GameObject
cd <unity-project-root> && node .unity-websocket/uw go create <name> [--parent <parent>] [--json]

# 销毁 GameObject
cd <unity-project-root> && node .unity-websocket/uw go destroy <name> [--json]

# 设置激活状态
cd <unity-project-root> && node .unity-websocket/uw go set-active <name> <true|false> [--json]

# 设置/移除父对象
cd <unity-project-root> && node .unity-websocket/uw go set-parent <name> [parent] [--json]

# 获取父对象信息
cd <unity-project-root> && node .unity-websocket/uw go get-parent <name> [--json]

# 获取子对象
cd <unity-project-root> && node .unity-websocket/uw go get-children <name> [--recursive] [--json]

# 查看层级树
cd <unity-project-root> && node .unity-websocket/uw hierarchy [--root-only] [--include-inactive] [--json]
```

### 变换 (Transform)
```bash
# 获取 transform 信息
cd <unity-project-root> && node .unity-websocket/uw tf get <name> [--json]

# 设置位置 (x,y,z)
cd <unity-project-root> && node .unity-websocket/uw tf set-position <name> <x,y,z> [--json]

# 设置旋转 (欧拉角，度数)
cd <unity-project-root> && node .unity-websocket/uw tf set-rotation <name> <x,y,z> [--json]

# 设置缩放
cd <unity-project-root> && node .unity-websocket/uw tf set-scale <name> <x,y,z> [--json]
```

### 组件管理 (Component Management)
```bash
# 列出 GameObject 上的组件
cd <unity-project-root> && node .unity-websocket/uw comp list <gameobject> [--include-disabled] [--json]

# 向 GameObject 添加组件
cd <unity-project-root> && node .unity-websocket/uw comp add <gameobject> <component-type> [--json]

# 从 GameObject 移除组件
cd <unity-project-root> && node .unity-websocket/uw comp remove <gameobject> <component-type> [--json]

# 启用/禁用组件
cd <unity-project-root> && node .unity-websocket/uw comp enable <gameobject> <component-type> [--json]
cd <unity-project-root> && node .unity-websocket/uw comp disable <gameobject> <component-type> [--json]

# 获取组件属性
cd <unity-project-root> && node .unity-websocket/uw comp get <gameobject> <component-type> [property] [--json]

# 设置组件属性
cd <unity-project-root> && node .unity-websocket/uw comp set <gameobject> <component-type> <property> <value> [--json]

# 检查组件 (显示所有属性)
cd <unity-project-root> && node .unity-websocket/uw comp inspect <gameobject> <component-type> [--json]

# 移动组件顺序
cd <unity-project-root> && node .unity-websocket/uw comp move-up <gameobject> <component-type> [--json]
cd <unity-project-root> && node .unity-websocket/uw comp move-down <gameobject> <component-type> [--json]

# 在 GameObject 之间复制组件
cd <unity-project-root> && node .unity-websocket/uw comp copy <source> <component-type> <target> [--json]
```

### 场景管理 (Scene Management)
```bash
# 获取当前场景信息
cd <unity-project-root> && node .unity-websocket/uw scene current [--json]

# 列出所有已加载的场景
cd <unity-project-root> && node .unity-websocket/uw scene list [--json]

# 加载场景
cd <unity-project-root> && node .unity-websocket/uw scene load <name> [--additive] [--json]

# 创建新场景
cd <unity-project-root> && node .unity-websocket/uw scene new [--empty] [--additive] [--json]

# 保存场景
cd <unity-project-root> && node .unity-websocket/uw scene save [path] [--scene <name>] [--json]

# 卸载场景
cd <unity-project-root> && node .unity-websocket/uw scene unload <name> [--remove] [--json]

# 设置活动场景 (多场景编辑)
cd <unity-project-root> && node .unity-websocket/uw scene set-active <name> [--json]
```

### 资产数据库与编辑器 (Asset Database & Editor)
```bash
# 刷新 AssetDatabase
cd <unity-project-root> && node .unity-websocket/uw editor refresh [--json]

# 重新编译脚本
cd <unity-project-root> && node .unity-websocket/uw editor recompile [--json]

# 重新导入资产
cd <unity-project-root> && node .unity-websocket/uw editor reimport <path> [--json]
```

### 控制台与日志 (Console & Logging)
```bash
# 获取控制台日志
cd <unity-project-root> && node .unity-websocket/uw console logs [--count <n>] [--errors-only] [--warnings] [--json]

# 清除控制台
cd <unity-project-root> && node .unity-websocket/uw console clear [--json]
```

### EditorPrefs 管理 (EditorPrefs Management)
```bash
# 获取 EditorPrefs 值
cd <unity-project-root> && node .unity-websocket/uw prefs get <key> [-t <type>] [-d <default>] [--json]

# 设置 EditorPrefs 值
cd <unity-project-root> && node .unity-websocket/uw prefs set <key> <value> [-t <type>] [--json]

# 删除 EditorPrefs 键
cd <unity-project-root> && node .unity-websocket/uw prefs delete <key> [--json]

# 检查键是否存在
cd <unity-project-root> && node .unity-websocket/uw prefs has <key> [--json]

# 删除所有 EditorPrefs 键
cd <unity-project-root> && node .unity-websocket/uw prefs delete-all [--json]

# 列出所有 EditorPrefs 键
cd <unity-project-root> && node .unity-websocket/uw prefs list [--json]
```

### 等待命令 (Wait Commands)
```bash
# 等待编译完成
cd <unity-project-root> && node .unity-websocket/uw wait compile [--timeout <ms>] [--json]

# 等待播放模式更改
cd <unity-project-root> && node .unity-websocket/uw wait playmode <enter|exit|pause> [--timeout <ms>] [--json]

# 休眠指定时长
cd <unity-project-root> && node .unity-websocket/uw wait sleep <seconds> [--timeout <ms>] [--json]

# 等待场景加载完成 (仅限播放模式)
cd <unity-project-root> && node .unity-websocket/uw wait scene [--timeout <ms>] [--json]
```

### 链式命令 (Chain Commands)
```bash
# 从 JSON 文件执行命令
cd <unity-project-root> && node .unity-websocket/uw chain execute <file> [--continue-on-error] [--timeout <ms>] [--json]

# 内联执行命令
cd <unity-project-root> && node .unity-websocket/uw chain exec <commands...> [--continue-on-error] [--timeout <ms>] [--json]

# 示例：带参数的内联命令
cd <unity-project-root> && node .unity-websocket/uw chain exec \
  "GameObject.Create:name=Player" \
  "GameObject.SetActive:instanceId=123,active=true" \
  --continue-on-error
```

### 预制件管理 (Prefab Management)
```bash
# 实例化预制件
cd <unity-project-root> && node .unity-websocket/uw prefab instantiate <path> [--name <name>] [--position <x,y,z>] [--json]

# 从场景对象创建预制件
cd <unity-project-root> && node .unity-websocket/uw prefab create <gameobject> <path> [--overwrite] [--json]

# 解包预制件实例
cd <unity-project-root> && node .unity-websocket/uw prefab unpack <gameobject> [--completely] [--json]

# 应用/还原预制件覆盖
cd <unity-project-root> && node .unity-websocket/uw prefab apply <gameobject> [--json]
cd <unity-project-root> && node .unity-websocket/uw prefab revert <gameobject> [--json]

# 创建预制件变体
cd <unity-project-root> && node .unity-websocket/uw prefab variant <sourcePath> <variantPath> [--json]

# 获取预制件覆盖
cd <unity-project-root> && node .unity-websocket/uw prefab overrides <gameobject> [--json]

# 获取源预制件信息
cd <unity-project-root> && node .unity-websocket/uw prefab source <gameobject> [--json]

# 检查是否为预制件实例
cd <unity-project-root> && node .unity-websocket/uw prefab is-instance <gameobject> [--json]

# 打开/关闭预制件编辑模式
cd <unity-project-root> && node .unity-websocket/uw prefab open <path> [--json]
cd <unity-project-root> && node .unity-websocket/uw prefab close [--json]

# 列出文件夹中的预制件
cd <unity-project-root> && node .unity-websocket/uw prefab list [--path <path>] [--json]
```

### 材质管理 (Material Management)
```bash
# 列出 GameObject 上的材质
cd <unity-project-root> && node .unity-websocket/uw material list <gameobject> [--shared] [--json]

# 获取/设置材质属性
cd <unity-project-root> && node .unity-websocket/uw material get <gameobject> <property> [--json]
cd <unity-project-root> && node .unity-websocket/uw material set <gameobject> <property> <value> [--json]

# 获取/设置材质颜色
cd <unity-project-root> && node .unity-websocket/uw material get-color <gameobject> [-p <property>] [--json]
cd <unity-project-root> && node .unity-websocket/uw material set-color <gameobject> --hex "#FF0000" [--json]

# 获取/设置着色器
cd <unity-project-root> && node .unity-websocket/uw material get-shader <gameobject> [--json]
cd <unity-project-root> && node .unity-websocket/uw material set-shader <gameobject> <shaderName> [--json]

# 获取/设置纹理
cd <unity-project-root> && node .unity-websocket/uw material get-texture <gameobject> [--json]
cd <unity-project-root> && node .unity-websocket/uw material set-texture <gameobject> <texturePath> [--json]
```

### 着色器管理 (Shader Management)
```bash
# 列出所有着色器
cd <unity-project-root> && node .unity-websocket/uw shader list [-f <filter>] [--builtin] [--json]

# 按名称查找着色器
cd <unity-project-root> && node .unity-websocket/uw shader find <name> [--json]

# 获取着色器属性
cd <unity-project-root> && node .unity-websocket/uw shader properties <shaderName> [--json]

# 获取/启用/禁用着色器关键字
cd <unity-project-root> && node .unity-websocket/uw shader keywords --global [--json]
cd <unity-project-root> && node .unity-websocket/uw shader keyword-enable <keyword> [--json]
cd <unity-project-root> && node .unity-websocket/uw shader keyword-disable <keyword> [--json]
cd <unity-project-root> && node .unity-websocket/uw shader keyword-status <keyword> [--json]
```

---

## 开发路线图

### 第 1 阶段 (当前) - 核心基础 ✅
- **30 个命令** 涵盖 8 个类别
- GameObject 操作，Transform 控制，场景管理
- 控制台日志，编辑器实用工具，EditorPrefs
- 等待条件，命令链

### 第 2 阶段 - 组件与材质系统 🔄
- **~140+ 个命令**
- 组件管理 (添加，移除，配置)
- 材质系统 (颜色，纹理，着色器)
- 预制件系统 (实例化，创建，覆盖)
- 资产数据库 (搜索，导入，依赖)

### 第 3 阶段 - 动画与物理 🔄
- **~170+ 个命令**
- 动画系统 (Animator, Curves, Events)
- 物理系统 (Rigidbody, Collider, Raycast)
- 光照系统 (Lights, Lightmaps, Probes)
- 相机系统 (FOV, Viewport, Screenshots)

### 第 4 阶段 - 高级功能 🔄
- **~100+ 个命令**
- 音频系统 (AudioSource, Mixer, 3D Audio)
- 导航与 AI (NavMesh, Agents, Obstacles)
- 粒子系统 (Emission, Modules, Simulation)
- 时间轴 (Playable Director, Tracks, Clips)

### 第 5 阶段 - 构建与工具 🔄
- **~100+ 个命令**
- 构建管道 (Build, Player Settings, Platforms)
- 项目设置 (Quality, Physics, Input, Graphics)
- 包管理器 (Install, Update, Remove)
- 版本控制 (Git, Plastic SCM)
- 分析器与性能 (CPU, GPU, Memory)
- 测试运行器 (Unit Tests, Code Coverage)
- 输入系统 (Actions, Bindings, Devices)
- UI 工具包 (Visual Elements, USS, UXML)

---

**路线图总计**: 25 个类别中的 500+ 个命令

有关包含所有选项和示例的详细命令文档，请参阅上面链接的特定类别文档文件。
