---
name: Unity Script Validator
description: 验证 C# 脚本的最佳实践、性能和 Unity 模式。在审查脚本或检查代码质量时使用。
allowed-tools: Read, Grep, Glob
---

# Unity 脚本验证器

根据 Unity 游戏开发的最佳实践和性能模式验证 Unity C# 脚本。

## 检查内容

- **字段声明**：使用 `[SerializeField] private` 代替 public 字段
- **组件缓存**：在 Awake/Start 中使用 GetComponent，而不是 Update（快约 100 倍）
- **字符串操作**：频繁拼接使用 StringBuilder
- **GameObject.Find**：缓存引用，避免在 Update 中使用（O(n) 操作）
- **代码组织**：#region 指令，一致的排序
- **XML 文档**：公共方法上的 `<summary>` 标签
- **Update 与 FixedUpdate**：物理/非物理的适当用法
- **协程**：间歇性任务优先于 Update

**提供**：发现的问题、具体修复、性能影响预估、重构后的代码示例。

## 兼容性

适用于 Unity 2019.4 LTS 及更高版本（包括 Unity 6）。

详见 [patterns.md](patterns.md) 和 [examples.md](examples.md) 了解详细优化技术。

## 何时使用 vs 其他组件

**使用此技能**：快速验证现有 Unity 脚本的最佳实践和常见问题

**使用 @unity-scripter agent**：编写新代码或从头实现 Unity 功能

**使用 @unity-refactor agent**：提高代码质量、应用设计模式或现代化遗留代码

**使用 @unity-performance agent**：深度性能分析、内存优化或特定平台调优

**使用 /unity:new-script 命令**：从生产级模板创建新脚本

## 相关技能

- **unity-scene-optimizer**：用于场景级性能分析
- **unity-template-generator**：用于生成经过验证的脚本模板
