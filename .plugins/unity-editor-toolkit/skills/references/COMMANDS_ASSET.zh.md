# 资产管理命令 (ScriptableObject)

用于创建、查询和修改 Unity ScriptableObject 资产的命令。完全支持数组/列表以及所有字段类型。

## 可用命令

| 命令 | 描述 |
|---------|-------------|
| `asset list-types` | 列出可用的 ScriptableObject 类型 |
| `asset create-so` | 创建新的 ScriptableObject 资产 |
| `asset get-fields` | 获取 ScriptableObject 的字段（支持数组展开） |
| `asset set-field` | 设置字段值（支持数组索引表示法） |
| `asset inspect` | 检查 ScriptableObject 及其完整元数据 |
| `asset add-element` | 向数组/列表字段添加元素 |
| `asset remove-element` | 从数组/列表字段移除元素 |
| `asset get-element` | 获取特定的数组元素 |
| `asset clear-array` | 清除数组/列表字段中的所有元素 |

---

## asset list-types

查询项目中可用的 ScriptableObject 类型。

### 用法

```bash
cd <unity-project-root> && node .unity-websocket/uw asset list-types [options]
```

### 选项

- `--filter <pattern>` - 按模式筛选类型（支持 * 通配符）
- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - WebSocket 连接超时时间（默认：30000）

### 示例

```bash
# 列出所有 ScriptableObject 类型
cd <unity-project-root> && node .unity-websocket/uw asset list-types

# 按模式筛选
cd <unity-project-root> && node .unity-websocket/uw asset list-types --filter "*Config*"
cd <unity-project-root> && node .unity-websocket/uw asset list-types --filter "Game*"

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw asset list-types --json
```

---

## asset create-so

创建新的 ScriptableObject 资产。

### 用法

```bash
cd <unity-project-root> && node .unity-websocket/uw asset create-so <typeName> <path> [options]
```

### 参数

- `<typeName>` - ScriptableObject 类型名称（全名或短名）
- `<path>` - 资产路径（例如 "Assets/Config/game.asset"）

### 选项

- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - WebSocket 连接超时时间（默认：30000）

### 示例

```bash
# 按短名创建 ScriptableObject
cd <unity-project-root> && node .unity-websocket/uw asset create-so GameConfig "Assets/Config/game.asset"

# 按全名创建 ScriptableObject
cd <unity-project-root> && node .unity-websocket/uw asset create-so "MyGame.GameConfig" "Config/settings"

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw asset create-so ItemData "Items/sword.asset" --json
```

### 注意事项

- 如果路径中没有 "Assets/" 前缀，会自动添加
- 如果路径中没有 ".asset" 扩展名，会自动添加
- 父目录会自动创建
- 类型名称可以是短名（如 "GameConfig"）或全名（如 "MyGame.GameConfig"）

---

## asset get-fields

查询 ScriptableObject 的字段。支持数组扩展选项。

### 用法

```bash
cd <unity-project-root> && node .unity-websocket/uw asset get-fields <path> [options]
```

### 参数

- `<path>` - 资产路径（例如 "Assets/Config/game.asset"）

### 选项

- `--expand` - 展开数组/列表元素
- `--depth <n>` - 嵌套展开的最大深度（默认：3）
- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - WebSocket 连接超时时间（默认：30000）

### 示例

```bash
# 获取基本字段
cd <unity-project-root> && node .unity-websocket/uw asset get-fields "Assets/Config/game.asset"

# 展开数组
cd <unity-project-root> && node .unity-websocket/uw asset get-fields "Config/game.asset" --expand

# 自定义深度展开
cd <unity-project-root> && node .unity-websocket/uw asset get-fields "Config/game.asset" --expand --depth 5

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw asset get-fields "Config/game.asset" --json
```

### 响应 (未使用 --expand)

```
✓ Fields for MyGame.GameConfig (Assets/Config/game.asset):

  Player Health (playerHealth)
    Type: Integer
    Value: 100

  Items (items)
    Type: Generic [Array: 3]
    Element Type: Generic
    Value: [Array: 3 elements]
```

### 响应 (使用 --expand)

```
✓ Fields for MyGame.GameConfig (Assets/Config/game.asset):

  Items (items)
    Type: Generic [Array: 3]
    Element Type: Generic
    Value: [Array: 3 elements]
      Element 0 ([0])
        Type: Generic
        Value: [Object: 2 fields]
          Item Name (itemName)
            Type: String
            Value: Sword
          Item Count (itemCount)
            Type: Integer
            Value: 5
```

---

## asset set-field

设置 ScriptableObject 的字段值。支持数组索引表示法。

### 用法

```bash
cd <unity-project-root> && node .unity-websocket/uw asset set-field <path> <fieldName> <value> [options]
```

### 参数

- `<path>` - 资产路径
- `<fieldName>` - 字段名称或路径（支持数组索引：`items[0]`, `items[2].name`）
- `<value>` - 新值（格式取决于字段类型）

### 选项

- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - WebSocket 连接超时时间（默认：30000）

### 字段名称模式

```bash
# 简单字段
"playerHealth"

# 数组元素（按索引）
"items[0]"

# 数组元素中的嵌套字段
"items[0].itemName"
"items[2].count"

# 多层嵌套
"enemies[1].stats.health"
```

### 各类型的数值格式

| 类型 | 格式 | 示例 |
|------|--------|---------|
| Integer | `"10"` | `"100"` |
| Float | `"1.5"` | `"3.14"` |
| String | `"text"` | `"Sword"` |
| Boolean | `"true"` 或 `"false"` | `"true"` |
| Enum | `"EnumValue"` 或 `"0"` | `"Active"` |
| Vector2 | `"x,y"` | `"1.0,2.0"` |
| Vector3 | `"x,y,z"` | `"1.0,2.0,3.0"` |
| Vector4 | `"x,y,z,w"` | `"1,2,3,4"` |
| Color | `"r,g,b,a"` | `"1,0,0,1"` |
| Quaternion | `"x,y,z,w"` | `"0,0,0,1"` |
| Rect | `"x,y,width,height"` | `"0,0,100,50"` |
| Bounds | `"centerX,Y,Z,sizeX,Y,Z"` | `"0,0,0,10,10,10"` |
| Vector2Int | `"x,y"` | `"10,20"` |
| Vector3Int | `"x,y,z"` | `"1,2,3"` |
| RectInt | `"x,y,width,height"` | `"0,0,100,50"` |
| BoundsInt | `"posX,Y,Z,sizeX,Y,Z"` | `"0,0,0,5,5,5"` |
| AnimationCurve | `"time:value;time:value"` | `"0:0;1:1"` |
| ObjectReference | `"Assets/path/to/asset"` | `"Assets/Prefabs/Player.prefab"` |
| ArraySize | `"3"` | 更改数组大小 |

### 示例

```bash
# 设置简单字段
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Config/game.asset" "playerHealth" "100"

# 设置数组元素
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Config/game.asset" "items[0]" "NewValue"

# 设置数组中的嵌套字段
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Config/game.asset" "items[0].itemName" "Sword"
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Config/game.asset" "items[2].count" "10"

# 设置 Vector3
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Config/game.asset" "spawnPosition" "10.5,0,5.2"

# 设置 Color
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Config/game.asset" "tintColor" "1,0,0,1"

# 设置 ObjectReference
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Config/game.asset" "playerPrefab" "Assets/Prefabs/Player.prefab"

# 更改数组大小
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Config/game.asset" "items.Array.size" "5"
```

---

## asset inspect

查询 ScriptableObject 的完整元数据和字段。

### 用法

```bash
cd <unity-project-root> && node .unity-websocket/uw asset inspect <path> [options]
```

### 参数

- `<path>` - 资产路径

### 选项

- `--expand` - 展开数组/列表元素
- `--depth <n>` - 嵌套展开的最大深度（默认：3）
- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - WebSocket 连接超时时间（默认：30000）

### 示例

```bash
# 检查资产
cd <unity-project-root> && node .unity-websocket/uw asset inspect "Assets/Config/game.asset"

# 展开数组检查
cd <unity-project-root> && node .unity-websocket/uw asset inspect "Config/game.asset" --expand

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw asset inspect "Config/game.asset" --json
```

---

## asset add-element

向数组/列表字段添加新元素。

### 用法

```bash
cd <unity-project-root> && node .unity-websocket/uw asset add-element <path> <fieldName> [options]
```

### 参数

- `<path>` - 资产路径
- `<fieldName>` - 数组字段名称

### 选项

- `--value <value>` - 新元素的初始值
- `--index <n>` - 插入位置（-1 = 末尾，默认：-1）
- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - WebSocket 连接超时时间（默认：30000）

### 示例

```bash
# 在末尾添加元素
cd <unity-project-root> && node .unity-websocket/uw asset add-element "Config/game.asset" "items"

# 添加带有初始值的元素
cd <unity-project-root> && node .unity-websocket/uw asset add-element "Config/game.asset" "items" --value "NewItem"

# 在特定索引处添加元素
cd <unity-project-root> && node .unity-websocket/uw asset add-element "Config/game.asset" "items" --index 0 --value "FirstItem"

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw asset add-element "Config/game.asset" "enemies" --json
```

---

## asset remove-element

从数组/列表字段移除元素。

### 用法

```bash
cd <unity-project-root> && node .unity-websocket/uw asset remove-element <path> <fieldName> <index> [options]
```

### 参数

- `<path>` - 资产路径
- `<fieldName>` - 数组字段名称
- `<index>` - 要移除的元素索引

### 选项

- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - WebSocket 连接超时时间（默认：30000）

### 示例

```bash
# 移除第一个元素
cd <unity-project-root> && node .unity-websocket/uw asset remove-element "Config/game.asset" "items" 0

# 移除最后一个元素（假设数组有5个元素）
cd <unity-project-root> && node .unity-websocket/uw asset remove-element "Config/game.asset" "items" 4

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw asset remove-element "Config/game.asset" "enemies" 2 --json
```

---

## asset get-element

获取数组/列表字段的特定元素。

### 用法

```bash
cd <unity-project-root> && node .unity-websocket/uw asset get-element <path> <fieldName> <index> [options]
```

### 参数

- `<path>` - 资产路径
- `<fieldName>` - 数组字段名称
- `<index>` - 要获取的元素索引

### 选项

- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - WebSocket 连接超时时间（默认：30000）

### 示例

```bash
# 获取第一个元素
cd <unity-project-root> && node .unity-websocket/uw asset get-element "Config/game.asset" "items" 0

# 获取特定元素
cd <unity-project-root> && node .unity-websocket/uw asset get-element "Config/game.asset" "enemies" 2

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw asset get-element "Config/game.asset" "items" 0 --json
```

---

## asset clear-array

移除数组/列表字段的所有元素。

### 用法

```bash
cd <unity-project-root> && node .unity-websocket/uw asset clear-array <path> <fieldName> [options]
```

### 参数

- `<path>` - 资产路径
- `<fieldName>` - 数组字段名称

### 选项

- `--json` - 以 JSON 格式输出
- `--timeout <ms>` - WebSocket 连接超时时间（默认：30000）

### 示例

```bash
# 清空数组
cd <unity-project-root> && node .unity-websocket/uw asset clear-array "Config/game.asset" "items"

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw asset clear-array "Config/game.asset" "enemies" --json
```

---

## 工作流示例

### 1. 创建和配置 ScriptableObject

```bash
# 步骤 1: 列出可用类型
cd <unity-project-root> && node .unity-websocket/uw asset list-types --filter "*Config*"

# 步骤 2: 创建新的 ScriptableObject
cd <unity-project-root> && node .unity-websocket/uw asset create-so GameConfig "Assets/Config/game.asset"

# 步骤 3: 查看字段
cd <unity-project-root> && node .unity-websocket/uw asset get-fields "Assets/Config/game.asset"

# 步骤 4: 设置字段值
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Assets/Config/game.asset" "playerHealth" "100"
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Assets/Config/game.asset" "playerName" "Hero"
```

### 2. 管理数组/列表字段

```bash
# 查看数组
cd <unity-project-root> && node .unity-websocket/uw asset get-fields "Config/game.asset" --expand

# 添加元素
cd <unity-project-root> && node .unity-websocket/uw asset add-element "Config/game.asset" "items" --value "Sword"
cd <unity-project-root> && node .unity-websocket/uw asset add-element "Config/game.asset" "items" --value "Shield"

# 修改数组元素
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Config/game.asset" "items[0]" "LegendarySword"

# 移除元素
cd <unity-project-root> && node .unity-websocket/uw asset remove-element "Config/game.asset" "items" 1

# 清空所有
cd <unity-project-root> && node .unity-websocket/uw asset clear-array "Config/game.asset" "items"
```

### 3. 使用嵌套对象

```bash
# 查看嵌套结构
cd <unity-project-root> && node .unity-websocket/uw asset get-fields "Config/game.asset" --expand --depth 5

# 修改嵌套字段
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Config/game.asset" "player.stats.health" "100"
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Config/game.asset" "enemies[0].name" "Goblin"
cd <unity-project-root> && node .unity-websocket/uw asset set-field "Config/game.asset" "enemies[0].stats.damage" "15"
```

---

## 支持的字段类型

支持所有 Unity SerializedPropertyType 值：

- **数值**: Integer, Float, Boolean, Character, LayerMask
- **向量**: Vector2, Vector3, Vector4, Vector2Int, Vector3Int
- **几何**: Rect, RectInt, Bounds, BoundsInt, Quaternion
- **图形**: Color, AnimationCurve, Gradient (只读)
- **引用**: ObjectReference, ExposedReference
- **特殊**: Enum, String, Hash128, ArraySize
- **复杂**: Generic (嵌套对象), ManagedReference

---

## 错误处理

### 资产未找到 (Asset Not Found)
```
Error: ScriptableObject not found at: Assets/Config/missing.asset
```
**解决方案**: 检查资产路径并确保资产存在

### 类型未找到 (Type Not Found)
```
Error: ScriptableObject type not found: InvalidType
```
**解决方案**: 使用 `asset list-types` 查找有效的类型名称

### 字段未找到 (Field Not Found)
```
Error: Field not found: invalidField
```
**解决方案**: 使用 `asset get-fields` 查看可用字段

### 数组索引越界 (Array Index Out of Range)
```
Error: Index 10 out of range (0-4)
```
**解决方案**: 使用 `asset get-fields` 或 `asset get-element` 检查数组大小

### 无效数值格式 (Invalid Value Format)
```
Error: Cannot convert '1.5' to integer
```
**解决方案**: 检查该字段类型的数值格式

---

## 最佳实践

1.  **类型发现**: 在使用 `create-so` 之前使用 `list-types`
2.  **字段检查**: 使用 `get-fields --expand` 查看结构
3.  **数组索引**: 访问元素前务必检查数组大小
4.  **嵌套路径**: 对嵌套对象使用点号表示法：`items[0].name`
5.  **撤销支持**: 所有修改都支持 Unity 的 Undo 系统
6.  **批量操作**: 为了效率，组合多个 `set-field` 调用
7.  **JSON 输出**: 使用 `--json` 进行程序化解析
8.  **错误处理**: 始终检查响应的成功/错误状态
