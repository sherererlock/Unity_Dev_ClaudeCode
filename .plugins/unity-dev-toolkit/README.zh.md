# Unity Dev Toolkit

> **您的 AI 驱动的 Unity 游戏开发助手**

> ⚠️ **实验性功能**
>
> 本插件目前处于 **实验阶段**。特性可能会发生变化，部分功能可能无法按预期工作。如果您遇到任何问题，请在 [GitHub Issues](https://github.com/Dev-GOM/claude-code-marketplace/issues) 上报告。
>
> **已知限制：**
> - 模板生成需要手动输入参数
> - 场景优化分析可能无法覆盖所有 Unity 版本
> - UI 系统选择（UGUI vs UI Toolkit）应根据项目需求决定
> - 技能由模型调用，可能无法在所有上下文中激活

> 一个全面的 Claude Code 插件，通过用于脚本编写、重构和优化的专门代理，加上智能自动化和生产级脚本模板，为您带来专家级的 Unity 开发辅助。

## 🌟 特性

本插件集成了三个强大的 Claude Code 功能，以增强您的 Unity 开发体验：

### 📝 斜杠命令 (Slash Commands)
快速访问 Unity 开发工具：
- `/unity:new-script` - 生成符合最佳实践的 Unity 脚本
- `/unity:optimize-scene` - 全面的场景性能分析
- `/unity:setup-test` - 创建完整的测试环境

### 🤖 专家代理 (Expert Agents)
用于 Unity 开发的专业 AI 助手：
- `@unity-scripter` - C# 脚本专家，编写整洁、高性能的代码
- `@unity-refactor` - 代码重构专家，提高质量和可维护性
- `@unity-performance` - 性能优化专家
- `@unity-architect` - 游戏系统架构顾问

### ⚡ 代理技能 (Agent Skills)
Claude 会在相关时自动使用的模型调用能力：
- **unity-script-validator** - 验证 Unity C# 脚本的最佳实践和性能
- **unity-scene-optimizer** - 分析场景的性能瓶颈
- **unity-template-generator** - 协助生成脚本模板
- **unity-ui-selector** - 根据项目需求指导 UGUI 与 UI Toolkit 的选择
- **unity-uitoolkit** - 协助 UI Toolkit 开发（UXML, USS, VisualElement API）
- **unity-compile-fixer** - 使用 VSCode 诊断检测并解决 Unity C# 编译错误
- **unity-test-runner** - 执行并分析 Unity 测试框架测试，提供详细的失败报告

## 🚀 安装

### 快速安装

```bash
# 添加市场（如果尚未添加）
/plugin marketplace add https://github.com/Dev-GOM/claude-code-marketplace.git

# 安装插件
/plugin install unity-dev-toolkit@dev-gom-plugins

# 重启 Claude Code
claude -r
```

### 验证安装

```bash
/plugin
```

您应该在已启用的插件列表中看到 "unity-dev-toolkit"。

## 📖 用法

### 创建 Unity 脚本

```bash
# 生成 MonoBehaviour 脚本
/unity:new-script MonoBehaviour PlayerController

# 生成 ScriptableObject
/unity:new-script ScriptableObject WeaponData

# 生成 Editor 脚本
/unity:new-script EditorScript CustomTool

# 生成测试脚本
/unity:new-script TestScript PlayerControllerTests
```

生成的脚本包括：
- ✅ Unity 最佳实践和约定
- ✅ 合理的区域（Region）组织
- ✅ XML 文档注释
- ✅ 性能意识模式
- ✅ 空值安全和验证
- ✅ 组件缓存
- ✅ 完整的生命周期方法

**生成的 MonoBehaviour 示例：**
```csharp
using UnityEngine;

namespace MyGame.Player
{
    /// <summary>
    /// 处理玩家移动和输入
    /// </summary>
    public class PlayerController : MonoBehaviour
    {
        #region Serialized Fields
        [SerializeField] private float moveSpeed = 5f;
        [SerializeField] private float jumpForce = 10f;
        #endregion

        #region Private Fields
        private Rigidbody rb;
        private bool isGrounded;
        #endregion

        #region Unity Lifecycle
        private void Awake()
        {
            rb = GetComponent<Rigidbody>();
        }

        private void Update()
        {
            HandleInput();
        }

        private void FixedUpdate()
        {
            ApplyMovement();
        }
        #endregion

        #region Private Methods
        private void HandleInput()
        {
            // 输入处理逻辑
        }

        private void ApplyMovement()
        {
            // 基于物理的移动
        }
        #endregion
    }
}
```

### 优化场景性能

```bash
# 分析当前场景
/unity:optimize-scene

# 分析特定场景
/unity:optimize-scene Assets/Scenes/GameLevel.unity

# 全项目分析
/unity:optimize-scene --full-project
```

优化分析涵盖：
- 🎨 **渲染 (Rendering)**：Draw calls（绘制调用）、批处理、材质、纹理
- ⚡ **物理 (Physics)**：Rigidbody、碰撞体、碰撞矩阵
- 📜 **脚本 (Scripting)**：Update 循环、组件缓存、GC 分配
- 💾 **内存 (Memory)**：纹理使用、资源加载、对象池
- 📱 **移动端 (Mobile)**：平台特定的优化

**分析输出示例：**
```markdown
# Unity 场景性能分析

## 当前指标
- Draw Calls: 250 ⚠️
- 三角形面数: 75,000 ⚠️
- 活跃 GameObjects: 450
- 脚本组件: 120

## 关键问题
1. 🔴 过多的 draw calls (250, 目标: <100)
2. 🔴 5 个未压缩的 4096x4096 纹理
3. 🟡 50+ 个对象缺少静态批处理 (static batching)

## 建议
1. 启用静态批处理...
2. 合并材质...
3. 实现对象池...

## 预估影响
- Draw calls: 250 → 80 (减少 68%)
- 帧时间: 25ms → 12ms (提升 52%)
```

### 设置测试

```bash
# 为脚本设置测试
/unity:setup-test PlayerController

# 设置 PlayMode 测试
/unity:setup-test playmode PlayerMovement

# 设置完整的测试环境
/unity:setup-test --full-project
```

生成的测试套件包括：
- ✅ 包含 Setup/TearDown 的完整测试结构
- ✅ 针对单个方法的单元测试
- ✅ 针对 Unity 生命周期的 PlayMode 测试
- ✅ 针对组件交互的集成测试
- ✅ 性能基准测试
- ✅ 边界情况覆盖
- ✅ 程序集定义文件 (Assembly definition files)

**测试示例：**
```csharp
[Test]
public void Jump_WhenGrounded_IncreasesYPosition()
{
    // Arrange (准备)
    var initialY = player.transform.position.y;

    // Act (执行)
    player.Jump();

    // Assert (断言)
    Assert.Greater(player.transform.position.y, initialY);
}
```

### 使用专家代理

您可以在对话中直接调用代理：

```
@unity-scripter create a player controller with WASD movement and jumping
（@unity-scripter 创建一个带有 WASD 移动和跳跃的玩家控制器）

@unity-performance analyze why my game is dropping to 30 fps
（@unity-performance 分析为什么我的游戏掉到了 30 fps）

@unity-architect how should I structure my inventory system?
（@unity-architect 我应该如何构建我的库存系统？）
```

**代理专长：**

**@unity-scripter**
- C# 脚本最佳实践
- Unity API 专业知识
- 组件架构
- 性能意识编码
- 代码组织

**@unity-refactor**
- 代码质量提升
- 设计模式应用
- 遗留代码现代化
- SOLID 原则
- 测试驱动重构

**@unity-performance**
- 剖析 (Profiling) 和基准测试
- 渲染优化
- 内存管理
- CPU/GPU 优化
- 平台特定调优

**@unity-architect**
- 系统设计模式
- 项目结构
- ScriptableObject 架构
- 依赖管理
- 可扩展的游戏系统

## 🔧 工作原理

### 代理技能系统

代理技能是 **模型调用 (model-invoked)** 的 - Claude 会根据您的请求自动决定何时使用它们。您不需要显式调用它们；它们会在相关时激活。

**1. 脚本验证技能**
当您要求 Claude 审查 Unity 脚本时，`unity-script-validator` 技能会自动：
- ✅ 检查公共字段（建议使用 `[SerializeField] private`）
- ✅ 检测 Update 循环中的 `GetComponent`
- ✅ 识别字符串连接问题
- ✅ 建议 XML 文档
- ✅ 推荐命名空间使用
- ✅ 检查缓存引用

**使用示例：**
```
You: Can you review this Unity script for best practices?
（您：您可以审查这个 Unity 脚本的最佳实践吗？）

Claude 激活 unity-script-validator 并提供：
🎮 Unity Script Analysis (Unity 脚本分析)

⚠️ Issues Found (发现的问题):
- GetComponent() called in Update - cache in Awake (在 Update 中调用了 GetComponent - 请在 Awake 中缓存)
- Public fields found - use [SerializeField] private (发现公共字段 - 请使用 [SerializeField] private)

💡 Suggestions (建议):
- Add XML documentation to public methods (为公共方法添加 XML 文档)
- Use #region directives to organize code (使用 #region 指令组织代码)
```

**2. 场景优化技能**
当讨论 Unity 场景性能时，`unity-scene-optimizer` 技能有助于分析：
- ⚠️ 过高的 GameObject 数量
- ⚠️ 过多的实时光源
- ⚠️ Draw call 优化
- ⚠️ 纹理压缩
- 💡 批处理机会

**3. UI 系统选择技能**
当开始 UI 开发时，`unity-ui-selector` 技能会根据以下因素指导您在 UGUI 和 UI Toolkit 之间进行选择：
- 目标 Unity 版本
- 项目复杂性
- 平台要求
- 团队经验

**4. 编译错误解决技能**
当 Unity 项目出现编译错误时，`unity-compile-fixer` 技能会自动：
- 🔍 从 VSCode 诊断（OmniSharp C# 语言服务器）收集错误
- 📊 针对常见 Unity 问题数据库分析错误模式
- 💡 提出上下文感知的解决方案供用户批准
- 🔧 应用修复，同时保留代码结构
- ✅ 验证 Unity .meta 文件的版本控制状态

**使用示例：**
```
You: My Unity project has compiler errors, can you fix them?
（您：我的 Unity 项目有编译器错误，您可以修复它们吗？）

Claude 激活 unity-compile-fixer 并提供：
🔍 Found 3 C# Compilation Errors (发现 3 个 C# 编译错误)

❌ CS0246 at PlayerController.cs:45
   The type or namespace name 'Rigidbody' could not be found
   （无法找到类型或命名空间名称 'Rigidbody'）

💡 Proposed Fix (建议的修复):
   Add 'using UnityEngine;' at the top of PlayerController.cs
   （在 PlayerController.cs 顶部添加 'using UnityEngine;'）

...
✅ Apply all fixes? [Yes/No] (应用所有修复？[是/否])
```

**5. 测试运行器技能**
当运行 Unity 测试时，`unity-test-runner` 技能会自动：
- 🔍 检测跨平台（Windows/macOS/Linux）的 Unity Editor 安装
- ⚙️ 配置测试参数（EditMode/PlayMode、类别、过滤器）
- 🚀 通过 Unity CLI 执行测试并设置适当的超时
- 📊 解析 NUnit XML 结果并提取失败详情
- 💡 针对常见测试模式分析失败原因
- 📝 生成带有文件:行号引用和修复建议的详细报告

**使用示例：**
```
You: Run all Unity tests in my project
（您：运行我项目中的所有 Unity 测试）

Claude 激活 unity-test-runner 并提供：
🧪 Unity Test Results (Unity 测试结果)

📊 Summary (摘要):
- Total Tests (总测试数): 10
- ✓ Passed (通过): 7 (70%)
- ✗ Failed (失败): 2 (20%)
- ⊘ Skipped (跳过): 1 (10%)
...
```

### 脚本模板

插件包含生产级模板：

**MonoBehaviour 模板** (`templates/MonoBehaviour.cs.template`)
- 完整的生命周期方法
- 区域组织
- 组件缓存
- XML 文档
- 验证辅助方法
- Gizmo 绘制

**ScriptableObject 模板** (`templates/ScriptableObject.cs.template`)
- CreateAssetMenu 属性
- 属性访问器
- 数据验证
- 克隆方法
- 自定义编辑器钩子

**Editor 脚本模板** (`templates/EditorScript.cs.template`)
- EditorWindow 结构
- 标签页系统
- 设置持久化
- 上下文菜单
- 进度条
- 资产实用工具

**测试脚本模板** (`templates/TestScript.cs.template`)
- 完整的测试结构
- Setup/TearDown
- PlayMode 测试
- 性能测试
- 边界情况处理
- 辅助方法

**编辑器 UI Toolkit 模板集**（3 个文件：C#, UXML, USS）
- `templates/EditorScriptUIToolkit.cs.template` - UI Toolkit EditorWindow
- `templates/EditorScriptUIToolkit.uxml.template` - UXML 结构
- `templates/EditorScriptUIToolkit.uss.template` - USS 样式
- 基于 VisualElement 的编辑器工具
- 用于元素引用的查询 API
- 事件处理系统
- EditorPrefs 设置持久化
- 针对暗色主题优化的样式

**运行时 UI Toolkit 模板集**（3 个文件：C#, UXML, USS）
- `templates/RuntimeUIToolkit.cs.template` - UIDocument MonoBehaviour
- `templates/RuntimeUIToolkit.uxml.template` - 游戏 UI 结构
- `templates/RuntimeUIToolkit.uss.template` - 游戏 UI 样式
- 完整的游戏 UI 系统（HUD、菜单、库存）
- UIDocument 集成
- 运行时事件处理
- 支持暂停的可见性控制
- 针对移动端的响应式设计

## 🎯 工作流程示例

以下是使用此插件的典型 Unity 开发工作流程：

```bash
# 1. 创建一个新的玩家控制器
/unity:new-script MonoBehaviour PlayerController
# Claude 生成一个完整的、带有文档的脚本，遵循 Unity 最佳实践

# 2. 寻求脚本专家的帮助
@unity-scripter add input handling with the new Input System
# 专家代理实现现代 Unity Input System

# 3. 要求 Claude 审查脚本
# Claude 自动使用 unity-script-validator 技能

# 4. 创建测试
/unity:setup-test PlayerController
# 生成完整的测试套件

# 5. 优化场景
/unity:optimize-scene Assets/Scenes/GameLevel.unity
# 提供全面的性能分析

# 6. 咨询架构师
@unity-architect how should I structure the enemy spawning system?
# 获取架构指导

# 7. 性能优化
@unity-performance the game is slow on mobile devices
# 获取平台特定的优化建议
```

## ⚙️ 配置

### 自定义模板

模板使用占位符，在生成过程中会被替换：
- `{{CLASS_NAME}}`：脚本类名
- `{{NAMESPACE}}`：命名空间
- `{{DESCRIPTION}}`：脚本描述
- `{{FILE_NAME}}`：输出文件名
- `{{MENU_PATH}}`：Unity 菜单路径
- `{{WINDOW_TITLE}}`：编辑器窗口标题

### 禁用技能

技能由 Claude 在相关时自动使用。为了防止使用特定技能，您可以暂时禁用插件：

```bash
/plugin disable unity-dev-toolkit
```

要重新启用：
```bash
/plugin enable unity-dev-toolkit
```

## 🎓 最佳实践

### 脚本组织

```
Assets/
├── Scripts/
│   ├── Runtime/
│   │   ├── Core/
│   │   ├── Player/
│   │   ├── Enemy/
│   │   └── Systems/
│   └── Editor/
│       └── Tools/
├── Data/
│   └── ScriptableObjects/
└── Tests/
    ├── EditMode/
    └── PlayMode/
```

### Unity 编码约定

```csharp
// ✅ Good (推荐)
[SerializeField] private float moveSpeed = 5f;
private Rigidbody rb;

void Awake()
{
    rb = GetComponent<Rigidbody>();  // Cache reference (缓存引用)
}

void Update()
{
    rb.velocity = ...;  // Use cached reference (使用缓存引用)
}

// ❌ Bad (不推荐)
public float moveSpeed = 5f;  // Public field (公共字段)

void Update()
{
    GetComponent<Rigidbody>().velocity = ...;  // Expensive! (开销大！)
}
```

### 性能模式

**对象池 (Object Pooling):**
```csharp
// Reuse objects instead of Instantiate/Destroy (重用对象而不是实例化/销毁)
public class BulletPool
{
    private Queue<Bullet> pool = new Queue<Bullet>();

    public Bullet Get()
    {
        if (pool.Count > 0)
        {
            var bullet = pool.Dequeue();
            bullet.gameObject.SetActive(true);
            return bullet;
        }
        return Instantiate(bulletPrefab);
    }

    public void Return(Bullet bullet)
    {
        bullet.gameObject.SetActive(false);
        pool.Enqueue(bullet);
    }
}
```

**避免分配 (Avoid Allocations):**
```csharp
// ❌ Bad: Allocates every frame (错误：每帧都进行内存分配)
void Update()
{
    string text = "Score: " + score.ToString();
}

// ✅ Good: No allocations (正确：无内存分配)
private StringBuilder sb = new StringBuilder(32);

void Update()
{
    sb.Clear();
    sb.Append("Score: ");
    sb.Append(score);
}
```

## 🐛 故障排除

### 插件不工作

1. 检查安装：
   ```bash
   /plugin
   ```

2. 验证 Node.js 是否已安装：
   ```bash
   node --version
   ```

3. 启用调试模式：
   ```bash
   claude --debug
   ```

### 技能未激活

技能由模型调用，Claude 决定何时使用它们。如果技能未激活：

1. 尝试让您的请求更具体
2. 提及关键字，如 "Unity script"（Unity 脚本）、"scene performance"（场景性能）或 "UI system"（UI 系统）
3. 检查插件是否已启用：`/plugin`
4. 重启 Claude Code：`claude -r`

### 代理无响应

1. 检查代理文件是否有有效的 YAML frontmatter
2. 使用正确的格式：`@unity-scripter`
3. 确保是 `.md` 扩展名，而不是 `.json`

## 🤝 贡献

欢迎贡献！您可以：

1. Fork 仓库
2. 添加新模板
3. 改进代理和技能
4. 增强命令
5. 分享您的改进

## 📄 许可证

Apache License 2.0 - 详见 [LICENSE](../../LICENSE)

## 🎮 Unity 版本兼容性

本插件适用于：
- ✅ Unity 2019.4 LTS 及更高版本
- ✅ Unity 2020.3 LTS
- ✅ Unity 2021.3 LTS
- ✅ Unity 2022.3 LTS
- ✅ Unity 6 (2023+)

## 📋 更新日志

### v1.3.0 (2025-10-22)
- 🔧 **新技能**：添加了 `unity-compile-fixer` 技能，用于自动检测和解决 C# 编译错误
- 🔍 **VSCode 集成**：利用 VSCode 诊断（OmniSharp）进行实时错误检测
- 📊 **错误模式数据库**：包含全面的 Unity C# 错误模式（CS0246, CS0029, CS1061 等）
- 💡 **智能解决方案**：根据错误分析提出上下文感知的修复建议
- ✅ **VCS 支持**：处理 Unity .meta 文件冲突和版本控制集成
- 📝 **分析脚本**：包含用于处理 VSCode 诊断的 Node.js 脚本

### v1.2.0 (2025-10-18)
- 🎨 **UI Toolkit 模板**：添加了完整的 UI Toolkit 模板，涵盖 Editor 和 Runtime（共 6 个文件）
- 📝 **Editor 模板**：带有 UXML/USS 的 EditorWindow（C#, UXML, USS）
- 🎮 **Runtime 模板**：带有 UXML/USS 的游戏 UI UIDocument（C#, UXML, USS）
- ⚡ **新技能**：添加了 `unity-uitoolkit` 技能以辅助 UI Toolkit 开发
- 📚 **模板数量**：从 7 个增加到 10 个生产级模板
- 🔗 **交叉引用**：更新了 Skills 以引用新的 UI Toolkit 功能

### v1.1.0 (2025-10-18)
- 🤖 **新代理**：添加了 `@unity-refactor` 代理，用于代码重构和质量提升
- 📝 **技能增强**：为所有 Skills 添加了“何时使用 vs 其他组件”部分
- 🔗 **组件集成**：明确了何时使用 Skills vs Agents vs Commands 的指导
- 📚 **文档**：改进了跨组件引用和使用模式

### v1.0.1 (2025-10-18)
- 📝 **技能文档优化**：简化了 SKILL.md 文件（834 → 197 行，减少 76%）
- 🎯 **渐进式披露**：应用了简洁技能文档的最佳实践
- 🗑️ **移除冗余**：消除了“何时使用此技能”部分（技能激活由描述字段决定）
- ⚡ **Token 效率**：减少了上下文大小，加快了技能加载和激活速度

### v1.0.0 (2025-10-18)
- 🎉 初始发布
- 📝 3 个斜杠命令：`/unity:new-script`, `/unity:optimize-scene`, `/unity:setup-test`
- 🤖 3 个专家代理：`@unity-scripter`, `@unity-performance`, `@unity-architect`
- ⚡ 4 个代理技能：`unity-script-validator`, `unity-scene-optimizer`, `unity-template-generator`, `unity-ui-selector`
- 📄 用于 MonoBehaviour, ScriptableObject, Editor 和 Test 脚本的生产级模板

## 🙏 致谢

为 Unity 和 Claude Code 社区创建，旨在通过智能 AI 辅助提高游戏开发生产力。

---

**祝 Unity 开发愉快！** 🚀🎮

如有问题或建议，请在 GitHub 上提交 issue。
