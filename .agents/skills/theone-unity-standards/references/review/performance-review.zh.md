# 性能代码审查

## 分配检查

### 检查清单
- [ ] 是否避免了在 Update/FixedUpdate 中进行内存分配？
- [ ] 是否对频繁实例化的对象使用了对象池？
- [ ] 字符串拼接是否经过优化？

### 常见违规

#### ❌ Update 中的内存分配
```csharp
// Bad: 每帧都在分配内存
private void Update()
{
    var enemies = new List<Enemy>(); // ❌ 每帧分配
    var position = new Vector3(0, 0, 0); // ❌ 不必要的分配
    var name = "Player: " + playerName; // ❌ 字符串分配
}
```

### ✅ 避免分配
```csharp
// Good: 重用集合
private List<Enemy> enemyCache = new List<Enemy>();

private void Update()
{
    this.enemyCache.Clear(); // 重用而非新建
}

// Good: 使用静态属性
private void Update()
{
    var position = Vector3.zero; // 无分配
}

// Good: 缓存字符串或使用 StringBuilder
private string cachedPlayerName;

private void UpdatePlayerName()
{
    this.cachedPlayerName = $"Player: {playerName}";
}
```

## 字符串操作

### 常见违规

#### ❌ 循环中的字符串拼接
```csharp
// Bad: 循环中进行字符串拼接
foreach (var player in players)
{
    var name = "Player: " + player.Name; // ❌ 产生垃圾内存
    Debug.Log(name);
}

// Bad: 重复拼接
var result = "";
for (int i = 0; i < 100; i++)
{
    result += i.ToString(); // ❌ 创建了 100 个字符串对象
}
```

### ✅ 优化的字符串操作
```csharp
// Good: 字符串插值（编译为优化代码）
foreach (var player in players)
{
    Debug.Log($"Player: {player.Name}");
}

// Good: 使用 StringBuilder 进行重复拼接
var builder = new StringBuilder();
for (int i = 0; i < 100; i++)
{
    builder.Append(i);
}
var result = builder.ToString();
```

## LINQ 性能

### 检查清单
- [ ] 是否在 Update/FixedUpdate 热路径中避免了使用 LINQ？
- [ ] LINQ 链是否经过优化（避免多次迭代）？

### 常见违规

#### ❌ 多次 LINQ 迭代
```csharp
// Bad: 多次 LINQ 迭代
private void Update()
{
    var active = enemies.Where(e => e.IsActive).ToList();
    var alive = active.Where(e => e.Health > 0).ToList();
    var close = alive.Where(e => Vector3.Distance(e.transform.position, transform.position) < 10f).ToList();
}
```

### ✅ 优化的迭代
```csharp
// Good: 单次迭代
private void Update()
{
    var closeEnemies = enemies
        .Where(e => e.IsActive
                 && e.Health > 0
                 && Vector3.Distance(e.transform.position, transform.position) < 10f)
        .ToList();
}

// Even better: 缓存并在热路径中使用 for 循环
private void Update()
{
    for (int i = 0; i < enemies.Count; i++)
    {
        var e = enemies[i];
        if (e.IsActive && e.Health > 0 && Vector3.Distance(e.transform.position, transform.position) < 10f)
        {
            // 处理
        }
    }
}
```

## GameObject 实例化

### 常见违规

#### ❌ 循环中实例化
```csharp
// Bad: 每次都实例化
private void SpawnEnemies()
{
    for (int i = 0; i < 10; i++)
    {
        Instantiate(enemyPrefab); // ❌ 开销大
    }
}
```

### ✅ 使用对象池
```csharp
// Good: 使用对象池
private void SpawnEnemies()
{
    for (int i = 0; i < 10; i++)
    {
        var enemy = objectPool.Get(); // ✅ 从池中重用
        enemy.Initialize();
    }
}

private void DespawnEnemy(GameObject enemy)
{
    objectPool.Return(enemy); // ✅ 返回池中
}
```

## 组件访问性能

### 常见违规

#### ❌ 重复调用 GetComponent
```csharp
// Bad: 循环中调用 GetComponent
private void Update()
{
    foreach (var enemy in enemies)
    {
        var health = enemy.GetComponent<Health>(); // ❌ 循环中开销大
        health.TakeDamage(1);
    }
}
```

### ✅ 缓存组件
```csharp
// Good: 缓存组件
private List<Health> enemyHealths = new List<Health>();

private void Start()
{
    foreach (var enemy in enemies)
    {
        enemyHealths.Add(enemy.GetComponent<Health>());
    }
}

private void Update()
{
    foreach (var health in enemyHealths)
    {
        health.TakeDamage(1);
    }
}
```

## 物理性能

### 常见违规

#### ❌ Update 中的射线检测
```csharp
// Bad: 每帧进行射线检测
private void Update()
{
    RaycastHit hit;
    if (Physics.Raycast(transform.position, transform.forward, out hit))
    {
        // 处理命中
    }
}
```

### ✅ 优化射线检测
```csharp
// Good: 带计时的射线检测
private float lastRaycastTime;
private const float RaycastInterval = 0.1f; // 每秒 10 次

private void Update()
{
    if (Time.time - this.lastRaycastTime < RaycastInterval)
        return;

    this.lastRaycastTime = Time.time;

    if (Physics.Raycast(transform.position, transform.forward, out var hit))
    {
        // 处理命中
    }
}
```

## 集合性能

### 常见违规

#### ❌ 错误的集合类型
```csharp
// Bad: 使用 List 进行频繁查找
private List<string> playerIds = new List<string>();

private bool HasPlayer(string id)
{
    return playerIds.Contains(id); // ❌ O(n) 查找
}
```

### ✅ 使用合适的集合
```csharp
// Good: 使用 HashSet 进行频繁查找
private HashSet<string> playerIds = new HashSet<string>();

private bool HasPlayer(string id)
{
    return playerIds.Contains(id); // ✅ O(1) 查找
}
```

## 装箱与拆箱

### 常见违规

#### ❌ 热路径中的装箱
```csharp
// Bad: 值类型装箱
private void Update()
{
    var dict = new Dictionary<string, object>();
    dict["health"] = 100; // ❌ int 装箱为 object
    dict["damage"] = 25.5f; // ❌ float 装箱为 object
}
```

### ✅ 避免装箱
```csharp
// Good: 使用泛型
private void Update()
{
    var healthDict = new Dictionary<string, int>();
    healthDict["health"] = 100; // ✅ 无装箱

    var damageDict = new Dictionary<string, float>();
    damageDict["damage"] = 25.5f; // ✅ 无装箱
}
```

## 完整性能示例

### ❌ 性能糟糕的代码
```csharp
public class CombatSystem : MonoBehaviour
{
    private void Update()
    {
        // ❌ 多次分配
        var enemies = new List<Enemy>();

        // ❌ 多次 LINQ 迭代
        var activeEnemies = FindObjectsOfType<Enemy>()
            .Where(e => e.IsActive)
            .ToList();
        var aliveEnemies = activeEnemies
            .Where(e => e.Health > 0)
            .ToList();

        // ❌ 循环中调用 GetComponent
        foreach (var enemy in aliveEnemies)
        {
            var health = enemy.GetComponent<Health>();

            // ❌ 字符串拼接
            var name = "Enemy: " + enemy.name;
            Debug.Log(name);

            // ❌ 创建新的 Vector3
            var direction = new Vector3(1, 0, 0);
            enemy.transform.position += direction;
        }

        // ❌ 每次都实例化
        if (aliveEnemies.Count < 5)
        {
            Instantiate(enemyPrefab);
        }
    }
}
```

### ✅ 优化后的性能代码
```csharp
public class CombatSystem : MonoBehaviour
{
    // ✅ 缓存集合
    private List<Enemy> enemies = new List<Enemy>();
    private List<Health> enemyHealths = new List<Health>();

    // ✅ 对象池
    private ObjectPool<Enemy> enemyPool;

    private void Start()
    {
        // ✅ 查找并缓存一次
        this.enemies.AddRange(FindObjectsOfType<Enemy>());

        // ✅ 缓存组件
        foreach (var enemy in this.enemies)
        {
            if (enemy.TryGetComponent<Health>(out var health))
            {
                this.enemyHealths.Add(health);
            }
        }

        this.enemyPool = new ObjectPool<Enemy>(
            () => Instantiate(enemyPrefab),
            e => e.gameObject.SetActive(true),
            e => e.gameObject.SetActive(false)
        );
    }

    private void Update()
    {
        // ✅ 使用 for 循环进行单次迭代
        for (int i = 0; i < this.enemies.Count; i++)
        {
            var enemy = this.enemies[i];
            if (!enemy.IsActive || enemy.Health <= 0)
                continue;

            // ✅ 已缓存生命值
            var health = this.enemyHealths[i];

            // ✅ 使用 Vector3.right
            enemy.transform.position += Vector3.right * Time.deltaTime;
        }

        // ✅ 使用对象池
        if (this.enemies.Count < 5)
        {
            var enemy = this.enemyPool.Get();
            this.enemies.Add(enemy);
        }
    }
}
```

## 审查严重等级

### 🔴 严重性能问题
- Update/FixedUpdate 中的内存分配
- 热路径中使用 LINQ 且无缓存
- 循环中使用 GetComponent/Find
- 实例化未使用对象池

### 🟡 重要性能问题
- 多次 LINQ 迭代
- 循环中的字符串拼接
- 用例使用了错误的集合类型
- 热路径中的装箱
- 无节流的重复射线检测

### 🟢 建议
- 可以使用 StringBuilder 构建字符串
- 可以使用 HashSet 进行更快的查找
- 可以更积极地进行缓存
- 可以使用对象池
