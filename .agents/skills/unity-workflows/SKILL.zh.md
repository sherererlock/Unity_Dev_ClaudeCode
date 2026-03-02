---
name: Unity Workflows
description: 当用户询问“Unity 编辑器脚本”、“自定义检查器”、“EditorWindow”、“PropertyDrawer”、“Unity 输入系统”、“新输入系统”、“UI 工具包”、“uGUI”、“Canvas”、“资产管理”、“AssetDatabase”、“构建管线”、“编辑器实用工具”，或需要关于 Unity 编辑器扩展、输入处理、UI 系统和工作流优化的指导时，应使用此技能。
version: 0.1.0
---

# Unity 工作流与编辑器工具

Unity 编辑器脚本、输入系统、UI 开发和资产管理的基本工作流。

## 概述

高效的 Unity 工作流可以加速开发并减少错误。本技能涵盖编辑器定制、现代输入处理、UI 系统和资产管线优化。

**核心工作流领域：**
- 编辑器脚本和自定义工具
- 输入系统（新版和旧版）
- UI 开发（UI Toolkit, uGUI）
- 资产管理和构建管线

## 编辑器脚本基础

### 菜单项 (Menu Items)

创建自定义菜单命令：

```csharp
using UnityEditor;

public static class CustomMenu
{
    [MenuItem("Tools/My Tool")]
    private static void ExecuteTool()
    {
        Debug.Log("Custom tool executed");
    }

    [MenuItem("Tools/My Tool", true)]  // 验证
    private static bool ValidateTool()
    {
        return Selection.activeGameObject != null;
    }

    // 快捷键：Ctrl+Shift+T (Windows), Cmd+Shift+T (Mac)
    [MenuItem("Tools/Quick Action %#t")]
    private static void QuickAction()
    {
        // 执行操作
    }
}
```

**快捷键：**
- `%` = Ctrl (Mac 上为 Cmd)
- `#` = Shift
- `&` = Alt
- `_g` = G 键

### 上下文菜单 (Context Menus)

游戏对象/组件的右键上下文菜单：

```csharp
[MenuItem("GameObject/My Custom Action", false, 10)]
private static void CustomAction()
{
    GameObject selected = Selection.activeGameObject;
    // 执行操作
}

// 组件上下文菜单
public class MyComponent : MonoBehaviour
{
    [ContextMenu("Do Something")]
    private void DoSomething()
    {
        Debug.Log("Context menu action");
    }

    [ContextMenuItem("Reset Value", "ResetValue")]
    [SerializeField] private int value;

    private void ResetValue()
    {
        value = 0;
    }
}
```

### 自定义检查器 (Custom Inspector)

重写自定义类型的检查器：

```csharp
[CustomEditor(typeof(MyScript))]
public class MyScriptEditor : Editor
{
    public override void OnInspectorGUI()
    {
        // 绘制默认检查器
        DrawDefaultInspector();

        MyScript script = (MyScript)target;

        // 自定义按钮
        if (GUILayout.Button("Execute Action"))
        {
            script.ExecuteAction();
        }

        // 自定义字段
        EditorGUILayout.LabelField("Custom Section", EditorStyles.boldLabel);
        script.customValue = EditorGUILayout.IntSlider("Custom Value", script.customValue, 0, 100);

        // 应用更改
        if (GUI.changed)
        {
            EditorUtility.SetDirty(target);
        }
    }
}
```

### 属性绘制器 (Property Drawers)

在检查器中自定义属性显示：

```csharp
[System.Serializable]
public class Range
{
    public float min;
    public float max;
}

[CustomPropertyDrawer(typeof(Range))]
public class RangeDrawer : PropertyDrawer
{
    public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
    {
        EditorGUI.BeginProperty(position, label, property);

        position = EditorGUI.PrefixLabel(position, GUIUtility.GetControlID(FocusType.Passive), label);

        var minProp = property.FindPropertyRelative("min");
        var maxProp = property.FindPropertyRelative("max");

        float halfWidth = position.width / 2;

        Rect minRect = new Rect(position.x, position.y, halfWidth - 5, position.height);
        Rect maxRect = new Rect(position.x + halfWidth, position.y, halfWidth - 5, position.height);

        EditorGUI.PropertyField(minRect, minProp, GUIContent.none);
        EditorGUI.PropertyField(maxRect, maxProp, GUIContent.none);

        EditorGUI.EndProperty();
    }
}
```

### 编辑器窗口 (EditorWindow)

创建自定义编辑器窗口：

```csharp
public class MyEditorWindow : EditorWindow
{
    private string textField = "";
    private int intField = 0;

    [MenuItem("Window/My Editor Window")]
    private static void ShowWindow()
    {
        var window = GetWindow<MyEditorWindow>();
        window.titleContent = new GUIContent("My Tool");
        window.minSize = new Vector2(300, 200);
        window.Show();
    }

    private void OnGUI()
    {
        GUILayout.Label("My Custom Tool", EditorStyles.boldLabel);

        textField = EditorGUILayout.TextField("Text Field", textField);
        intField = EditorGUILayout.IntField("Int Field", intField);

        if (GUILayout.Button("Execute"))
        {
            Execute();
        }
    }

    private void Execute()
    {
        Debug.Log($"Executed with: {textField}, {intField}");
    }
}
```

### 资产数据库操作 (AssetDatabase Operations)

以编程方式操作资产：

```csharp
using UnityEditor;

public static class AssetUtilities
{
    [MenuItem("Assets/Create Prefab From Selection")]
    private static void CreatePrefab()
    {
        GameObject selected = Selection.activeGameObject;
        if (selected == null)
            return;

        string path = $"Assets/Prefabs/{selected.name}.prefab";

        // 创建预制件
        PrefabUtility.SaveAsPrefabAsset(selected, path);

        AssetDatabase.Refresh();
    }

    public static T LoadAsset<T>(string path) where T : UnityEngine.Object
    {
        return AssetDatabase.LoadAssetAtPath<T>(path);
    }

    public static void CreateFolder(string path)
    {
        if (!AssetDatabase.IsValidFolder(path))
        {
            AssetDatabase.CreateFolder(System.IO.Path.GetDirectoryName(path), System.IO.Path.GetFileName(path));
        }
    }
}
```

## Unity 输入系统 (Unity Input System)

### 新输入系统设置

**安装：Window > Package Manager > Input System**

**创建输入动作 (Input Actions)：**
1. Create > Input Actions
2. 添加动作映射 (Action Maps)（玩家、UI 等）
3. 添加动作 (Actions)（移动、跳跃、开火等）
4. 绑定输入（键盘、手柄等）
5. 生成 C# 类 (Inspector > Generate C# Class)

### 使用输入动作

```csharp
using UnityEngine.InputSystem;

public class PlayerController : MonoBehaviour
{
    private PlayerInput playerInput;
    private InputAction moveAction;
    private InputAction jumpAction;

    private void Awake()
    {
        playerInput = GetComponent<PlayerInput>();

        moveAction = playerInput.actions["Move"];
        jumpAction = playerInput.actions["Jump"];
    }

    private void OnEnable()
    {
        jumpAction.performed += OnJump;
    }

    private void OnDisable()
    {
        jumpAction.performed -= OnJump;
    }

    private void Update()
    {
        Vector2 moveInput = moveAction.ReadValue<Vector2>();
        Move(moveInput);
    }

    private void OnJump(InputAction.CallbackContext context)
    {
        Jump();
    }

    private void Move(Vector2 input) { }
    private void Jump() { }
}
```

### 输入动作资产（生成的 C#）

```csharp
// 从 Input Actions 资产生成 C# 类后
private PlayerInputActions inputActions;

private void Awake()
{
    inputActions = new PlayerInputActions();
}

private void OnEnable()
{
    inputActions.Player.Enable();
    inputActions.Player.Jump.performed += OnJump;
}

private void OnDisable()
{
    inputActions.Player.Disable();
    inputActions.Player.Jump.performed -= OnJump;
}

private void Update()
{
    Vector2 move = inputActions.Player.Move.ReadValue<Vector2>();
}
```

**优点：**
- 类型安全的动作访问
- 自动补全
- 编译时检查

### 旧版输入系统 (Legacy Input System)

```csharp
// 仍然可用，适用于基础游戏
private void Update()
{
    // 键盘
    if (Input.GetKeyDown(KeyCode.Space))
        Jump();

    // 鼠标
    if (Input.GetMouseButtonDown(0))
        Fire();

    // 轴（在 Edit > Project Settings > Input Manager 中配置）
    float horizontal = Input.GetAxis("Horizontal");
    float vertical = Input.GetAxis("Vertical");

    Move(new Vector2(horizontal, vertical));
}
```

**适用于：**
- 简单原型
- 移动端触摸输入
- 旧项目

**新输入系统更适合：**
- 多平台支持
- 可重新绑定的控制
- 复杂的输入处理

## UI 系统

### uGUI（基于画布的 UI）

标准 Unity UI 系统：

```csharp
using UnityEngine.UI;
using TMPro;  // TextMeshPro

public class UIManager : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI scoreText;
    [SerializeField] private Button playButton;
    [SerializeField] private Slider healthSlider;

    private void Start()
    {
        playButton.onClick.AddListener(OnPlayClicked);
    }

    private void OnDestroy()
    {
        playButton.onClick.RemoveListener(OnPlayClicked);
    }

    public void UpdateScore(int score)
    {
        scoreText.text = $"Score: {score}";
    }

    public void UpdateHealth(float health, float maxHealth)
    {
        healthSlider.value = health / maxHealth;
    }

    private void OnPlayClicked()
    {
        Debug.Log("Play button clicked");
    }
}
```

**画布渲染模式：**
- **Screen Space - Overlay**：UI 始终在最上层，无需相机（最快）
- **Screen Space - Camera**：UI 由相机渲染，支持后期处理
- **World Space**：游戏世界中的 3D UI

**UI 优化：**
- 将静态/动态 UI 分离到不同的画布
- 禁用非交互元素的射线检测目标 (raycast targets)
- 使用精灵图集 (sprite atlases)
- 最小化布局组 (Layout Groups) 的使用（开销大）

### UI Toolkit（现代 UI）

运行时和编辑器 UI（Unity 2021+）：

**UXML（UI 结构）：**
```xml
<ui:UXML>
    <ui:VisualElement name="root">
        <ui:Label text="Score: 0" name="scoreLabel"/>
        <ui:Button text="Play" name="playButton"/>
    </ui:VisualElement>
</ui:UXML>
```

**USS（样式）：**
```css
.root {
    flex-grow: 1;
    background-color: rgb(50, 50, 50);
}

#scoreLabel {
    font-size: 24px;
    color: white;
}
```

**C#（逻辑）：**
```csharp
using UnityEngine.UIElements;

public class UIController : MonoBehaviour
{
    private UIDocument uiDocument;
    private Label scoreLabel;
    private Button playButton;

    private void Awake()
    {
        uiDocument = GetComponent<UIDocument>();

        var root = uiDocument.rootVisualElement;
        scoreLabel = root.Q<Label>("scoreLabel");
        playButton = root.Q<Button>("playButton");

        playButton.clicked += OnPlayClicked;
    }

    public void UpdateScore(int score)
    {
        scoreLabel.text = $"Score: {score}";
    }

    private void OnPlayClicked()
    {
        Debug.Log("Play clicked");
    }
}
```

**优点：**
- 更快的性能（保留模式）
- 更适合复杂 UI
- 现代类似 Web 的工作流（等同于 HTML/CSS）
- 与编辑器 UI 共享

**适用于：**
- 新项目 (Unity 2021+)
- 复杂 UI
- 编辑器工具

## 资产管理

### Addressables（可寻址系统）

异步资产加载和内存管理：

**设置：**
1. Window > Asset Management > Addressables > Groups
2. 将资产标记为 Addressable
3. 组织到组中

**加载资产：**
```csharp
using UnityEngine.AddressableAssets;
using UnityEngine.ResourceManagement.AsyncOperations;

public class AssetLoader : MonoBehaviour
{
    private async void Start()
    {
        // 异步加载资产
        var handle = Addressables.LoadAssetAsync<GameObject>("Enemy");
        await handle.Task;

        if (handle.Status == AsyncOperationStatus.Succeeded)
        {
            GameObject prefab = handle.Result;
            Instantiate(prefab);
        }

        // 完成后释放
        Addressables.Release(handle);
    }

    // 直接实例化
    private async void SpawnEnemy()
    {
        var handle = Addressables.InstantiateAsync("Enemy");
        await handle.Task;

        GameObject enemy = handle.Result;
        // 使用 enemy
    }
}
```

**优点：**
- 异步加载（无帧卡顿）
- 内存管理
- 远程内容更新
- 构建优化

**适用于：**
- 大型游戏
- DLC/实时更新
- 手机游戏（减少安装包大小）

### Resources 文件夹（旧版）

简单的资产加载：

```csharp
// 从 Resources 文件夹加载
GameObject prefab = Resources.Load<GameObject>("Prefabs/Enemy");

// 所有资产都会包含在构建中 - 低效
```

**避免：** 新项目请改用 Addressables。

## 构建管线

### 构建设置

```csharp
using UnityEditor;
using UnityEditor.Build.Reporting;

public static class BuildUtility
{
    [MenuItem("Build/Build Windows")]
    private static void BuildWindows()
    {
        BuildPlayerOptions options = new BuildPlayerOptions
        {
            scenes = new[] { "Assets/Scenes/MainMenu.unity", "Assets/Scenes/Game.unity" },
            locationPathName = "Builds/Windows/Game.exe",
            target = BuildTarget.StandaloneWindows64,
            options = BuildOptions.None
        };

        BuildReport report = BuildPipeline.BuildPlayer(options);

        if (report.summary.result == BuildResult.Succeeded)
        {
            Debug.Log($"Build succeeded: {report.summary.totalSize} bytes");
        }
        else
        {
            Debug.LogError($"Build failed: {report.summary.result}");
        }
    }
}
```

### 构建预处理

```csharp
using UnityEditor.Build;
using UnityEditor.Build.Reporting;

public class BuildPreprocessor : IPreprocessBuildWithReport
{
    public int callbackOrder => 0;

    public void OnPreprocessBuild(BuildReport report)
    {
        Debug.Log("Preprocessing build...");

        // 验证资产
        // 更新版本号
        // 生成构建信息
    }
}
```

## 最佳实践

✅ **建议 (DO)：**
- 使用编辑器脚本自动化重复任务
- 为复杂组件实现自定义检查器
- 为多平台项目使用新输入系统
- 使用分离的画布来优化静态/动态内容的 UI
- 对大型游戏和移动端使用 Addressables
- 为策划/美术创建编辑器工具
- 在构建前验证资产

❌ **不建议 (DON'T)：**
- 硬编码编辑器路径（使用 AssetDatabase）
- 忘记取消订阅输入系统的动作
- 过度混合使用布局组（UI 性能杀手）
- 在新项目中使用 Resources 文件夹（使用 Addressables）
- 在 Update 循环中创建 UI
- 跳过关键脚本的编辑器验证

**黄金法则**：使用编辑器工具自动化工作流。花在创建工具上的时间会在迭代中成倍地节省回来。

---

应用这些工作流优化和现代系统，以实现高效、可扩展的 Unity 开发。
