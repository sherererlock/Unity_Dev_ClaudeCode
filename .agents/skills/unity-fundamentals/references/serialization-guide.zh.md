# Unity 序列化 - 高级指南

## 序列化规则

### Unity 序列化的内容

✅ **支持：**
- 原始类型 (int, float, bool, string, enum)
- Unity 对象引用 (GameObject, Transform, Material 等)
- 带有可序列化字段的结构体 (Structs)
- 可序列化类型的数组 (Arrays) 和列表 (Lists)
- 带有 `[Serializable]` 特性的自定义类

❌ **不支持：**
- 属性 (Properties, get/set)
- 字典 (Dictionaries)
- 接口 (Interfaces)
- 泛型类型 (Generic types，List<T> 除外)
- 静态字段 (Static fields)
- 常量 (const) 或只读 (readonly) 字段

### 序列化特性 (Attributes)

```csharp
// 使私有字段在检视面板中可见
[SerializeField] private int health = 100;

// 在检视面板中隐藏公共字段
[HideInInspector] public bool debugMode;

// 不序列化的公共字段
[System.NonSerialized] public float tempValue;

// 使类可序列化
[System.Serializable]
public class SaveData
{
    public int level;
    public float playtime;
}
```

## 检视面板组织 (Inspector Organization)

### 标题和工具提示 (Headers and Tooltips)

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

### 范围和最大/最小值 (Range and Min/Max)

```csharp
[Range(0f, 1f)]
[SerializeField] private float volume = 0.8f;

[Min(0)]
[SerializeField] private int minHealth = 1;
```

### 多行文本和文本区域 (Multiline and TextArea)

```csharp
[Multiline(3)]
[SerializeField] private string description;

[TextArea(5, 10)]  // 最小/最大行数
[SerializeField] private string dialogue;
```

### 上下文菜单 (Context Menu)

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

## 自定义可序列化类

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

## ScriptableObject 序列化

ScriptableObject 是序列化到磁盘的数据资产：

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

**优点：**
- 数据作为资产文件存在
- 跨场景共享数据
- 运行时编辑不影响资产（注意：在编辑器中运行时修改会保留）
- 检视面板编辑

## 预制件序列化 (Prefab Serialization)

### 实例覆盖 (Instance Overrides)

修改预制件实例时：
- **粗体蓝色** - 被覆盖的值
- **应用 (Apply)** - 推送到预制件
- **还原 (Revert)** - 丢弃覆盖

### 嵌套预制件引用 (Nested Prefab References)

预制件可以引用其他预制件：

```csharp
[SerializeField] private GameObject weaponPrefab;  // 序列化的预制件引用
```

## 属性绘制器 (Property Drawers)

自定义类型的检视面板显示：

```csharp
[System.Serializable]
public class Stat
{
    public string name;
    [Range(1, 100)]
    public int value;
}

// Editor 文件夹中的 PropertyDrawer
#if UNITY_EDITOR
[CustomPropertyDrawer(typeof(Stat))]
public class StatDrawer : PropertyDrawer
{
    public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
    {
        // 自定义检视面板渲染
    }
}
#endif
```

## JSON 序列化

用于保存/加载系统：

```csharp
[System.Serializable]
public class SaveData
{
    public int level;
    public float health;
    public Vector3 position;
}

// 保存
SaveData data = new SaveData { level = 5, health = 75f };
string json = JsonUtility.ToJson(data);
File.WriteAllText(savePath, json);

// 加载
string json = File.ReadAllText(savePath);
SaveData data = JsonUtility.FromJson<SaveData>(json);
```

## 性能考量

- **序列化字段** 会增加场景文件大小
- **大型数组** 会拖慢检视面板
- **深层嵌套** 影响性能
- **引用 Resources** 会使资产保留在内存中

## 最佳实践

✅ **建议：**
- 使用 `[SerializeField] private` 代替 `public`
- 使用 `[Header]` 特性进行组织
- 添加 `[Tooltip]` 以增加清晰度
- 使用 ScriptableObject 共享数据

❌ **不建议：**
- 序列化超出需要的内容
- 使用公共字段进行封装（应使用 `[SerializeField] private`）
- 在 MonoBehaviour 中序列化大型数据结构
- 从 ScriptableObject 引用场景对象

## 常见问题

**数值重置 (Values resetting)：** 检查字段是否正确序列化
**引用为空 (References null)：** 在检视面板中已赋值但脚本重新编译了？
**更改未保存 (Changes not saving)：** 确保修改数值后保存了场景
**预制件覆盖丢失 (Prefab overrides lost)：** 在重新加载场景前应用更改
