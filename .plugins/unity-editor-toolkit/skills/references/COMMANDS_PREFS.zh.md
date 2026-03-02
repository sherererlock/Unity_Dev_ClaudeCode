# Unity Editor Toolkit - EditorPrefs 管理命令

阅读、写入和管理 Unity EditorPrefs（持久化编辑器设置）的完整参考。

**最后更新**：2025-01-15

---

## cd <unity-project-root> && node .unity-websocket/uw prefs get

通过键获取 EditorPrefs 值。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefs get <key> [options]
```

**参数：**
```
<key>                  EditorPrefs 键名
```

**选项：**
```
-t, --type <type>      值类型：string|int|float|bool（默认值："string"）
-d, --default <value>  键不存在时的默认值
--json                 以 JSON 格式输出
-h, --help             显示命令帮助
```

**示例：**
```bash
# 获取字符串值
cd <unity-project-root> && node .unity-websocket/uw prefs get "UnityEditorToolkit.DatabaseConfig"

# 获取带有默认值的整数值
cd <unity-project-root> && node .unity-websocket/uw prefs get "MyInt" -t int -d "0"

# 获取布尔值并输出为 JSON
cd <unity-project-root> && node .unity-websocket/uw prefs get "MyBool" -t bool --json
```

---

## cd <unity-project-root> && node .unity-websocket/uw prefs set

通过键设置 EditorPrefs 值。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefs set <key> <value> [options]
```

**参数：**
```
<key>                  EditorPrefs 键名
<value>                要设置的值
```

**选项：**
```
-t, --type <type>      值类型：string|int|float|bool（默认值："string"）
--json                 以 JSON 格式输出
-h, --help             显示命令帮助
```

**示例：**
```bash
# 设置字符串值
cd <unity-project-root> && node .unity-websocket/uw prefs set "MyKey" "MyValue"

# 设置整数值
cd <unity-project-root> && node .unity-websocket/uw prefs set "MyInt" "42" -t int

# 设置布尔值
cd <unity-project-root> && node .unity-websocket/uw prefs set "MyBool" "true" -t bool

# 设置浮点值并输出为 JSON
cd <unity-project-root> && node .unity-websocket/uw prefs set "MyFloat" "3.14" -t float --json
```

---

## cd <unity-project-root> && node .unity-websocket/uw prefs delete

删除 EditorPrefs 键。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefs delete <key> [options]
```

**参数：**
```
<key>                  要删除的 EditorPrefs 键名
```

**选项：**
```
--json                 以 JSON 格式输出
-h, --help             显示命令帮助
```

**⚠️ 警告：**
删除操作不可逆。请确保键名正确。

**示例：**
```bash
# 删除键
cd <unity-project-root> && node .unity-websocket/uw prefs delete "MyKey"

# 删除并输出 JSON
cd <unity-project-root> && node .unity-websocket/uw prefs delete "OldConfig" --json
```

---

## cd <unity-project-root> && node .unity-websocket/uw prefs clear

删除所有 EditorPrefs（警告：不可逆）。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefs clear [options]
```

**选项：**
```
--force                跳过确认提示（必需）
--json                 以 JSON 格式输出
-h, --help             显示命令帮助
```

**⚠️ 危险：**
此命令将删除整个 Unity 编辑器的**所有** EditorPrefs 数据，而不仅仅是当前项目的数据。请务必极度谨慎使用。**请务必先备份您的设置！**

**示例：**
```bash
# 尝试清除（没有 --force 将显示警告）
cd <unity-project-root> && node .unity-websocket/uw prefs clear

# 强制清除所有 EditorPrefs
cd <unity-project-root> && node .unity-websocket/uw prefs clear --force

# 清除并输出 JSON
cd <unity-project-root> && node .unity-websocket/uw prefs clear --force --json
```

---

## cd <unity-project-root> && node .unity-websocket/uw prefs has

检查 EditorPrefs 键是否存在。如果键存在，还会返回其类型和值。

**用法：**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefs has <key> [options]
```

**参数：**
```
<key>                  要检查的 EditorPrefs 键名
```

**选项：**
```
--json                 以 JSON 格式输出
-h, --help             显示命令帮助
```

**输出：**
```
Key: <key-name>
Exists: Yes/No
Type: string|int|float|bool      （仅当存在时）
Value: <value>                    （仅当存在时）
```

**示例：**
```bash
# 检查键是否存在（如果存在则显示值）
cd <unity-project-root> && node .unity-websocket/uw prefs has "UnityEditorToolkit.Database.EnableWAL"
# 输出：
# Key: UnityEditorToolkit.Database.EnableWAL
# Exists: Yes
# Type: bool
# Value: true

# 检查不存在的键
cd <unity-project-root> && node .unity-websocket/uw prefs has "NonExistentKey"
# 输出：
# Key: NonExistentKey
# Exists: No

# 检查并输出 JSON
cd <unity-project-root> && node .unity-websocket/uw prefs has "MyKey" --json
```

---

## 常见用例

### 调试 DatabaseConfig

```bash
# 检查配置是否存在
cd <unity-project-root> && node .unity-websocket/uw prefs has "UnityEditorToolkit.DatabaseConfig"

# 获取当前配置（JSON 格式）
cd <unity-project-root> && node .unity-websocket/uw prefs get "UnityEditorToolkit.DatabaseConfig"

# 重置配置（删除）
cd <unity-project-root> && node .unity-websocket/uw prefs delete "UnityEditorToolkit.DatabaseConfig"
```

### 手动配置

```bash
# 设置自定义数据库路径
cd <unity-project-root> && node .unity-websocket/uw prefs set "CustomDatabasePath" "D:/MyData/db.sqlite"

# 启用特性标志
cd <unity-project-root> && node .unity-websocket/uw prefs set "FeatureEnabled" "true" -t bool

# 设置数值设置
cd <unity-project-root> && node .unity-websocket/uw prefs set "MaxConnections" "10" -t int
```

---

## EditorPrefs 存储位置

EditorPrefs 是按 Unity 编辑器存储的（而不是按项目）：

- **Windows**: `HKEY_CURRENT_USER\Software\Unity\UnityEditor\CompanyName\ProductName`
- **macOS**: `~/Library/Preferences/com.CompanyName.ProductName.plist`
- **Linux**: `~/.config/unity3d/CompanyName/ProductName/prefs`

**注意**：CompanyName（公司名称）和 ProductName（产品名称）来自您的 Unity 项目设置（Project Settings）。

---

## 另请参阅

- [连接与状态命令](./COMMANDS_CONNECTION_STATUS.md)
- [GameObject 与层级视图命令](./COMMANDS_GAMEOBJECT_HIERARCHY.md)
- [Transform 命令](./COMMANDS_TRANSFORM.md)
- [场景管理命令](./COMMANDS_SCENE.md)
- [Asset Database 与编辑器命令](./COMMANDS_EDITOR.md)
- [控制台与日志命令](./COMMANDS_CONSOLE.md)
