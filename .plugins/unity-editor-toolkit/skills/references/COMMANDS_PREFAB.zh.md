# Unity Editor Toolkit - Prefab 命令

完整的 Prefab（预制件）操作及管理命令参考。

**最后更新**: 2025-01-26

---

## prefab instantiate

在场景中实例化预制件。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefab instantiate <path> [options]
```

**参数:**
```
<path>                 预制件资源路径 (例如: "Assets/Prefabs/Player.prefab")
```

**选项:**
```
--name <name>          指定实例名称
--position <x,y,z>     生成位置 (例如: "0,1,0")
--rotation <x,y,z>     旋转值 (欧拉角, 例如: "0,90,0")
--parent <gameobject>  指定父 GameObject
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时 (默认值: 30000)
-h, --help             显示命令帮助
```

**示例:**
```bash
# 基本实例化
cd <unity-project-root> && node .unity-websocket/uw prefab instantiate "Assets/Prefabs/Enemy.prefab"

# 指定名称和位置
cd <unity-project-root> && node .unity-websocket/uw prefab instantiate "Assets/Prefabs/Player.prefab" --name "Player1" --position "0,1,0"

# 指定父对象
cd <unity-project-root> && node .unity-websocket/uw prefab instantiate "Assets/Prefabs/Item.prefab" --parent "ItemContainer"

# 指定位置和旋转
cd <unity-project-root> && node .unity-websocket/uw prefab instantiate "Assets/Prefabs/Car.prefab" --position "10,0,5" --rotation "0,180,0"
```

---

## prefab create

从场景中的 GameObject 创建预制件。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefab create <gameobject> <path> [options]
```

**参数:**
```
<gameobject>           GameObject 名称或路径
<path>                 保存预制件的路径 (例如: "Assets/Prefabs/MyPrefab.prefab")
```

**选项:**
```
--overwrite            覆盖现有预制件
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时 (默认值: 30000)
-h, --help             显示命令帮助
```

**示例:**
```bash
# 创建新预制件
cd <unity-project-root> && node .unity-websocket/uw prefab create "Player" "Assets/Prefabs/Player.prefab"

# 覆盖现有预制件
cd <unity-project-root> && node .unity-websocket/uw prefab create "Enemy" "Assets/Prefabs/Enemy.prefab" --overwrite

# 保存到子文件夹
cd <unity-project-root> && node .unity-websocket/uw prefab create "Boss" "Assets/Prefabs/Enemies/Boss.prefab"
```

**重要:**
- 创建预制件后，原始 GameObject 将链接为预制件实例。
- 路径中不存在的文件夹将自动创建。

---

## prefab unpack

解包预制件实例。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefab unpack <gameobject> [options]
```

**参数:**
```
<gameobject>           预制件实例名称或路径
```

**选项:**
```
--completely           完全解包 (包括嵌套的预制件也全部解包)
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时 (默认值: 30000)
-h, --help             显示命令帮助
```

**示例:**
```bash
# 仅解包最外层 (默认)
cd <unity-project-root> && node .unity-websocket/uw prefab unpack "Player"

# 完全解包 (包含所有嵌套预制件)
cd <unity-project-root> && node .unity-websocket/uw prefab unpack "ComplexPrefab" --completely
```

**解包模式 (Unpack Modes):**
| 模式 | 说明 |
|------|------|
| OutermostRoot (默认) | 仅解包最外层预制件，保留嵌套预制件 |
| Completely | 完全解包，包括所有嵌套预制件 |

---

## prefab apply

将预制件实例的覆盖（Override）应用到原始预制件。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefab apply <gameobject> [options]
```

**参数:**
```
<gameobject>           预制件实例名称或路径
```

**选项:**
```
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时 (默认值: 30000)
-h, --help             显示命令帮助
```

**示例:**
```bash
# 应用覆盖
cd <unity-project-root> && node .unity-websocket/uw prefab apply "Player"

# JSON 响应
cd <unity-project-root> && node .unity-websocket/uw prefab apply "Enemy" --json
```

**重要:**
- 所有覆盖都将保存到原始预制件中。
- 会影响其他场景中的同一预制件实例。
- 可以使用 Ctrl+Z (Undo) 撤销。

---

## prefab revert

还原预制件实例的覆盖（Override）。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefab revert <gameobject> [options]
```

**参数:**
```
<gameobject>           预制件实例名称或路径
```

**选项:**
```
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时 (默认值: 30000)
-h, --help             显示命令帮助
```

**示例:**
```bash
# 还原所有覆盖
cd <unity-project-root> && node .unity-websocket/uw prefab revert "Player"
```

---

## prefab variant

从现有预制件创建变体 (Variant)。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefab variant <sourcePath> <variantPath> [options]
```

**参数:**
```
<sourcePath>           原始预制件路径
<variantPath>          变体保存路径
```

**选项:**
```
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时 (默认值: 30000)
-h, --help             显示命令帮助
```

**示例:**
```bash
# 创建变体
cd <unity-project-root> && node .unity-websocket/uw prefab variant "Assets/Prefabs/Enemy.prefab" "Assets/Prefabs/EnemyBoss.prefab"

# 在其他文件夹创建变体
cd <unity-project-root> && node .unity-websocket/uw prefab variant "Assets/Prefabs/Base/Character.prefab" "Assets/Prefabs/Variants/Warrior.prefab"
```

**重要:**
- 变体继承原始预制件的更改。
- 可以在变体中覆盖个别属性。

---

## prefab overrides

查询预制件实例的覆盖列表。

**别名:** `prefab get-overrides`

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefab overrides <gameobject> [options]
```

**参数:**
```
<gameobject>           预制件实例名称或路径
```

**选项:**
```
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时 (默认值: 30000)
-h, --help             显示命令帮助
```

**输出图标:**
```
●  PropertyOverride (属性变更)
+  AddedComponent (添加的组件)
-  RemovedComponent (移除的组件)
★  AddedGameObject (添加的子对象)
```

**示例:**
```bash
# 查询覆盖列表
cd <unity-project-root> && node .unity-websocket/uw prefab overrides "Player"

# 以 JSON 格式查询
cd <unity-project-root> && node .unity-websocket/uw prefab overrides "Enemy" --json
```

---

## prefab source

查询预制件实例的原始预制件路径。

**别名:** `prefab get-source`

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefab source <gameobject> [options]
```

**参数:**
```
<gameobject>           GameObject 名称或路径
```

**选项:**
```
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时 (默认值: 30000)
-h, --help             显示命令帮助
```

**示例:**
```bash
# 查询原始预制件路径
cd <unity-project-root> && node .unity-websocket/uw prefab source "Player"
```

**输出:**
```
✓ Prefab info for 'Player':
  Is Prefab Instance: Yes
  Prefab Path: Assets/Prefabs/Player.prefab
  Prefab Type: Regular
  Status: Connected
```

---

## prefab is-instance

检查 GameObject 是否为预制件实例。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefab is-instance <gameobject> [options]
```

**参数:**
```
<gameobject>           GameObject 名称或路径
```

**选项:**
```
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时 (默认值: 30000)
-h, --help             显示命令帮助
```

**示例:**
```bash
# 检查是否为预制件实例
cd <unity-project-root> && node .unity-websocket/uw prefab is-instance "Player"
```

**预制件类型 (Prefab Types):**
| 类型 | 说明 |
|------|------|
| NotAPrefab | 不是预制件 |
| Regular | 普通预制件 |
| Model | 模型预制件 (FBX 等) |
| Variant | 预制件变体 |
| MissingAsset | 原始资源缺失 |

---

## prefab open

以编辑模式打开预制件。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefab open <path> [options]
```

**参数:**
```
<path>                 预制件资源路径
```

**选项:**
```
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时 (默认值: 30000)
-h, --help             显示命令帮助
```

**示例:**
```bash
# 打开预制件编辑模式
cd <unity-project-root> && node .unity-websocket/uw prefab open "Assets/Prefabs/Player.prefab"
```

**重要:**
- 在预制件编辑模式下，是在预制件上下文中操作，而非场景上下文。
- 更改不会自动保存。
- 使用 `prefab close` 退出编辑模式。

---

## prefab close

关闭预制件编辑模式并返回场景。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefab close [options]
```

**选项:**
```
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时 (默认值: 30000)
-h, --help             显示命令帮助
```

**示例:**
```bash
# 关闭预制件编辑模式
cd <unity-project-root> && node .unity-websocket/uw prefab close
```

---

## prefab list

列出文件夹内的所有预制件。

**用法:**
```bash
cd <unity-project-root> && node .unity-websocket/uw prefab list [options]
```

**选项:**
```
--path <path>          搜索文件夹路径 (默认值: "Assets")
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时 (默认值: 30000)
-h, --help             显示命令帮助
```

**输出图标:**
```
●  普通预制件
◇  Variant 预制件
```

**示例:**
```bash
# 列出所有预制件
cd <unity-project-root> && node .unity-websocket/uw prefab list

# 仅搜索特定文件夹
cd <unity-project-root> && node .unity-websocket/uw prefab list --path "Assets/Prefabs/Characters"

# JSON 格式
cd <unity-project-root> && node .unity-websocket/uw prefab list --json
```

---

## Tips & Best Practices (提示与最佳实践)

### 预制件工作流

```bash
# 1. 在场景中创建对象并设置
cd <unity-project-root> && node .unity-websocket/uw go create "NewCharacter"
cd <unity-project-root> && node .unity-websocket/uw comp add "NewCharacter" Rigidbody
cd <unity-project-root> && node .unity-websocket/uw comp add "NewCharacter" BoxCollider

# 2. 保存为预制件
cd <unity-project-root> && node .unity-websocket/uw prefab create "NewCharacter" "Assets/Prefabs/NewCharacter.prefab"

# 3. 实例化预制件
cd <unity-project-root> && node .unity-websocket/uw prefab instantiate "Assets/Prefabs/NewCharacter.prefab" --position "10,0,0"
```

### Variant (变体) 活用

```bash
# 1. 创建基础敌人预制件
cd <unity-project-root> && node .unity-websocket/uw prefab create "BaseEnemy" "Assets/Prefabs/Enemies/BaseEnemy.prefab"

# 2. 创建变体
cd <unity-project-root> && node .unity-websocket/uw prefab variant "Assets/Prefabs/Enemies/BaseEnemy.prefab" "Assets/Prefabs/Enemies/FastEnemy.prefab"
cd <unity-project-root> && node .unity-websocket/uw prefab variant "Assets/Prefabs/Enemies/BaseEnemy.prefab" "Assets/Prefabs/Enemies/TankEnemy.prefab"

# 3. 实例化变体并修改
cd <unity-project-root> && node .unity-websocket/uw prefab instantiate "Assets/Prefabs/Enemies/FastEnemy.prefab" --name "FastEnemy1"
```

### 覆盖 (Override) 管理

```bash
# 1. 确认覆盖
cd <unity-project-root> && node .unity-websocket/uw prefab overrides "Player"

# 2. 将覆盖应用到原始预制件
cd <unity-project-root> && node .unity-websocket/uw prefab apply "Player"

# 或者取消覆盖
cd <unity-project-root> && node .unity-websocket/uw prefab revert "Player"
```

---

## Troubleshooting (故障排除)

### 找不到预制件
**问题**: `Prefab not found: Assets/Prefabs/MyPrefab.prefab`

**解决方法**:
- 检查路径是否正确 (区分大小写)
- 确认是否包含 `.prefab` 扩展名
- 确认资源是否实际存在

### 不是预制件实例
**问题**: `GameObject is not a prefab instance`

**解决方法**:
- 先使用 `prefab is-instance` 进行确认
- 如果是已经解包的对象，则不是预制件
- 在场景中直接创建的对象不是预制件实例

### 预制件创建失败
**问题**: `Prefab already exists: ... Use --overwrite to replace.`

**解决方法**:
- 同一路径下已存在预制件
- 可使用 `--overwrite` 选项进行覆盖

---

## Related Commands (相关命令)

- [GameObject & Hierarchy Commands](./COMMANDS_GAMEOBJECT_HIERARCHY.md) - GameObject 创建、层级结构管理
- [Component Commands](./COMMANDS_COMPONENT.md) - 组件操作
- [Transform Commands](./COMMANDS_TRANSFORM.md) - 位置、旋转、缩放操作
- [Asset Commands](./COMMANDS_ASSET.md) - 资源及 ScriptableObject 管理
