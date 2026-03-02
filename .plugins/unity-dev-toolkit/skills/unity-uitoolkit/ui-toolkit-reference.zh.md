# UI Toolkit 快速参考

## 常用 VisualElement 类型

### 输入控件 (Input Controls)
- `TextField` - 单行/多行文本输入
- `IntegerField`, `FloatField`, `Vector3Field` - 数值输入
- `Toggle` - 布尔复选框
- `Button` - 可点击按钮
- `Slider` - 带可选输入字段的数值滑动条
- `EnumField` - 枚举值下拉菜单
- `ObjectField` - Unity 对象引用选择器

### 布局容器 (Layout Containers)
- `VisualElement` - 通用容器 (类似 `<div>`)
- `ScrollView` - 可滚动区域
- `Foldout` - 可折叠部分
- `TwoPaneSplitView` - 可调整大小的分裂面板
- `ListView` - 带虚拟化的数据驱动列表

### 显示元素 (Display Elements)
- `Label` - 文本显示
- `Image` - 精灵/纹理显示
- `HelpBox` - 信息/警告/错误消息框
- `ProgressBar` - 进度指示器

## USS Flexbox 布局

```css
.container {
    flex-direction: row; /* 或 column */
    justify-content: flex-start; /* flex-end, center, space-between */
    align-items: stretch; /* flex-start, flex-end, center */
    flex-grow: 1;
    flex-shrink: 0;
}
```

## USS 常用属性

```css
/* 间距 */
margin: 10px;
padding: 5px 10px;

/* 尺寸 */
width: 200px;
height: 100px;
min-width: 50px;
max-height: 300px;

/* 背景 */
background-color: rgb(50, 50, 50);
background-image: url('path/to/image.png');

/* 边框 */
border-width: 1px;
border-color: rgba(255, 255, 255, 0.2);
border-radius: 4px;

/* 文本 */
color: rgb(200, 200, 200);
font-size: 14px;
-unity-font-style: bold; /* 或 italic */
-unity-text-align: middle-center;
```

## 查询 API 示例

```csharp
// 按名称查询 (必须在 UXML 中设置 name 属性)
var button = root.Q<Button>("my-button");

// 按类名查询
var items = root.Query<VisualElement>(className: "item").ToList();

// 获取第一个匹配项
var firstLabel = root.Q<Label>();

// 获取所有匹配项
var allButtons = root.Query<Button>().ToList();

// 复杂查询
var activeItems = root.Query<VisualElement>()
    .Where(e => e.ClassListContains("active"))
    .ToList();
```

## 事件处理

```csharp
// 按钮点击
button.clicked += () => Debug.Log("Clicked!");

// 值改变
textField.RegisterValueChangedCallback(evt => {
    Debug.Log($"Changed: {evt.previousValue} -> {evt.newValue}");
});

// 鼠标事件
element.RegisterCallback<MouseDownEvent>(evt => {
    Debug.Log($"Mouse down at {evt.localMousePosition}");
});

// 清理
void OnDestroy() {
    button.clicked -= OnButtonClick;
}
```

## 数据绑定

```csharp
// 绑定到 SerializedObject
var so = new SerializedObject(targetObject);
rootVisualElement.Bind(so);

// 手动绑定
var property = so.FindProperty("fieldName");
var field = new PropertyField(property);
field.BindProperty(property);
```

## 自定义 VisualElement

```csharp
public class CustomElement : VisualElement
{
    public new class UxmlFactory : UxmlFactory<CustomElement, UxmlTraits> { }

    public new class UxmlTraits : VisualElement.UxmlTraits
    {
        UxmlStringAttribute customAttribute = new UxmlStringAttribute
            { name = "custom-value" };

        public override void Init(VisualElement ve, IUxmlAttributes bag,
            CreationContext cc)
        {
            base.Init(ve, bag, cc);
            ((CustomElement)ve).customValue = customAttribute.GetValueFromBag(bag, cc);
        }
    }

    private string customValue;

    public CustomElement()
    {
        AddToClassList("custom-element");
    }
}
```

## 性能提示

1.  **使用 USS 类** 而不是内联样式
2.  **缓存 VisualElement 引用** 而不是重复查询
3.  **使用 ListView** 处理大型列表 (虚拟化)
4.  **避免过度重建** - 仅更新更改的元素
5.  **使用 USS 变量** 以获得可维护的主题
6.  **最小化 UXML 嵌套** 以获得更好的性能

## 常见陷阱

❌ **在 CreateGUI 完成之前查询**
```csharp
// 错误
void OnEnable() {
    var button = rootVisualElement.Q<Button>(); // null!
}

// 正确
public void CreateGUI() {
    visualTree.CloneTree(rootVisualElement);
    var button = rootVisualElement.Q<Button>(); // works!
}
```

❌ **忘记命名元素**
```xml
<!-- 错误: 无法按名称查询 -->
<ui:Button text="Click" />

<!-- 正确 -->
<ui:Button name="my-button" text="Click" />
```

❌ **未清理事件**
```csharp
// 内存泄漏!
button.clicked += OnClick;

// 正确
void OnDestroy() {
    button.clicked -= OnClick;
}
```

## 资源

- Unity 手册: UI Toolkit
- Unity 脚本 API: UnityEngine.UIElements
- Unity 论坛: UI Toolkit 版块
- 示例项目: GitHub 上的 UI Toolkit 示例
