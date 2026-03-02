---
name: generate-script
description: 遵循 Unity 最佳实践生成 Unity 脚本模板（MonoBehaviour, ScriptableObject, Editor 脚本）
argument-hint: "[script-type] [script-name]"
allowed-tools: [Write, Read, Bash]
---

# Unity 脚本生成器

生成遵循最佳实践和规范的 Unity 脚本模板。

## 目的

创建结构正确的 Unity 脚本，包含：
- 正确的命名空间和文件组织
- Unity 生命周期方法（如适用）
- 最佳实践模式（缓存、序列化）
- 针对编辑器脚本的正确 EditorOnly 保护

## 脚本类型

可用的脚本类型：
1. **MonoBehaviour** - 标准 Unity 组件
2. **ScriptableObject** - 数据资产
3. **EditorWindow** - 自定义编辑器窗口
4. **CustomEditor** - 自定义检视面板（Inspector）
5. **PropertyDrawer** - 自定义属性绘制器

## 使用说明

### 第 1 步：收集信息

询问用户：
1. **脚本类型**（如果未提供）：MonoBehaviour, ScriptableObject, EditorWindow, CustomEditor, 或 PropertyDrawer
2. **脚本名称**（如果未提供）：帕斯卡命名法（PascalCase）名称（例如 "PlayerController", "WeaponData"）
3. **命名空间**（可选）：项目命名空间（默认：使用当前目录的项目名称）
4. **位置**（可选）：创建脚本的位置（默认：Assets/Scripts/）

**交互式提示：**
```
如果未提供脚本类型："What type of script? (MonoBehaviour/ScriptableObject/EditorWindow/CustomEditor/PropertyDrawer)"
如果未提供脚本名称："What should the script be called?"
如果需要命名空间："Use custom namespace? (default: ProjectName)"
如果需要位置："Where should the script be created? (default: Assets/Scripts/)"
```

### 第 2 步：验证输入

检查：
- 脚本名称是否为有效的 C# 标识符（帕斯卡命名法，无空格）
- 目标目录是否存在
- 文件是否已存在（如果存在则发出警告）

### 第 3 步：生成脚本

根据类型使用适当的模板创建脚本文件。

### MonoBehaviour 模板

```csharp
using UnityEngine;

namespace ProjectNamespace
{
    public class ScriptName : MonoBehaviour
    {
        [Header("Settings")]
        [SerializeField] private float exampleValue = 1f;

        private void Awake()
        {
            // 初始化组件引用
        }

        private void Start()
        {
            // 在所有 Awake 调用之后初始化
        }

        private void Update()
        {
            // 帧更新逻辑
        }
    }
}
```

### ScriptableObject 模板

```csharp
using UnityEngine;

namespace ProjectNamespace
{
    [CreateAssetMenu(fileName = "ScriptName", menuName = "ProjectName/ScriptName")]
    public class ScriptName : ScriptableObject
    {
        [Header("Data")]
        [SerializeField] private string dataName;
        [SerializeField] private int value;

        public string DataName => dataName;
        public int Value => value;
    }
}
```

### EditorWindow 模板

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;

namespace ProjectNamespace.Editor
{
    public class ScriptName : EditorWindow
    {
        private string textField = "";
        private int intField = 0;

        [MenuItem("Window/ProjectName/ScriptName")]
        private static void ShowWindow()
        {
            var window = GetWindow<ScriptName>();
            window.titleContent = new GUIContent("ScriptName");
            window.minSize = new Vector2(400, 300);
            window.Show();
        }

        private void OnGUI()
        {
            GUILayout.Label("ScriptName", EditorStyles.boldLabel);

            textField = EditorGUILayout.TextField("Text Field", textField);
            intField = EditorGUILayout.IntField("Int Field", intField);

            if (GUILayout.Button("Execute"))
            {
                Execute();
            }
        }

        private void Execute()
        {
            Debug.Log($"Executed: {textField}, {intField}");
        }
    }
}
#endif
```

### CustomEditor 模板

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;

namespace ProjectNamespace.Editor
{
    [CustomEditor(typeof(TargetScript))]
    public class TargetScriptEditor : Editor
    {
        public override void OnInspectorGUI()
        {
            DrawDefaultInspector();

            TargetScript script = (TargetScript)target;

            EditorGUILayout.Space();

            if (GUILayout.Button("Custom Action"))
            {
                script.CustomAction();
                EditorUtility.SetDirty(target);
            }
        }
    }
}
#endif
```

### PropertyDrawer 模板

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;

namespace ProjectNamespace.Editor
{
    [CustomPropertyDrawer(typeof(PropertyType))]
    public class PropertyTypeDrawer : PropertyDrawer
    {
        public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
        {
            EditorGUI.BeginProperty(position, label, property);

            position = EditorGUI.PrefixLabel(position, GUIUtility.GetControlID(FocusType.Passive), label);

            // 在此处绘制自定义属性字段

            EditorGUI.EndProperty();
        }

        public override float GetPropertyHeight(SerializedProperty property, GUIContent label)
        {
            return EditorGUIUtility.singleLineHeight;
        }
    }
}
#endif
```

### 第 4 步：创建文件

使用 Write 工具在指定位置创建脚本：
- 对于运行时脚本：`Assets/Scripts/ScriptName.cs`
- 对于编辑器脚本：`Assets/Scripts/Editor/ScriptName.cs`

如果目录不存在，则创建必要的目录。

### 第 5 步：确认

通知用户：
- 脚本创建成功
- 文件位置
- 下一步操作（例如，对于 MonoBehaviour 是 "Attach to GameObject"（附加到游戏对象），对于 ScriptableObject 是 "Create asset"（创建资产））

## 应用的最佳实践

生成的脚本包含：
- **正确的命名空间**：组织代码
- **SerializeField 优于 public**：封装
- **Header 属性**：检视面板组织
- **缓存模式**：在 Awake 中获取组件 (GetComponent)
- **EditorOnly 保护**：编辑器脚本使用 #if UNITY_EDITOR
- **CreateAssetMenu**：用于 ScriptableObjects
- **MenuItem**：用于编辑器工具

## 示例

**示例 1：生成 MonoBehaviour**
```
User: /unity-dev:generate-script
Claude: What type of script? (MonoBehaviour/ScriptableObject/EditorWindow/CustomEditor/PropertyDrawer)
User: MonoBehaviour
Claude: What should the script be called?
User: PlayerController
Claude: Created Assets/Scripts/PlayerController.cs with standard MonoBehaviour template
```

**示例 2：生成 ScriptableObject**
```
User: /unity-dev:generate-script ScriptableObject WeaponData
Claude: Created Assets/Scripts/WeaponData.cs with CreateAssetMenu attribute
Next steps: Create asset via Assets > Create > ProjectName > WeaponData
```

**示例 3：生成 EditorWindow**
```
User: /unity-dev:generate-script EditorWindow LevelEditor
Claude: Created Assets/Scripts/Editor/LevelEditor.cs
Access via Window > ProjectName > LevelEditor
```

## 注意事项

- 在创建之前，始终使用 Read 工具检查文件是否已存在
- 如果目录不存在，使用 Bash 创建目录
- 对于编辑器脚本，自动将其放置在 Editor/ 子文件夹中
- 为所有编辑器脚本包含 #if UNITY_EDITOR 保护
- 使用项目目录名称作为默认命名空间

对所有生成的脚本遵循 Unity 命名约定和最佳实践。
