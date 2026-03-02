# Unity Editor Toolkit - 控制台与日志命令

Unity 控制台日志检索和清理命令的完整参考。

**最后更新**：2025-01-13

---

## cd <unity-project-root> && node .unity-websocket/uw console logs

获取 Unity 控制台日志。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw console logs [选项]
```

**选项：**
```
-n, --limit <number>   获取最近日志的数量（默认：50）
-e, --errors-only      仅显示错误和异常
-w, --warnings         在输出中包含警告
-t, --type <type>      按日志类型过滤：error, warning, log, exception, assert
-f, --filter <text>    按文本过滤日志（不区分大小写）
-v, --verbose          显示完整堆栈跟踪（默认：仅前 5 行）
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间，单位为毫秒（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 获取最近 50 条日志
cd <unity-project-root> && node .unity-websocket/uw console logs

# 仅获取错误
cd <unity-project-root> && node .unity-websocket/uw console logs --errors-only

# 获取最近 100 条日志，包含警告
cd <unity-project-root> && node .unity-websocket/uw console logs --limit 100 --warnings

# 按文本过滤日志
cd <unity-project-root> && node .unity-websocket/uw console logs --filter "player"

# 获取特定类型的日志
cd <unity-project-root> && node .unity-websocket/uw console logs --type error

# 获取包含完整堆栈跟踪的详细输出
cd <unity-project-root> && node .unity-websocket/uw console logs --verbose --errors-only

# 获取 JSON 输出
cd <unity-project-root> && node .unity-websocket/uw console logs --json
```

---

## cd <unity-project-root> && node .unity-websocket/uw console clear

清除 Unity 控制台。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw console clear [选项]
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间，单位为毫秒（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 清除控制台
cd <unity-project-root> && node .unity-websocket/uw console clear

# 获取 JSON 输出
cd <unity-project-root> && node .unity-websocket/uw console clear --json
```

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
cd <unity-project-root> && node .unity-websocket/uw --verbose console logs

# 使用特定端口
cd <unity-project-root> && node .unity-websocket/uw --port 9501 console logs
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
cd <unity-project-root> && node .unity-websocket/uw console logs --timeout 60000
```

### 错误处理

命令返回相应的退出代码：
- `0`：成功
- `1`：错误（连接失败、命令失败、无效参数等）

检查错误消息以获取故障详情。

---

**另请参阅：**
- [QUICKSTART.md](../../QUICKSTART.md) - 快速设置和首个命令
- [COMMANDS.md](./COMMANDS.md) - 完整命令路线图
- [API_COMPATIBILITY.md](../../API_COMPATIBILITY.md) - Unity 版本兼容性
- [TEST_GUIDE.md](../../TEST_GUIDE.md) - Unity C# 服务器测试指南
