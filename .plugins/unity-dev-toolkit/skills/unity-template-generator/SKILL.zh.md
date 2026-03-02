---
name: Unity Template Generator
description: 生成可用于生产环境的 C# 脚本模板（MonoBehaviour、ScriptableObject、Editor、测试）。在创建新脚本或设置项目结构时使用。
allowed-tools: Write, Read, Glob
---

# Unity 模板生成器

协助生成遵循最佳实践和 Unity 规范的可用于生产环境的 Unity C# 脚本模板。

## 可用模板

**MonoBehaviour** - 包含生命周期方法、序列化字段、组件缓存和 Gizmo 辅助功能的 GameObject 组件。

**ScriptableObject** - 带有 `[CreateAssetMenu]`、验证、封装和克隆方法的数据资产。

**编辑器脚本** - 自定义检查器或窗口。询问 UGUI 与 UI Toolkit 的偏好（参见 [unity-ui-selector](../unity-ui-selector/SKILL.md)）。

**测试脚本** - 带有 Setup/TearDown、`[UnityTest]`、性能测试和 Arrange-Act-Assert 模式的 NUnit/PlayMode 测试。

## 模板特性

所有模板均包含：
- Unity 编码规范（`[SerializeField]`、PascalCase、XML 文档）
- 性能模式（组件缓存、不在 Update 中使用 GetComponent）
- 代码组织（#region 指令、一致的排序）
- 安全特性（空值检查、OnValidate、Gizmos）

占位符：`{{CLASS_NAME}}`、`{{NAMESPACE}}`、`{{DESCRIPTION}}`、`{{MENU_PATH}}`、`{{FILE_NAME}}`

参见 [template-reference.md](template-reference.md) 了解详细的自定义选项。

## 何时使用 vs 其他组件

**使用此技能的场景**：讨论模板选项、了解模板特性或获取脚本结构指导时

**使用 @unity-scripter 代理的场景**：编写具有特定要求的自定义脚本或实现复杂的 Unity 功能时

**使用 @unity-refactor 代理的场景**：改进现有脚本或重构代码以获得更好的可维护性时

**使用 /unity:new-script 命令的场景**：实际使用特定参数从模板生成脚本文件时

**使用 /unity:setup-test 命令的场景**：设置包含测试脚本的完整测试环境时

## 相关技能

- **unity-script-validator**：验证生成的脚本
- **unity-ui-selector**：帮助为编辑器脚本选择 UI 系统
- **unity-uitoolkit**：在生成编辑器脚本时协助 UI Toolkit 的实现
