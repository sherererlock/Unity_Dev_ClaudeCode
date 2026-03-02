# Unity Editor Toolkit - 场景管理命令

场景管理命令的完整参考。

**最后更新**：2025-01-25
**命令数量**：7 个命令

---

## cd <unity-project-root> && node .unity-websocket/uw scene current

获取当前活动场景信息。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw scene current [选项]
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间，单位为毫秒（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 获取当前场景信息
cd <unity-project-root> && node .unity-websocket/uw scene current

# 获取 JSON 输出
cd <unity-project-root> && node .unity-websocket/uw scene current --json
```

---

## cd <unity-project-root> && node .unity-websocket/uw scene list

列出所有已加载的场景。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw scene list [选项]
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间，单位为毫秒（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 列出所有场景
cd <unity-project-root> && node .unity-websocket/uw scene list

# 获取 JSON 输出
cd <unity-project-root> && node .unity-websocket/uw scene list --json
```

---

## cd <unity-project-root> && node .unity-websocket/uw scene load

按名称或路径加载场景。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw scene load <name> [选项]
```

**参数：**
```
<name>                 场景名称或路径（不含 .unity 扩展名）
```

**选项：**
```
-a, --additive         以叠加方式加载场景（不卸载当前场景）
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间，单位为毫秒（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 加载场景（替换当前场景）
cd <unity-project-root> && node .unity-websocket/uw scene load "MainMenu"

# 以叠加方式加载场景
cd <unity-project-root> && node .unity-websocket/uw scene load "UIOverlay" --additive

# 按路径加载场景
cd <unity-project-root> && node .unity-websocket/uw scene load "Assets/Scenes/Level1"
```

---

## cd <unity-project-root> && node .unity-websocket/uw scene new

创建一个新场景。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw scene new [选项]
```

**选项：**
```
-e, --empty            创建空场景（无默认摄像机/灯光）
-a, --additive         添加新场景而不替换当前场景
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间，单位为毫秒（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 创建带有默认对象（主摄像机、方向光）的新场景
cd <unity-project-root> && node .unity-websocket/uw scene new

# 创建空场景
cd <unity-project-root> && node .unity-websocket/uw scene new --empty

# 以叠加方式创建新场景（保留当前场景）
cd <unity-project-root> && node .unity-websocket/uw scene new --additive

# 以叠加方式创建空场景
cd <unity-project-root> && node .unity-websocket/uw scene new -e -a
```

---

## cd <unity-project-root> && node .unity-websocket/uw scene save

将场景保存到磁盘。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw scene save [path] [选项]
```

**参数：**
```
[path]                 “另存为”的可选路径（例如 "Assets/Scenes/NewScene.unity"）
```

**选项：**
```
-s, --scene <name>     要保存的特定场景名称（默认：活动场景）
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间，单位为毫秒（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 保存活动场景
cd <unity-project-root> && node .unity-websocket/uw scene save

# 另存为 - 保存到新位置
cd <unity-project-root> && node .unity-websocket/uw scene save "Assets/Scenes/Level2.unity"

# 保存特定场景（多场景编辑）
cd <unity-project-root> && node .unity-websocket/uw scene save --scene "UIScene"
```

---

## cd <unity-project-root> && node .unity-websocket/uw scene unload

从编辑器中卸载场景。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw scene unload <name> [选项]
```

**参数：**
```
<name>                 要卸载的场景名称或路径
```

**选项：**
```
-r, --remove           完全移除场景（默认：仅卸载/关闭）
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间，单位为毫秒（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 按名称卸载场景
cd <unity-project-root> && node .unity-websocket/uw scene unload "UIOverlay"

# 按路径卸载场景
cd <unity-project-root> && node .unity-websocket/uw scene unload "Assets/Scenes/Level1.unity"

# 完全移除场景
cd <unity-project-root> && node .unity-websocket/uw scene unload "TempScene" --remove
```

**注意**：无法卸载仅剩的最后一个场景。必须始终至少加载一个场景。

---

## cd <unity-project-root> && node .unity-websocket/uw scene set-active

将场景设置为活动场景（用于多场景编辑）。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw scene set-active <name> [选项]
```

**参数：**
```
<name>                 要设置为活动的场景名称或路径
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间，单位为毫秒（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 按名称设置活动场景
cd <unity-project-root> && node .unity-websocket/uw scene set-active "MainScene"

# 按路径设置活动场景
cd <unity-project-root> && node .unity-websocket/uw scene set-active "Assets/Scenes/Level1.unity"
```

**注意**：目标场景必须先加载才能设置为活动场景。

---

## 全局选项

所有命令都支持这些全局选项：

```
-V, --version          输出版本号
-v, --verbose          启用详细日志记录
-p, --port <number>    Unity WebSocket 端口（覆盖自动检测）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 检查 CLI 版本
cd <unity-project-root> && node .unity-websocket/uw --version

# 启用详细日志记录
cd <unity-project-root> && node .unity-websocket/uw --verbose scene current

# 使用特定端口
cd <unity-project-root> && node .unity-websocket/uw --port 9501 scene load "Level1"
```

---

## 说明

### 端口自动检测

Unity Editor Toolkit CLI 通过读取 Unity 项目目录中的 `.unity-websocket/server-status.json` 自动检测 Unity WebSocket 服务器端口。仅在以下情况下需要指定 `--port`：
- 运行多个 Unity 编辑器实例
- 服务器使用非默认端口范围

### JSON 输出

所有命令都支持 `--json` 标志以进行机器可读输出。适用于：
- CI/CD 管道
- 自动化脚本
- 与其他工具集成

### 超时配置

默认超时为 30 秒（30000 毫秒）。对于可能需要更长时间的操作，请增加超时时间：

```bash
# 为复杂操作设置更长的超时时间
cd <unity-project-root> && node .unity-websocket/uw scene load "Level1" --timeout 60000
```

### 错误处理

命令返回适当的退出代码：
- `0`：成功
- `1`：错误（连接失败、命令失败、参数无效等）

检查错误消息以获取有关失败的详细信息。

---

**另请参阅：**
- [QUICKSTART.md](../../QUICKSTART.md) - 快速设置和首个命令
- [COMMANDS.md](./COMMANDS.md) - 完整命令路线图
- [API_COMPATIBILITY.md](../../API_COMPATIBILITY.md) - Unity 版本兼容性
- [TEST_GUIDE.md](../../TEST_GUIDE.md) - Unity C# 服务器测试指南
