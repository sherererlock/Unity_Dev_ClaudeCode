# Unity Editor Toolkit - Unity Package

通过 WebSocket 实现实时的 Unity 编辑器控制，用于 Claude Code 集成。

## 安装

### 推荐：Unity Package Manager (UPM)

1. 打开 Unity 编辑器
2. 菜单栏：Window → Package Manager
3. 点击 `+` → Add package from git URL
4. 输入：`https://github.com/Dev-GOM/claude-code-marketplace.git?path=/plugins/unity-editor-toolkit/skills/assets/unity-package`
5. 点击 Add

### 替代方案：Assets 文件夹

为了更容易自定义，将此文件夹复制到 `Assets/UnityEditorToolkit/`

> **注意**：UPM 安装到 `Packages/` 文件夹（只读），而 Assets 方式允许直接修改。

## 快速开始

### 1. 打开服务器窗口

1. Unity 菜单：`Tools > Unity Editor Toolkit > Server Window`
2. 一个新窗口将出现在编辑器中

### 2. 配置插件脚本路径

1. **Plugin Scripts Path**：自动从用户主目录检测 (`~/.claude/plugins/...`)
2. 如果未检测到，点击 "Browse" 手动选择
3. 路径应指向：`unity-editor-toolkit/skills/scripts`

### 3. 安装 CLI（一次性设置）

1. 点击 "Install CLI" 按钮
2. 这将构建 WebSocket 服务器和 TypeScript CLI
3. 等待安装完成（可能需要 1-2 分钟）
4. 控制台显示："✓ CLI installation completed"

### 4. 服务器自动启动

1. Unity 编辑器打开时服务器自动启动
2. **端口**：从 9500-9600 范围自动分配（无需手动配置）
3. **状态文件**：`{ProjectRoot}/.unity-websocket/server-status.json`
4. CLI 会自动从该文件检测正确的端口

### 5. 从 Claude Code 连接

在 Claude Code 中安装 Unity Editor Toolkit 插件：

```bash
# 添加市场
/plugin marketplace add https://github.com/Dev-GOM/claude-code-marketplace.git

# 安装插件
/plugin install unity-editor-toolkit@dev-gom-plugins
```

使用 CLI 命令：

```bash
# 查找游戏对象
cd <unity-project-root> node .unity-websocket/uw go find "Player"

# 设置位置
cd <unity-project-root> node .unity-websocket/uw tf set-position "Player" "0,5,10"

# 加载场景
cd <unity-project-root> node .unity-websocket/uw scene load "GameScene"

# 获取控制台日志
cd <unity-project-root> node .unity-websocket/uw console logs
```

## 要求

### Unity 版本
- **Unity 2020.3 或更高版本**
- 完全兼容 **Unity 6+ (2023.x+)**

### 依赖项
- websocket-sharp 库（见下方依赖项部分）

### 测试框架（用于运行测试）
- **Unity 2019.2+**：自动包含在所有项目中
- **Unity 6+ (2023.x+)**：核心包（版本锁定到编辑器）+ 新特性

## 依赖项

### websocket-sharp

此包需要 websocket-sharp 用于 WebSocket 通信。

**安装：**

1. 从以下网址下载 websocket-sharp：https://github.com/sta/websocket-sharp/releases
2. 解压 `websocket-sharp.dll`
3. 复制到 `Packages/com.devgom.unity-editor-toolkit/ThirdParty/websocket-sharp/`
4. Unity 将自动导入 DLL

**替代方案：**

通过 NuGet for Unity 添加：
1. 安装 NuGet for Unity：https://github.com/GlitchEnzo/NuGetForUnity
2. 打开 NuGet 窗口
3. 搜索 "websocket-sharp"
4. 安装

## 支持的命令

### GameObject (5 个命令)
- `GameObject.Find` - 按名称查找 GameObject
- `GameObject.Create` - 创建新 GameObject
- `GameObject.Destroy` - 销毁 GameObject
- `GameObject.SetActive` - 设置激活状态

### Transform (6 个命令)
- `Transform.GetPosition` - 获取世界坐标
- `Transform.SetPosition` - 设置世界坐标
- `Transform.GetRotation` - 获取旋转（欧拉角）
- `Transform.SetRotation` - 设置旋转
- `Transform.GetScale` - 获取本地缩放
- `Transform.SetScale` - 设置本地缩放

### Scene (3 个命令)
- `Scene.GetCurrent` - 获取当前场景信息
- `Scene.GetAll` - 获取所有已加载场景
- `Scene.Load` - 加载场景（单一或叠加）

### Console (2 个命令)
- `Console.GetLogs` - 获取带过滤的控制台日志
- `Console.Clear` - 清除控制台

### Hierarchy (1 个命令)
- `Hierarchy.Get` - 获取 GameObject 层级树

## API 示例

### 查找 GameObject

**请求：**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "GameObject.Find",
  "params": { "name": "Player" }
}
```

**响应：**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
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

### 设置位置

**请求：**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "Transform.SetPosition",
  "params": {
    "name": "Player",
    "position": { "x": 0, "y": 5, "z": 10 }
  }
}
```

**响应：**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": { "success": true }
}
```

## 安全性

- **仅限本地主机**：服务器仅接受来自 127.0.0.1 的连接
- **仅限编辑器模式**：服务器仅在编辑器中运行，不在构建版本中运行
- **支持撤销**：所有操作均支持 Unity 的撤销（Undo）系统

## 故障排除

### 服务器无法启动

1. 检查控制台是否有错误消息
2. 验证端口 9500 未被占用
3. 确保已安装 websocket-sharp.dll
4. 尝试不同的端口号

### 无法连接

1. 验证服务器是否正在运行（检查控制台）
2. 确认 WebSocket URL：`ws://127.0.0.1:9500`
3. 检查防火墙设置
4. 确保 Unity 编辑器已打开（编辑模式或播放模式）

### 命令失败

1. 检查控制台获取错误详情
2. 验证 GameObject 名称是否正确
3. 确保场景已加载
4. 检查参数格式是否匹配 API

## 编辑器窗口

通过 Unity 菜单访问服务器控制：

**Tools → Unity Editor Toolkit → Server Window**

特性：
- 服务器状态监控
- 插件脚本路径配置
- CLI 安装和构建
- 快速访问文档

## 性能

- 极低开销：每条命令约 1-2ms
- 支持多个并发客户端
- 日志限制为 1000 条
- 线程安全操作

## 已知限制

- 仅限编辑器模式（构建版本中不可用）
- 单一场景激活命令执行
- GameObject 查找仅限于活动场景
- 控制台日志限制为最近 1000 条

## 未来特性

请参阅 [COMMANDS.md](../COMMANDS.md) 了解计划中的 500+ 命令，包括：
- 组件操作
- 材质编辑
- 预制件实例化
- 资源数据库查询
- 动画控制
- 物理模拟
- 以及更多...

## 许可证

Apache License 2.0

## 链接

- [GitHub 仓库](https://github.com/Dev-GOM/claude-code-marketplace)
- [插件文档](../README.md)
- [命令参考](../COMMANDS.md)
- [问题追踪](https://github.com/Dev-GOM/claude-code-marketplace/issues)

## 支持

如有问题、疑问或功能请求，请访问：
https://github.com/Dev-GOM/claude-code-marketplace/issues
