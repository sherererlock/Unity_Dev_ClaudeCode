# Unity Prefab Workflows - Complete Guide

Comprehensive guide to prefabs in Unity: creation, variants, nesting, overrides, programmatic usage, and best practices for maintainable game development.

## Prefab Fundamentals

### What Are Prefabs?

Prefabs are reusable GameObject templates stored as assets. Changes to the prefab asset propagate to all instances.

**Key concepts:**
- **Prefab Asset**: Template stored in Project window (.prefab file)
- **Prefab Instance**: Copy of prefab in scene (blue text in Hierarchy)
- **Instance Override**: Modification to specific instance (bold blue text)
- **Prefab Variant**: Prefab that inherits from another prefab

### Why Use Prefabs?

1. **Reusability**: Create once, use many times
2. **Consistency**: Changes propagate to all instances
3. **Efficiency**: Faster than duplicating GameObjects
4. **Organization**: Modular, maintainable assets
5. **Runtime Spawning**: Instantiate at runtime

## Creating Prefabs

### Basic Prefab Creation

Drag GameObject from Hierarchy to Project window:

```
Hierarchy:          Project:
Enemy GameObject -> Assets/Prefabs/Enemy.prefab
```

**Visual indicators:**
- **Original GameObject**: Turns blue in Hierarchy
- **Prefab Asset**: Shows in Project window with blue cube icon

### Creating Prefab from Code

```csharp
#if UNITY_EDITOR
using UnityEditor;

public class PrefabCreator
{
    [MenuItem("Tools/Create Prefab")]
    private static void CreatePrefab()
    {
        GameObject selected = Selection.activeGameObject;
        if (selected == null)
            return;

        string path = "Assets/Prefabs/" + selected.name + ".prefab";
        PrefabUtility.SaveAsPrefabAsset(selected, path);
    }
}
#endif
```

### Unpacking Prefabs

Break prefab connection to edit independently:

**Right-click prefab instance:**
- **Unpack Prefab**: Disconnect this instance (stays in hierarchy)
- **Unpack Prefab Completely**: Unpack including nested prefabs

**From code:**
```csharp
#if UNITY_EDITOR
GameObject instance = PrefabUtility.InstantiatePrefab(prefab) as GameObject;
PrefabUtility.UnpackPrefabInstance(instance, PrefabUnpackMode.Completely, InteractionMode.AutomatedAction);
#endif
```

## Prefab Instances

### Instance Overrides

When modifying prefab instance, values show in **bold blue**:

**Override types:**
- Transform changes (position, rotation, scale)
- Component property changes
- Added components
- Removed components
- GameObject hierarchy changes

**Managing overrides:**
- **Revert**: Discard override, match prefab
- **Apply**: Push override to prefab asset (affects all instances)
- **Apply to Prefab Variant**: Create variant with override

### Instance Override Workflow

```
1. Select prefab instance in scene
2. Modify property (turns bold blue)
3. Right-click property or use Overrides dropdown
4. Choose:
   - Revert: Discard change
   - Apply to Prefab: Update asset
   - Apply as Override in Variant: Create variant
```

**Viewing all overrides:**
- Open Overrides dropdown at top of Inspector
- See all overrides in one place
- Apply/revert selectively or all at once

### Auto-Save vs Manual Apply

Unity 2020+ auto-saves prefab changes when entering Prefab Mode. For instances:

**Manual apply required:**
- Instance overrides don't auto-apply to prefab
- Prevents accidental changes to all instances
- Explicit Apply action needed

## Prefab Variants

### Creating Variants

Variants inherit from base prefab with specific overrides:

**Method 1: From existing prefab**
1. Right-click prefab in Project
2. Create > Prefab Variant
3. Modify variant as needed

**Method 2: From instance override**
1. Modify prefab instance
2. Overrides dropdown > Apply as Override in Variant
3. Creates variant with that override

**Hierarchy example:**
```
Enemy (Base Prefab)
├── FastEnemy (Variant: moveSpeed = 10)
├── TankEnemy (Variant: health = 200)
└── FlyingEnemy (Variant: added FlyingMovement component)
```

### Variant Inheritance

Changes to base prefab propagate to variants:

```
Base Enemy:
- maxHealth = 100
- moveSpeed = 5

FastEnemy Variant:
- Inherits maxHealth = 100 from base
- Overrides moveSpeed = 10

If base changes maxHealth to 150:
- FastEnemy automatically gets maxHealth = 150
- FastEnemy keeps moveSpeed = 10 override
```

**Breaking inheritance:**
- Override property in variant
- Property no longer updates from base
- Can revert override to re-inherit

### Variant Use Cases

**Character variations:**
```
Character (Base)
├── PlayerCharacter (Variant: Player scripts)
├── EnemyCharacter (Variant: AI scripts)
│   ├── MeleeEnemy (Variant: Melee components)
│   └── RangedEnemy (Variant: Ranged components)
```

**UI prefabs:**
```
Button (Base)
├── PrimaryButton (Variant: Color, larger)
├── SecondaryButton (Variant: Different color)
└── DangerButton (Variant: Red color, warning icon)
```

## Nested Prefabs

### Prefab Composition

Prefabs can contain other prefabs:

```
Car (Prefab)
├── Wheel (Prefab) x4
├── Engine (Prefab)
├── Door (Prefab) x4
└── Seat (Prefab) x4
```

**Benefits:**
- Modular design
- Reuse subcomponents
- Edit nested prefabs independently
- Changes propagate correctly

### Editing Nested Prefabs

**Option 1: Edit nested prefab directly**
1. Double-click nested prefab in Project
2. Opens in Prefab Mode
3. Changes affect all uses of that prefab

**Option 2: Edit within parent prefab**
1. Open parent prefab in Prefab Mode
2. Select nested prefab instance
3. Use Overrides system as normal

**Option 3: Select and open**
1. Select nested prefab instance in scene/parent
2. Click "Open" in Inspector
3. Opens nested prefab in Prefab Mode

### Nested Prefab Overrides

Overrides work recursively:

```
House (Prefab)
└── Door (Nested Prefab)
    └── Handle (Nested Prefab)
        └── material (Property)

Scene Instance:
House Instance
└── Door Instance (override: color = red)
    └── Handle Instance (override: scale = 1.2)
```

Both overrides maintained independently.

## Prefab Mode

### Entering Prefab Mode

**Open prefab for editing:**
- Double-click prefab in Project window
- Select instance, click "Open Prefab" in Inspector
- Right-click instance > Open Prefab

**Visual indicators:**
- Blue breadcrumb bar at top
- Scene shows only prefab contents
- Isolated editing environment

### Prefab Mode Benefits

1. **Isolated editing**: See only prefab, not scene clutter
2. **Context preservation**: Prefab position matches scene usage
3. **Safe changes**: Preview before applying
4. **Auto-save**: Changes save automatically on exit

### Exiting Prefab Mode

- Click scene name in breadcrumb
- Click arrow in breadcrumb
- Close tab (saves automatically)

## Programmatic Prefab Usage

### Instantiating Prefabs

```csharp
public class PrefabSpawner : MonoBehaviour
{
    [SerializeField] private GameObject prefab;

    private void Start()
    {
        // Basic instantiation
        GameObject instance = Instantiate(prefab);

        // With position and rotation
        GameObject instance2 = Instantiate(prefab, transform.position, Quaternion.identity);

        // With parent
        GameObject instance3 = Instantiate(prefab, transform);

        // With all parameters
        GameObject instance4 = Instantiate(prefab, new Vector3(0, 5, 0), Quaternion.Euler(0, 90, 0), transform);
    }
}
```

### Configuring After Instantiation

```csharp
public class EnemySpawner : MonoBehaviour
{
    [SerializeField] private GameObject enemyPrefab;
    [SerializeField] private Transform spawnPoint;

    private void SpawnEnemy()
    {
        // Instantiate
        GameObject enemy = Instantiate(enemyPrefab, spawnPoint.position, Quaternion.identity);

        // Configure instance
        enemy.name = "Enemy_" + Time.time;  // Unique name
        enemy.transform.SetParent(transform);  // Parent to spawner

        // Get and configure components
        var enemyAI = enemy.GetComponent<EnemyAI>();
        if (enemyAI != null)
        {
            enemyAI.SetTarget(FindPlayerTransform());
            enemyAI.SetDifficulty(currentDifficulty);
        }

        var health = enemy.GetComponent<Health>();
        if (health != null)
        {
            health.SetMaxHealth(100);
        }
    }

    private Transform FindPlayerTransform()
    {
        return GameObject.FindWithTag("Player")?.transform;
    }

    private int currentDifficulty = 1;
}
```

### Destroying Prefab Instances

```csharp
// Immediate destruction (use sparingly)
Destroy(enemyInstance);

// Delayed destruction (better for cleanup)
Destroy(enemyInstance, 2f);  // Destroy after 2 seconds

// Immediate destruction (Editor and Runtime)
DestroyImmediate(enemyInstance);  // Use only in Editor code
```

### Prefab Pooling

Reuse instances instead of destroying:

```csharp
public class PrefabPool : MonoBehaviour
{
    [SerializeField] private GameObject prefab;
    [SerializeField] private int initialPoolSize = 10;

    private Queue<GameObject> availablePool = new Queue<GameObject>();
    private List<GameObject> allInstances = new List<GameObject>();

    private void Awake()
    {
        // Pre-instantiate pool
        for (int i = 0; i < initialPoolSize; i++)
        {
            CreateNewInstance();
        }
    }

    private GameObject CreateNewInstance()
    {
        GameObject instance = Instantiate(prefab, transform);
        instance.SetActive(false);
        availablePool.Enqueue(instance);
        allInstances.Add(instance);
        return instance;
    }

    public GameObject Get()
    {
        GameObject instance;

        if (availablePool.Count > 0)
        {
            instance = availablePool.Dequeue();
        }
        else
        {
            // Pool exhausted - create new
            instance = CreateNewInstance();
        }

        instance.SetActive(true);
        return instance;
    }

    public void Return(GameObject instance)
    {
        instance.SetActive(false);
        availablePool.Enqueue(instance);
    }

    private void OnDestroy()
    {
        // Clean up all instances
        foreach (var instance in allInstances)
        {
            if (instance != null)
                Destroy(instance);
        }
    }
}
```

**Usage:**
```csharp
public class BulletSpawner : MonoBehaviour
{
    [SerializeField] private PrefabPool bulletPool;

    private void Fire()
    {
        GameObject bullet = bulletPool.Get();
        bullet.transform.position = firePoint.position;

        // Return to pool after 3 seconds
        StartCoroutine(ReturnAfterDelay(bullet, 3f));
    }

    private IEnumerator ReturnAfterDelay(GameObject obj, float delay)
    {
        yield return new WaitForSeconds(delay);
        bulletPool.Return(obj);
    }
}
```

## Prefab Workflow Best Practices

### Organization

**Folder structure:**
```
Assets/
└── Prefabs/
    ├── Characters/
    │   ├── Player/
    │   └── Enemies/
    ├── Environment/
    │   ├── Props/
    │   └── Buildings/
    ├── UI/
    │   ├── Panels/
    │   └── Buttons/
    └── Effects/
        ├── Particles/
        └── Audio/
```

**Naming conventions:**
- Use descriptive names: `EnemyOrc`, not `Enemy1`
- Variants include base name: `ButtonPrimary`, `ButtonSecondary`
- Prefabs for spawning: `BulletProjectile`, `ExplosionEffect`

### Version Control

**Good practices:**
- Commit prefab changes separately from scenes
- Document breaking changes in commit messages
- Use prefab variants instead of duplicating
- Test prefab changes before committing

**Merge conflicts:**
- Prefab files are YAML (mergeable)
- Use Unity's Smart Merge tool
- When conflicts occur, prefer "theirs" for prefabs, "yours" for scenes

### Testing Changes

**Before applying to prefab:**
1. Test override in scene
2. Verify all systems still work
3. Check dependent prefabs/scenes
4. Apply to prefab
5. Test all instances

**After applying:**
1. Check other scenes using prefab
2. Verify prefab variants still work
3. Test runtime instantiation

### Prefab Variants vs Duplication

✅ **Use variants for:**
- Similar objects with minor differences
- Maintaining connection to base
- Changes that should propagate

❌ **Don't use variants for:**
- Completely different objects
- Temporary test prefabs
- One-off modifications

### Prefab References

**Scene references:**
- Prefabs CAN reference scene objects in Prefab Mode
- References break when prefab used in different scene
- Solution: Initialize references at runtime

**Prefab-to-prefab references:**
```csharp
public class Weapon : MonoBehaviour
{
    [SerializeField] private GameObject bulletPrefab;  // ✅ GOOD - Prefab reference

    private void Fire()
    {
        Instantiate(bulletPrefab, firePoint.position, firePoint.rotation);
    }
}
```

**ScriptableObject references:**
```csharp
[CreateAssetMenu]
public class WeaponData : ScriptableObject
{
    public int damage;
    public float fireRate;
    public GameObject projectilePrefab;
}

public class Weapon : MonoBehaviour
{
    [SerializeField] private WeaponData data;  // ✅ GOOD - ScriptableObject reference
}
```

## Advanced Prefab Techniques

### Prefab-Specific Scripts

Scripts that only run in prefab instances:

```csharp
public class PrefabInitializer : MonoBehaviour
{
    private void Awake()
    {
#if UNITY_EDITOR
        // Check if this is a prefab asset (not instance)
        if (UnityEditor.PrefabUtility.IsPartOfPrefabAsset(gameObject))
        {
            // Don't run initialization in prefab asset
            return;
        }
#endif

        // Initialize prefab instance
        InitializeComponents();
    }

    private void InitializeComponents()
    {
        // Runtime initialization
    }
}
```

### Editor-Only Prefab Utilities

```csharp
#if UNITY_EDITOR
using UnityEditor;

public class PrefabUtilities
{
    [MenuItem("Tools/Prefabs/Select All Instances")]
    private static void SelectAllInstances()
    {
        GameObject selected = Selection.activeGameObject;
        if (selected == null)
            return;

        GameObject prefabAsset = PrefabUtility.GetCorrespondingObjectFromSource(selected);
        if (prefabAsset == null)
            return;

        // Find all instances of this prefab
        var instances = new List<GameObject>();
        foreach (var go in GameObject.FindObjectsOfType<GameObject>())
        {
            if (PrefabUtility.GetCorrespondingObjectFromSource(go) == prefabAsset)
            {
                instances.Add(go);
            }
        }

        Selection.objects = instances.ToArray();
    }

    [MenuItem("Tools/Prefabs/Update All Instances")]
    private static void UpdateAllInstances()
    {
        GameObject prefabAsset = Selection.activeGameObject;
        if (prefabAsset == null || !PrefabUtility.IsPartOfPrefabAsset(prefabAsset))
            return;

        // Find all instances and revert overrides
        foreach (var go in GameObject.FindObjectsOfType<GameObject>())
        {
            if (PrefabUtility.GetCorrespondingObjectFromSource(go) == prefabAsset)
            {
                PrefabUtility.RevertPrefabInstance(go, InteractionMode.AutomatedAction);
            }
        }
    }
}
#endif
```

### Prefab Validation

Validate prefab structure in Editor:

```csharp
#if UNITY_EDITOR
using UnityEditor;

public class PrefabValidator
{
    [MenuItem("Tools/Prefabs/Validate Selected")]
    private static void ValidateSelectedPrefab()
    {
        GameObject selected = Selection.activeGameObject;
        if (selected == null)
        {
            Debug.LogError("No prefab selected");
            return;
        }

        // Check if prefab
        if (!PrefabUtility.IsPartOfPrefabAsset(selected) &&
            PrefabUtility.GetPrefabInstanceStatus(selected) == PrefabInstanceStatus.NotAPrefab)
        {
            Debug.LogError("Selected object is not a prefab");
            return;
        }

        // Validate required components
        bool valid = true;

        if (selected.GetComponent<Renderer>() == null)
        {
            Debug.LogWarning("Prefab missing Renderer component");
            valid = false;
        }

        if (selected.GetComponent<Collider>() == null)
        {
            Debug.LogWarning("Prefab missing Collider component");
            valid = false;
        }

        if (valid)
            Debug.Log("Prefab validation passed");
    }
}
#endif
```

## Common Prefab Patterns

### Factory Pattern

```csharp
public class EnemyFactory : MonoBehaviour
{
    [System.Serializable]
    public class EnemyPrefabData
    {
        public string enemyType;
        public GameObject prefab;
        public int poolSize;
    }

    [SerializeField] private List<EnemyPrefabData> enemyPrefabs;

    private Dictionary<string, PrefabPool> pools = new Dictionary<string, PrefabPool>();

    private void Awake()
    {
        // Create pools for each enemy type
        foreach (var data in enemyPrefabs)
        {
            var poolObj = new GameObject($"{data.enemyType}_Pool");
            poolObj.transform.SetParent(transform);

            var pool = poolObj.AddComponent<PrefabPool>();
            // Configure pool with prefab and size
            pools[data.enemyType] = pool;
        }
    }

    public GameObject SpawnEnemy(string enemyType, Vector3 position)
    {
        if (pools.TryGetValue(enemyType, out var pool))
        {
            GameObject enemy = pool.Get();
            enemy.transform.position = position;
            return enemy;
        }

        Debug.LogError($"Enemy type {enemyType} not found");
        return null;
    }
}
```

### Prefab Registry

```csharp
[CreateAssetMenu]
public class PrefabRegistry : ScriptableObject
{
    [System.Serializable]
    public class PrefabEntry
    {
        public string id;
        public GameObject prefab;
    }

    [SerializeField] private List<PrefabEntry> prefabs;

    private Dictionary<string, GameObject> prefabDict;

    public void Initialize()
    {
        prefabDict = new Dictionary<string, GameObject>();
        foreach (var entry in prefabs)
        {
            prefabDict[entry.id] = entry.prefab;
        }
    }

    public GameObject GetPrefab(string id)
    {
        if (prefabDict == null)
            Initialize();

        return prefabDict.TryGetValue(id, out var prefab) ? prefab : null;
    }
}
```

## Troubleshooting

### Missing Prefab References

**Symptom**: Prefab reference shows "None" or "Missing"

**Causes:**
1. Prefab was deleted
2. Prefab was moved/renamed without updating reference
3. GUID changed (rare)

**Solution:**
1. Find original prefab or replacement
2. Reassign reference
3. For multiple instances, use Find & Replace References

### Broken Prefab Connection

**Symptom**: Instance no longer links to prefab (black text instead of blue)

**Cause**: Prefab asset was deleted or moved

**Solution:**
1. Find prefab asset and restore it
2. Or: Create new prefab from instance
3. Or: Reconnect using PrefabUtility.ReconnectToLastPrefab (Editor)

### Overrides Not Applying

**Symptom**: Applied overrides don't affect other instances

**Cause:**
1. Applied to variant instead of base
2. Other instances have overrides on same property
3. Prefab file not saved

**Solution:**
1. Check you're applying to correct prefab
2. Revert overrides on other instances
3. Save project/scene

### Prefab Instance Corruption

**Symptom**: Instance behaves strangely, properties reset

**Cause**: Merge conflict or file corruption

**Solution:**
1. Delete instance
2. Re-instantiate from prefab
3. Reapply necessary overrides

## Best Practices Summary

✅ **DO:**
- Use prefabs for all reusable GameObjects
- Create variants for similar objects
- Test overrides before applying to prefab
- Organize prefabs in logical folders
- Use prefab pooling for frequently spawned objects
- Document breaking changes
- Validate prefabs before committing

❌ **DON'T:**
- Duplicate prefabs instead of using variants
- Reference scene objects from prefabs
- Apply untested overrides to prefabs
- Create deep nested prefab hierarchies (>3 levels)
- Forget to test all instances after changes
- Mix prefabs and regular GameObjects for same purpose

Follow these workflows for maintainable, efficient prefab usage in Unity projects.
