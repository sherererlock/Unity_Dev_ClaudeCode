---
name: create-editor-tool
description: Scaffold Unity editor tools (EditorWindow, CustomEditor, PropertyDrawer, Menu items) with proper structure
argument-hint: "[tool-type] [name]"
allowed-tools: [Write, Read, Bash]
---

# Unity Editor Tool Creator

Scaffold complete Unity editor tools following Unity best practices.

## Purpose

Create fully functional Unity editor tools with:
- Proper EditorOnly guards (#if UNITY_EDITOR)
- Menu integration
- Standard UI layouts
- Best practice patterns

## Tool Types

Available tool types:
1. **EditorWindow** - Custom editor window with GUI
2. **CustomEditor** - Custom Inspector for components
3. **PropertyDrawer** - Custom property display
4. **MenuItem** - Menu command utility
5. **SceneGUI** - Scene view overlay tool

## Usage Instructions

### Step 1: Gather Information

Ask the user for:
1. **Tool type** (if not provided)
2. **Tool name** (if not provided)
3. **Target type** (for CustomEditor/PropertyDrawer): Which script/type to customize
4. **Menu path** (optional): Where to place menu item (default: Tools/ProjectName/)

### Step 2: Generate Tool

Create appropriate editor tool based on type.

### EditorWindow Template

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;
using System.Collections.Generic;

namespace ProjectNamespace.Editor
{
    public class ToolNameWindow : EditorWindow
    {
        // Settings
        private Vector2 scrollPosition;
        private string searchText = "";

        // State
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
            // Initialize when window opens
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

            if (GUILayout.Button("Refresh", EditorStyles.toolbarButton, GUILayout.Width(60)))
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

            EditorGUILayout.LabelField("Settings", EditorStyles.boldLabel);

            // Tool content here
            EditorGUILayout.HelpBox("Tool functionality goes here", MessageType.Info);

            EditorGUILayout.Space();

            if (GUILayout.Button("Execute Action", GUILayout.Height(30)))
            {
                ExecuteAction();
            }

            EditorGUILayout.EndScrollView();
        }

        private void DrawFooter()
        {
            EditorGUILayout.BeginHorizontal();
            EditorGUILayout.LabelField($"Objects: {targetObjects.Count}", EditorStyles.miniLabel);
            EditorGUILayout.EndHorizontal();
        }

        private void RefreshData()
        {
            targetObjects.Clear();
            // Refresh logic
            Repaint();
        }

        private void ExecuteAction()
        {
            Debug.Log("Action executed");
            EditorUtility.DisplayDialog("ToolName", "Action completed", "OK");
        }
    }
}
#endif
```

### CustomEditor Template

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
            // Cache serialized properties
            exampleProperty = serializedObject.FindProperty("exampleField");
        }

        public override void OnInspectorGUI()
        {
            serializedObject.Update();

            // Custom header
            EditorGUILayout.LabelField("TargetScript Settings", EditorStyles.boldLabel);
            EditorGUILayout.Space();

            // Basic properties
            DrawBasicSection();

            EditorGUILayout.Space();

            // Advanced foldout
            showAdvanced = EditorGUILayout.Foldout(showAdvanced, "Advanced Settings", true);
            if (showAdvanced)
            {
                EditorGUI.indentLevel++;
                DrawAdvancedSection();
                EditorGUI.indentLevel--;
            }

            EditorGUILayout.Space();

            // Custom buttons
            DrawActionButtons();

            serializedObject.ApplyModifiedProperties();
        }

        private void DrawBasicSection()
        {
            EditorGUILayout.PropertyField(exampleProperty);
        }

        private void DrawAdvancedSection()
        {
            EditorGUILayout.HelpBox("Advanced settings", MessageType.None);
        }

        private void DrawActionButtons()
        {
            TargetScript script = (TargetScript)target;

            EditorGUILayout.BeginHorizontal();

            if (GUILayout.Button("Reset", GUILayout.Height(25)))
            {
                Undo.RecordObject(target, "Reset TargetScript");
                script.Reset();
                EditorUtility.SetDirty(target);
            }

            if (GUILayout.Button("Execute", GUILayout.Height(25)))
            {
                script.Execute();
            }

            EditorGUILayout.EndHorizontal();
        }

        // Scene view handles
        private void OnSceneGUI()
        {
            TargetScript script = (TargetScript)target;

            // Draw handles in scene view
            Handles.color = Color.yellow;
            Handles.DrawWireDisc(script.transform.position, Vector3.up, 1f);
        }
    }
}
#endif
```

### PropertyDrawer Template

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

            // Draw label
            position = EditorGUI.PrefixLabel(position, GUIUtility.GetControlID(FocusType.Passive), label);

            // Don't indent child fields
            int indent = EditorGUI.indentLevel;
            EditorGUI.indentLevel = 0;

            // Calculate rects
            float fieldWidth = position.width / 2 - padding;
            Rect leftRect = new Rect(position.x, position.y, fieldWidth, position.height);
            Rect rightRect = new Rect(position.x + fieldWidth + padding, position.y, fieldWidth, position.height);

            // Draw fields
            SerializedProperty field1 = property.FindPropertyRelative("field1");
            SerializedProperty field2 = property.FindPropertyRelative("field2");

            EditorGUI.PropertyField(leftRect, field1, GUIContent.none);
            EditorGUI.PropertyField(rightRect, field2, GUIContent.none);

            // Restore indent
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

### MenuItem Template

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
                EditorUtility.DisplayDialog("Error", "Action cannot be executed", "OK");
                return;
            }

            if (EditorUtility.DisplayDialog("Confirm", "Execute action on selected objects?", "Execute", "Cancel"))
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
                // Perform action on each object
                Debug.Log($"Processing: {obj.name}");
            }

            EditorUtility.DisplayDialog("Success", $"Processed {Selection.gameObjects.Length} objects", "OK");
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

### SceneGUI Template

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

            // Draw UI overlay
            GUILayout.BeginArea(new Rect(10, 10, 200, 100));
            GUILayout.Box("Scene Tool Active");
            if (GUILayout.Button("Execute"))
            {
                ExecuteSceneTool();
            }
            GUILayout.EndArea();

            Handles.EndGUI();

            // Draw scene handles
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
            Debug.Log("Scene tool executed");
        }
    }
}
#endif
```

### Step 3: Create File

Place editor tools in correct location:
- **EditorWindow/CustomEditor/PropertyDrawer**: `Assets/Scripts/Editor/ToolName.cs`
- Create Editor folder if it doesn't exist
- Ensure #if UNITY_EDITOR guards

### Step 4: Confirm

Inform user:
- Tool created successfully
- How to access (menu path)
- Next steps if applicable

## Best Practices Applied

All generated tools include:
- **#if UNITY_EDITOR guards**: Editor-only compilation
- **Proper namespaces**: ProjectName.Editor
- **Menu integration**: Easily accessible
- **Undo support**: Where applicable (Undo.RecordObject)
- **Validation**: Menu item validation
- **Help messages**: EditorGUILayout.HelpBox
- **Proper layouts**: BeginHorizontal/EndHorizontal
- **SerializedProperty**: For CustomEditor (safer than direct field access)

## Examples

**Example 1: Create EditorWindow**
```
User: /unity-dev:create-editor-tool EditorWindow LevelBuilder
Claude: Created Assets/Scripts/Editor/LevelBuilderWindow.cs
Access via: Tools/ProjectName/LevelBuilder
```

**Example 2: Create CustomEditor**
```
User: /unity-dev:create-editor-tool CustomEditor EnemyAI
Claude: What is the target script to customize?
User: EnemyController
Claude: Created Assets/Scripts/Editor/EnemyControllerEditor.cs
Automatically shows when EnemyController selected
```

**Example 3: Create MenuItem utility**
```
User: /unity-dev:create-editor-tool MenuItem AlignObjects
Claude: Created Assets/Scripts/Editor/AlignObjectsUtility.cs
Access via: Tools/ProjectName/AlignObjects Action
```

## Notes

- All editor scripts use #if UNITY_EDITOR guards
- Placed in Editor/ subfolder automatically
- Uses Undo system for reversible operations
- Includes validation functions where appropriate
- Provides user feedback via dialogs

Follow Unity editor scripting best practices for all generated tools.
