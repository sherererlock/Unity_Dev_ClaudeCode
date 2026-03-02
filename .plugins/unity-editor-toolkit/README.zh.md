# Unity Editor Toolkit

> **⚠️ 状态**: 🧪 实验性 (v0.15.1) - **需要 Unity 6+**
>
> **此插件目前处于实验阶段。API 和功能可能会发生变化。**
> **数据库功能需要 Unity 6 或更高版本**（嵌入式 SQLite，无需安装）

用于 Claude Code 的完整 Unity 编辑器控制和自动化工具包，集成了 SQLite 数据库。通过实时 WebSocket 自动化，指挥 25 个类别的 500 多个 Unity 编辑器功能 - GameObject、组件、场景、材质、物理、动画等。

## 最近更新

查看 [CHANGELOG.md](./CHANGELOG.md) 获取完整的发布说明。

## 功能

- **500+ 命令**: 跨越 25 个 Unity 编辑器类别的全面控制
- **实时 WebSocket**: 即时双向通信 (端口 9500-9600)
- **SQLite 数据库集成**: 具有基于 GUID 持久性的实时 GameObject 同步
  - **基于 GUID 的标识**: 跨 Unity 会话的持久 GameObject 跟踪
  - **多场景支持**: 同时同步所有加载的场景
  - **命令模式**: 数据库操作的撤销/重做支持
  - **自动迁移**: 自动模式迁移系统
  - **批处理操作**: 高效的批量插入、更新和删除（500 个对象/批次）
- **GameObject & 层级视图**: 创建、销毁、操作、查询层级结构并带有树状可视化
- **Transform 控制**: 对位置、旋转、缩放进行精确的 Vector3 操作
- **组件管理**: 添加、移除、配置组件并访问属性
- **场景管理**: 加载、保存、合并多个场景并控制构建设置
- **材质 & 渲染**: 材质、着色器、纹理、渲染器属性
- **Prefab 系统**: 实例化、创建、覆盖、变体管理
- **资产数据库**: 搜索、导入、依赖项、标签、Bundle 分配
- **动画**: 播放、Animator 参数、曲线、事件
- **物理**: Rigidbody、Collider、Raycast、模拟控制
- **控制台日志**: 带有过滤、导出、流式传输的实时日志
- **编辑器自动化**: 播放模式、窗口聚焦、选择、场景视图控制
- **构建 & 部署**: 构建管线、播放器设置、平台切换
- **高级功能**: 光照、相机、音频、导航、粒子、Timeline、UI Toolkit
- **安全加固**: 防御路径遍历、命令注入、JSON 注入、SQL 注入
- **跨平台**: 完全支持 Windows、macOS、Linux

## 安装

此插件是 [Dev GOM Plugins](https://github.com/Dev-GOM/claude-code-marketplace) 市场的一部分。

### 从市场安装（推荐）

```bash
# 添加市场
/plugin marketplace add https://github.com/Dev-GOM/claude-code-marketplace.git

# 安装插件
/plugin install unity-editor-toolkit@dev-gom-plugins
```

### 直接安装

```bash
# 直接从仓库安装
/plugin add https://github.com/Dev-GOM/claude-code-marketplace/tree/main/plugins/unity-editor-toolkit
```

## 使用方法

### Unity 设置

1. **安装 Unity 包**:

   首先安装依赖包，然后安装主包：

   **第 1 步：安装依赖项**
   - 打开 Unity 编辑器
   - Window（窗口） → Package Manager（包管理器）
   - 点击 `+` → Add package from git URL...（从 git URL 添加包...）
   - 按顺序添加以下包：

   ```
   https://github.com/Cysharp/UniTask.git?path=src/UniTask/Assets/Plugins/UniTask
   ```

   ```
   https://github.com/gilzoide/unity-sqlite-net.git#1.3.2
   ```

   **第 2 步：安装 Unity Editor Toolkit**
   - 安装完两个依赖包后，添加主包：

   ```
   https://github.com/Dev-GOM/claude-code-marketplace.git?path=/plugins/unity-editor-toolkit/skills/assets/unity-package
   ```

   **替代方法：**
   - 通过 Package Manager 从 `skills/assets/unity-package` 添加 Unity 包（从磁盘添加包）
   - 或者直接将包文件夹复制到项目的 `Packages` 目录中

2. **设置 WebSocket 服务器**:
   - 从菜单打开 `Unity Editor Toolkit Server` 窗口：`Tools > Unity Editor Toolkit > Server Window`
   - 配置插件脚本路径（默认：从用户主文件夹自动检测）
   - 点击 "Install CLI" 构建 WebSocket 服务器（一次性设置）
   - Unity 编辑器打开时服务器会自动启动

3. **数据库设置**（可选）:
   - 在服务器窗口中，切换到 "Database" 标签页
   - 点击 "Connect" 初始化 SQLite 数据库
   - 数据库文件位置：`{ProjectRoot}/.unity-websocket/unity-editor.db`
   - 点击 "Start Sync" 启用实时 GameObject 同步（1秒间隔）
   - **GUID 组件**: GameObject 会自动标记持久性 GUID
   - **多场景**: 所有加载的场景都会自动同步
   - **分析**: 查看同步统计、数据库健康状况和撤销/重做历史

4. **服务器状态**:
   - 端口：自动分配（9500-9600 范围）
   - 状态文件：`{ProjectRoot}/.unity-websocket/server-status.json`
   - CLI 会自动从该文件检测正确的端口

### 更新包

要将 Unity Editor Toolkit 更新到最新版本：

1. 打开 Unity 编辑器
2. Window（窗口） → Package Manager（包管理器）
3. 点击 `+` → Add package from git URL...（从 git URL 添加包...）
4. 输入：

   ```
   https://github.com/Dev-GOM/claude-code-marketplace.git?path=/plugins/unity-editor-toolkit/skills/assets/unity-package
   ```

5. 点击 Add（这将更新现有包）

**注意：** 除非发布说明中指定，否则依赖包（UniTask, unity-sqlite-net）通常不需要更新。

### CLI 命令

```bash
# 基本用法
cd <unity-project-root> && node .unity-websocket/uw <command> [options]

# 显示所有可用命令
cd <unity-project-root> && node .unity-websocket/uw --help

# 显示特定命令的帮助
cd <unity-project-root> && node .unity-websocket/uw <command> --help
```

**目前已实现**: 15 个类别中的 86 个命令

查看 [COMMANDS.md](./skills/references/COMMANDS.md) 获取完整的命令参考。

#### 命令类别

查看 [COMMANDS.md](./skills/references/COMMANDS.md) 获取完整的命令参考，包括：

- **Component**: 添加、移除、配置组件并访问属性
- **Material**: 颜色、纹理、着色器、渲染器设置
- **Prefab**: 实例化、创建、覆盖管理
- **Asset Database**: 搜索、导入、依赖项
- **Animation**: Animator 参数、剪辑、曲线
- **Physics**: Rigidbody、Collider、Raycast、模拟
- **Lighting**: 灯光、光照贴图、反射探针
- **Camera**: FOV、视口、截图
- **Audio**: AudioSource、混音器、3D 音频
- **Navigation**: NavMesh、代理、障碍物
- **Particles**: 发射、模块、模拟
- **Timeline**: Playable director、轨道、剪辑
- **Build**: 构建管线、播放器设置
- **Profiler**: 性能数据、内存快照
- **Test Runner**: 单元测试、代码覆盖率
- **还有 10+ 个更多类别...**

### 命令示例

**创建并配置 GameObject:**
```bash
cd <unity-project-root> && node .unity-websocket/uw go create "Enemy" && \
cd <unity-project-root> && node .unity-websocket/uw tf set-position "Enemy" "10,0,5" && \
cd <unity-project-root> && node .unity-websocket/uw tf set-rotation "Enemy" "0,45,0"
```

**实例化 Prefab 并修改:**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefab instantiate "Prefabs/Player" --position "0,1,0" && \
cd <unity-project-root> && node .unity-websocket/uw material set-color "Player" "_Color" "0,1,0,1"
```

**加载场景并激活 GameObject:**
```bash
cd <unity-project-root> && node .unity-websocket/uw scene load "Level1" && \
cd <unity-project-root> && node .unity-websocket/uw go set-active "Boss" true
```

**实时监控控制台错误:**
```bash
cd <unity-project-root> && node .unity-websocket/uw console stream --filter error
```

**批量创建 GameObject:**
```bash
for i in {1..10}; do
  cd <unity-project-root> && node .unity-websocket/uw go create "Cube_$i" && \
  cd <unity-project-root> && node .unity-websocket/uw tf set-position "Cube_$i" "$i,0,0"
done
```

## 架构

### 组件

- **Unity C# Server**: 带有 JSON-RPC 2.0 处理框架的 WebSocket 服务器
- **Server Status Sync**: 通过 `.unity-websocket/server-status.json` 自动发现端口
- **WebSocket Client**: 带有自动重连和超时处理的 TypeScript 实现
- **CLI Framework**: 带有模块化命令架构的 Commander.js
- **Security Layer**: 多层输入验证和注入防御

### 通信协议

基于 WebSocket 的 JSON-RPC 2.0:

**请求:**
```json
{
  "jsonrpc": "2.0",
  "id": "req_1",
  "method": "GameObject.Find",
  "params": { "name": "Player" }
}
```

**响应:**
```json
{
  "jsonrpc": "2.0",
  "id": "req_1",
  "result": {
    "name": "Player",
    "instanceId": 12345,
    "path": "/Player",
    "active": true,
    "tag": "Player",
    "layer": 0
  }
}
```

### 端口分配与发现

- **范围**: 9500-9600 (100 个端口)
- **无冲突**: 避免与 Browser Pilot (9222-9322) 和 Blender Toolkit (9400-9500) 冲突
- **自动检测**: Unity 服务器将端口写入 `.unity-websocket/server-status.json`
- **CLI 发现**: 自动从状态文件读取端口（无需手动配置）
- **心跳**: 服务器每 5 秒更新一次状态以进行连接健康监测

## 安全性

纵深防御安全实现：

- **路径遍历保护**: 带有 `..` 检测的 `path.resolve()` 验证
- **命令注入防御**: 经过净化的 npm 执行和环境隔离
- **JSON 注入预防**: 对所有结构进行运行时类型验证
- **日志注入防御**: 消息净化防止日志被篡改
- **SQL 注入预防**: 所有数据库操作均使用参数化查询
  - 小批量（≤100 项）使用单独的 DELETE 语句
  - 大批量（>100 项）使用临时表模式
- **事务安全**: 嵌套事务检测和优雅回退
- **WebSocket 安全**: 仅限本地主机连接
- **端口验证**: 强制执行 9500-9600 范围
- **原子操作**: 无竞争条件的锁获取 (`{ flag: 'wx' }`)
- **内存安全**: 正确的事件监听器清理和域重新加载 (Domain Reload) 安全检查

## 开发

### 项目结构

```
unity-editor-toolkit/
├── .claude-plugin/
│   └── plugin.json              # 插件元数据
├── skills/
│   ├── SKILL.md                 # 技能文档
│   ├── scripts/
│   │   ├── src/
│   │   │   ├── cli/
│   │   │   │   ├── cli.ts       # 主要 CLI 入口点
│   │   │   │   └── commands/    # 命令实现
│   │   │   ├── constants/
│   │   │   │   └── index.ts     # 集中式常量
│   │   │   ├── unity/
│   │   │   │   ├── client.ts    # WebSocket 客户端
│   │   │   │   └── protocol.ts  # JSON-RPC 类型
│   │   │   └── utils/
│   │   │       ├── config.ts    # 配置管理
│   │   │       └── logger.ts    # 日志工具
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── references/              # 文档
│   │   ├── QUICKSTART.md        # 快速入门指南
│   │   ├── QUICKSTART.ko.md     # 韩语快速入门指南
│   │   ├── COMMANDS.md          # 完整命令参考 (500+)
│   │   ├── COMMANDS.ko.md       # 韩语命令参考
│   │   ├── API_COMPATIBILITY.md # Unity 版本兼容性
│   │   ├── TEST_GUIDE.md        # 测试指南
│   │   └── TEST_GUIDE.ko.md     # 韩语测试指南
│   └── assets/                  # Unity 包
│       └── unity-package/       # Unity C# WebSocket 服务器
│           ├── Runtime/         # 核心处理程序 & 协议
│           ├── Editor/          # 编辑器窗口
│           ├── Tests/           # 单元测试 (66 个测试)
│           ├── ThirdParty/      # websocket-sharp
│           └── package.json     # Unity 包清单
├── README.md                    # 本文件
└── README.ko.md                 # 韩语 README
```

### 构建

```bash
cd skills/scripts
npm install
npm run build
```

### 测试

端到端测试需要 Unity C# 服务器实现。单元测试即将推出。

## 开发路线图

**阶段 2 (当前)**: 15 个类别中的 86 个命令 (GameObject, Transform, Component, Scene, Prefab, Material, Shader, Asset 等)
**阶段 3**: 动画、物理、光照 - 150+ 命令
**阶段 4**: 构建、Profiler、Test Runner - 100+ 命令
**阶段 5**: 高级功能 (Timeline, UI Toolkit, VCS) - 150+ 命令

查看 [COMMANDS.md](./skills/references/COMMANDS.md) 获取详细路线图。

## 状态

### Unity C# 服务器包 ✅
- [x] 使用 websocket-sharp 的 WebSocket 服务器
- [x] JSON-RPC 2.0 处理框架
- [x] 命令路由和执行
- [x] Unity Package Manager 集成
- [x] 服务器状态同步 (`.unity-websocket/server-status.json`)
- [x] 自动端口发现
- [x] 基于主文件夹的插件路径检测
- [x] 使用 UniTask async/await 的 SQLite 数据库集成
- [x] 基于 GUID 的 GameObject 标识（跨会话持久）
- [x] 多场景同步支持
- [x] 带有撤销/重做支持的命令模式
- [x] 自动迁移系统
- [x] 批处理操作（500 个对象/批次）
- [x] SQL 注入预防（参数化查询）
- [x] 事务嵌套预防

### 命令 (500+)
- [x] GameObject & Hierarchy (15 个命令)
- [x] Transform (8 个命令)
- [x] Scene Management (3 个命令)
- [x] Console & Logging (2 个命令)
- [x] EditorPrefs Management (6 个命令)
- [x] Wait Commands (4 个命令)
- [x] Chain Commands (2 个命令)
- [ ] Component (20+ 个命令)
- [ ] Material & Rendering (25+ 个命令)
- [ ] Prefab (15+ 个命令)
- [ ] Asset Database (20+ 个命令)
- [ ] Animation (20+ 个命令)
- [ ] Physics (20+ 个命令)
- [ ] Lighting (15+ 个命令)
- [ ] Camera (15+ 个命令)
- [ ] Audio (15+ 个命令)
- [ ] Navigation & AI (15+ 个命令)
- [ ] Particle System (15+ 个命令)
- [ ] Timeline (10+ 个命令)
- [ ] Build & Player (15+ 个命令)
- [ ] Project Settings (20+ 个命令)
- [ ] Package Manager (10+ 个命令)
- [ ] Version Control (10+ 个命令)
- [ ] Profiler & Performance (15+ 个命令)
- [ ] Test Runner (10+ 个命令)
- [ ] Input System (10+ 个命令)
- [ ] UI Toolkit (10+ 个命令)
- [ ] Utility Commands (20+ 个命令)

## 许可证

Apache License 2.0 - 查看 [LICENSE](../../LICENSE) 获取详情

## 相关插件

- [Browser Pilot](../browser-pilot) - 通过 Chrome DevTools Protocol 进行浏览器自动化
- [Blender Toolkit](../blender-toolkit) - Blender 3D 自动化和场景管理
- [Unity Dev Toolkit](../unity-dev-toolkit) - Unity 开发实用工具和编译错误修复

## 文档

- [COMMANDS.md](./skills/references/COMMANDS.md) - 完整命令参考 (500+ 命令)
- [COMMANDS.ko.md](./skills/references/COMMANDS.ko.md) - 韩语命令参考
- [DATABASE_GUIDE.md](./skills/references/DATABASE_GUIDE.md) - 数据库使用指南

---

**版本**: 0.15.1
**Unity 版本**: Unity 6+ (数据库功能需要嵌入式 SQLite 支持)
**最后更新**: 2025-12-02
**作者**: Dev GOM
**市场**: [dev-gom-plugins](https://github.com/Dev-GOM/claude-code-marketplace)
