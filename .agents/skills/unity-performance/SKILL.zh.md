---
name: Unity Performance
description: 当用户询问“Unity 性能”、“优化”、“GC 分配”、“对象池”、“缓存”、“Update 循环优化”、“内存管理”、“性能分析”、“帧率”、“垃圾回收”或需要 Unity 游戏性能最佳实践指导时，应使用此技能。
version: 0.1.0
---

# Unity 性能优化

针对 Unity 游戏的必要性能优化技术，涵盖内存管理、CPU 优化、渲染和性能分析策略。

## 概述

性能对于所有平台上的 Unity 游戏都至关重要。性能不佳表现为低帧率、卡顿、加载时间长和崩溃。本技能涵盖适用于所有 Unity 项目的经过验证的优化技术。

**核心优化领域：**
- CPU 优化（Update 循环、缓存、对象池）
- 内存管理（减少 GC、分配模式）
- 渲染优化（批处理、剔除、LOD）
- 性能分析与测量（识别瓶颈）

## 引用缓存

最常见的 Unity 性能错误是重复进行昂贵的查找。缓存所有引用以避免多余的操作。

### GetComponent 缓存

切勿重复调用 GetComponent —— 在 Awake 中缓存结果：

```csharp
// ❌ 坏 - 每帧调用 GetComponent
private void Update()
{
    GetComponent<Rigidbody>().velocity = Vector3.forward;  // 慢！
}

// ✅ 好 - 缓存一次
private Rigidbody rb;

private void Awake()
{
    rb = GetComponent<Rigidbody>();
}

private void Update()
{
    rb.velocity = Vector3.forward;  // 快
}
```

**性能影响**：GetComponent 比缓存引用慢 10-100 倍。

### Transform 缓存

缓存 `transform` 访问，特别是对于频繁访问的 GameObject：

```csharp
// ❌ 坏 - 属性访问开销
private void Update()
{
    transform.position += Vector3.forward * Time.deltaTime;
    transform.rotation = Quaternion.identity;
}

// ✅ 好 - 缓存 transform 引用
private Transform myTransform;

private void Awake()
{
    myTransform = transform;
}

private void Update()
{
    myTransform.position += Vector3.forward * Time.deltaTime;
    myTransform.rotation = Quaternion.identity;
}
```

**原因**：`transform` 属性有开销。缓存引用消除了重复查找。

### Find 方法缓存

切勿在 Update 中使用 Find 方法 —— 缓存结果：

```csharp
// ❌ 坏 - 每帧查找（极慢）
private void Update()
{
    GameObject player = GameObject.Find("Player");
    Transform target = GameObject.FindWithTag("Enemy").transform;
}

// ✅ 好 - 在 Start 中缓存
private GameObject player;
private Transform target;

private void Start()
{
    player = GameObject.Find("Player");
    target = GameObject.FindWithTag("Enemy")?.transform;
}

private void Update()
{
    // 使用缓存的引用
}
```

**性能影响**：Find 方法扫描整个场景层级。比缓存引用慢 100-1000 倍。

### Material 缓存

访问 `renderer.material` 会创建新的 Material 实例 —— 缓存以避免泄漏：

```csharp
// ❌ 坏 - 每帧创建新 Material（内存泄漏）
private void Update()
{
    GetComponent<Renderer>().material.color = Color.red;  // 创建新 Material！
}

// ✅ 好 - 缓存 material 引用
private Material material;

private void Awake()
{
    material = GetComponent<Renderer>().material;
}

private void Update()
{
    material.color = Color.red;  // 修改缓存的 Material
}

private void OnDestroy()
{
    // 清理实例化的 material
    if (material != null)
        Destroy(material);
}
```

**关键**：访问 `.material` 会创建新实例。使用 `.sharedMaterial` 进行只读访问以避免实例化。

## 对象池 (Object Pooling)

Instantiate 和 Destroy 均很昂贵。重用对象而不是重复创建/销毁。

### 基础池实现

```csharp
public class ObjectPool : MonoBehaviour
{
    [SerializeField] private GameObject prefab;
    [SerializeField] private int initialSize = 10;

    private Queue<GameObject> pool = new Queue<GameObject>();

    private void Awake()
    {
        // 预实例化对象
        for (int i = 0; i < initialSize; i++)
        {
            GameObject obj = Instantiate(prefab);
            obj.SetActive(false);
            pool.Enqueue(obj);
        }
    }

    public GameObject Get()
    {
        if (pool.Count > 0)
        {
            GameObject obj = pool.Dequeue();
            obj.SetActive(true);
            return obj;
        }

        // 池已耗尽 - 创建新的
        return Instantiate(prefab);
    }

    public void Return(GameObject obj)
    {
        obj.SetActive(false);
        pool.Enqueue(obj);
    }
}
```

**适用于：**
- 子弹、投射物
- 粒子效果
- UI 元素（工具提示、伤害数字）
- 波次游戏中的敌人
- 音频源

**性能提升**：比 Instantiate/Destroy 快 10-50 倍，消除 GC 峰值。

### 池模式用法

```csharp
public class BulletSpawner : MonoBehaviour
{
    [SerializeField] private ObjectPool bulletPool;
    [SerializeField] private Transform firePoint;

    private void Fire()
    {
        // 从池中获取
        GameObject bullet = bulletPool.Get();
        bullet.transform.position = firePoint.position;
        bullet.transform.rotation = firePoint.rotation;

        // 3秒后返回
        StartCoroutine(ReturnToPoolAfterDelay(bullet, 3f));
    }

    private IEnumerator ReturnToPoolAfterDelay(GameObject obj, float delay)
    {
        yield return new WaitForSeconds(delay);
        bulletPool.Return(obj);
    }
}
```

## Update 循环优化

Update、FixedUpdate 和 LateUpdate 被频繁调用 —— 尽量减少这些方法中完成的工作。

### 移除空的 Update 方法

```csharp
// ❌ 坏 - 空方法仍有开销
private void Update() { }
private void FixedUpdate() { }

// ✅ 好 - 移除未使用的方法
// (如果不需要，则没有 Update/FixedUpdate)
```

**性能**：即使为空，Unity 也会调用所有 Update 方法。移除以减少开销。

### 降低 Update 频率

并非所有逻辑都需要每帧运行：

```csharp
// ❌ 坏 - 每帧进行昂贵的检查
private void Update()
{
    CheckForNearbyEnemies();  // 昂贵的射线检测/距离检查
}

// ✅ 好 - 每 N 帧检查一次
private int frameCounter = 0;
private const int checkInterval = 10;

private void Update()
{
    frameCounter++;
    if (frameCounter >= checkInterval)
    {
        frameCounter = 0;
        CheckForNearbyEnemies();
    }
}

// ✅ 更好 - 使用 InvokeRepeating 或 Coroutine
private void Start()
{
    InvokeRepeating(nameof(CheckForNearbyEnemies), 0f, 0.2f);  // 每 0.2 秒
}
```

**替代方案：协程 (Coroutines)**
```csharp
private void Start()
{
    StartCoroutine(CheckEnemiesRoutine());
}

private IEnumerator CheckEnemiesRoutine()
{
    while (true)
    {
        CheckForNearbyEnemies();
        yield return new WaitForSeconds(0.2f);
    }
}
```

### 事件驱动架构

用事件代替轮询：

```csharp
// ❌ 坏 - 每帧轮询状态变化
private bool wasGrounded;

private void Update()
{
    bool grounded = IsGrounded();
    if (grounded != wasGrounded)
    {
        OnGroundedChanged(grounded);
    }
    wasGrounded = grounded;
}

// ✅ 好 - 事件驱动
public event Action<bool> OnGroundedChanged;

private bool isGrounded;

private void SetGrounded(bool grounded)
{
    if (isGrounded != grounded)
    {
        isGrounded = grounded;
        OnGroundedChanged?.Invoke(grounded);
    }
}
```

## 减少垃圾回收 (Garbage Collection Reduction)

避免在频繁调用的方法中进行分配，以防止 GC 峰值。

### 字符串拼接

```csharp
// ❌ 坏 - 每帧分配字符串
private void Update()
{
    string message = "Health: " + health;  // 字符串分配
    scoreText.text = "Score: " + score;    // 字符串分配
}

// ✅ 好 - 使用 StringBuilder 或字符串插值
private StringBuilder sb = new StringBuilder();

private void UpdateUI()
{
    sb.Clear();
    sb.Append("Health: ").Append(health);
    healthText.text = sb.ToString();
}

// ✅ 替代方案 - 缓存格式化字符串
private void UpdateHealth(int newHealth)
{
    health = newHealth;
    healthText.text = health.ToString();  // 分配比拼接少
}
```

### 集合分配

```csharp
// ❌ 坏 - 每帧分配新列表
private void Update()
{
    List<Enemy> nearbyEnemies = new List<Enemy>();  // GC 分配！
    FindNearbyEnemies(nearbyEnemies);
}

// ✅ 好 - 重用列表
private List<Enemy> nearbyEnemies = new List<Enemy>();

private void Update()
{
    nearbyEnemies.Clear();  // 重用现有列表
    FindNearbyEnemies(nearbyEnemies);
}
```

### 数组/列表最佳实践

```csharp
// ❌ 坏 - ToArray 分配内存
private void Update()
{
    GameObject[] enemies = enemyList.ToArray();  // GC 分配！
}

// ✅ 好 - 直接迭代列表
private void Update()
{
    for (int i = 0; i < enemyList.Count; i++)
    {
        Enemy enemy = enemyList[i];
        // 处理 enemy
    }
}

// ✅ 好 - 使用 foreach (对 List 无分配)
private void Update()
{
    foreach (var enemy in enemyList)
    {
        // 处理 enemy
    }
}
```

### 协程分配

```csharp
// ❌ 坏 - 每次调用分配 WaitForSeconds
private IEnumerator DelayedAction()
{
    yield return new WaitForSeconds(1f);  // 每次都新分配
}

// ✅ 好 - 缓存 WaitForSeconds
private WaitForSeconds oneSecondWait = new WaitForSeconds(1f);

private IEnumerator DelayedAction()
{
    yield return oneSecondWait;  // 重用缓存的 wait
}
```

## 组件访问模式

### 最小化组件查询

```csharp
// ❌ 坏 - 多次 GetComponent 调用
private void OnTriggerEnter(Collider other)
{
    if (other.GetComponent<Enemy>() != null)
    {
        other.GetComponent<Enemy>().TakeDamage(10);  // 调用了两次！
    }
}

// ✅ 好 - 单次 GetComponent 配合模式匹配
private void OnTriggerEnter(Collider other)
{
    if (other.TryGetComponent<Enemy>(out var enemy))
    {
        enemy.TakeDamage(10);  // 调用一次
    }
}
```

### 碰撞时的组件缓存

```csharp
// ❌ 坏 - 每次碰撞都调用 GetComponent
private void OnTriggerEnter(Collider other)
{
    var damageable = other.GetComponent<IDamageable>();
    if (damageable != null)
        damageable.TakeDamage(10);
}

// ✅ 好 - 在触发进入时缓存组件
private Dictionary<Collider, IDamageable> damageableCache = new Dictionary<Collider, IDamageable>();

private void OnTriggerEnter(Collider other)
{
    if (!damageableCache.TryGetValue(other, out var damageable))
    {
        damageable = other.GetComponent<IDamageable>();
        damageableCache[other] = damageable;  // 为未来碰撞缓存
    }

    damageable?.TakeDamage(10);
}

private void OnTriggerExit(Collider other)
{
    damageableCache.Remove(other);  // 清理缓存
}
```

## 物理优化

### 基于层的碰撞

配置物理层碰撞矩阵 (Physics Layer Collision Matrix) 以防止不必要的碰撞检查：

**Edit > Project Settings > Physics > Layer Collision Matrix**

```csharp
// 设置层
Layer 8: Player
Layer 9: Enemies
Layer 10: Projectiles
Layer 11: Environment

// 禁用不必要的碰撞：
- Player vs Player (禁用)
- Enemies vs Enemies (禁用)
- Projectiles vs Projectiles (禁用)
```

**性能提升**：减少 30-50% 的物理开销。

### 射线检测 (Raycast) 优化

```csharp
// ❌ 坏 - Raycast 检查所有物体
bool hit = Physics.Raycast(origin, direction, out RaycastHit hitInfo);

// ✅ 好 - 层掩码 (Layer mask) 限制检查
int layerMask = 1 << LayerMask.NameToLayer("Enemy");
bool hit = Physics.Raycast(origin, direction, out RaycastHit hitInfo, maxDistance, layerMask);

// ✅ 更好 - 缓存层掩码
private int enemyLayerMask;

private void Awake()
{
    enemyLayerMask = 1 << LayerMask.NameToLayer("Enemy");
}

private void Fire()
{
    bool hit = Physics.Raycast(origin, direction, out RaycastHit hitInfo, maxDistance, enemyLayerMask);
}
```

### 刚体休眠 (Rigidbody Sleep)

当刚体不移动时让其休眠：

```csharp
// 当速度 < 阈值时，Rigidbody 自动休眠
// 在 Edit > Project Settings > Physics 中配置

// 需要时手动唤醒
private void ApplyForce()
{
    if (rb.IsSleeping())
        rb.WakeUp();

    rb.AddForce(force);
}
```

## 性能分析 (Profiling)

先测量，后优化。使用 Unity Profiler 识别实际瓶颈。

**打开 Profiler: Window > Analysis > Profiler**

### 关键 Profiler 指标

**CPU 使用率：**
- 渲染 (DrawCalls, SetPass calls)
- 脚本 (Update, FixedUpdate, Coroutines)
- 物理 (FixedUpdate.PhysicsFixedUpdate)
- GC.Alloc (垃圾回收分配)

**内存：**
- 总分配 (Total Allocated)
- GC 分配 (GC Allocated)
- 纹理内存
- 网格内存

### 性能分析工作流

1.  **识别瓶颈**：运行 Profiler，找到开销大的帧
2.  **深入分析**：点击峰值，查看调用层级
3.  **测量基准**：记录当前性能
4.  **应用优化**：进行针对性更改
5.  **测量改进**：比较前/后
6.  **重复**：寻找下一个瓶颈

### 深度分析 (Deep Profile)

启用 **Deep Profile** 以获取详细的调用堆栈（影响性能）：

**警告**：Deep Profile 会显著减慢游戏速度。仅用于小场景或针对性分析。

### 自定义 Profiler 标记

测量特定代码段：

```csharp
using Unity.Profiling;

public class AIController : MonoBehaviour
{
    private static readonly ProfilerMarker s_PathfindingMarker = new ProfilerMarker("AI.Pathfinding");
    private static readonly ProfilerMarker s_DecisionMarker = new ProfilerMarker("AI.DecisionMaking");

    private void Update()
    {
        s_PathfindingMarker.Begin();
        CalculatePath();
        s_PathfindingMarker.End();

        s_DecisionMarker.Begin();
        MakeDecision();
        s_DecisionMarker.End();
    }

    private void CalculatePath() { }
    private void MakeDecision() { }
}
```

在 Profiler 中显示自定义标记以进行精确测量。

## 性能预算

为每个系统设定性能目标：

**目标：60 FPS (每帧 16.67ms)**
- 渲染：6ms
- 脚本：4ms
- 物理：2ms
- UI：1ms
- 音频：0.5ms
- 其他：3ms

使用 Profiler 监控并优化超出预算的系统。

## 平台特定优化

### 移动端优化

**主要关注点：**
- 较低的 CPU/GPU 功率
- 内存限制
- 电池寿命
- 触摸输入开销

**移动端特定优化：**
- 减少 Draw Call（移动端 < 100）
- 降低纹理分辨率
- 禁用阴影或使用简单阴影
- 减少粒子数量
- 使用遮挡剔除 (Occlusion Culling)
- 优化 UI (Canvas batching)

### PC/主机优化

**空间更大但仍需优化：**
- 目标最低 60 FPS
- 允许更高的质量设置
- 监控 VRAM 使用情况
- 在最低配置硬件上进行分析

## 附加资源

### 参考文件

有关详细的性能技术，请参阅：
- **`references/memory-optimization.md`** - 高级 GC 减少、分配模式
- **`references/rendering-optimization.md`** - Draw Call 批处理、GPU 优化、着色器
- **`references/physics-optimization.md`** - 碰撞优化、Rigidbody 最佳实践
- **`references/profiling-guide.md`** - 完整的性能分析工作流、工具、分析

## 快速参考

**缓存优先级：**
1. Transform 引用
2. GetComponent 结果
3. Find 结果
4. Material 实例
5. 协程中的 WaitForSeconds

**避免在 Update 中使用：**
- GetComponent
- Find 方法
- 字符串拼接
- 新分配 (New allocations)
- 物理射线检测 (Raycast)（谨慎使用）

**优化前务必进行性能分析：**
- 测量基准
- 识别瓶颈
- 应用针对性修复
- 测量改进

---

持续应用这些性能实践，以在所有平台上实现流畅、响应迅速的 Unity 游戏。
