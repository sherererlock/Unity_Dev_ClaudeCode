# 链式命令参考 (Chain Commands Reference)

顺序执行多个 Unity 命令并进行错误处理。

## 命令格式 (Command Format)

```bash
cd <unity-project-root>
node .unity-websocket/uw.js chain <subcommand> [options]
```

## 子命令 (Subcommands)

### chain execute

从 JSON 文件执行命令。

**用法 (Usage):**
```bash
node .unity-websocket/uw.js chain execute <file> [options]
```

**参数 (Arguments):**
- `<file>` - 包含命令的 JSON 文件路径

**选项 (Options):**
- `--json` - 以 JSON 格式输出
- `--stop-on-error` - 遇到首个错误时停止 (默认: true)
- `--continue-on-error` - 即使命令失败也继续执行
- `--timeout <ms>` - 超时时间，单位毫秒 (默认: 300000 = 5 分钟)

**JSON 文件格式 (JSON File Format):**
```json
[
  {
    "method": "Editor.Refresh",
    "parameters": null
  },
  {
    "method": "GameObject.Create",
    "parameters": {
      "name": "TestObject"
    }
  },
  {
    "method": "Console.Clear"
  }
]
```

或者使用包装器:
```json
{
  "commands": [
    { "method": "Editor.Refresh" },
    { "method": "Console.Clear" }
  ]
}
```

**示例 (Examples):**
```bash
# 从文件执行命令
cd <unity-project-root> && node .unity-websocket/uw.js chain execute commands.json

# 遇到错误继续执行
cd <unity-project-root> && node .unity-websocket/uw.js chain execute commands.json --continue-on-error

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw.js chain execute commands.json --json
```

---

### chain exec

内联执行命令 (不使用 JSON 文件)。

**用法 (Usage):**
```bash
node .unity-websocket/uw.js chain exec <commands...> [options]
```

**参数 (Arguments):**
- `<commands...>` - 一个或多个命令，格式为: `method:param1=value1,param2=value2`

**选项 (Options):**
- `--json` - 以 JSON 格式输出
- `--stop-on-error` - 遇到首个错误时停止 (默认: true)
- `--continue-on-error` - 即使命令失败也继续执行
- `--timeout <ms>` - 超时时间，单位毫秒 (默认: 300000 = 5 分钟)

**命令格式 (Command Format):**
- 简单: `"Editor.Refresh"`
- 带参数: `"GameObject.Create:name=Test"`
- 多参数: `"GameObject.SetActive:instanceId=123,active=true"`

**参数解析 (Parameter Parsing):**
- 字符串: `name=MyObject`
- 数字: `instanceId=123`
- 布尔值: `active=true` 或 `active=false`

**示例 (Examples):**
```bash
# 简单命令
cd <unity-project-root> && node .unity-websocket/uw.js chain exec "Editor.Refresh" "Console.Clear"

# 带参数的命令
cd <unity-project-root> && node .unity-websocket/uw.js chain exec \
  "GameObject.Create:name=Player" \
  "GameObject.SetActive:instanceId=123,active=true"

# 遇到错误继续执行
cd <unity-project-root> && node .unity-websocket/uw.js chain exec \
  "Editor.Refresh" \
  "GameObject.Find:path=InvalidPath" \
  "Console.Clear" \
  --continue-on-error
```

---

## 重要说明 (Important Notes)

### 支持的命令 (Supported Commands)

Chain 仅支持 **立即响应 (immediate response)** 命令。以下命令 **不支持**:

❌ **等待命令 (Wait commands)** (延迟响应):
- `wait compile`
- `wait playmode`
- `wait sleep`
- `wait scene`

✅ **所有其他命令** (立即响应):
- GameObject 命令
- Transform 命令
- Scene 命令
- Console 命令
- Editor 命令
- Prefs 命令

**等待命令的替代方案 (Workaround for Wait):**
```bash
# 不要在链式命令中使用 Wait 命令:
cd <unity-project-root>
node .unity-websocket/uw.js wait compile
node .unity-websocket/uw.js chain exec "Editor.Refresh" "Console.Clear"
```

### 错误处理 (Error Handling)

**遇到错误停止 (默认):**
```bash
# 在第一次失败时停止
node .unity-websocket/uw.js chain exec "Editor.Refresh" "Invalid.Command" "Console.Clear"
# 结果: Refresh 成功, Invalid.Command 失败, Console.Clear 被跳过
```

**遇到错误继续:**
```bash
# 尽管有失败仍继续
node .unity-websocket/uw.js chain exec "Editor.Refresh" "Invalid.Command" "Console.Clear" --continue-on-error
# 结果: Refresh 成功, Invalid.Command 失败, Console.Clear 成功
```

### 超时行为 (Timeout Behavior)

- 默认超时: 整个链 **300 秒 (5 分钟)**
- 每个命令有其自己的执行时间
- 总耗时会在响应中报告

---

## 输出格式 (Output Format)

### CLI 输出 (CLI Output)

**成功 (Success):**
```
✓ Chain execution completed
  Total commands: 3
  Executed: 3
  Total time: 0.015s

  [1] ✓ Editor.Refresh (0.010s)
  [2] ✓ GameObject.Create (0.003s)
  [3] ✓ Console.Clear (0.002s)
```

**包含错误 (With Errors):**
```
✓ Chain execution completed
  Total commands: 3
  Executed: 2
  Total time: 0.012s

  [1] ✓ Editor.Refresh (0.010s)
  [2] ✗ GameObject.Find (0.002s)
      Error: GameObject not found: InvalidPath
```

### JSON 输出 (JSON Output)

**成功 (Success):**
```json
{
  "success": true,
  "totalCommands": 3,
  "executedCommands": 3,
  "totalElapsed": 0.015,
  "results": [
    {
      "index": 0,
      "method": "Editor.Refresh",
      "success": true,
      "result": { "success": true, "message": "AssetDatabase refreshed" },
      "elapsed": 0.010
    },
    {
      "index": 1,
      "method": "GameObject.Create",
      "success": true,
      "result": { "instanceId": 12345, "name": "TestObject" },
      "elapsed": 0.003
    },
    {
      "index": 2,
      "method": "Console.Clear",
      "success": true,
      "result": { "success": true, "cleared": 10 },
      "elapsed": 0.002
    }
  ]
}
```

**包含错误 (With Errors):**
```json
{
  "success": true,
  "totalCommands": 3,
  "executedCommands": 2,
  "totalElapsed": 0.012,
  "results": [
    {
      "index": 0,
      "method": "Editor.Refresh",
      "success": true,
      "result": { "success": true, "message": "AssetDatabase refreshed" },
      "elapsed": 0.010
    },
    {
      "index": 1,
      "method": "GameObject.Find",
      "success": false,
      "error": "GameObject not found: InvalidPath",
      "elapsed": 0.002
    }
  ]
}
```

---

## 常见工作流 (Common Workflows)

### 清理工作流 (Cleanup Workflow)

```json
{
  "commands": [
    { "method": "Console.Clear" },
    { "method": "Editor.Refresh" },
    { "method": "Scene.Load", "parameters": { "name": "MainScene" } }
  ]
}
```

```bash
cd <unity-project-root> && node .unity-websocket/uw.js chain execute cleanup.json
```

### GameObject 批量创建 (GameObject Batch Creation)

```bash
cd <unity-project-root> && node .unity-websocket/uw.js chain exec \
  "GameObject.Create:name=Player" \
  "GameObject.Create:name=Enemy" \
  "GameObject.Create:name=Pickup" \
  "Console.Clear"
```

### 容错清理 (Error-Tolerant Cleanup)

```json
{
  "commands": [
    { "method": "GameObject.Destroy", "parameters": { "path": "OldObject1" } },
    { "method": "GameObject.Destroy", "parameters": { "path": "OldObject2" } },
    { "method": "GameObject.Destroy", "parameters": { "path": "OldObject3" } },
    { "method": "Console.Clear" }
  ]
}
```

```bash
# 即使某些对象不存在也继续执行
cd <unity-project-root> && node .unity-websocket/uw.js chain execute cleanup.json --continue-on-error
```

### CI/CD 管道 (CI/CD Pipeline)

```bash
#!/bin/bash
cd /path/to/unity/project

# 清理
node .unity-websocket/uw.js chain exec "Console.Clear" "Editor.Refresh"

# 等待编译
node .unity-websocket/uw.js wait compile

# 运行测试 (示例)
node .unity-websocket/uw.js chain exec \
  "Scene.Load:name=TestScene" \
  "GameObject.Find:path=TestRunner" \
  "Console.Clear"
```
