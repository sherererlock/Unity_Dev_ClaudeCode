---
name: Unity UI Toolkit
description: 协助 Unity UI Toolkit 开发 - 包括 UXML 结构、USS 样式、C# VisualElement 操作、数据绑定和自定义控件。在实现 UI Toolkit 界面时使用。
allowed-tools: Read, Write, Glob
---

# Unity UI Toolkit

协助 Unity UI Toolkit 开发，包括 UXML 标记、USS 样式、C# VisualElement API 和现代 UI 模式。

## 此技能的帮助范围

### UXML 结构
- 正确的元素层级和命名约定
- 常用控件：TextField（文本框）、Button（按钮）、Toggle（切换开关）、Slider（滑块）、ObjectField（对象字段）、ListView（列表视图）
- 布局容器：VisualElement、ScrollView（滚动视图）、Foldout（折叠页）、TwoPaneSplitView（双窗格拆分视图）
- 使用模板和绑定的数据驱动 UI

### USS 样式
- 基于类的样式和选择器
- Flexbox 布局（flex-direction、justify-content、align-items）
- USS 变量和深色主题优化
- 伪类（:hover、:active、:disabled）
- 过渡和动画

### C# VisualElement API
- 查询 API：`rootElement.Q<Button>("my-button")`
- 事件处理：`.clicked +=` 和 `.RegisterValueChangedCallback()`
- 使用构造函数动态创建 UI
- 使用 `Bind()` 和 `SerializedObject` 进行数据绑定

### 最佳实践
- UXML 用于结构，USS 用于样式，C# 用于逻辑
- 为元素命名以便通过查询 API 访问
- 使用类进行样式设置，而非内联样式
- 在字段中缓存 VisualElement 引用
- 在 `OnDestroy()` 中进行正确的事件清理

## 常用模式

**编辑器窗口设置：**
```csharp
public void CreateGUI() {
    var visualTree = AssetDatabase.LoadAssetAtPath<VisualTreeAsset>("path/to.uxml");
    visualTree.CloneTree(rootVisualElement);

    var button = rootVisualElement.Q<Button>("my-button");
    button.clicked += OnButtonClick;
}
```

**USS 类切换：**
```csharp
element.AddToClassList("active");
element.RemoveFromClassList("active");
element.ToggleInClassList("active");
```

**数据绑定：**
```csharp
var so = new SerializedObject(target);
rootVisualElement.Bind(so);
```

## Unity 版本要求

- **Unity 2021.2+** 用于运行时 UI Toolkit
- **Unity 2019.4+** 用于仅编辑器 UI Toolkit（功能受限）

有关完整的 API 文档，请参阅 [ui-toolkit-reference.md](ui-toolkit-reference.md)。

## 何时使用 vs 其他组件

**使用此技能的情况**：构建 UI Toolkit 界面、编写 UXML/USS 或在 C# 中操作 VisualElement

**使用 unity-ui-selector 技能的情况**：为项目在 UGUI 和 UI Toolkit 之间进行选择时

**使用 @unity-scripter 代理的情况**：实现复杂的 UI 逻辑或自定义 VisualElement 控件时

**使用 EditorScriptUIToolkit 模板的情况**：生成带有 UXML/USS 文件的新 UI Toolkit 编辑器窗口时

## 相关技能

- **unity-ui-selector**：帮助在 UGUI 和 UI Toolkit 之间进行选择
- **unity-template-generator**：生成 UI Toolkit 编辑器脚本模板
- **unity-script-validator**：验证 UI Toolkit 代码模式
