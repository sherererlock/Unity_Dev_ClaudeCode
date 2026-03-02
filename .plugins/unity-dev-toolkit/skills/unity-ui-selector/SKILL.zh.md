---
name: Unity UI System Selector
description: 指导项目在 UGUI 和 UI Toolkit 之间进行选择。在选择 UI 框架或迁移 UI 系统时使用。
---

# Unity UI 系统选择器

帮助您为 Unity 项目选择合适的 UI 系统，并提供 UGUI 和 UI Toolkit 的实施指南。

## 两种 UI 系统

**UGUI（传统）** - 基于 GameObject（2014）。成熟，适用于所有 Unity 版本，社区庞大。弱点：复杂 UI 性能较差，样式受限，无实时重载。

**UI Toolkit（现代）** - 保留模式，受 Web 启发的 UXML/USS（2021.2+）。性能更好，支持实时重载和数据绑定。弱点：需要 2021.2+ 版本，社区较小，3D 世界空间 UI 受限。

## 决策框架

**使用 UGUI，如果：**
- Unity < 2021.2
- 简单的 UI（菜单、HUD）
- 需要 3D 世界空间 UI
- 团队熟悉 UGUI / 截止日期紧迫
- 遗留项目

**使用 UI Toolkit，如果：**
- Unity 2021.2+ 且为新项目（面向未来）
- 复杂/数据驱动的 UI（库存、技能树）
- 编辑器工具（检查器、窗口） - **强烈推荐**
- 有 Web 开发背景（HTML/CSS）
- 大规模 UI（MMO、策略游戏）

如有疑问：对于 Unity 2021.2+ 上的新项目，**推荐使用 UI Toolkit**。

## 比较

| 特性 | UGUI | UI Toolkit |
|---------|------|-----------|
| **版本** | 4.6+ | 2021.2+ |
| **性能** | 简单 UI | 所有 UI |
| **样式** | Inspector | 类 CSS 的 USS |
| **布局** | 手动/组 | 类 Flexbox |
| **编辑器工具** | 良好 | 优秀 |
| **运行时 UI** | 优秀 | 良好 |
| **3D 世界 UI** | 优秀 | 有限 |

## 迁移

参阅 [migration-guide.md](migration-guide.md) 了解 UGUI → UI Toolkit 迁移策略（中型项目需 3-4 个月）。

## UI 系统支持矩阵

| Unity 版本 | UGUI | UI Toolkit（编辑器） | UI Toolkit（运行时） |
|--------------|------|-------------------|---------------------|
| 2019.4 LTS | ✅ 完整 | ✅ 基础 | ❌ 无 |
| 2020.3 LTS | ✅ 完整 | ✅ 良好 | ⚠️ 实验性 |
| 2021.3 LTS | ✅ 完整 | ✅ 优秀 | ✅ 生产级 |
| 2022.3 LTS+ | ✅ 完整 | ✅ 主要 | ✅ 完整 |

## 何时使用 vs 其他组件

**使用此技能，当**：在 UGUI 和 UI Toolkit 之间做选择、了解 UI 系统的权衡或规划 UI 迁移时

**使用 @unity-scripter 代理，当**：实现 UI 组件、编写自定义 UI 脚本或转换 UI 代码时

**使用 @unity-architect 代理，当**：设计复杂的 UI 架构、规划 UI 数据流或构建大规模 UI 系统时

**使用 /unity:new-script 命令，当**：使用 UI Toolkit 或 UGUI 模板生成编辑器脚本时

## 相关技能

- **unity-uitoolkit**：协助 UI Toolkit 实现（UXML、USS、VisualElement API）
- **unity-template-generator**：使用选定的 UI 系统生成编辑器脚本
- **unity-script-validator**：验证 UI 代码模式
