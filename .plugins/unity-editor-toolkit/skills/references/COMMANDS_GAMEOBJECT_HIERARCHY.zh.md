# Unity 编辑器工具包 - 游戏对象与层级命令

游戏对象操作和层级查询命令的完整参考。

**最后更新**：2025-01-13

---

## cd <unity-project-root> && node .unity-websocket/uw go find

按名称或层级路径查找游戏对象。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw go find <name> [options]
```

**参数：**
```
<name>                 游戏对象名称或路径（例如 "Player" 或 "Environment/Trees/Oak"）
```

**选项：**
```
-c, --with-components  在输出中包含组件列表
--with-children        包含子对象层级
--full                 包含所有详细信息（组件 + 子对象）
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间（毫秒）（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 按名称查找游戏对象
cd <unity-project-root> && node .unity-websocket/uw go find "Player"

# 使用完整层级路径查找
cd <unity-project-root> && node .unity-websocket/uw go find "Environment/Terrain/Trees"

# 包含组件信息
cd <unity-project-root> && node .unity-websocket/uw go find "Player" --with-components

# 获取 JSON 格式的所有详细信息
cd <unity-project-root> && node .unity-websocket/uw go find "Enemy" --full --json
```

---

## cd <unity-project-root> && node .unity-websocket/uw go create

创建新的游戏对象。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw go create <name> [options]
```

**参数：**
```
<name>                 新游戏对象的名称
```

**选项：**
```
-p, --parent <parent>  父游戏对象名称或路径
--primitive <type>     创建基本体：cube, sphere, cylinder, capsule, plane, quad
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间（毫秒）（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 创建空游戏对象
cd <unity-project-root> && node .unity-websocket/uw go create "NewObject"

# 创建带父对象的游戏对象
cd <unity-project-root> && node .unity-websocket/uw go create "Child" --parent "Parent"

# 创建基本体
cd <unity-project-root> && node .unity-websocket/uw go create "MyCube" --primitive cube

# 创建嵌套对象
cd <unity-project-root> && node .unity-websocket/uw go create "Enemy" --parent "Enemies/Group1"
```

---

## cd <unity-project-root> && node .unity-websocket/uw go destroy

销毁游戏对象。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw go destroy <name> [options]
```

**参数：**
```
<name>                 要销毁的游戏对象名称或路径
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间（毫秒）（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 销毁游戏对象
cd <unity-project-root> && node .unity-websocket/uw go destroy "OldObject"

# 销毁嵌套的游戏对象
cd <unity-project-root> && node .unity-websocket/uw go destroy "Enemies/Enemy1"

# 获取 JSON 响应
cd <unity-project-root> && node .unity-websocket/uw go destroy "Temp" --json
```

---

## cd <unity-project-root> && node .unity-websocket/uw go set-active

设置游戏对象的激活状态。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw go set-active <name> <active> [options]
```

**参数：**
```
<name>                 游戏对象名称或路径
<active>               激活状态：true 或 false
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间（毫秒）（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 激活游戏对象
cd <unity-project-root> && node .unity-websocket/uw go set-active "Player" true

# 停用游戏对象
cd <unity-project-root> && node .unity-websocket/uw go set-active "Enemy" false

# 设置嵌套游戏对象的状态
cd <unity-project-root> && node .unity-websocket/uw go set-active "UI/Menu/Settings" false
```

---

## cd <unity-project-root> && node .unity-websocket/uw go set-parent

设置或移除游戏对象的父对象。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw go set-parent <name> [parent] [options]
```

**参数：**
```
<name>                 游戏对象名称或路径
[parent]               父游戏对象名称（省略以移除父对象）
```

**选项：**
```
--world-position-stays <bool>  保持世界坐标位置（默认：true）
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间（毫秒）（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 设置父对象（将 "Weapon" 附加到 "Player"）
cd <unity-project-root> && node .unity-websocket/uw go set-parent "Weapon" "Player"

# 移除父对象（分离到根目录）
cd <unity-project-root> && node .unity-websocket/uw go set-parent "Weapon"

# 设置父对象但不保持世界坐标位置
cd <unity-project-root> && node .unity-websocket/uw go set-parent "Child" "Parent" --world-position-stays false

# 重新设置嵌套对象的父对象
cd <unity-project-root> && node .unity-websocket/uw go set-parent "UI/Menu" "Canvas"
```

---

## cd <unity-project-root> && node .unity-websocket/uw go get-parent

获取游戏对象的父对象信息。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw go get-parent <name> [options]
```

**参数：**
```
<name>                 游戏对象名称或路径
```

**选项：**
```
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间（毫秒）（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 获取父对象信息
cd <unity-project-root> && node .unity-websocket/uw go get-parent "Weapon"

# 检查对象是否有父对象
cd <unity-project-root> && node .unity-websocket/uw go get-parent "Player" --json

# 获取嵌套对象的父对象
cd <unity-project-root> && node .unity-websocket/uw go get-parent "Enemies/Enemy1"
```

---

## cd <unity-project-root> && node .unity-websocket/uw go get-children

获取游戏对象的子对象。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw go get-children <name> [options]
```

**参数：**
```
<name>                 游戏对象名称或路径
```

**选项：**
```
-r, --recursive        获取所有后代（不仅是直接子对象）
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间（毫秒）（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 获取直接子对象
cd <unity-project-root> && node .unity-websocket/uw go get-children "Player"

# 递归获取所有后代
cd <unity-project-root> && node .unity-websocket/uw go get-children "Player" --recursive

# 获取 JSON 输出的子对象
cd <unity-project-root> && node .unity-websocket/uw go get-children "Canvas" --json

# 统计嵌套对象
cd <unity-project-root> && node .unity-websocket/uw go get-children "Environment" --recursive --json
```

---

## cd <unity-project-root> && node .unity-websocket/uw hierarchy

使用树状可视化查询 Unity 游戏对象层级。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw hierarchy [options]
```

**选项：**
```
-r, --root-only        仅显示根游戏对象（无子对象）
-i, --include-inactive 在输出中包含非激活的游戏对象
-a, --active-only      仅显示激活的游戏对象（与 -i 相反）
-d, --depth <n>        限制层级深度（例如 2 表示 2 层）
-f, --filter <name>    按名称过滤游戏对象（不区分大小写）
-c, --with-components  包含每个游戏对象的组件信息
--json                 以 JSON 格式输出
--timeout <ms>         连接超时时间（毫秒）（默认：30000）
-h, --help             显示命令帮助
```

**示例：**
```bash
# 查看完整层级
cd <unity-project-root> && node .unity-websocket/uw hierarchy

# 仅显示根游戏对象
cd <unity-project-root> && node .unity-websocket/uw hierarchy --root-only

# 限制深度为 2 层
cd <unity-project-root> && node .unity-websocket/uw hierarchy --depth 2

# 按名称过滤
cd <unity-project-root> && node .unity-websocket/uw hierarchy --filter "enemy"

# 仅显示带组件的激活游戏对象
cd <unity-project-root> && node .unity-websocket/uw hierarchy --active-only --with-components

# 获取 JSON 输出
cd <unity-project-root> && node .unity-websocket/uw hierarchy --json
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
cd <unity-project-root> && node .unity-websocket/uw --verbose hierarchy

# 使用特定端口
cd <unity-project-root> && node .unity-websocket/uw --port 9501 go find "Player"
```

---

## 注意事项

### 端口自动检测

Unity Editor Toolkit CLI 会通过读取 Unity 项目目录中的 `.unity-websocket/server-status.json` 自动检测 Unity WebSocket 服务器端口。仅在以下情况下需要指定 `--port`：
- 运行多个 Unity 编辑器实例
- 服务器使用非默认端口范围

### JSON 输出

所有命令均支持 `--json` 标志以进行机器可读输出。适用于：
- CI/CD 管道
- 自动化脚本
- 与其他工具集成

### 超时配置

默认超时时间为 30 秒（30000 毫秒）。对于可能耗时较长的操作，请增加超时时间：

```bash
# 复杂操作的较长超时时间
cd <unity-project-root> && node .unity-websocket/uw hierarchy --timeout 60000
```

### 错误处理

命令返回相应的退出代码：
- `0`: 成功
- `1`: 错误（连接失败、命令失败、参数无效等）

检查错误消息以获取失败详情。

---

**另请参阅：**
- [QUICKSTART.md](../../QUICKSTART.md) - 快速设置和首个命令
- [COMMANDS.md](./COMMANDS.md) - 完整命令路线图
- [API_COMPATIBILITY.md](../../API_COMPATIBILITY.md) - Unity 版本兼容性
- [TEST_GUIDE.md](../../TEST_GUIDE.md) - Unity C# 服务器测试指南
