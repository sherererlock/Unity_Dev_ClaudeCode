# Unity Editor Toolkit - Transform 命令

Transform 操作命令的完整参考。

**最后更新**：2025-01-13

---

## cd <unity-project-root> && node .unity-websocket/uw tf get

获取 Transform 信息（位置、旋转、缩放）。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw tf get <name> [options]
```

**参数：**
```
<name>                 GameObject 名称或路径
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间，单位为毫秒（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 获取 Transform 信息
cd <unity-project-root> && node .unity-websocket/uw tf get "Player"

# 获取嵌套 GameObject 的 Transform
cd <unity-project-root> && node .unity-websocket/uw tf get "Environment/Trees/Oak"

# 获取 JSON 输出
cd <unity-project-root> && node .unity-websocket/uw tf get "Enemy" --json
```

---

## cd <unity-project-root> && node .unity-websocket/uw tf set-position

设置 Transform 位置。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw tf set-position <name> <position> [options]
```

**参数：**
```
<name>                 GameObject 名称或路径
<position>             位置，格式为 "x,y,z"（例如："0,5,10"）
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间，单位为毫秒（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 设置位置
cd <unity-project-root> && node .unity-websocket/uw tf set-position "Player" "0,5,10"

# 设置带有负值的位置
cd <unity-project-root> && node .unity-websocket/uw tf set-position "Enemy" "-5.5,0,3.2"

# 设置嵌套 GameObject 的位置
cd <unity-project-root> && node .unity-websocket/uw tf set-position "UI/Menu/Button" "100,50,0"
```

---

## cd <unity-project-root> && node .unity-websocket/uw tf set-rotation

使用欧拉角设置 Transform 旋转。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw tf set-rotation <name> <rotation> [options]
```

**参数：**
```
<name>                 GameObject 名称或路径
<rotation>             欧拉角，格式为 "x,y,z"，单位为度（例如："0,90,0"）
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间，单位为毫秒（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 在 Y 轴上旋转 90 度
cd <unity-project-root> && node .unity-websocket/uw tf set-rotation "Player" "0,90,0"

# 设置所有轴的旋转
cd <unity-project-root> && node .unity-websocket/uw tf set-rotation "Camera" "30,45,0"

# 重置旋转
cd <unity-project-root> && node .unity-websocket/uw tf set-rotation "Enemy" "0,0,0"
```

---

## cd <unity-project-root> && node .unity-websocket/uw tf set-scale

设置 Transform 缩放。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw tf set-scale <name> <scale> [options]
```

**参数：**
```
<name>                 GameObject 名称或路径
<scale>                缩放，格式为 "x,y,z"（例如："2,2,2"）
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间，单位为毫秒（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 尺寸加倍
cd <unity-project-root> && node .unity-websocket/uw tf set-scale "Cube" "2,2,2"

# 仅在 Y 轴上缩放
cd <unity-project-root> && node .unity-websocket/uw tf set-scale "Platform" "1,0.5,1"

# 非均匀缩放
cd <unity-project-root> && node .unity-websocket/uw tf set-scale "Wall" "5,3,0.2"
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
cd <unity-project-root> && node .unity-websocket/uw --verbose tf get "Player"

# 使用特定端口
cd <unity-project-root> && node .unity-websocket/uw --port 9501 tf set-position "Enemy" "0,0,0"
```

---

## 注意事项

### 端口自动检测

Unity Editor Toolkit CLI 通过读取 Unity 项目目录中的 `.unity-websocket/server-status.json` 自动检测 Unity WebSocket 服务器端口。仅在以下情况下需要指定 `--port`：
- 运行多个 Unity Editor 实例
- 服务器使用非默认端口范围

### JSON 输出

所有命令都支持 `--json` 标志以进行机器可读输出。适用于：
- CI/CD 管道
- 自动化脚本
- 与其他工具集成

### 超时配置

默认超时时间为 30 秒（30000毫秒）。对于可能需要更长时间的操作，请增加此值：

```bash
# 为复杂操作设置更长的超时时间
cd <unity-project-root> && node .unity-websocket/uw tf get "Player" --timeout 60000
```

### 错误处理

命令返回相应的退出代码：
- `0`：成功
- `1`：错误（连接失败、命令失败、参数无效等）

检查错误消息以获取失败详情。

---

**另请参阅：**
- [QUICKSTART.md](../../QUICKSTART.md) - 快速设置和首个命令
- [COMMANDS.md](./COMMANDS.md) - 完整命令路线图
- [API_COMPATIBILITY.md](../../API_COMPATIBILITY.md) - Unity 版本兼容性
- [TEST_GUIDE.md](../../TEST_GUIDE.md) - Unity C# 服务器测试指南
