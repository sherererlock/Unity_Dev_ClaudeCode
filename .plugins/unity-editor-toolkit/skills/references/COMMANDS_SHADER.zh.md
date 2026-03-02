# Unity Editor Toolkit - Shader 命令

Shader 查询及关键字管理命令参考。

**Last Updated**: 2025-12-02

---

## shader list

列出项目中的所有 Shader。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw shader list [options]
```

**Options:**
```
-f, --filter <pattern> 按名称过滤（部分匹配）
-b, --builtin          包含内置 Shader
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 列出项目内的所有 Shader
cd <unity-project-root> && node .unity-websocket/uw shader list

# 搜索包含 "Standard" 名称的 Shader
cd <unity-project-root> && node .unity-websocket/uw shader list -f "Standard"

# 包含内置 Shader 一起搜索
cd <unity-project-root> && node .unity-websocket/uw shader list --builtin

# 仅搜索 Unlit 系列 Shader
cd <unity-project-root> && node .unity-websocket/uw shader list -f "Unlit"
```

**Output:**
```
Shaders (15):

Name                                              Properties  Queue   Type
-------------------------------------------------------------------------------------
Standard                                          15          2000    Built-in
Mobile/Diffuse                                    2           2000    Built-in
Custom/MyShader                                   8           3000    Project
```

---

## shader find

按名称搜索 Shader 并查询详细信息。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw shader find <name> [options]
```

**Arguments:**
```
<name>                 Shader 名称（必须完全匹配）
                       例: "Standard", "Unlit/Color", "Custom/MyShader"
```

**Options:**
```
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 查询 Standard Shader 信息
cd <unity-project-root> && node .unity-websocket/uw shader find "Standard"

# 查询 Unlit/Color Shader
cd <unity-project-root> && node .unity-websocket/uw shader find "Unlit/Color"

# 查询自定义 Shader
cd <unity-project-root> && node .unity-websocket/uw shader find "Custom/Water"
```

**Output:**
```
Shader Info:
  Name: Standard
  Path: Built-in
  Type: Built-in
  Render Queue: 2000
  Properties: 15

  Properties:
    - _Color (Color): Color
    - _MainTex (Texture): Albedo
    - _Metallic (Range): Metallic
    - _Glossiness (Range): Smoothness
```

---

## shader properties

详细查询 Shader 的所有属性。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw shader properties <shaderName> [options]
```

**Arguments:**
```
<shaderName>           Shader 名称
```

**Options:**
```
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 查询 Standard Shader 属性
cd <unity-project-root> && node .unity-websocket/uw shader properties "Standard"

# 查询 URP Lit Shader 属性
cd <unity-project-root> && node .unity-websocket/uw shader properties "Universal Render Pipeline/Lit"
```

**Output:**
```
Shader: Standard
Properties (15):

Name                          Type        Description
--------------------------------------------------------------------------------
_Color                        Color       Color
_MainTex                      Texture     Albedo
_Cutoff                       Range       Alpha Cutoff [0 - 1]
_Glossiness                   Range       Smoothness [0 - 1]
_Metallic                     Range       Metallic [0 - 1]
_BumpScale                    Float       Scale
_BumpMap                      Texture     Normal Map
```

---

## shader keywords

查询 Shader 关键字。可以查询全局关键字或特定 Shader 的关键字。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw shader keywords [options]
```

**Options:**
```
-g, --global           查询全局关键字（不能与 --shader 一起使用）
-s, --shader <name>    查询特定 Shader 的关键字
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Note:** 必须指定 `--global` 或 `--shader <name>` 中的一个。

**Examples:**
```bash
# 查询全局 Shader 关键字
cd <unity-project-root> && node .unity-websocket/uw shader keywords --global

# 查询 Standard Shader 的关键字
cd <unity-project-root> && node .unity-websocket/uw shader keywords -s "Standard"
```

**Output:**
```
Global Keywords (12):

Name                                    Type                Valid
-----------------------------------------------------------------
_ALPHATEST_ON                           UserDefined         Yes
_NORMALMAP                              UserDefined         Yes
_EMISSION                               UserDefined         Yes
INSTANCING_ON                           BuiltinAutoStripped Yes
```

---

## shader keyword-enable

启用全局 Shader 关键字。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw shader keyword-enable <keyword> [options]
```

**Arguments:**
```
<keyword>              要启用的关键字名称（区分大小写）
```

**Options:**
```
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 启用 HDR 关键字
cd <unity-project-root> && node .unity-websocket/uw shader keyword-enable "HDR_ON"

# 启用 Soft Particles
cd <unity-project-root> && node .unity-websocket/uw shader keyword-enable "_SOFT_PARTICLES"
```

---

## shader keyword-disable

禁用全局 Shader 关键字。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw shader keyword-disable <keyword> [options]
```

**Arguments:**
```
<keyword>              要禁用的关键字名称（区分大小写）
```

**Options:**
```
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 禁用 HDR 关键字
cd <unity-project-root> && node .unity-websocket/uw shader keyword-disable "HDR_ON"

# 禁用 Fog
cd <unity-project-root> && node .unity-websocket/uw shader keyword-disable "FOG_LINEAR"
```

---

## shader keyword-status

确认全局 Shader 关键字的启用状态。

**Usage:**
```bash
cd <unity-project-root> && node .unity-websocket/uw shader keyword-status <keyword> [options]
```

**Arguments:**
```
<keyword>              要确认的关键字名称（区分大小写）
```

**Options:**
```
--json                 以 JSON 格式输出
--timeout <ms>         WebSocket 连接超时（默认值：30000）
-h, --help             显示命令帮助
```

**Examples:**
```bash
# 确认 HDR 关键字状态
cd <unity-project-root> && node .unity-websocket/uw shader keyword-status "HDR_ON"
```

**Output:**
```
Keyword "HDR_ON" is enabled
```

---

## Common Shader Keywords

常用 Shader 关键字：

| Keyword | Description |
|---------|-------------|
| `_NORMALMAP` | 启用 Normal Map |
| `_EMISSION` | 启用 Emission |
| `_ALPHATEST_ON` | Alpha Test (Cutout) 模式 |
| `_ALPHABLEND_ON` | Alpha Blend (Transparent) 模式 |
| `_ALPHAPREMULTIPLY_ON` | Premultiplied Alpha 模式 |
| `_METALLICGLOSSMAP` | 使用 Metallic Gloss Map |
| `_SPECGLOSSMAP` | 使用 Specular Gloss Map |
| `_PARALLAXMAP` | 启用 Parallax Map |
| `FOG_LINEAR` | Linear Fog |
| `FOG_EXP` | Exponential Fog |
| `FOG_EXP2` | Exponential Squared Fog |
| `INSTANCING_ON` | 启用 GPU Instancing |

---

## Shader Property Types

| Type | Description | Example Values |
|------|-------------|----------------|
| `Color` | RGBA 颜色 | `(1, 0, 0, 1)` |
| `Vector` | 4D 向量 | `(1, 2, 3, 4)` |
| `Float` | 单个数字 | `1.5` |
| `Range` | 范围限制数字 | `0.5` (min-max) |
| `Texture` | 2D 纹理 | Texture2D |
| `Int` | 整数 | `5` |

---

## JSON Output Examples

### shader list
```json
{
  "success": true,
  "count": 3,
  "shaders": [
    {
      "name": "Standard",
      "path": "Built-in",
      "isBuiltin": true,
      "propertyCount": 15,
      "renderQueue": 2000
    },
    {
      "name": "Custom/Water",
      "path": "Assets/Shaders/Water.shader",
      "isBuiltin": false,
      "propertyCount": 8,
      "renderQueue": 3000
    }
  ]
}
```

### shader keywords
```json
{
  "success": true,
  "global": true,
  "count": 5,
  "keywords": [
    {
      "name": "_NORMALMAP",
      "type": "UserDefined",
      "isValid": true
    },
    {
      "name": "_EMISSION",
      "type": "UserDefined",
      "isValid": true
    }
  ]
}
```

---

## Render Queue Ranges

| Range | Name | Description |
|-------|------|-------------|
| 0-999 | Background | 背景（天空盒等） |
| 1000-1999 | Geometry-1 | 几何体前部 |
| 2000-2449 | Geometry | 默认不透明物体 |
| 2450-2499 | AlphaTest | Alpha 测试物体 |
| 2500-2999 | Transparent-1 | 透明前部 |
| 3000-3999 | Transparent | 默认透明物体 |
| 4000-4999 | Overlay | UI, 覆盖层 |
