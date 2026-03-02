# Unity 内存优化 - 完整指南

减少垃圾回收（Garbage Collection）、管理内存分配以及优化 Unity 游戏内存使用的高级技巧。

## 垃圾回收（Garbage Collection）基础

### Unity 的 GC 如何工作

Unity 使用 Boehm-Demers-Weiser 垃圾回收器：

**关键特征：**
- **非分代（Non-generational）**：扫描所有托管内存
- **Stop-the-world**：在回收期间暂停游戏执行
- **标记清除（Mark-and-sweep）**：标记可达对象，清除不可达对象
- **增量式（Incremental）**：Unity 2019+ 支持增量 GC（将工作分散到多个帧）

**GC 触发条件：**
- 堆内存已满（自动）
- 显式调用 `GC.Collect()`
- 场景加载
- 操作系统内存压力

**性能影响**：GC 峰值会导致掉帧、卡顿。

### 分配来源

常见的分配来源（按严重程度排序）：

1.  **字符串操作**（连接、格式化）
2.  **装箱（Boxing）**（值类型 → 引用类型）
3.  **集合**（List、Dictionary 扩容、LINQ）
4.  **闭包（Closures）**（Lambda 捕获、委托）
5.  **协程（Coroutines）**（yield 语句、WaitForSeconds）
6.  **Unity API 分配**（GetComponents、Find、foreach）

## 字符串优化

### StringBuilder 模式

```csharp
// ❌ 坏 - 分配多个字符串
private void UpdateUI()
{
    string text = "Player: " + playerName + " | Health: " + health + "/" + maxHealth;
    uiText.text = text;
}

// ✅ 好 - 重用 StringBuilder
private StringBuilder sb = new StringBuilder(100);  // 预分配容量

private void UpdateUI()
{
    sb.Clear();
    sb.Append("Player: ").Append(playerName)
      .Append(" | Health: ").Append(health)
      .Append("/").Append(maxHealth);
    uiText.text = sb.ToString();
}
```

**容量**：预分配 StringBuilder 容量以防止调整大小时产生分配。

### 字符串驻留（String Interning）

缓存频繁使用的字符串：

```csharp
// ❌ 坏 - 创建新字符串
private void LogState(string state)
{
    if (state == "Idle") { }
    if (state == "Running") { }
}

// ✅ 好 - 缓存字符串
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

// ✅ 更好 - 使用枚举代替字符串
private enum State { Idle, Running, Attacking }
private State currentState;
```

### 字符串格式化

```csharp
// ❌ 坏 - string.Format 会产生分配
private void Update()
{
    text.text = string.Format("Score: {0}", score);  // 产生分配
}

// ✅ 好 - 直接连接或缓存格式
private void UpdateScore(int newScore)
{
    score = newScore;
    text.text = "Score: " + score.ToString();  // 分配较少
}

// ✅ 最好 - 除非改变否则不更新
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

## 集合优化

### List 容量管理

预分配 List 容量以防止调整大小：

```csharp
// ❌ 坏 - List 动态增长，导致分配
private List<Enemy> enemies = new List<Enemy>();  // 容量 = 0，增长为 4, 8, 16...

// ✅ 好 - 预分配预期容量
private List<Enemy> enemies = new List<Enemy>(100);  // 容量 = 100

// ✅ 最好 - 重用 List，清除而不是新建
private List<Enemy> tempEnemies = new List<Enemy>(50);

private void FindNearbyEnemies()
{
    tempEnemies.Clear();  // 重用现有 List
    // 填充 tempEnemies
}
```

**容量规则：**
- 为已知/估计的大小预分配容量
- 在批量添加之前使用 `EnsureCapacity()`
- 清除并重用而不是创建新列表

### Dictionary 容量

```csharp
// ❌ 坏 - Dictionary 增长，导致重新哈希（rehashing）
private Dictionary<int, Enemy> enemyLookup = new Dictionary<int, Enemy>();

// ✅ 好 - 预分配容量
private Dictionary<int, Enemy> enemyLookup = new Dictionary<int, Enemy>(200);
```

### 数组池化（Array Pooling）

重用数组而不是分配新数组：

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
        System.Array.Clear(array, 0, array.Length);  // 清除数据
        pool.Push(array);
    }
}

// 用法
private ArrayPool<Vector3> vectorPool = new ArrayPool<Vector3>(100, 5);

private void ProcessPositions()
{
    Vector3[] positions = vectorPool.Rent();

    // 使用数组

    vectorPool.Return(positions);
}
```

**Unity 2021+**：使用来自 .NET 的 `System.Buffers.ArrayPool<T>`。

## 避免装箱（Boxing Avoidance）

### 值类型装箱

当值类型转换为引用类型时会发生装箱：

```csharp
// ❌ 坏 - 装箱分配
private void LogValue(object value)  // object 参数导致装箱
{
    Debug.Log(value);
}

int health = 100;
LogValue(health);  // 装箱: int → object

// ✅ 好 - 泛型方法避免装箱
private void LogValue<T>(T value)
{
    Debug.Log(value);
}

int health = 100;
LogValue(health);  // 无装箱
```

### 值类型字典

```csharp
// ❌ 坏 - 枚举器（Enumerator）产生分配
foreach (var kvp in dictionary)  // 分配枚举器
{
    Process(kvp.Key, kvp.Value);
}

// ✅ 好 - For 循环避免分配（针对 List）
for (int i = 0; i < list.Count; i++)
{
    Process(list[i]);
}

// ✅ 好 - 使用结构体枚举器（Unity 2021+）
foreach (var kvp in dictionary)
{
    // 现代 Unity 使用结构体枚举器（无分配）
}
```

## LINQ 分配

LINQ 方法分配严重 - 避免在热路径（hot paths）中使用：

```csharp
// ❌ 坏 - LINQ 分配闭包、枚举器
private void Update()
{
    var nearbyEnemies = enemies.Where(e => Vector3.Distance(e.position, player.position) < range)
                                .OrderBy(e => e.health)
                                .ToList();  // 多次分配！
}

// ✅ 好 - 手动循环
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

    // 如果需要，手动排序
    nearbyEnemies.Sort((a, b) => a.health.CompareTo(b.health));
}
```

**LINQ 分配来源：**
- `Where()` - 分配谓词闭包
- `Select()` - 分配转换
- `OrderBy()` - 分配比较器
- `ToList()`, `ToArray()` - 分配集合

**仅在以下情况使用 LINQ：**
- 初始化代码（Awake, Start）
- 编辑器代码
- 不频繁的操作（场景加载、保存）

## 协程优化（Coroutine Optimization）

### 缓存 Yield 指令

```csharp
// ❌ 坏 - 每次调用都分配 WaitForSeconds
private IEnumerator SpawnEnemies()
{
    while (true)
    {
        SpawnEnemy();
        yield return new WaitForSeconds(1f);  // 每次迭代都有新分配
    }
}

// ✅ 好 - 缓存 yield 指令
private readonly WaitForSeconds oneSecond = new WaitForSeconds(1f);
private readonly WaitForSeconds halfSecond = new WaitForSeconds(0.5f);
private readonly WaitForEndOfFrame endOfFrame = new WaitForEndOfFrame();

private IEnumerator SpawnEnemies()
{
    while (true)
    {
        SpawnEnemy();
        yield return oneSecond;  // 重用缓存实例
    }
}
```

### 空 Yield 缓存

```csharp
// ❌ 坏 - yield return null 在某些 Unity 版本中仍会分配
private IEnumerator DoWork()
{
    yield return null;  // 可能分配
}

// ✅ 好 - 缓存空 yield
private static readonly WaitForEndOfFrame waitFrame = new WaitForEndOfFrame();

private IEnumerator DoWork()
{
    yield return waitFrame;  // 无分配
}
```

### 协程池化

对于频繁启动/停止的协程，使用带有状态机的 Update 代替。

## 闭包和 Lambda 优化

### Lambda 捕获

捕获变量的 Lambda 会分配闭包：

```csharp
// ❌ 坏 - Lambda 捕获 'damage'，分配闭包
private void DealDamageToAll(int damage)
{
    enemies.ForEach(enemy => enemy.TakeDamage(damage));  // 分配闭包
}

// ✅ 好 - 使用 for 循环，无 lambda
private void DealDamageToAll(int damage)
{
    for (int i = 0; i < enemies.Count; i++)
    {
        enemies[i].TakeDamage(damage);
    }
}

// ✅ 替代方案 - 实例方法（无捕获）
private int currentDamage;

private void DealDamageToAll(int damage)
{
    currentDamage = damage;
    enemies.ForEach(DealDamageToEnemy);  // 无捕获，无分配
}

private void DealDamageToEnemy(Enemy enemy)
{
    enemy.TakeDamage(currentDamage);
}
```

### 委托缓存

缓存委托实例：

```csharp
// ❌ 坏 - 每次都创建新委托
public event Action OnDeath;

private void Subscribe()
{
    OnDeath += () => Debug.Log("Died");  // 新委托分配
}

// ✅ 好 - 缓存委托
private Action onDeathHandler;

private void Awake()
{
    onDeathHandler = HandleDeath;  // 缓存一次
}

private void Subscribe()
{
    OnDeath += onDeathHandler;  // 重用缓存的委托
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

## Unity API 分配

### GetComponents 变体

```csharp
// ❌ 坏 - GetComponents 分配数组
private void Update()
{
    Collider[] colliders = GetComponents<Collider>();  // 分配数组
}

// ✅ 好 - GetComponents 配合 List（如果 List 有容量则无分配）
private List<Collider> colliders = new List<Collider>(10);

private void Update()
{
    GetComponents(colliders);  // 重用 List
}
```

### 物理查询

```csharp
// ❌ 坏 - Physics.RaycastAll 分配数组
private void Update()
{
    RaycastHit[] hits = Physics.RaycastAll(origin, direction);  // 分配
}

// ✅ 好 - Physics.RaycastNonAlloc
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

**NonAlloc 变体：**
- `Physics.RaycastNonAlloc()`
- `Physics.SphereCastNonAlloc()`
- `Physics.OverlapSphereNonAlloc()`
- `Physics2D.RaycastNonAlloc()`

### GameObject.Find 分配

```csharp
// ❌ 坏 - FindObjectsOfType 每次调用都分配数组
private void Update()
{
    Enemy[] enemies = FindObjectsOfType<Enemy>();  // 分配！
}

// ✅ 好 - 维护列表，注册/注销模式
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

// 用法 - 无分配
private void Update()
{
    var enemies = EnemyManager.AllEnemies;
    for (int i = 0; i < enemies.Count; i++)
    {
        // 处理 enemy
    }
}
```

## 结构体优化

对小型、值语义类型使用结构体：

```csharp
// 值语义数据 - 使用 struct
public struct DamageInfo
{
    public int damage;
    public Vector3 hitPoint;
    public Vector3 hitNormal;
    public GameObject attacker;
}

// 引用语义行为 - 使用 class
public class Enemy : MonoBehaviour
{
    public void TakeDamage(DamageInfo damageInfo)
    {
        health -= damageInfo.damage;
    }
}
```

**结构体准则：**
- 小尺寸（<16 字节最理想）
- 值语义（拷贝是有意义的）
- 首选不可变（Immutable）
- 避免大结构体（拷贝开销）

## 内存分析

### 使用 Memory Profiler

**Window > Analysis > Memory Profiler**

**捕获快照：**
1. 启动时拍摄快照
2. 玩一会儿
3. 拍摄第二个快照
4. 比较以发现泄漏

**寻找：**
- 增长的托管堆
- 泄漏的纹理/网格
- 保留的事件处理程序
- 未回收的对象

### 检测泄漏

**常见泄漏源：**
1. **未关闭的事件处理程序**
```csharp
// 泄漏 - 从未取消订阅
private void Start()
{
    GameManager.OnLevelComplete += HandleComplete;  // 从未移除！
}
```

2. **静态引用**
```csharp
// 泄漏 - 静态持有引用
private static Player currentPlayer;  // 从未清除，阻止 GC
```

3. **委托引用**
```csharp
// 泄漏 - 匿名方法捕获 'this'
private void Start()
{
    GameManager.OnUpdate += () => UpdatePlayer();  // 捕获 this，阻止 GC
}
```

### GC.GetTotalMemory

监控托管内存增长：

```csharp
private void LogMemory()
{
    long memory = GC.GetTotalMemory(false);
    Debug.Log($"Managed memory: {memory / (1024 * 1024)} MB");
}
```

## 对象池模式

### 通用池

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

### 粒子池

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

## 平台特定内存

### 移动端内存限制

移动设备 RAM 有限（总共 2-4GB，应用仅获部分）：

**移动端优化：**
- 压缩纹理（ASTC, ETC2）
- 降低纹理分辨率
- 减少网格密度
- 卸载未使用的资源
- 使用 AssetBundles 进行流式传输
- 使用 Xcode/Android Profiler 进行监控

### 纹理内存

纹理消耗大量内存：

**计算：**
- RGBA32：宽 × 高 × 4 字节
- RGB24：宽 × 高 × 3 字节
- ASTC 6×6：宽 × 高 / 36 字节

**1024×1024 RGBA32**：4 MB
**1024×1024 ASTC 6×6**：178 KB（小 23 倍）

**优化：**
- 使用压缩格式（ASTC, DXT, ETC）
- 对 3D 纹理启用 Mipmap
- 对 UI 禁用 Mipmap
- 对相似纹理使用纹理数组（Texture Arrays）
- 使用 AssetBundles 流式传输纹理

## 最佳实践总结

✅ **该做：**
- 预分配集合容量
- 在协程中缓存 WaitForSeconds
- 使用 NonAlloc 物理 API
- 为生成的对象实现对象池
- 定期使用 Memory Profiler 进行分析
- 对小型值类型使用结构体
- 在 OnDisable/OnDestroy 中取消订阅所有事件
- 使用 StringBuilder 缓存字符串连接

❌ **不该做：**
- 在 Update/FixedUpdate 中使用 LINQ
- 在 Update 中创建新集合
- 在热路径中连接字符串
- 不必要地装箱值类型
- 忘记取消订阅事件
- 重复使用 FindObjectsOfType
- 在循环中访问 GetComponents
- 分配捕获变量的 Lambda

**黄金法则**：先分析，后优化。测量变更的影响。

遵循这些内存优化模式，以实现流畅、无 GC 峰值的 Unity 游戏。
