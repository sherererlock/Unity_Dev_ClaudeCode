# Unity Editor Toolkit - 组件命令 (Component Commands)

完整的组件操作及管理命令参考手册。

**最后更新**: 2025-01-25

---

## comp list

列出 GameObject 的所有组件。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw comp list <gameobject> [options]
```

**参数:**
```
<gameobject>           GameObject 名称或路径 (例如: "Player" 或 "Environment/Player")
```

**选项:**
```
--include-disabled     包含禁用的组件 (默认: 仅显示启用的组件)
--type-only           仅显示组件类型 (不包含详细信息)
--json                以 JSON 格式输出
--timeout <ms>        WebSocket 连接超时 (默认: 30000)
-h, --help            显示命令帮助
```

**输出图标:**
```
●  启用的内置组件
○  禁用的组件
★  MonoBehaviour (自定义脚本)
```

**示例:**
```bash
# 列出 GameObject 的所有启用组件
cd <unity-project-root> && node .unity-websocket/uw comp list "Player"

# 包含禁用组件一起列出
cd <unity-project-root> && node .unity-websocket/uw comp list "Enemy" --include-disabled

# 仅显示类型
cd <unity-project-root> && node .unity-websocket/uw comp list "Character" --type-only

# 以 JSON 格式输出
cd <unity-project-root> && node .unity-websocket/uw comp list "Player" --json
```

---

## comp add

向 GameObject 添加组件。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw comp add <gameobject> <component> [options]
```

**参数:**
```
<gameobject>           GameObject 名称或路径
<component>            要添加的组件类型 (例如: Rigidbody, BoxCollider, AudioSource)
```

**选项:**
```
--json                以 JSON 格式输出
--timeout <ms>        WebSocket 连接超时 (默认: 30000)
-h, --help            显示命令帮助
```

**支持的组件类型:**
- Unity 内置: Rigidbody, Rigidbody2D, BoxCollider, SphereCollider, CapsuleCollider, MeshCollider, AudioSource, AudioListener, Camera, Light 等。
- 物理: Rigidbody, Rigidbody2D, Joint, SpringJoint, HingeJoint, ConfigurableJoint 等。
- 渲染: MeshRenderer, SkinnedMeshRenderer, SpriteRenderer, LineRenderer, TrailRenderer 等。
- 音频: AudioSource, AudioListener, AudioReverbFilter 等。
- 自定义: 所有 MonoBehaviour 脚本

**示例:**
```bash
# 添加 Rigidbody
cd <unity-project-root> && node .unity-websocket/uw comp add "Player" Rigidbody

# 添加 BoxCollider
cd <unity-project-root> && node .unity-websocket/uw comp add "Enemy" BoxCollider

# 添加自定义脚本
cd <unity-project-root> && node .unity-websocket/uw comp add "Character" PlayerController

# 以 JSON 格式接收响应
cd <unity-project-root> && node .unity-websocket/uw comp add "Item" Collider --json
```

**重要:**
- Transform 组件会自动包含在所有 GameObject 中，因此无法添加。
- 如果已存在相同类型的组件，则不会重复添加。

---

## comp remove

从 GameObject 移除组件。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw comp remove <gameobject> <component> [options]
```

**参数:**
```
<gameobject>           GameObject 名称或路径
<component>            要移除的组件类型
```

**选项:**
```
--force               无需确认直接移除
--json                以 JSON 格式输出
--timeout <ms>        WebSocket 连接超时 (默认: 30000)
-h, --help            显示命令帮助
```

**示例:**
```bash
# 移除组件
cd <unity-project-root> && node .unity-websocket/uw comp remove "Player" AudioSource

# 移除 Rigidbody
cd <unity-project-root> && node .unity-websocket/uw comp remove "Enemy" Rigidbody

# 无需确认直接移除
cd <unity-project-root> && node .unity-websocket/uw comp remove "Temp" BoxCollider --force
```

**重要:**
- Transform 是必需组件，无法移除。
- 可以使用 Ctrl+Z (Undo) 撤销。

---

## comp enable

启用 GameObject 的组件。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw comp enable <gameobject> <component> [options]
```

**参数:**
```
<gameobject>           GameObject 名称或路径
<component>            要启用的组件类型
```

**选项:**
```
--json                以 JSON 格式输出
--timeout <ms>        WebSocket 连接超时 (默认: 30000)
-h, --help            显示命令帮助
```

**示例:**
```bash
# 启用 AudioSource
cd <unity-project-root> && node .unity-websocket/uw comp enable "Player" AudioSource

# 启用自定义脚本
cd <unity-project-root> && node .unity-websocket/uw comp enable "Character" PlayerController

# JSON 响应
cd <unity-project-root> && node .unity-websocket/uw comp enable "Enemy" AIController --json
```

**重要:**
- 仅继承自 Behaviour 的组件 (MonoBehaviour, Renderer, AudioSource 等) 可以启用/禁用。

---

## comp disable

禁用 GameObject 的组件。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw comp disable <gameobject> <component> [options]
```

**参数:**
```
<gameobject>           GameObject 名称或路径
<component>            要禁用的组件类型
```

**选项:**
```
--json                以 JSON 格式输出
--timeout <ms>        WebSocket 连接超时 (默认: 30000)
-h, --help            显示命令帮助
```

**示例:**
```bash
# 禁用 AudioSource
cd <unity-project-root> && node .unity-websocket/uw comp disable "Player" AudioSource

# 禁用自定义脚本
cd <unity-project-root> && node .unity-websocket/uw comp disable "Enemy" EnemyAI

# 禁用渲染器
cd <unity-project-root> && node .unity-websocket/uw comp disable "Visual" MeshRenderer --json
```

**重要:**
- 仅继承自 Behaviour 的组件可以禁用。
- 如果 GameObject 被禁用，所有组件将自动处于禁用状态。

---

## comp get

查询组件的属性值。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw comp get <gameobject> <component> [property] [options]
```

**参数:**
```
<gameobject>           GameObject 名称或路径
<component>            组件类型
[property]             特定属性名称 (省略则显示所有属性)
```

**选项:**
```
--json                以 JSON 格式输出
--timeout <ms>        WebSocket 连接超时 (默认: 30000)
-h, --help            显示命令帮助
```

**示例:**
```bash
# 查询 Rigidbody 的所有属性
cd <unity-project-root> && node .unity-websocket/uw comp get "Player" Rigidbody

# 查询特定属性值
cd <unity-project-root> && node .unity-websocket/uw comp get "Player" Rigidbody mass

# 查询 Transform 位置
cd <unity-project-root> && node .unity-websocket/uw comp get "Enemy" Transform localPosition

# 以 JSON 格式输出
cd <unity-project-root> && node .unity-websocket/uw comp get "Character" Camera fieldOfView --json
```

**支持的属性类型:**
- 基础类型: int, float, bool, string
- Unity 类型: Vector2, Vector3, Vector4, Color, Rect, Bounds
- Enum: 枚举值
- Reference: GameObject/资源引用

---

## comp set

设置组件的属性值。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw comp set <gameobject> <component> <property> <value> [options]
```

**参数:**
```
<gameobject>           GameObject 名称或路径
<component>            组件类型
<property>             属性名称
<value>                新值 (逗号分隔的向量或普通值)
```

**选项:**
```
--json                以 JSON 格式输出
--timeout <ms>        WebSocket 连接超时 (默认: 30000)
-h, --help            显示命令帮助
```

**值格式:**
```
Integer/Float:        123 或 45.67
Boolean:              true 或 false
String:               "Hello World" 或 HelloWorld
Vector3:              "1,2,3" (无空格)
Color:                "1,0,0,1" (R,G,B,A)
Enum:                 枚举名称 (例如: "Linear")
ObjectReference:      "GameObjectName" 或 "GameObject:Component" 或 "Assets/path/to/asset.ext"
```

**ObjectReference 格式 (组件/资源引用):**
| 格式 | 示例 | 说明 |
|------|------|------|
| `null` | `"null"` | 解除引用 |
| `GameObject` | `"Player"` | GameObject 引用 |
| `GameObject:Component` | `"GameHUD:UnityEngine.UIElements.UIDocument"` | 组件引用 |
| `Asset Path` | `"Assets/Materials/Red.mat"` | 资源引用 |

**⚠️ 重要: 引用组件时需要完整的命名空间**
```
✅ "GameHUD:UnityEngine.UIElements.UIDocument"   (完整命名空间)
❌ "GameHUD:UIDocument"                           (不起作用)
```

**示例:**
```bash
# 设置 Rigidbody mass
cd <unity-project-root> && node .unity-websocket/uw comp set "Player" Rigidbody mass 2.5

# 禁用重力
cd <unity-project-root> && node .unity-websocket/uw comp set "Floating" Rigidbody useGravity false

# 设置位置
cd <unity-project-root> && node .unity-websocket/uw comp set "Enemy" Transform localPosition "5,10,0"

# 设置 Camera FOV
cd <unity-project-root> && node .unity-websocket/uw comp set "MainCamera" Camera fieldOfView 75

# 设置颜色 (R,G,B,A)
cd <unity-project-root> && node .unity-websocket/uw comp set "Material" SpriteRenderer color "1,0,0,1"

# 设置 Enum 值
cd <unity-project-root> && node .unity-websocket/uw comp set "Collider" Rigidbody constraints "FreezeRotationZ"

# 设置 ObjectReference - GameObject 引用
cd <unity-project-root> && node .unity-websocket/uw comp set "Enemy" AIController target "Player"

# 设置 ObjectReference - 组件引用 (必需完整命名空间)
cd <unity-project-root> && node .unity-websocket/uw comp set "HUD" GameView uiDocument "GameHUD:UnityEngine.UIElements.UIDocument"

# 设置 ObjectReference - 资源引用
cd <unity-project-root> && node .unity-websocket/uw comp set "Enemy" SpriteRenderer sprite "Assets/Sprites/Enemy.png"

# 设置 ObjectReference - 解除引用
cd <unity-project-root> && node .unity-websocket/uw comp set "Item" Pickup targetObject "null"
```

**重要:**
- 可以使用 Ctrl+Z (Undo) 撤销。
- 响应中包含修改前的旧值。

---

## comp inspect

显示组件的所有属性和状态。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw comp inspect <gameobject> <component> [options]
```

**参数:**
```
<gameobject>           GameObject 名称或路径
<component>            组件类型
```

**选项:**
```
--json                以 JSON 格式输出
--timeout <ms>        WebSocket 连接超时 (默认: 30000)
-h, --help            显示命令帮助
```

**示例:**
```bash
# 检查 Rigidbody 整体
cd <unity-project-root> && node .unity-websocket/uw comp inspect "Player" Rigidbody

# 检查 Transform
cd <unity-project-root> && node .unity-websocket/uw comp inspect "Enemy" Transform

# 检查自定义脚本
cd <unity-project-root> && node .unity-websocket/uw comp inspect "Character" PlayerController

# 输出为 JSON 并保存到文件
cd <unity-project-root> && node .unity-websocket/uw comp inspect "Item" Collider --json > component.json
```

**输出格式:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ComponentName
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ● Type: Full.Type.Name
  ● Enabled: true

    propertyName1  : PropertyType = value1
    propertyName2  : PropertyType = value2
    ...
```

---

## comp move-up

在组件列表中向上移动组件。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw comp move-up <gameobject> <component> [options]
```

**参数:**
```
<gameobject>           GameObject 名称或路径
<component>            要移动的组件类型
```

**选项:**
```
--json                以 JSON 格式输出
--timeout <ms>        WebSocket 连接超时 (默认: 30000)
-h, --help            显示命令帮助
```

**示例:**
```bash
# 将 Rigidbody 向上移动
cd <unity-project-root> && node .unity-websocket/uw comp move-up "Player" Rigidbody

# 将 BoxCollider 向上移动
cd <unity-project-root> && node .unity-websocket/uw comp move-up "Enemy" BoxCollider

# 先确认列表
cd <unity-project-root> && node .unity-websocket/uw comp list "Character"
# 然后移动
cd <unity-project-root> && node .unity-websocket/uw comp move-up "Character" CustomScript
```

**重要:**
- Transform 始终固定在第一个位置，无法移动。
- Inspector 中的显示顺序会改变。
- 某些组件可能会受到执行顺序的影响。

---

## comp move-down

在组件列表中向下移动组件。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw comp move-down <gameobject> <component> [options]
```

**参数:**
```
<gameobject>           GameObject 名称或路径
<component>            要移动的组件类型
```

**选项:**
```
--json                以 JSON 格式输出
--timeout <ms>        WebSocket 连接超时 (默认: 30000)
-h, --help            显示命令帮助
```

**示例:**
```bash
# 将 Rigidbody 向下移动
cd <unity-project-root> && node .unity-websocket/uw comp move-down "Player" Rigidbody

# 重新排列组件顺序
cd <unity-project-root> && node .unity-websocket/uw comp move-down "Character" AudioSource
cd <unity-project-root> && node .unity-websocket/uw comp move-down "Character" AudioSource

# 确认最终结果
cd <unity-project-root> && node .unity-websocket/uw comp list "Character"
```

**重要:**
- 如果已经在最后位置，则无法移动。
- Transform 始终位于第一个位置。

---

## comp copy

将一个 GameObject 的组件复制到另一个 GameObject。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw comp copy <source> <component> <target> [options]
```

**参数:**
```
<source>               源 GameObject 名称或路径
<component>            要复制的组件类型
<target>               目标 GameObject 名称或路径
```

**选项:**
```
--json                以 JSON 格式输出
--timeout <ms>        WebSocket 连接超时 (默认: 30000)
-h, --help            显示命令帮助
```

**示例:**
```bash
# 复制 Rigidbody
cd <unity-project-root> && node .unity-websocket/uw comp copy "Player" Rigidbody "Enemy"

# 复制多个组件
cd <unity-project-root> && node .unity-websocket/uw comp copy "Template" BoxCollider "New"
cd <unity-project-root> && node .unity-websocket/uw comp copy "Template" Rigidbody "New"
cd <unity-project-root> && node .unity-websocket/uw comp copy "Template" AudioSource "New"

# 复制脚本
cd <unity-project-root> && node .unity-websocket/uw comp copy "Character" PlayerController "Character2"

# 确认结果
cd <unity-project-root> && node .unity-websocket/uw comp list "New"
```

**重要:**
- 源组件的所有属性都将被复制。
- 如果目标 GameObject 中已存在相同类型的组件，不会覆盖，而是新增。
- 可以使用 Ctrl+Z (Undo) 撤销。

---

## 技巧与最佳实践

### 组件顺序优化
```bash
# 1. 通过列表确认当前顺序
cd <unity-project-root> && node .unity-websocket/uw comp list "Player"

# 2. 按所需顺序排列
cd <unity-project-root> && node .unity-websocket/uw comp move-up "Player" Rigidbody
cd <unity-project-root> && node .unity-websocket/uw comp move-up "Player" BoxCollider

# 3. 最终确认
cd <unity-project-root> && node .unity-websocket/uw comp list "Player"
```

### 创建 GameObject 模板
```bash
# 1. 向一个 GameObject 添加并设置所有组件
cd <unity-project-root> && node .unity-websocket/uw comp add "Template" Rigidbody
cd <unity-project-root> && node .unity-websocket/uw comp add "Template" BoxCollider
cd <unity-project-root> && node .unity-websocket/uw comp add "Template" AudioSource
cd <unity-project-root> && node .unity-websocket/uw comp set "Template" Rigidbody mass 2.0

# 2. 复制到其他 GameObject
cd <unity-project-root> && node .unity-websocket/uw comp copy "Template" Rigidbody "Object1"
cd <unity-project-root> && node .unity-websocket/uw comp copy "Template" BoxCollider "Object1"
cd <unity-project-root> && node .unity-websocket/uw comp copy "Template" AudioSource "Object1"
```

### 批量设置更改
```bash
# 一次性更改多个组件的属性 (脚本)
for obj in "Enemy1" "Enemy2" "Enemy3"; do
  cd <unity-project-root> && node .unity-websocket/uw comp set "$obj" Rigidbody mass 1.5
  cd <unity-project-root> && node .unity-websocket/uw comp set "$obj" AudioSource volume 0.8
done
```

---

## 故障排除

### 找不到组件类型
**问题**: `Component type not found`

**解决方案**:
- 确认组件名称是否准确 (区分大小写)
- 例如: `Rigidbody` (O), `rigidbody` (X), `RigidBody` (X)
- 如果需要命名空间: `UnityEngine.Rigidbody`

### 无法更改属性
**问题**: `Failed to set property`

**解决方案**:
- 确认属性名称是否准确
- 无法更改未在 Inspector 中显示的内部属性 (以 `m_` 开头)
- 确认值格式是否正确 (例如: Vector3 为 "x,y,z" 格式)

### 无法移除组件
**问题**: `Cannot remove component`

**解决方案**:
- Transform 是必需组件，无法移除
- 某些组件可能因 RequireComponent 而无法移除

### 找不到 ObjectReference
**问题**: `ObjectReference not found: 'GameHUD:UIDocument'`

**解决方案**:
- **引用组件时必需完整的命名空间**
  - ✅ `"GameHUD:UnityEngine.UIElements.UIDocument"`
  - ❌ `"GameHUD:UIDocument"`
- 常见的 Unity 命名空间:
  - `UnityEngine.` - 基本组件 (Rigidbody, Collider 等)
  - `UnityEngine.UI.` - uGUI 组件 (Image, Text 等)
  - `UnityEngine.UIElements.` - UI Toolkit 组件 (UIDocument 等)
- 确认 GameObject 名称是否准确
- 禁用的 GameObject 无法通过 GameObject.Find 找到

---

## 相关命令

- [GameObject & Hierarchy Commands](./COMMANDS_GAMEOBJECT_HIERARCHY.md) - GameObject 创建, 层级结构管理
- [Transform Commands](./COMMANDS_TRANSFORM.md) - 位置, 旋转, 缩放操作
- [Scene Commands](./COMMANDS_SCENE.md) - 场景管理
- [Asset Commands](./COMMANDS_ASSET.md) - 资源及 ScriptableObject 管理
