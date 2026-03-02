# Unity Serialization - Advanced Guide

## Serialization Rules

### What Unity Serializes

✅ **Supported:**
- Primitive types (int, float, bool, string, enum)
- Unity object references (GameObject, Transform, Material, etc.)
- Structs with serializable fields
- Arrays and Lists of serializable types
- Custom classes with [Serializable] attribute

❌ **Not Supported:**
- Properties (get/set)
- Dictionaries
- Interfaces
- Generic types (except List<T>)
- Static fields
- const or readonly fields

### Serialization Attributes

```csharp
// Make private field visible in Inspector
[SerializeField] private int health = 100;

// Hide public field from Inspector
[HideInInspector] public bool debugMode;

// Non-serialized public field
[System.NonSerialized] public float tempValue;

// Make class serializable
[System.Serializable]
public class SaveData
{
    public int level;
    public float playtime;
}
```

## Inspector Organization

### Headers and Tooltips

```csharp
[Header("Movement")]
[Tooltip("Speed in units per second")]
[SerializeField] private float moveSpeed = 5f;

[Tooltip("Rotation speed in degrees per second")]
[SerializeField] private float rotationSpeed = 180f;

[Header("Combat")]
[Range(1, 100)]
[SerializeField] private int attackDamage = 25;
```

### Range and Min/Max

```csharp
[Range(0f, 1f)]
[SerializeField] private float volume = 0.8f;

[Min(0)]
[SerializeField] private int minHealth = 1;
```

### Multiline and TextArea

```csharp
[Multiline(3)]
[SerializeField] private string description;

[TextArea(5, 10)]  // min/max lines
[SerializeField] private string dialogue;
```

### Context Menu

```csharp
[ContextMenu("Reset Health")]
private void ResetHealth()
{
    health = maxHealth;
}

[ContextMenuItem("Reset", "ResetSpeed")]
[SerializeField] private float speed = 5f;

private void ResetSpeed()
{
    speed = 5f;
}
```

## Custom Serializable Classes

```csharp
[System.Serializable]
public class CharacterStats
{
    public int strength;
    public int agility;
    public int intelligence;

    [Header("Derived Stats")]
    public int health;
    public float moveSpeed;
}

public class Character : MonoBehaviour
{
    [SerializeField] private CharacterStats stats;

    private void Start()
    {
        Debug.Log($"Strength: {stats.strength}");
    }
}
```

## ScriptableObject Serialization

ScriptableObjects are data assets serialized to disk:

```csharp
[CreateAssetMenu(fileName = "CharacterData", menuName = "Game/Character Data")]
public class CharacterDataSO : ScriptableObject
{
    public string characterName;
    public int maxHealth;
    public float moveSpeed;
    public Sprite portrait;
}
```

**Benefits:**
- Data lives as asset file
- Share data across scenes
- Edit at runtime without affecting asset
- Inspector editing

## Prefab Serialization

### Instance Overrides

When modifying prefab instance:
- **Bold blue** - Overridden value
- **Apply** - Push to prefab
- **Revert** - Discard override

### Nested Prefab References

Prefabs can reference other prefabs:

```csharp
[SerializeField] private GameObject weaponPrefab;  // Serialized prefab reference
```

## Property Drawers

Custom Inspector display for types:

```csharp
[System.Serializable]
public class Stat
{
    public string name;
    [Range(1, 100)]
    public int value;
}

// PropertyDrawer in Editor folder
#if UNITY_EDITOR
[CustomPropertyDrawer(typeof(Stat))]
public class StatDrawer : PropertyDrawer
{
    public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
    {
        // Custom Inspector rendering
    }
}
#endif
```

## JSON Serialization

For save/load systems:

```csharp
[System.Serializable]
public class SaveData
{
    public int level;
    public float health;
    public Vector3 position;
}

// Save
SaveData data = new SaveData { level = 5, health = 75f };
string json = JsonUtility.ToJson(data);
File.WriteAllText(savePath, json);

// Load
string json = File.ReadAllText(savePath);
SaveData data = JsonUtility.FromJson<SaveData>(json);
```

## Performance Considerations

- **Serialized fields** add to scene file size
- **Large arrays** slow down Inspector
- **Deep nesting** impacts performance
- **References to Resources** keep assets in memory

## Best Practices

✅ **Do:**
- Use `[SerializeField] private` over `public`
- Organize with `[Header]` attributes
- Add `[Tooltip]` for clarity
- Use ScriptableObjects for shared data

❌ **Don't:**
- Serialize more than needed
- Use public fields for encapsulation
- Serialize large data structures in MonoBehaviour
- Reference scene objects from ScriptableObjects

## Common Issues

**Values resetting:** Check if field is serialized correctly
**References null:** Assigned in Inspector but script recompiled?
**Changes not saving:** Ensure scene saved after modifying values
**Prefab overrides lost:** Apply changes before scene reload
