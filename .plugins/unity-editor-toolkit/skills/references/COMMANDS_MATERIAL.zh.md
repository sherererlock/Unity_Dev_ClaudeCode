# Unity Editor Toolkit - 材质 (Material) 命令

用于操作和管理 Material 属性的命令参考。

**Last Updated**: 2025-12-02

---

## material list

列出 GameObject 上连接的所有 Material。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw material list <gameobject> [options]
```

**Arguments:**
```
<gameobject>           GameObject 名称或路径（例如："Player" 或 "Environment/Cube"）
```

**Options:**
```
--shared               使用共享 Material（未实例化的原始材质）
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 列出 GameObject 的所有 Material
cd <unity-project-root> && node .unity-websocket/uw material list "Cube"

# 共享 Material 列表
cd <unity-project-root> && node .unity-websocket/uw material list "Player" --shared

# 以 JSON 格式输出
cd <unity-project-root> && node .unity-websocket/uw material list "Enemy" --json
```

---

## material get

查询 Material 属性值。支持 Float, Int, Range 类型属性。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw material get <gameobject> <property> [options]
```

**Arguments:**
```
<gameobject>           GameObject 名称或路径
<property>             Material 属性名称（例如："_Glossiness", "_Metallic"）
```

**Options:**
```
-m, --material <index> Material 索引（默认值：0）
--shared               使用共享 Material
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 查询 Smoothness 值
cd <unity-project-root> && node .unity-websocket/uw material get "Cube" "_Glossiness"

# 查询第二个 Material 的 Metallic 值
cd <unity-project-root> && node .unity-websocket/uw material get "Character" "_Metallic" -m 1
```

---

## material set

设置 Material 属性值。支持 Float, Int, Range 类型属性。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw material set <gameobject> <property> <value> [options]
```

**Arguments:**
```
<gameobject>           GameObject 名称或路径
<property>             Material 属性名称
<value>                要设置的值（数字）
```

**Options:**
```
-m, --material <index> Material 索引（默认值：0）
--shared               修改共享 Material
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 将 Smoothness 设置为 0.8
cd <unity-project-root> && node .unity-websocket/uw material set "Cube" "_Glossiness" 0.8

# 将 Metallic 设置为 1.0
cd <unity-project-root> && node .unity-websocket/uw material set "Player" "_Metallic" 1.0
```

---

## material get-color

查询 Material 的颜色属性。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw material get-color <gameobject> [options]
```

**Arguments:**
```
<gameobject>           GameObject 名称或路径
```

**Options:**
```
-p, --property <name>  颜色属性名称（默认值："_Color"）
-m, --material <index> Material 索引（默认值：0）
--shared               使用共享 Material
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 查询基本颜色
cd <unity-project-root> && node .unity-websocket/uw material get-color "Cube"

# 查询 Emission 颜色
cd <unity-project-root> && node .unity-websocket/uw material get-color "Neon" -p "_EmissionColor"
```

---

## material set-color

设置 Material 的颜色属性。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw material set-color <gameobject> [options]
```

**Arguments:**
```
<gameobject>           GameObject 名称或路径
```

**Options:**
```
-p, --property <name>  颜色属性名称（默认值："_Color"）
-m, --material <index> Material 索引（默认值：0）
--hex <color>          Hex 颜色代码（例如："#FF0000" 或 "FF0000FF"）
--r <value>            红 (0.0 ~ 1.0)
--g <value>            绿 (0.0 ~ 1.0)
--b <value>            蓝 (0.0 ~ 1.0)
--a <value>            Alpha (0.0 ~ 1.0, 默认值：1.0)
--shared               修改共享 Material
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 使用 Hex 代码设置红色
cd <unity-project-root> && node .unity-websocket/uw material set-color "Cube" --hex "#FF0000"

# 使用 RGBA 设置半透明蓝色
cd <unity-project-root> && node .unity-websocket/uw material set-color "Glass" --r 0 --g 0 --b 1 --a 0.5

# 设置 Emission 颜色
cd <unity-project-root> && node .unity-websocket/uw material set-color "Neon" -p "_EmissionColor" --hex "#00FF00"
```

---

## material get-shader

查询应用于 Material 的 Shader 信息。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw material get-shader <gameobject> [options]
```

**Arguments:**
```
<gameobject>           GameObject 名称或路径
```

**Options:**
```
-m, --material <index> Material 索引（默认值：0）
--shared               使用共享 Material
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 查询 Shader 信息
cd <unity-project-root> && node .unity-websocket/uw material get-shader "Cube"

# 查询第二个 Material 的 Shader
cd <unity-project-root> && node .unity-websocket/uw material get-shader "Character" -m 1
```

---

## material set-shader

更改 Material 的 Shader。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw material set-shader <gameobject> <shaderName> [options]
```

**Arguments:**
```
<gameobject>           GameObject 名称或路径
<shaderName>           Shader 名称（例如："Standard", "Unlit/Color"）
```

**Options:**
```
-m, --material <index> Material 索引（默认值：0）
--shared               修改共享 Material
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 更改为 Standard Shader
cd <unity-project-root> && node .unity-websocket/uw material set-shader "Cube" "Standard"

# 更改为 Unlit Shader
cd <unity-project-root> && node .unity-websocket/uw material set-shader "UIElement" "Unlit/Texture"
```

---

## material get-texture

查询 Material 的纹理属性。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw material get-texture <gameobject> [options]
```

**Arguments:**
```
<gameobject>           GameObject 名称或路径
```

**Options:**
```
-p, --property <name>  纹理属性名称（默认值："_MainTex"）
-m, --material <index> Material 索引（默认值：0）
--shared               使用共享 Material
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 查询主纹理信息
cd <unity-project-root> && node .unity-websocket/uw material get-texture "Cube"

# 查询 Normal Map 纹理
cd <unity-project-root> && node .unity-websocket/uw material get-texture "Character" -p "_BumpMap"
```

---

## material set-texture

设置 Material 的纹理。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw material set-texture <gameobject> <texturePath> [options]
```

**Arguments:**
```
<gameobject>           GameObject 名称或路径
<texturePath>          基于 Assets 文件夹的纹理路径（例如："Assets/Textures/Wood.png"）
```

**Options:**
```
-p, --property <name>  纹理属性名称（默认值："_MainTex"）
-m, --material <index> Material 索引（默认值：0）
--scale-x <value>      纹理 X 缩放
--scale-y <value>      纹理 Y 缩放
--offset-x <value>     纹理 X 偏移
--offset-y <value>     纹理 Y 偏移
--shared               修改共享 Material
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 设置主纹理
cd <unity-project-root> && node .unity-websocket/uw material set-texture "Cube" "Assets/Textures/Stone.png"

# 设置 Normal Map
cd <unity-project-root> && node .unity-websocket/uw material set-texture "Wall" "Assets/Textures/Brick_Normal.png" -p "_BumpMap"

# 设置纹理平铺
cd <unity-project-root> && node .unity-websocket/uw material set-texture "Floor" "Assets/Textures/Tile.png" --scale-x 4 --scale-y 4
```

---

## Common Material Properties

Standard Shader 中常用的属性：

| Property Name | Type | Description |
|---------------|------|-------------|
| `_Color` | Color | Albedo 颜色 |
| `_MainTex` | Texture | Albedo 纹理 |
| `_Metallic` | Float | 金属度 (0-1) |
| `_Glossiness` | Float | 平滑度/光泽度 (0-1) |
| `_BumpMap` | Texture | Normal Map |
| `_BumpScale` | Float | Normal Map 强度 |
| `_EmissionColor` | Color | Emission 颜色 |
| `_EmissionMap` | Texture | Emission 纹理 |
| `_OcclusionMap` | Texture | Ambient Occlusion 贴图 |
| `_OcclusionStrength` | Float | AO 强度 (0-1) |

---

## JSON Output Example

```json
{
  "success": true,
  "gameObject": "Cube",
  "material": "Default-Material",
  "propertyName": "_Color",
  "color": {
    "r": 1.0,
    "g": 0.5,
    "b": 0.0,
    "a": 1.0,
    "hex": "FF8000FF"
  }
}
```
