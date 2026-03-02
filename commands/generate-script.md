---
name: generate-script
description: Generate Unity script templates (MonoBehaviour, ScriptableObject, Editor scripts) following Unity best practices
argument-hint: "[script-type] [script-name]"
allowed-tools: [Write, Read, Bash]
---

# Unity Script Generator

Generate Unity script templates that follow best practices and conventions.

## Purpose

Create properly structured Unity scripts with:
- Correct namespaces and file organization
- Unity lifecycle methods (where appropriate)
- Best practice patterns (caching, serialization)
- Proper EditorOnly guards for editor scripts

## Script Types

Available script types:
1. **MonoBehaviour** - Standard Unity component
2. **ScriptableObject** - Data asset
3. **EditorWindow** - Custom editor window
4. **CustomEditor** - Custom Inspector
5. **PropertyDrawer** - Custom property drawer

## Usage Instructions

### Step 1: Gather Information

Ask the user for:
1. **Script type** (if not provided): MonoBehaviour, ScriptableObject, EditorWindow, CustomEditor, or PropertyDrawer
2. **Script name** (if not provided): PascalCase name (e.g., "PlayerController", "WeaponData")
3. **Namespace** (optional): Project namespace (default: use project name from current directory)
4. **Location** (optional): Where to create the script (default: Assets/Scripts/)

**Interactive prompting:**
```
If script type not provided: "What type of script? (MonoBehaviour/ScriptableObject/EditorWindow/CustomEditor/PropertyDrawer)"
If script name not provided: "What should the script be called?"
If namespace desired: "Use custom namespace? (default: ProjectName)"
If location needed: "Where should the script be created? (default: Assets/Scripts/)"
```

### Step 2: Validate Input

Check:
- Script name is valid C# identifier (PascalCase, no spaces)
- Target directory exists
- File doesn't already exist (warn if it does)

### Step 3: Generate Script

Create script file with appropriate template based on type.

### MonoBehaviour Template

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
            // Initialize component references
        }

        private void Start()
        {
            // Initialize after all Awake calls
        }

        private void Update()
        {
            // Frame update logic
        }
    }
}
```

### ScriptableObject Template

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

### EditorWindow Template

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

### CustomEditor Template

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
        public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
        {
            EditorGUI.BeginProperty(position, label, property);

            position = EditorGUI.PrefixLabel(position, GUIUtility.GetControlID(FocusType.Passive), label);

            // Draw custom property fields here

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

### Step 4: Create File

Use Write tool to create script at specified location:
- For runtime scripts: `Assets/Scripts/ScriptName.cs`
- For editor scripts: `Assets/Scripts/Editor/ScriptName.cs`

Create necessary directories if they don't exist.

### Step 5: Confirm

Inform user:
- Script created successfully
- File location
- Next steps (e.g., "Attach to GameObject" for MonoBehaviour, "Create asset" for ScriptableObject)

## Best Practices Applied

Scripts generated include:
- **Proper namespaces**: Organized code
- **SerializeField over public**: Encapsulation
- **Header attributes**: Inspector organization
- **Caching pattern**: GetComponent in Awake
- **EditorOnly guards**: #if UNITY_EDITOR for editor scripts
- **CreateAssetMenu**: For ScriptableObjects
- **MenuItem**: For editor tools

## Examples

**Example 1: Generate MonoBehaviour**
```
User: /unity-dev:generate-script
Claude: What type of script? (MonoBehaviour/ScriptableObject/EditorWindow/CustomEditor/PropertyDrawer)
User: MonoBehaviour
Claude: What should the script be called?
User: PlayerController
Claude: Created Assets/Scripts/PlayerController.cs with standard MonoBehaviour template
```

**Example 2: Generate ScriptableObject**
```
User: /unity-dev:generate-script ScriptableObject WeaponData
Claude: Created Assets/Scripts/WeaponData.cs with CreateAssetMenu attribute
Next steps: Create asset via Assets > Create > ProjectName > WeaponData
```

**Example 3: Generate EditorWindow**
```
User: /unity-dev:generate-script EditorWindow LevelEditor
Claude: Created Assets/Scripts/Editor/LevelEditor.cs
Access via Window > ProjectName > LevelEditor
```

## Notes

- Always use Read tool to check if file already exists before creating
- Create directories with Bash if they don't exist
- For editor scripts, automatically place in Editor/ subfolder
- Include #if UNITY_EDITOR guards for all editor scripts
- Use project directory name as default namespace

Follow Unity naming conventions and best practices for all generated scripts.
