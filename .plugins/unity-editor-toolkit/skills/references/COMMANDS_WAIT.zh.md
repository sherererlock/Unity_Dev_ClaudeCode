# 等待命令参考 (Wait Commands Reference)

等待各种 Unity 条件满足后再继续执行。

## 命令格式 (Command Format)

```bash
cd <unity-project-root>
node .unity-websocket/uw.js wait <subcommand> [options]
```

## 子命令 (Subcommands)

### wait compile

等待 Unity 编译完成。

**用法 (Usage):**
```bash
node .unity-websocket/uw.js wait compile [options]
```

**选项 (Options):**
- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - 超时时间，单位毫秒（默认：300000 = 5 分钟）

**示例 (Examples):**
```bash
# 等待编译完成
cd <unity-project-root> && node .unity-websocket/uw.js wait compile

# 自定义超时时间（30 秒）
cd <unity-project-root> && node .unity-websocket/uw.js wait compile --timeout 30000
```

**使用场景 (Use Cases):**
- 在修改代码后等待
- 确保在运行测试前编译已完成
- 顺序自动化工作流

---

### wait playmode

等待特定的播放模式状态。

**用法 (Usage):**
```bash
node .unity-websocket/uw.js wait playmode <state> [options]
```

**参数 (Arguments):**
- `<state>` - 目标状态：`enter`（进入）、`exit`（退出）或 `pause`（暂停）

**选项 (Options):**
- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - 超时时间，单位毫秒（默认：300000 = 5 分钟）

**示例 (Examples):**
```bash
# 等待播放模式开始
cd <unity-project-root> && node .unity-websocket/uw.js wait playmode enter

# 等待播放模式退出
cd <unity-project-root> && node .unity-websocket/uw.js wait playmode exit

# 等待暂停
cd <unity-project-root> && node .unity-websocket/uw.js wait playmode pause
```

**使用场景 (Use Cases):**
- 与播放模式测试同步
- 在发送命令前等待游戏开始
- 需要播放模式的自动化工作流

---

### wait sleep

休眠指定的持续时间。

**用法 (Usage):**
```bash
node .unity-websocket/uw.js wait sleep <seconds> [options]
```

**参数 (Arguments):**
- `<seconds>` - 持续时间，单位秒（可以是小数，例如 0.5 表示半秒）

**选项 (Options):**
- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - 超时时间，单位毫秒（必须大于休眠持续时间）

**示例 (Examples):**
```bash
# 休眠 2 秒
cd <unity-project-root> && node .unity-websocket/uw.js wait sleep 2

# 休眠半秒
cd <unity-project-root> && node .unity-websocket/uw.js wait sleep 0.5

# 带自定义超时的休眠
cd <unity-project-root> && node .unity-websocket/uw.js wait sleep 5 --timeout 10000
```

**使用场景 (Use Cases):**
- 在命令之间添加延迟
- 等待 UI 动画
- 自动化中的速率限制

---

### wait scene

等待场景加载完成（仅限播放模式）。

**用法 (Usage):**
```bash
node .unity-websocket/uw.js wait scene [options]
```

**选项 (Options):**
- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - 超时时间，单位毫秒（默认：300000 = 5 分钟）

**示例 (Examples):**
```bash
# 等待场景加载
cd <unity-project-root> && node .unity-websocket/uw.js wait scene
```

**要求 (Requirements):**
- Unity 必须处于播放模式
- 场景必须正在加载或刚刚加载

**使用场景 (Use Cases):**
- 加载新场景后等待
- 确保在运行测试前场景已就绪
- 顺序场景加载工作流

---

## 重要注意事项 (Important Notes)

### 延迟响应模型 (Delayed Response Model)

等待命令使用**延迟响应**模型：
1. 命令发送给 Unity
2. Unity 注册等待条件
3. 连接保持打开状态
4. Unity 每帧监控条件
5. 当条件满足或超时时发送响应

### 超时行为 (Timeout Behavior)

- 默认超时：**300 秒（5 分钟）**
- 如果在超时前条件满足：成功响应
- 如果发生超时：带有超时消息的错误响应
- 建议：将超时设置得比预期等待时间长

### 域重载处理 (Domain Reload Handling)

如果 Unity 在等待期间编译脚本（域重载）：
- 所有挂起的等待请求都会**自动取消**
- 客户端收到错误："Script compilation started, request cancelled"（脚本编译开始，请求已取消）
- 这可以防止孤立的等待请求

### 服务器停止处理 (Server Stop Handling)

如果 Unity WebSocket 服务器在等待期间停止：
- 所有挂起的等待请求都会**自动取消**
- 客户端收到错误："Server stopping"（服务器正在停止）

---

## JSON 输出格式 (JSON Output Format)

当使用 `--json` 标志时：

**成功 (Success):**
```json
{
  "success": true,
  "type": "sleep",
  "seconds": 2,
  "message": "Slept for 2 seconds"
}
```

**错误 (Error):**
```json
{
  "jsonrpc": "2.0",
  "id": "...",
  "error": {
    "code": -32000,
    "message": "Wait condition timed out after 300 seconds"
  }
}
```

---

## 常见工作流 (Common Workflows)

### 等待编译然后执行 (Wait for Compilation then Execute)

```bash
# 等待编译，然后刷新 AssetDatabase
cd <unity-project-root>
node .unity-websocket/uw.js wait compile
node .unity-websocket/uw.js editor refresh
```

### 顺序场景加载 (Sequential Scene Loading)

```bash
# 加载场景，等待它，然后执行操作
cd <unity-project-root>
node .unity-websocket/uw.js scene load MainMenu
node .unity-websocket/uw.js wait scene
node .unity-websocket/uw.js gameobject find "StartButton"
```

### 速率受限的自动化 (Rate-Limited Automation)

```bash
# 执行操作，等待，重复
cd <unity-project-root>
for i in {1..5}; do
  node .unity-websocket/uw.js console logs --limit 10
  node .unity-websocket/uw.js wait sleep 1
done
```
