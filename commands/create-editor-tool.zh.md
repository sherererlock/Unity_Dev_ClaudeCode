---
name: create-editor-tool
description: 搭建 Unity 编辑器工具（EditorWindow、CustomEditor、PropertyDrawer、菜单项），并提供正确的结构
argument-hint: "[tool-type] [name]"
allowed-tools: [Write, Read, Bash]
---

# Unity 编辑器工具生成器

遵循 Unity 最佳实践，搭建完整的 Unity 编辑器工具。

## 目的

创建功能齐全的 Unity 编辑器工具，包含：
- 正确的 EditorOnly 保护（#if UNITY_EDITOR）
- 菜单集成
- 标准 UI 布局
- 最佳实践模式

## 工具类型

可用的工具类型：
1. **EditorWindow** - 自定义编辑器窗口，带 GUI
2. **CustomEditor** - 组件的自定义检视面板（Inspector）
3. **PropertyDrawer** - 自定义属性显示
4. **MenuItem** - 菜单命令实用程序
5. **SceneGUI** - 场景视图覆盖工具

## 使用说明

### 第 1 步：收集信息

向用户询问：
1. **工具类型**（如果未提供）
2. **工具名称**（如果未提供）
3. **目标类型**（针对 CustomEditor/PropertyDrawer）：要自定义哪个脚本/类型
4. **菜单路径**（可选）：菜单项放置的位置（默认：Tools/ProjectName/）

### 第 2 步：生成工具

根据类型创建适当的编辑器工具。

### EditorWindow 模板

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;
using System.Collections.Generic;

namespace ProjectNamespace.Editor
{
    public class ToolNameWindow : EditorWindow
    {
        // 设置
        private Vector2 scrollPosition;
        private string searchText = "";

        // 状态
        private List<GameObject> targetObjects = new List<GameObject>();

        [MenuItem("Tools/ProjectName/ToolName")]
        private static void ShowWindow()
        {
            var window = GetWindow<ToolNameWindow>();
            window.titleContent = new GUIContent("ToolName");
            window.minSize = new Vector2(400, 300);
            window.Show();
        }

        private void OnEnable()
        {
            // 窗口打开时初始化
            RefreshData();
        }

        private void OnGUI()
        {
            DrawToolbar();
            DrawMainContent();
            DrawFooter();
        }

        private void DrawToolbar()
        {
            EditorGUILayout.BeginHorizontal(EditorStyles.toolbar);

            if (GUILayout.Button("刷新", EditorStyles.toolbarButton, GUILayout.Width(60)))
            {
                RefreshData();
            }

            GUILayout.FlexibleSpace();

            searchText = EditorGUILayout.TextField(searchText, EditorStyles.toolbarSearchField);

            EditorGUILayout.EndHorizontal();
        }

        private void DrawMainContent()
        {
            scrollPosition = EditorGUILayout.BeginScrollView(scrollPosition);

            EditorGUILayout.LabelField("设置", EditorStyles.boldLabel);

            // 工具内容在此处
            EditorGUILayout.HelpBox("工具功能代码写在这里", MessageType.Info);

            EditorGUILayout.Space();

            if (GUILayout.Button("执行操作", GUILayout.Height(30)))
            {
                ExecuteAction();
            }

            EditorGUILayout.EndScrollView();
        }

        private void DrawFooter()
        {
            EditorGUILayout.BeginHorizontal();
            EditorGUILayout.LabelField($"对象数: {targetObjects.Count}", EditorStyles.miniLabel);
            EditorGUILayout.EndHorizontal();
        }

        private void RefreshData()
        {
            targetObjects.Clear();
            // 刷新逻辑
            Repaint();
        }

        private void ExecuteAction()
        {
            Debug.Log("操作已执行");
            EditorUtility.DisplayDialog("ToolName", "操作完成", "确定");
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
    [CanEditMultipleObjects]
    public class TargetScriptEditor : Editor
    {
        private SerializedProperty exampleProperty;
        private bool showAdvanced = false;

        private void OnEnable()
        {
            // 缓存序列化属性
            exampleProperty = serializedObject.FindProperty("exampleField");
        }

        public override void OnInspectorGUI()
        {
            serializedObject.Update();

            // 自定义标题
            EditorGUILayout.LabelField("TargetScript 设置", EditorStyles.boldLabel);
            EditorGUILayout.Space();

            // 基本属性
            DrawBasicSection();

            EditorGUILayout.Space();

            // 高级设置折叠页
            showAdvanced = EditorGUILayout.Foldout(showAdvanced, "高级设置", true);
            if (showAdvanced)
            {
                EditorGUI.indentLevel++;
                DrawAdvancedSection();
                EditorGUI.indentLevel--;
            }

            EditorGUILayout.Space();

            // 自定义按钮
            DrawActionButtons();

            serializedObject.ApplyModifiedProperties();
        }

        private void DrawBasicSection()
        {
            EditorGUILayout.PropertyField(exampleProperty);
        }

        private void DrawAdvancedSection()
        {
            EditorGUILayout.HelpBox("高级设置", MessageType.None);
        }

        private void DrawActionButtons()
        {
            TargetScript script = (TargetScript)target;

            EditorGUILayout.BeginHorizontal();

            if (GUILayout.Button("重置", GUILayout.Height(25)))
            {
                Undo.RecordObject(target, "重置 TargetScript");
                script.Reset();
                EditorUtility.SetDirty(target);
            }

            if (GUILayout.Button("执行", GUILayout.Height(25)))
            {
                script.Execute();
            }

            EditorGUILayout.EndHorizontal();
        }

        // 场景视图控制柄
        private void OnSceneGUI()
        {
            TargetScript script = (TargetScript)target;

            // 在场景视图中绘制控制柄
            Handles.color = Color.yellow;
            Handles.DrawWireDisc(script.transform.position, Vector3.up, 1f);
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
        private const float padding = 2f;
        private const float lineHeight = 18f;

        public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
        {
            EditorGUI.BeginProperty(position, label, property);

            // 绘制标签
            position = EditorGUI.PrefixLabel(position, GUIUtility.GetControlID(FocusType.Passive), label);

            // 不要缩进子字段
            int indent = EditorGUI.indentLevel;
            EditorGUI.indentLevel = 0;

            // 计算矩形区域
            float fieldWidth = position.width / 2 - padding;
            Rect leftRect = new Rect(position.x, position.y, fieldWidth, position.height);
            Rect rightRect = new Rect(position.x + fieldWidth + padding, position.y, fieldWidth, position.height);

            // 绘制字段
            SerializedProperty field1 = property.FindPropertyRelative("field1");
            SerializedProperty field2 = property.FindPropertyRelative("field2");

            EditorGUI.PropertyField(leftRect, field1, GUIContent.none);
            EditorGUI.PropertyField(rightRect, field2, GUIContent.none);

            // 恢复缩进
            EditorGUI.indentLevel = indent;

            EditorGUI.EndProperty();
        }

        public override float GetPropertyHeight(SerializedProperty property, GUIContent label)
        {
            return lineHeight;
        }
    }
}
#endif
```

### MenuItem 模板

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;

namespace ProjectNamespace.Editor
{
    public static class ToolNameUtility
    {
        [MenuItem("Tools/ProjectName/ToolName Action")]
        private static void ExecuteAction()
        {
            if (!ValidateAction())
            {
                EditorUtility.DisplayDialog("错误", "无法执行操作", "确定");
                return;
            }

            if (EditorUtility.DisplayDialog("确认", "对选定对象执行操作？", "执行", "取消"))
            {
                PerformAction();
            }
        }

        [MenuItem("Tools/ProjectName/ToolName Action", true)]
        private static bool ValidateAction()
        {
            return Selection.gameObjects.Length > 0;
        }

        private static void PerformAction()
        {
            Undo.RecordObjects(Selection.gameObjects, "ToolName Action");

            foreach (GameObject obj in Selection.gameObjects)
            {
                // 对每个对象执行操作
                Debug.Log($"正在处理: {obj.name}");
            }

            EditorUtility.DisplayDialog("成功", $"已处理 {Selection.gameObjects.Length} 个对象", "确定");
        }

        [MenuItem("GameObject/ToolName/Add Component", false, 10)]
        private static void AddComponentToSelected()
        {
            foreach (GameObject obj in Selection.gameObjects)
            {
                Undo.AddComponent<TargetComponent>(obj);
            }
        }
    }
}
#endif
```

### SceneGUI 模板

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;

namespace ProjectNamespace.Editor
{
    [InitializeOnLoad]
    public static class SceneViewTool
    {
        private static bool isEnabled = false;

        static SceneViewTool()
        {
            SceneView.duringSceneGui += OnSceneGUI;
        }

        [MenuItem("Tools/ProjectName/Toggle SceneView Tool")]
        private static void ToggleTool()
        {
            isEnabled = !isEnabled;
            Menu.SetChecked("Tools/ProjectName/Toggle SceneView Tool", isEnabled);
            SceneView.RepaintAll();
        }

        [MenuItem("Tools/ProjectName/Toggle SceneView Tool", true)]
        private static bool ToggleToolValidate()
        {
            Menu.SetChecked("Tools/ProjectName/Toggle SceneView Tool", isEnabled);
            return true;
        }

        private static void OnSceneGUI(SceneView sceneView)
        {
            if (!isEnabled)
                return;

            Handles.BeginGUI();

            // 绘制 UI 覆盖层
            GUILayout.BeginArea(new Rect(10, 10, 200, 100));
            GUILayout.Box("场景工具已激活");
            if (GUILayout.Button("执行"))
            {
                ExecuteSceneTool();
            }
            GUILayout.EndArea();

            Handles.EndGUI();

            // 绘制场景控制柄
            DrawSceneHandles();
        }

        private static void DrawSceneHandles()
        {
            foreach (GameObject obj in Selection.gameObjects)
            {
                Handles.color = Color.cyan;
                Handles.DrawWireDisc(obj.transform.position, Vector3.up, 1f);
            }
        }

        private static void ExecuteSceneTool()
        {
            Debug.Log("场景工具已执行");
        }
    }
}
#endif
```

### 第 3 步：创建文件

将编辑器工具放置在正确的位置：
- **EditorWindow/CustomEditor/PropertyDrawer**: `Assets/Scripts/Editor/ToolName.cs`
- 如果 Editor 文件夹不存在，则创建它
- 确保包含 #if UNITY_EDITOR 保护

### 第 4 步：确认

通知用户：
- 工具创建成功
- 如何访问（菜单路径）
- 下一步操作（如果适用）

## 应用的最佳实践

所有生成的工具均包含：
- **#if UNITY_EDITOR 保护**：仅在编辑器中编译
- **正确的命名空间**：ProjectNamespace.Editor
- **菜单集成**：易于访问
- **Undo 支持**：在适用的情况下（Undo.RecordObject）
- **验证**：菜单项验证
- **帮助消息**：EditorGUILayout.HelpBox
- **正确的布局**：BeginHorizontal/EndHorizontal
- **SerializedProperty**：用于 CustomEditor（比直接字段访问更安全）

## 示例

**示例 1：创建 EditorWindow**
```
User: /unity-dev:create-editor-tool EditorWindow LevelBuilder
Claude: 已创建 Assets/Scripts/Editor/LevelBuilderWindow.cs
通过以下路径访问：Tools/ProjectName/LevelBuilder
```

**示例 2：创建 CustomEditor**
```
User: /unity-dev:create-editor-tool CustomEditor EnemyAI
Claude: 要自定义的目标脚本是什么？
User: EnemyController
Claude: 已创建 Assets/Scripts/Editor/EnemyControllerEditor.cs
当选择 EnemyController 时自动显示
```

**示例 3：创建 MenuItem 实用程序**
```
User: /unity-dev:create-editor-tool MenuItem AlignObjects
Claude: 已创建 Assets/Scripts/Editor/AlignObjectsUtility.cs
通过以下路径访问：Tools/ProjectName/AlignObjects Action
```

## 注意

- 所有编辑器脚本均使用 #if UNITY_EDITOR 保护
- 自动放置在 Editor/ 子文件夹中
- 使用 Undo 系统进行可逆操作
- 在适当的地方包含验证函数
- 通过对话框提供用户反馈

对于所有生成的工具，请遵循 Unity 编辑器脚本编写最佳实践。
