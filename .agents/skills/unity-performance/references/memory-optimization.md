# Unity Memory Optimization - Complete Guide

Advanced techniques for reducing garbage collection, managing allocations, and optimizing memory usage in Unity games.

## Garbage Collection Fundamentals

### How Unity's GC Works

Unity uses the Boehm-Demers-Weiser garbage collector:

**Key characteristics:**
- **Non-generational**: Scans all managed memory
- **Stop-the-world**: Pauses game execution during collection
- **Mark-and-sweep**: Marks reachable objects, sweeps unreachable
- **Incremental**: Unity 2019+ supports incremental GC (spreads work across frames)

**GC trigger conditions:**
- Heap fills up (automatic)
- Explicit GC.Collect() call
- Scene load
- Memory pressure from OS

**Performance impact**: GC spikes cause frame drops, stuttering.

### Allocation Sources

Common allocation sources ranked by severity:

1. **String operations** (concatenation, formatting)
2. **Boxing** (value types → reference types)
3. **Collections** (List, Dictionary growth, LINQ)
4. **Closures** (lambda captures, delegates)
5. **Coroutines** (yield statements, WaitForSeconds)
6. **Unity API allocations** (GetComponents, Find, foreach)

## String Optimization

### StringBuilder Pattern

```csharp
// ❌ BAD - Allocates multiple strings
private void UpdateUI()
{
    string text = "Player: " + playerName + " | Health: " + health + "/" + maxHealth;
    uiText.text = text;
}

// ✅ GOOD - StringBuilder reuse
private StringBuilder sb = new StringBuilder(100);  // Pre-allocate capacity

private void UpdateUI()
{
    sb.Clear();
    sb.Append("Player: ").Append(playerName)
      .Append(" | Health: ").Append(health)
      .Append("/").Append(maxHealth);
    uiText.text = sb.ToString();
}
```

**Capacity**: Pre-allocate StringBuilder capacity to prevent resizing allocations.

### String Interning

Cache frequently used strings:

```csharp
// ❌ BAD - Creates new strings
private void LogState(string state)
{
    if (state == "Idle") { }
    if (state == "Running") { }
}

// ✅ GOOD - Cache strings
private static class States
{
    public static readonly string Idle = "Idle";
    public static readonly string Running = "Running";
    public static readonly string Attacking = "Attacking";
}

private void LogState(string state)
{
    if (state == States.Idle) { }
    if (state == States.Running) { }
}

// ✅ BETTER - Use enums instead of strings
private enum State { Idle, Running, Attacking }
private State currentState;
```

### String Formatting

```csharp
// ❌ BAD - string.Format allocates
private void Update()
{
    text.text = string.Format("Score: {0}", score);  // Allocates
}

// ✅ GOOD - Direct concatenation or cached format
private void UpdateScore(int newScore)
{
    score = newScore;
    text.text = "Score: " + score.ToString();  // Less allocation
}

// ✅ BEST - Avoid updates unless changed
private int lastScore = -1;

private void UpdateScore(int newScore)
{
    if (score != lastScore)
    {
        score = newScore;
        lastScore = newScore;
        text.text = "Score: " + score.ToString();
    }
}
```

## Collection Optimization

### List Capacity Management

Pre-allocate List capacity to prevent resizing:

```csharp
// ❌ BAD - List grows dynamically, causes allocations
private List<Enemy> enemies = new List<Enemy>();  // Capacity = 0, grows to 4, 8, 16...

// ✅ GOOD - Pre-allocate expected capacity
private List<Enemy> enemies = new List<Enemy>(100);  // Capacity = 100

// ✅ BEST - Reuse list, clear instead of creating new
private List<Enemy> tempEnemies = new List<Enemy>(50);

private void FindNearbyEnemies()
{
    tempEnemies.Clear();  // Reuse existing list
    // Populate tempEnemies
}
```

**Capacity rules:**
- Pre-allocate capacity for known/estimated sizes
- Use `EnsureCapacity()` before bulk additions
- Clear and reuse instead of creating new lists

### Dictionary Capacity

```csharp
// ❌ BAD - Dictionary grows, causes rehashing
private Dictionary<int, Enemy> enemyLookup = new Dictionary<int, Enemy>();

// ✅ GOOD - Pre-allocate capacity
private Dictionary<int, Enemy> enemyLookup = new Dictionary<int, Enemy>(200);
```

### Array Pooling

Reuse arrays instead of allocating new ones:

```csharp
public class ArrayPool<T>
{
    private readonly Stack<T[]> pool = new Stack<T[]>();
    private readonly int arraySize;

    public ArrayPool(int arraySize, int initialCount)
    {
        this.arraySize = arraySize;
        for (int i = 0; i < initialCount; i++)
        {
            pool.Push(new T[arraySize]);
        }
    }

    public T[] Rent()
    {
        return pool.Count > 0 ? pool.Pop() : new T[arraySize];
    }

    public void Return(T[] array)
    {
        System.Array.Clear(array, 0, array.Length);  // Clear data
        pool.Push(array);
    }
}

// Usage
private ArrayPool<Vector3> vectorPool = new ArrayPool<Vector3>(100, 5);

private void ProcessPositions()
{
    Vector3[] positions = vectorPool.Rent();

    // Use array

    vectorPool.Return(positions);
}
```

**Unity 2021+**: Use `System.Buffers.ArrayPool<T>` from .NET.

## Boxing Avoidance

### Value Type Boxing

Boxing occurs when value types convert to reference types:

```csharp
// ❌ BAD - Boxing allocations
private void LogValue(object value)  // object parameter causes boxing
{
    Debug.Log(value);
}

int health = 100;
LogValue(health);  // Boxing: int → object

// ✅ GOOD - Generic method avoids boxing
private void LogValue<T>(T value)
{
    Debug.Log(value);
}

int health = 100;
LogValue(health);  // No boxing
```

### Dictionary with Value Types

```csharp
// ❌ BAD - Enumerator allocates
foreach (var kvp in dictionary)  // Allocates enumerator
{
    Process(kvp.Key, kvp.Value);
}

// ✅ GOOD - For loop avoids allocation (for List)
for (int i = 0; i < list.Count; i++)
{
    Process(list[i]);
}

// ✅ GOOD - Use struct enumerator (Unity 2021+)
foreach (var kvp in dictionary)
{
    // Modern Unity uses struct enumerator (no allocation)
}
```

## LINQ Allocation

LINQ methods allocate heavily - avoid in hot paths:

```csharp
// ❌ BAD - LINQ allocates closures, enumerators
private void Update()
{
    var nearbyEnemies = enemies.Where(e => Vector3.Distance(e.position, player.position) < range)
                                .OrderBy(e => e.health)
                                .ToList();  // Multiple allocations!
}

// ✅ GOOD - Manual loop
private List<Enemy> nearbyEnemies = new List<Enemy>(50);

private void Update()
{
    nearbyEnemies.Clear();

    for (int i = 0; i < enemies.Count; i++)
    {
        if (Vector3.Distance(enemies[i].position, player.position) < range)
        {
            nearbyEnemies.Add(enemies[i]);
        }
    }

    // Manual sort if needed
    nearbyEnemies.Sort((a, b) => a.health.CompareTo(b.health));
}
```

**LINQ allocation sources:**
- `Where()` - allocates predicate closure
- `Select()` - allocates transformation
- `OrderBy()` - allocates comparer
- `ToList()`, `ToArray()` - allocates collection

**Use LINQ only in:**
- Initialization code (Awake, Start)
- Editor code
- Infrequent operations (scene load, save)

## Coroutine Optimization

### Caching Yield Instructions

```csharp
// ❌ BAD - Allocates WaitForSeconds every call
private IEnumerator SpawnEnemies()
{
    while (true)
    {
        SpawnEnemy();
        yield return new WaitForSeconds(1f);  // New allocation each iteration
    }
}

// ✅ GOOD - Cache yield instructions
private readonly WaitForSeconds oneSecond = new WaitForSeconds(1f);
private readonly WaitForSeconds halfSecond = new WaitForSeconds(0.5f);
private readonly WaitForEndOfFrame endOfFrame = new WaitForEndOfFrame();

private IEnumerator SpawnEnemies()
{
    while (true)
    {
        SpawnEnemy();
        yield return oneSecond;  // Reuse cached instance
    }
}
```

### Null Yield Caching

```csharp
// ❌ BAD - yield return null still allocates in some Unity versions
private IEnumerator DoWork()
{
    yield return null;  // May allocate
}

// ✅ GOOD - Cache null yield
private static readonly WaitForEndOfFrame waitFrame = new WaitForEndOfFrame();

private IEnumerator DoWork()
{
    yield return waitFrame;  // No allocation
}
```

### Coroutine Pooling

For frequently started/stopped coroutines, use Update with state machine instead.

## Closure and Lambda Optimization

### Lambda Captures

Lambdas that capture variables allocate closures:

```csharp
// ❌ BAD - Lambda captures 'damage', allocates closure
private void DealDamageToAll(int damage)
{
    enemies.ForEach(enemy => enemy.TakeDamage(damage));  // Allocates closure
}

// ✅ GOOD - Use for loop, no lambda
private void DealDamageToAll(int damage)
{
    for (int i = 0; i < enemies.Count; i++)
    {
        enemies[i].TakeDamage(damage);
    }
}

// ✅ ALTERNATIVE - Instance method (no capture)
private int currentDamage;

private void DealDamageToAll(int damage)
{
    currentDamage = damage;
    enemies.ForEach(DealDamageToEnemy);  // No capture, no allocation
}

private void DealDamageToEnemy(Enemy enemy)
{
    enemy.TakeDamage(currentDamage);
}
```

### Delegate Caching

Cache delegate instances:

```csharp
// ❌ BAD - Creates new delegate each time
public event Action OnDeath;

private void Subscribe()
{
    OnDeath += () => Debug.Log("Died");  // New delegate allocation
}

// ✅ GOOD - Cache delegate
private Action onDeathHandler;

private void Awake()
{
    onDeathHandler = HandleDeath;  // Cache once
}

private void Subscribe()
{
    OnDeath += onDeathHandler;  // Reuse cached delegate
}

private void Unsubscribe()
{
    OnDeath -= onDeathHandler;
}

private void HandleDeath()
{
    Debug.Log("Died");
}
```

## Unity API Allocations

### GetComponents Variants

```csharp
// ❌ BAD - GetComponents allocates array
private void Update()
{
    Collider[] colliders = GetComponents<Collider>();  // Allocates array
}

// ✅ GOOD - GetComponents with list (no allocation if list has capacity)
private List<Collider> colliders = new List<Collider>(10);

private void Update()
{
    GetComponents(colliders);  // Reuses list
}
```

### Physics Queries

```csharp
// ❌ BAD - Physics.RaycastAll allocates array
private void Update()
{
    RaycastHit[] hits = Physics.RaycastAll(origin, direction);  // Allocates
}

// ✅ GOOD - Physics.RaycastNonAlloc
private RaycastHit[] hits = new RaycastHit[10];

private void Update()
{
    int hitCount = Physics.RaycastNonAlloc(origin, direction, hits);

    for (int i = 0; i < hitCount; i++)
    {
        ProcessHit(hits[i]);
    }
}
```

**NonAlloc variants:**
- `Physics.RaycastNonAlloc()`
- `Physics.SphereCastNonAlloc()`
- `Physics.OverlapSphereNonAlloc()`
- `Physics2D.RaycastNonAlloc()`

### GameObject.Find Allocations

```csharp
// ❌ BAD - FindObjectsOfType allocates array every call
private void Update()
{
    Enemy[] enemies = FindObjectsOfType<Enemy>();  // Allocates!
}

// ✅ GOOD - Maintain list, register/unregister pattern
public class EnemyManager : MonoBehaviour
{
    private static List<Enemy> allEnemies = new List<Enemy>();

    public static void Register(Enemy enemy)
    {
        allEnemies.Add(enemy);
    }

    public static void Unregister(Enemy enemy)
    {
        allEnemies.Remove(enemy);
    }

    public static IReadOnlyList<Enemy> AllEnemies => allEnemies;
}

public class Enemy : MonoBehaviour
{
    private void OnEnable()
    {
        EnemyManager.Register(this);
    }

    private void OnDisable()
    {
        EnemyManager.Unregister(this);
    }
}

// Usage - no allocation
private void Update()
{
    var enemies = EnemyManager.AllEnemies;
    for (int i = 0; i < enemies.Count; i++)
    {
        // Process enemy
    }
}
```

## Struct Optimization

Use structs for small, value-semantic types:

```csharp
// Value-semantic data - use struct
public struct DamageInfo
{
    public int damage;
    public Vector3 hitPoint;
    public Vector3 hitNormal;
    public GameObject attacker;
}

// Reference-semantic behavior - use class
public class Enemy : MonoBehaviour
{
    public void TakeDamage(DamageInfo damageInfo)
    {
        health -= damageInfo.damage;
    }
}
```

**Struct guidelines:**
- Small size (<16 bytes ideal)
- Value semantics (copies make sense)
- Immutable preferred
- Avoid large structs (copying overhead)

## Memory Profiling

### Using Memory Profiler

**Window > Analysis > Memory Profiler**

**Capture snapshots:**
1. Take snapshot at startup
2. Play for a while
3. Take second snapshot
4. Compare to find leaks

**Look for:**
- Growing managed heap
- Leaked textures/meshes
- Retained event handlers
- Uncollected objects

### Detecting Leaks

**Common leak sources:**
1. **Unclosed event handlers**
```csharp
// LEAK - Never unsubscribes
private void Start()
{
    GameManager.OnLevelComplete += HandleComplete;  // Never removed!
}
```

2. **Static references**
```csharp
// LEAK - Static holds reference
private static Player currentPlayer;  // Never cleared, prevents GC
```

3. **Delegate references**
```csharp
// LEAK - Anonymous method captures 'this'
private void Start()
{
    GameManager.OnUpdate += () => UpdatePlayer();  // Captures this, prevents GC
}
```

### GC.GetTotalMemory

Monitor managed memory growth:

```csharp
private void LogMemory()
{
    long memory = GC.GetTotalMemory(false);
    Debug.Log($"Managed memory: {memory / (1024 * 1024)} MB");
}
```

## Object Pooling Patterns

### Generic Pool

```csharp
public class GenericPool<T> where T : MonoBehaviour
{
    private T prefab;
    private Queue<T> pool = new Queue<T>();
    private Transform parent;

    public GenericPool(T prefab, int initialSize, Transform parent = null)
    {
        this.prefab = prefab;
        this.parent = parent;

        for (int i = 0; i < initialSize; i++)
        {
            T instance = Object.Instantiate(prefab, parent);
            instance.gameObject.SetActive(false);
            pool.Enqueue(instance);
        }
    }

    public T Get()
    {
        if (pool.Count > 0)
        {
            T instance = pool.Dequeue();
            instance.gameObject.SetActive(true);
            return instance;
        }

        return Object.Instantiate(prefab, parent);
    }

    public void Return(T instance)
    {
        instance.gameObject.SetActive(false);
        instance.transform.SetParent(parent);
        pool.Enqueue(instance);
    }
}
```

### Particle Pool

```csharp
public class ParticlePool : MonoBehaviour
{
    [SerializeField] private ParticleSystem particlePrefab;
    [SerializeField] private int poolSize = 20;

    private Queue<ParticleSystem> pool = new Queue<ParticleSystem>();

    private void Awake()
    {
        for (int i = 0; i < poolSize; i++)
        {
            ParticleSystem ps = Instantiate(particlePrefab, transform);
            ps.Stop(true, ParticleSystemStopBehavior.StopEmittingAndClear);
            pool.Enqueue(ps);
        }
    }

    public ParticleSystem PlayEffect(Vector3 position, Quaternion rotation)
    {
        ParticleSystem ps = pool.Count > 0 ? pool.Dequeue() : Instantiate(particlePrefab, transform);

        ps.transform.position = position;
        ps.transform.rotation = rotation;
        ps.Play(true);

        StartCoroutine(ReturnWhenFinished(ps));
        return ps;
    }

    private IEnumerator ReturnWhenFinished(ParticleSystem ps)
    {
        yield return new WaitWhile(() => ps.isPlaying);

        ps.Stop(true, ParticleSystemStopBehavior.StopEmittingAndClear);
        pool.Enqueue(ps);
    }
}
```

## Platform-Specific Memory

### Mobile Memory Constraints

Mobile devices have limited RAM (2-4GB total, app gets portion):

**Mobile optimizations:**
- Compress textures (ASTC, ETC2)
- Lower texture resolution
- Reduce mesh density
- Unload unused assets
- Use asset bundles for streaming
- Monitor with Xcode/Android Profiler

### Texture Memory

Textures consume significant memory:

**Calculation:**
- RGBA32: width × height × 4 bytes
- RGB24: width × height × 3 bytes
- ASTC 6×6: width × height / 36 bytes

**1024×1024 RGBA32**: 4 MB
**1024×1024 ASTC 6×6**: 178 KB (23× smaller)

**Optimization:**
- Use compressed formats (ASTC, DXT, ETC)
- Enable mipmaps for 3D textures
- Disable mipmaps for UI
- Use Texture Arrays for similar textures
- Stream textures with AssetBundles

## Best Practices Summary

✅ **DO:**
- Pre-allocate collection capacities
- Cache WaitForSeconds in coroutines
- Use NonAlloc Physics APIs
- Implement object pooling for spawned objects
- Profile with Memory Profiler regularly
- Use structs for small value types
- Unsubscribe from all events in OnDisable/OnDestroy
- Cache string concatenations with StringBuilder

❌ **DON'T:**
- Use LINQ in Update/FixedUpdate
- Create new collections in Update
- Concatenate strings in hot paths
- Box value types unnecessarily
- Forget to unsubscribe from events
- Use FindObjectsOfType repeatedly
- Access GetComponents in loops
- Allocate lambdas that capture variables

**Golden rule**: Profile before optimizing. Measure impact of changes.

Follow these memory optimization patterns for smooth, GC-spike-free Unity games.
