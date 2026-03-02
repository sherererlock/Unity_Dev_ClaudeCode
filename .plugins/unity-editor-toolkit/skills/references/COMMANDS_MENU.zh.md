# 菜单执行命令

用于以编程方式执行和查询 Unity Editor 菜单项的命令。

## 可用命令

| 命令 | 描述 |
|---------|-------------|
| `menu run` | 通过路径执行 Unity Editor 菜单项 |
| `menu list` | 列出可用的菜单项，支持过滤 |

---

## menu run

通过路径执行 Unity Editor 菜单项。

### 用法

```bash
cd <unity-project-root> && node .unity-websocket/uw menu run <menuPath> [options]
```

### 参数

- `<menuPath>` - 菜单项路径 (例如 "Window/General/Console", "Assets/Refresh")

### 选项

- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - WebSocket 连接超时时间（毫秒） (默认: 30000)

### 示例

```bash
# 打开控制台窗口
cd <unity-project-root> && node .unity-websocket/uw menu run "Window/General/Console"

# 刷新资源数据库 (AssetDatabase)
cd <unity-project-root> && node .unity-websocket/uw menu run "Assets/Refresh"

# 打开项目设置 (Project Settings)
cd <unity-project-root> && node .unity-websocket/uw menu run "Edit/Project Settings..."

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw menu run "Window/General/Inspector" --json
```

### 响应

**成功:**
```json
{
  "success": true,
  "menuPath": "Window/General/Console",
  "message": "Menu item 'Window/General/Console' executed successfully"
}
```

**错误:**
```json
{
  "error": "Menu item not found: Invalid/Menu/Path"
}
```

### 注意事项

- 菜单路径必须完全匹配（区分大小写）
- 某些菜单项可能需要特定的 Unity Editor 状态
- 菜单执行是同步的 - 会等待完成
- 并非所有菜单项都可以通过编程方式执行

---

## menu list

查询可用的 Unity Editor 菜单项。

### 用法

```bash
cd <unity-project-root> && node .unity-websocket/uw menu list [options]
```

### 选项

- `--filter <pattern>` - 按模式过滤菜单项（支持 * 通配符）
- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - WebSocket 连接超时时间（毫秒） (默认: 30000)

### 示例

```bash
# 列出所有已知的菜单项
cd <unity-project-root> && node .unity-websocket/uw menu list

# 按模式过滤 (通配符)
cd <unity-project-root> && node .unity-websocket/uw menu list --filter "*Window*"
cd <unity-project-root> && node .unity-websocket/uw menu list --filter "Assets/*"
cd <unity-project-root> && node .unity-websocket/uw menu list --filter "*General*"

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw menu list --filter "*Console*" --json
```

### 响应

**成功:**
```json
{
  "success": true,
  "menuItems": [
    "Window/General/Console",
    "Window/General/Inspector",
    "Window/General/Hierarchy",
    "Window/General/Project",
    "Assets/Refresh",
    "Edit/Project Settings..."
  ],
  "count": 6,
  "filter": "*General*"
}
```

### 过滤模式

- `*pattern` - 以模式结尾
- `pattern*` - 以模式开头
- `*pattern*` - 包含模式
- `pattern` - 精确匹配或包含

### 注意事项

- 返回预定义的常用 Unity 菜单项列表
- 通配符过滤不区分大小写
- 某些菜单项可能因 Unity 版本而异
- 在 `menu run` 命令中使用精确的菜单路径

---

## 常用菜单路径

### Window 菜单
```
Window/General/Console
Window/General/Inspector
Window/General/Hierarchy
Window/General/Project
Window/General/Scene
Window/General/Game
Window/Analysis/Profiler
Window/Package Manager
```

### Assets 菜单
```
Assets/Refresh
Assets/Reimport
Assets/Reimport All
Assets/Find References In Scene
```

### Edit 菜单
```
Edit/Project Settings...
Edit/Preferences...
Edit/Play
Edit/Pause
Edit/Step
```

### GameObject 菜单
```
GameObject/Create Empty
GameObject/Create Empty Child
GameObject/3D Object/Cube
GameObject/3D Object/Sphere
GameObject/Light/Directional Light
```

---

## 用例

### 1. 打开编辑器窗口
```bash
# 打开控制台进行调试
cd <unity-project-root> && node .unity-websocket/uw menu run "Window/General/Console"

# 打开性能分析器 (Profiler) 进行性能分析
cd <unity-project-root> && node .unity-websocket/uw menu run "Window/Analysis/Profiler"
```

### 2. 资源管理
```bash
# 在外部更改后刷新资源数据库 (AssetDatabase)
cd <unity-project-root> && node .unity-websocket/uw menu run "Assets/Refresh"

# 重新导入所有资源
cd <unity-project-root> && node .unity-websocket/uw menu run "Assets/Reimport All"
```

### 3. 编辑器设置
```bash
# 打开项目设置 (Project Settings)
cd <unity-project-root> && node .unity-websocket/uw menu run "Edit/Project Settings..."

# 打开首选项 (Preferences)
cd <unity-project-root> && node .unity-websocket/uw menu run "Edit/Preferences..."
```

### 4. 播放模式控制
```bash
# 进入播放模式 (Play mode)
cd <unity-project-root> && node .unity-websocket/uw menu run "Edit/Play"

# 暂停
cd <unity-project-root> && node .unity-websocket/uw menu run "Edit/Pause"

# 逐帧步进
cd <unity-project-root> && node .unity-websocket/uw menu run "Edit/Step"
```

---

## 错误处理

### 菜单未找到
```
Error: Menu item not found: Invalid/Path
```
**解决方案**: 使用 `menu list` 查找有效的菜单路径

### 菜单执行失败
```
Error: Failed to execute menu: <reason>
```
**解决方案**: 检查 Unity Editor 状态和菜单项可用性

### 连接失败
```
Error: Unity server not running
```
**解决方案**: 确保 Unity Editor WebSocket 服务器正在运行

---

## 最佳实践

1. **验证菜单路径**: 使用 `menu list` 验证正确的路径
2. **区分大小写**: 菜单路径区分大小写
3. **错误处理**: 始终检查响应是否成功/出错
4. **超时**: 为缓慢的菜单操作增加超时时间
5. **JSON 输出**: 使用 `--json` 进行程序化解析
