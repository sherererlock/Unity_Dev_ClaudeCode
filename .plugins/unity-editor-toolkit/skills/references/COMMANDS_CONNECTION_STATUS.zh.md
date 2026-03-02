# Unity Editor Toolkit - 连接与状态命令

连接与状态命令的完整参考。

**最后更新**：2025-01-13

---

## cd <unity-project-root> && node .unity-websocket/uw status

检查 Unity WebSocket 连接状态。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw status
```

**选项：**
```
-p, --port <number>    Unity WebSocket 端口（默认：从状态文件自动检测）
-v, --verbose          启用详细日志记录
-h, --help             显示命令帮助
```

**示例：**
```bash
# 检查默认连接
cd <unity-project-root> && node .unity-websocket/uw status

# 检查特定端口
cd <unity-project-root> && node .unity-websocket/uw --port 9500 status
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
cd <unity-project-root> && node .unity-websocket/uw --verbose status

# 使用特定端口
cd <unity-project-root> && node .unity-websocket/uw --port 9501 status
```

---

## 注意事项

### 端口自动检测

Unity Editor Toolkit CLI 会通过读取 Unity 项目目录中的 `.unity-websocket/server-status.json` 自动检测 Unity WebSocket 服务器端口。仅在以下情况需要指定 `--port`：
- 运行多个 Unity Editor 实例时
- 服务器使用非默认端口范围时

### JSON 输出

所有命令均支持 `--json` 标志以输出机器可读格式。适用于：
- CI/CD 管道
- 自动化脚本
- 与其他工具集成

### 超时配置

默认超时时间为 30 秒（30000ms）。对于可能需要更长时间的操作，请增加超时时间：

```bash
# 针对复杂操作延长超时时间
cd <unity-project-root> && node .unity-websocket/uw status --timeout 60000
```

### 错误处理

命令返回相应的退出代码：
- `0`：成功
- `1`：错误（连接失败、命令失败、参数无效等）

查看错误消息以了解失败详情。

---

**另请参阅：**
- [QUICKSTART.md](../../QUICKSTART.md) - 快速设置与首次命令
- [COMMANDS.md](./COMMANDS.md) - 完整命令路线图
- [API_COMPATIBILITY.md](../../API_COMPATIBILITY.md) - Unity 版本兼容性
- [TEST_GUIDE.md](../../TEST_GUIDE.md) - Unity C# 服务器测试指南
