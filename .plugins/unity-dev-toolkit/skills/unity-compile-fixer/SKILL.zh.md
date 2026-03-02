---
name: unity-compile-fixer
description: 使用 VSCode 诊断检测并解决 Unity C# 编译错误。当 Unity 项目出现需要诊断和自动修复的编译错误时使用此技能。分析来自 VSCode 语言服务器的错误，基于错误模式提出解决方案，并处理 Unity 项目的版本控制冲突。
---

# Unity 编译修复器

## 概述

本技能利用 VSCode 的诊断系统，实现 Unity C# 编译错误的自动检测和解决。它从 OmniSharp C# 语言服务器收集实时错误，对照精心策划的常见 Unity 问题数据库分析错误模式，并在应用修复前提出结合语境的解决方案供用户批准。

## 何时使用此技能

在以下情况使用此技能：
- Unity 项目在 VSCode 中报告 C# 编译错误
- 需要诊断 Unity 编译器错误（CS* 错误代码）的根本原因
- 想要针对常见 Unity 脚本问题获取自动修复建议
- 处理集成了版本控制（Git, Unity Collaborate, Plastic SCM）的 Unity 项目
- 需要处理 Unity .meta 文件冲突

**用户请求示例：**
- "检查 Unity 编译错误并帮我修复"
- "我的 Unity 项目有编译器错误，你能诊断并修复吗？"
- "Unity 脚本无法编译，哪里出问题了？"
- "修复我 Unity 项目中的 C# 错误"

## 工作流程

调用此技能时请遵循以下工作流程：

### 1. 检测编译错误

使用 `mcp__ide__getDiagnostics` 工具从 VSCode 收集错误：

```typescript
// 收集所有项目诊断信息
mcp__ide__getDiagnostics()

// 或针对特定的 Unity 脚本文件
mcp__ide__getDiagnostics({ uri: "file:///path/to/PlayerController.cs" })
```

过滤诊断信息以关注 Unity 相关错误：
- **严重程度 (Severity)**：仅处理 `severity: "Error"` 的错误（忽略警告）
- **来源 (Source)**：仅处理 `source: "csharp"`（OmniSharp C# 诊断）
- **错误代码 (Error Codes)**：关注 CS* 编译器错误代码（例如 CS0246, CS0029, CS1061）

### 2. 分析错误模式

对于每个检测到的错误：

1. **提取错误信息：**
   - 从 `uri` 和 `range` 获取文件路径和行号
   - 从 `message` 获取错误代码（例如 "CS0246"）
   - 获取完整的错误消息文本

2. **匹配错误模式数据库：**
   - 加载 `references/error-patterns.json`
   - 查找错误代码条目（例如 CS0246）
   - 检索常见原因和解决方案

3. **读取受影响的文件上下文：**
   - 使用 Read 工具加载包含错误的文件
   - 检查周围代码的上下文
   - 识别缺失的导入、错误的类型或 API 误用

### 3. 生成解决方案建议

为每个错误创建一个结构化的修复建议：

```markdown
**错误**: CS0246 at PlayerController.cs:45
**消息**: 找不到类型或命名空间名称 'Rigidbody'

**分析**:
- 缺少 UnityEngine 命名空间的 using 指令
- 常见的 Unity API 使用模式

**建议方案**:
在 PlayerController.cs 顶部添加 `using UnityEngine;`

**所需更改**:
- 文件: Assets/Scripts/PlayerController.cs
- 操作: 在第 1 行插入 using 指令
```

### 4. 用户确认

在应用任何修复之前：

1. **以清晰、结构化的格式展示所有建议的解决方案**
2. **使用 AskUserQuestion 工具获取用户批准：**
   - 列出每个错误和建议的修复
   - 允许用户批准全部、选择特定修复或取消
3. **等待明确确认** - 不要自动应用修复

### 5. 应用已批准的修复

对于每个已批准的修复：

1. **使用 Edit 工具**修改受影响的文件
2. **保留代码格式**和现有结构
3. **应用最小更改** - 仅修复特定错误

示例：
```typescript
Edit({
  file_path: "Assets/Scripts/PlayerController.cs",
  old_string: "public class PlayerController : MonoBehaviour",
  new_string: "using UnityEngine;\n\npublic class PlayerController : MonoBehaviour"
})
```

### 6. 验证版本控制状态

应用修复后：

1. **检查 .meta 文件冲突：**
   - 使用 Grep 搜索 Unity .meta 文件
   - 验证脚本 GUID 未更改
   - 检查合并冲突标记 (<<<<<<, ======, >>>>>>)

2. **报告 VCS 状态：**
   - 列出已修改的文件
   - 警告任何 .meta 文件问题
   - 如有需要，建议 git 操作

### 7. 重新验证编译

应用修复后：

1. **重新运行诊断**，使用 `mcp__ide__getDiagnostics()`
2. **比较前后错误数量**
3. **向用户报告结果：**
   - 已修复的错误数量
   - 剩余错误（如有）
   - 成功率

## 错误模式数据库

本技能依赖 `references/error-patterns.json` 进行错误分析。此数据库包含：

- **错误代码**: CS* 编译器错误代码
- **描述**: 人类可读的解释
- **常见原因**: 此错误在 Unity 中出现的典型原因
- **解决方案**: 分步修复说明
- **Unity 特定说明**: Unity API 注意事项

要分析错误，请加载数据库并匹配错误代码：

```typescript
Read({ file_path: "references/error-patterns.json" })
// 解析 JSON 并查找 errorCode 条目
```

## 分析脚本

本技能在 `scripts/` 目录下包含用于复杂错误分析的 Node.js 脚本：

### scripts/analyze-diagnostics.js

处理 VSCode 诊断 JSON 输出并提取 Unity 相关的错误。

**用法：**
```bash
node scripts/analyze-diagnostics.js <diagnostics-json-file>
```

**输出：**
- 过滤后的 Unity C# 编译错误列表
- 按类型分类的错误（缺失导入、类型错误、API 问题）
- 严重程度和文件位置信息

此脚本可以独立运行，也可以在需要详细分析时从 SKILL.md 工作流程中调用。

## 最佳实践

使用此技能时：

1. **从全项目诊断开始** - 使用不带参数的 `mcp__ide__getDiagnostics()` 获取完整的错误概况
2. **按严重程度和依赖关系通过优先处理错误** - 在处理下游错误之前先修复基础错误（缺失导入）
3. **批量处理相关修复** - 将来自同一文件的错误分组以进行高效编辑
4. **始终验证 VCS 状态** - Unity .meta 文件对于版本控制至关重要
5. **修复后重新验证** - 确保错误确实已解决

## 资源

### scripts/analyze-diagnostics.js
用于处理 VSCode 诊断并过滤 Unity 特定 C# 错误的 Node.js 脚本。

### references/error-patterns.json
包含解决方案和 Unity 特定指导的常见 Unity C# 编译错误精心策划数据库。
