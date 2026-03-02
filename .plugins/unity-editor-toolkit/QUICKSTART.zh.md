**Language**: English | [한국어](./QUICKSTART.ko.md)

---

# Unity 编辑器工具包 - 快速入门指南

从安装到首次执行命令的完整设置指南。

## 先决条件

- Unity 2020.3 或更高版本
- 已安装 Claude Code
- 基本熟悉 Unity 编辑器

## 安装步骤

### 1. 安装 Claude Code 插件

打开 Claude Code 设置并添加：

```json
{
  "plugins": {
    "marketplaces": [
      {
        "name": "dev-gom-plugins",
        "url": "https://github.com/Dev-GOM/claude-code-marketplace"
      }
    ],
    "enabled": ["dev-gom-plugins:unity-editor-toolkit"]
  }
}
```

### 2. 安装 Unity 包 (通过包管理器)

1. 打开 Unity 编辑器
2. 转到 `Window → Package Manager` (窗口 → 包管理器)
3. 点击左上角的 `+` 按钮 → `Add package from git URL` (从 git URL 添加包)
4. 输入此 URL：
   ```
   https://github.com/Dev-GOM/claude-code-marketplace.git?path=/plugins/unity-editor-toolkit/skills/assets/unity-package
   ```
5. 点击 `Add` (添加) 并等待安装完成

> **替代方案**：如果您更喜欢安装在 Assets 文件夹中（以便于自定义），请将 `plugins/unity-editor-toolkit/skills/assets/unity-package/` 复制到 `Assets/UnityEditorToolkit/`

### 3. 安装 websocket-sharp DLL

该包需要 websocket-sharp DLL。在包管理器中找到安装脚本：

1. 在包管理器中，选择 "Unity Editor Toolkit"
2. 查找 "Samples" (示例) 部分并导入 "Installation Scripts" (安装脚本)
3. 或者手动导航至：
   ```
   Packages/com.devgom.unity-editor-toolkit/ThirdParty/websocket-sharp/
   ```

**Windows**: 双击 `install.bat`
**macOS/Linux**: 在终端中运行 `./install.sh`

**手动安装** (如果自动安装失败)：
1. 下载：https://github.com/sta/websocket-sharp/releases/download/1.0.3-rc11/websocket-sharp.dll
2. 放置于：`Packages/com.devgom.unity-editor-toolkit/ThirdParty/websocket-sharp/websocket-sharp.dll`

### 4. 设置 WebSocket 服务器

1. 打开 Unity 编辑器工具包服务器窗口：
   - 在 Unity 菜单中：`Tools > Unity Editor Toolkit > Server Window`
   - 一个新窗口将出现在编辑器中

2. 配置插件脚本路径：
   - **Plugin Scripts Path** (插件脚本路径)：自动从用户主文件夹检测 (`~/.claude/plugins/...`)
   - 如果未检测到，点击 "Browse" (浏览) 手动选择
   - 路径应指向：`unity-editor-toolkit/skills/scripts`

3. 安装 CLI (一次性设置)：
   - 点击 "Install CLI" (安装 CLI) 按钮
   - 这将构建 WebSocket 服务器和 TypeScript CLI
   - 等待安装完成（可能需要 1-2 分钟）
   - 控制台显示："✓ CLI installation completed" (CLI 安装完成)

4. 服务器自动启动：
   - Unity 编辑器打开时服务器自动启动
   - **Port** (端口)：从 9500-9600 范围自动分配（无需手动配置）
   - **Status file** (状态文件)：`{ProjectRoot}/.unity-websocket/server-status.json`
   - CLI 会自动从该文件检测正确的端口
   - 检查控制台显示：`✓ Unity Editor Server started on ws://127.0.0.1:XXXX`

## 初步命令

在 Claude Code 中打开终端并尝试这些命令：

### 1. 检查连接状态

```bash
cd <unity-project-root> node .unity-websocket/uw status
```

预期输出：
```
✓ Connected to Unity Editor
WebSocket: ws://127.0.0.1:9500
Status: Running
```

### 2. 查找游戏对象 (GameObject)

```bash
cd <unity-project-root> node .unity-websocket/uw go find "Main Camera"
```

预期输出：
```
✓ GameObject found:
  Name: Main Camera
  Instance ID: 12345
  Path: /Main Camera
  Active: true
  Tag: MainCamera
  Layer: 0
```

### 3. 创建新游戏对象

```bash
cd <unity-project-root> node .unity-websocket/uw go create "TestCube"
```

检查 Unity 层级视图 - 您应该会看到一个新的 "TestCube" 游戏对象！

### 4. 设置位置

```bash
cd <unity-project-root> node .unity-websocket/uw tf set-position "TestCube" "5,2,3"
```

在 Unity 场景视图中，"TestCube" 移动到位置 (5, 2, 3)。

### 5. 获取场景信息

```bash
cd <unity-project-root> node .unity-websocket/uw scene current
```

显示有关当前活动场景的信息。

### 6. 查看层级结构

```bash
cd <unity-project-root> node .unity-websocket/uw hierarchy
```

以树状格式显示整个游戏对象层级结构。

### 7. 获取控制台日志

```bash
cd <unity-project-root> node .unity-websocket/uw console logs --count 10
```

显示最近 10 条控制台日志条目。

## 验证清单

- [ ] Claude Code 插件已安装并启用
- [ ] Unity 包已成功导入
- [ ] websocket-sharp.dll 位置正确
- [ ] Unity 编辑器工具包服务器窗口已打开 (`Tools > Unity Editor Toolkit > Server Window`)
- [ ] 插件脚本路径配置正确
- [ ] CLI 已成功安装（点击 "Install CLI" 按钮）
- [ ] 服务器已自动启动（检查控制台启动消息）
- [ ] 状态文件已创建：`.unity-websocket/server-status.json`
- [ ] 控制台显示 "✓ Unity Editor Server started on ws://127.0.0.1:XXXX"
- [ ] `cd <unity-project-root> node .unity-websocket/uw status` 命令工作正常
- [ ] 可以创建/查找游戏对象
- [ ] 可以修改变换组件 (transforms)
- [ ] Unity 控制台无错误

## 故障排除

### "Server not found" (未找到服务器) 或 "Connection refused" (连接被拒绝)

**检查：**
1. Unity 编辑器已打开
2. 控制台显示服务器启动消息
3. 项目根目录下存在状态文件：`.unity-websocket/server-status.json`
4. 端口范围 9500-9600 未被防火墙阻止
5. 服务器窗口显示 "Server Status: Running" (服务器状态：运行中)

**修复：**
```bash
# 检查 Unity 项目根目录下的状态文件
ls -la .unity-websocket/

# 如果需要，手动指定端口
cd <unity-project-root> node .unity-websocket/uw --port 9500 status
```

在 Unity 服务器窗口中，检查 "Server Status" (服务器状态)，必要时重启。

### "Assembly 'websocket-sharp' not found" (未找到程序集 'websocket-sharp')

**修复：**
1. 验证 DLL 位置：`ThirdParty/websocket-sharp/websocket-sharp.dll`
2. 重启 Unity 编辑器
3. 检查控制台是否有导入错误
4. 尝试重新导入：右键点击包 → Reimport (重新导入)

### 命令超时或失败

**检查：**
1. 游戏对象名称正确（区分大小写）
2. 场景已加载
3. Unity 未进入错误状态
4. 服务器仍在运行（检查控制台）

**修复：**
```bash
# 先检查服务器状态
cd <unity-project-root> node .unity-websocket/uw status

# 尝试简单命令
cd <unity-project-root> node .unity-websocket/uw go find "Main Camera"
```

### Unity 控制台显示错误

**常见问题：**

**"NullReferenceException"**
- 游戏对象名称不存在
- 场景未加载
- 组件未找到

**"JsonException"**
- 命令参数格式错误
- 检查文档中的参数格式

**"SocketException"**
- 端口已被占用
- 防火墙阻止连接
- 尝试不同端口

## 下一步

### 了解更多命令

请参阅 [COMMANDS.md](./skills/references/COMMANDS.md) 获取完整的 500+ 命令路线图。

**当前可用 (18 个命令)：**
- 连接与状态：Status (1 个命令)
- 游戏对象与层级：Find, Create, Destroy, SetActive, Hierarchy (5 个命令)
- 变换组件：Get, SetPosition, SetRotation, SetScale (4 个命令)
- 场景：Current, List, Load (3 个命令)
- 资产数据库与编辑器：Refresh, Recompile, Reimport (3 个命令)
- 控制台：Logs, Clear (2 个命令)

### 高级用法

**批处理操作：**
```bash
# 创建多个立方体
for i in {1..5}; do
  cd <unity-project-root> node .unity-websocket/uw go create "Cube_$i"
  cd <unity-project-root> node .unity-websocket/uw tf set-position "Cube_$i" "$i,0,0"
done
```

**脚本集成：**
```bash
# 将层级结构保存到文件
cd <unity-project-root> node .unity-websocket/uw hierarchy > hierarchy.json

# 实时监控控制台
cd <unity-project-root> node .unity-websocket/uw console stream --filter error
```

### 编辑器窗口

访问服务器控制面板：

`Tools → Unity Editor Toolkit → Server Window`

功能：
- 安装 CLI (一次性设置)
- 配置插件脚本路径
- 查看服务器状态 (Running/Stopped)
- 查看连接信息 (端口, 状态文件位置)
- 手动启动/停止服务器
- 访问文档

## 支持

**问题反馈：**
https://github.com/Dev-GOM/claude-code-marketplace/issues

**文档：**
- [完整自述文件](./README.md)
- [命令路线图](./skills/references/COMMANDS.md) - 500+ 命令路线图
- **已实现的命令类别：**
  - [连接与状态](./skills/references/COMMANDS_CONNECTION_STATUS.md)
  - [游戏对象与层级](./skills/references/COMMANDS_GAMEOBJECT_HIERARCHY.md)
  - [变换组件](./skills/references/COMMANDS_TRANSFORM.md)
  - [场景管理](./skills/references/COMMANDS_SCENE.md)
  - [资产数据库与编辑器](./skills/references/COMMANDS_EDITOR.md)
  - [控制台与日志](./skills/references/COMMANDS_CONSOLE.md)
- [Unity 包文档](./skills/assets/unity-package/README.md)

---

**恭喜！** 🎉 您已成功设置 Unity 编辑器工具包。现在您可以直接从 Claude Code 控制 Unity 编辑器了！
