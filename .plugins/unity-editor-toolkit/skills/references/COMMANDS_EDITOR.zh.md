# Unity Editor Toolkit - Asset Database 与编辑器实用工具命令

Asset Database 刷新、重新编译和资源重新导入命令的完整参考。

**最后更新**：2025-01-13

---

## cd <unity-project-root> && node .unity-websocket/uw editor refresh

刷新 Unity AssetDatabase（生成/更新 meta 文件，触发编译）。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw editor refresh [options]
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间（毫秒）（默认：30000）
-h, --help             显示命令帮助
```

**⚠️ 重要提示：**
运行 `refresh` 后，您应检查 Unity 编辑器的编译状态。Unity 的增量编译会自动编译已更改的程序集。

**示例：**
```bash
# 刷新 AssetDatabase
cd <unity-project-root> && node .unity-websocket/uw editor refresh

# 获取 JSON 输出
cd <unity-project-root> && node .unity-websocket/uw editor refresh --json
```

---

## cd <unity-project-root> && node .unity-websocket/uw editor recompile

请求脚本重新编译。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw editor recompile [options]
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间（毫秒）（默认：30000）
-h, --help             显示命令帮助
```

**⚠️ 重要提示：**
Unity 的增量编译会自动重新编译已更改的程序集。此命令强制进行重新编译检查。运行后请检查 Unity 编辑器的编译状态。

**示例：**
```bash
# 请求重新编译
cd <unity-project-root> && node .unity-websocket/uw editor recompile

# 获取 JSON 输出
cd <unity-project-root> && node .unity-websocket/uw editor recompile --json
```

---

## cd <unity-project-root> && node .unity-websocket/uw editor reimport

重新导入特定资源（触发该程序集的重新编译）。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw editor reimport <path> [options]
```

**参数：**
```
<path>                 相对于 Assets 文件夹的资源路径（例如 "XLua" 或 "Scripts/Player.cs"）
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间（毫秒）（默认：30000）
-h, --help             显示命令帮助
```

**⚠️ 重要提示：**
运行 `reimport` 后，请检查 Unity 编辑器的重新导入和编译状态。

**示例：**
```bash
# 重新导入文件夹
cd <unity-project-root> && node .unity-websocket/uw editor reimport "XLua"

# 重新导入特定资源
cd <unity-project-root> && node .unity-websocket/uw editor reimport "Scripts/Player.cs"

# 重新导入 .asmdef（重新编译特定程序集）
cd <unity-project-root> && node .unity-websocket/uw editor reimport "MyPlugin/Editor/MyPlugin.Editor.asmdef"
```

---

## cd <unity-project-root> && node .unity-websocket/uw editor execute

执行标记有 `[ExecutableMethod]` 属性的静态方法。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw editor execute <commandName> [options]
```

**参数：**
```
<commandName>          要执行的命令名称（例如 reinstall-cli）
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间（毫秒）（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 重新安装 CLI
cd <unity-project-root> && node .unity-websocket/uw editor execute reinstall-cli

# 先列出可用命令
cd <unity-project-root> && node .unity-websocket/uw editor list

# 获取 JSON 输出
cd <unity-project-root> && node .unity-websocket/uw editor execute reinstall-cli --json
```

**安全性：**
只有显式标记为 `[ExecutableMethod]` 属性的方法才能被执行。这防止了任意代码执行。

---

## cd <unity-project-root> && node .unity-websocket/uw editor list

列出所有可通过 `editor execute` 命令使用的可执行方法。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw editor list [options]
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间（毫秒）（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 列出所有可执行方法
cd <unity-project-root> && node .unity-websocket/uw editor list

# 获取 JSON 输出
cd <unity-project-root> && node .unity-websocket/uw editor list --json
```

**输出格式：**
```
✓ Found 1 executable method(s):

  reinstall-cli
    Reinstall Unity Editor Toolkit CLI
    UnityEditorToolkit.Editor.EditorServerWindow.ReinstallCLI
```

---

## 全局选项

所有命令均支持以下全局选项：

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
cd <unity-project-root> && node .unity-websocket/uw --verbose editor refresh

# 使用特定端口
cd <unity-project-root> && node .unity-websocket/uw --port 9501 editor recompile
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

默认超时时间为 30 秒（30000 毫秒）。对于可能需要更长时间的操作，请增加超时时间：

```bash
# 为复杂操作设置更长的超时时间
cd <unity-project-root> && node .unity-websocket/uw editor refresh --timeout 60000
```

### 错误处理

命令返回适当的退出代码：
- `0`: 成功
- `1`: 错误（连接失败、命令失败、参数无效等）

检查错误消息以获取有关失败的详细信息。

---

**另请参阅：**
- [QUICKSTART.md](../../QUICKSTART.md) - 快速设置和首个命令
- [COMMANDS.md](./COMMANDS.md) - 完整命令路线图
- [API_COMPATIBILITY.md](../../API_COMPATIBILITY.md) - Unity 版本兼容性
- [TEST_GUIDE.md](../../TEST_GUIDE.md) - Unity C# 服务器测试指南
