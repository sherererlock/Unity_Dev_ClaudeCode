# Unity 物理优化 - 完整指南

优化 Unity 物理系统以获得更好性能的完整指南：涵盖碰撞优化、刚体 (Rigidbody) 最佳实践、射线检测 (Raycasting) 和物理设置。

## 物理更新基础 (Physics Update Fundamentals)

### FixedUpdate 计时 (FixedUpdate Timing)

物理更新在固定的时间步长下运行（默认：0.02秒 = 50 Hz）：

```csharp
// 配置时间步长
// Edit > Project Settings > Time > Fixed Timestep
Time.fixedDeltaTime = 0.02f;  // 50 FPS 的物理更新
```

**性能影响：**
- 较低的时间步长 (0.01 = 100 Hz)：更精确，CPU 开销更大
- 较高的时间步长 (0.04 = 25 Hz)：精度较低，CPU 开销较小

**建议：**
- **桌面端 (Desktop)**：0.02秒 (50 Hz)
- **移动端 (Mobile)**：0.025-0.033秒 (30-40 Hz)
- **VR**：0.0111秒 (90 Hz) 以匹配显示刷新率

### FixedUpdate 开销 (FixedUpdate Cost)

FixedUpdate 的运行独立于帧率：

**60 FPS 的游戏，50 Hz 的物理更新：**
- 某些帧：执行 0 次 FixedUpdate
- 某些帧：执行 1 次 FixedUpdate
- 某些帧：执行 2 次 FixedUpdate（追赶进度）

**性能分析路径：**
```
Profiler > CPU > FixedUpdate.PhysicsFixedUpdate
```

如果开销很高，说明需要进行物理优化。

## 基于层级的碰撞 (Layer-Based Collision)

这是最有效的物理优化手段。

### 物理层级碰撞矩阵 (Physics Layer Collision Matrix)

**配置路径：Edit > Project Settings > Physics > Layer Collision Matrix**

**设置示例：**
```
Layer 8:  Player (玩家)
Layer 9:  Enemies (敌人)
Layer 10: Projectiles (投射物)
Layer 11: Environment (环境)
Layer 12: Triggers (触发器)
Layer 13: Ragdolls (布娃娃)
```

**禁用不必要的碰撞：**
- Player vs Player ❌
- Enemies vs Enemies ❌
- Projectiles vs Projectiles ❌
- Triggers vs Triggers ❌
- Ragdolls vs Ragdolls ❌ (除非有特殊需求)

**仅启用需要的碰撞：**
- Player vs Environment ✅
- Player vs Enemies ✅
- Projectiles vs Enemies ✅
- Projectiles vs Environment ✅

**性能提升**：减少 30-70% 的碰撞检测计算。

### 设置层级 (Setting Layers)

```csharp
// 在代码中设置层级
gameObject.layer = LayerMask.NameToLayer("Player");

// 递归设置层级
private void SetLayerRecursive(GameObject obj, int layer)
{
    obj.layer = layer;
    foreach (Transform child in obj.transform)
    {
        SetLayerRecursive(child.gameObject, layer);
    }
}

// 使用方法
SetLayerRecursive(character, LayerMask.NameToLayer("Player"));
```

### 触发器层级 (Trigger Layers)

将触发器碰撞体分离到专用层级：

```csharp
// 触发器层级设置
Layer 12: Triggers

// 在碰撞矩阵中禁用 Triggers vs Triggers
// 启用 Triggers vs Player, Enemies 等
```

**好处：**
- 明确控制触发器的交互对象
- 防止意外的触发器与触发器之间的碰撞

## 刚体优化 (Rigidbody Optimization)

### 刚体休眠 (Rigidbody Sleep)

当速度低于阈值时，刚体会进入休眠状态：

**配置路径：Edit > Project Settings > Physics**
- **Sleep Threshold**：速度低于此值时刚体休眠（默认：0.005）
- **Default Contact Offset**：表面碰撞容差（默认：0.01）

**休眠行为：**
```csharp
// 检查是否在休眠
if (rb.IsSleeping())
{
    // 刚体未被模拟 - 节省性能
}

// 强制休眠
rb.Sleep();

// 唤醒
rb.WakeUp();

// 防止休眠
rb.sleepThreshold = 0;  // 永不休眠
```

**优化点：**
- 休眠的刚体不消耗物理 CPU 资源
- 让静止的物体进入休眠
- 仅在需要时唤醒

**手动唤醒示例：**
```csharp
public class Projectile : MonoBehaviour
{
    private Rigidbody rb;

    private void Awake()
    {
        rb = GetComponent<Rigidbody>();
    }

    public void Launch(Vector3 force)
    {
        rb.WakeUp();  // 施加力之前确保唤醒
        rb.AddForce(force, ForceMode.Impulse);
    }
}
```

### 刚体约束 (Rigidbody Constraints)

冻结不必要的轴：

```csharp
// 冻结 XZ 轴的旋转（适用于角色控制器）
rb.constraints = RigidbodyConstraints.FreezeRotationX | RigidbodyConstraints.FreezeRotationZ;

// 冻结 Y 轴位置（适用于俯视游戏）
rb.constraints = RigidbodyConstraints.FreezePositionY;

// 冻结所有旋转（防止倾倒）
rb.constraints = RigidbodyConstraints.FreezeRotation;
```

**性能**：约束减少了自由度，从而简化了物理计算。

### 连续与离散碰撞 (Continuous vs Discrete Collision)

**碰撞检测模式：**

**Discrete (离散)**（默认，最快）：
- 检查当前位置的碰撞
- 快速移动的物体可能会穿过薄的障碍物（穿隧效应）
- 最适合移动缓慢或体积大的物体

**Continuous (连续)**（中等开销）：
- 使用扫描式 (swept) 碰撞检测
- 防止快速投射物发生穿隧
- 最适合玩家或重要物体

**Continuous Dynamic (连续动态)**（昂贵）：
- 针对静态和动态物体都进行连续检测
- 精度最高，开销最大
- 谨慎使用（仅用于关键的快速物体）

**Continuous Speculative (连续推测)**（良好的平衡）：
- Unity 2020.2+，推荐用于大多数情况
- 良好的防穿隧效果，开销低于 Continuous
- 适用于触发器

```csharp
// 设置碰撞检测模式
rb.collisionDetectionMode = CollisionDetectionMode.ContinuousSpeculative;
```

**建议：**
- 静态/缓慢物体：Discrete
- 玩家/投射物：Continuous Speculative
- 关键的快速物体：Continuous Dynamic（谨慎使用）

### 插值 (Interpolation)

平滑刚体在 FixedUpdate 调用之间的移动：

```csharp
// None (默认) - 无平滑，可能显得卡顿
rb.interpolation = RigidbodyInterpolation.None;

// Interpolate - 基于上一帧进行平滑
rb.interpolation = RigidbodyInterpolation.Interpolate;

// Extrapolate - 预测下一帧（有更高的延迟风险）
rb.interpolation = RigidbodyInterpolation.Extrapolate;
```

**使用 Interpolate 的情况：**
- 玩家角色
- 摄像机跟随的物体
- 可见的物理物体

**使用 None 的情况：**
- 特效/粒子
- 远离摄像机的物体
- 性能至关重要时

**开销**：CPU 开销极小，主要带来视觉平滑的收益。

## 碰撞体优化 (Collider Optimization)

### 碰撞体复杂度 (Collider Complexity)

碰撞体类型按性能排名（从快到慢）：

1.  **Sphere Collider (球体碰撞体)** - 最快
2.  **Capsule Collider (胶囊体碰撞体)** - 非常快
3.  **Box Collider (盒体碰撞体)** - 快
4.  **Mesh Collider (convex) (网格碰撞体-凸面)** - 中等
5.  **Mesh Collider (non-convex) (网格碰撞体-非凸面)** - 慢

**建议：**
- 尽可能使用原始碰撞体（球体、胶囊体、盒体）
- 组合多个原始碰撞体代替网格碰撞体
- 仅在必要时使用凸面网格碰撞体
- 避免使用非凸面网格碰撞体（仅限静态物体）

### 复合碰撞体 (Compound Colliders)

使用多个简单的碰撞体代替复杂的网格碰撞体：

```csharp
// ❌ 坏 - 复杂的网格碰撞体（慢）
MeshCollider complexCollider;

// ✅ 好 - 复合原始碰撞体（快）
Character (GameObject)
├── Body (Capsule Collider)
├── Head (Sphere Collider)
└── Weapon (Box Collider)
```

**好处：**
- 更快的碰撞检测
- 在物体数量多时性能更好
- 对碰撞层级有更多控制权

### 网格碰撞体最佳实践 (Mesh Collider Best Practices)

**凸面网格碰撞体 (Convex mesh colliders)：**
```csharp
meshCollider.convex = true;
```

**动态物体的要求：**
- Rigidbody + Mesh Collider 必须是凸面 (convex) 的
- 非凸面碰撞体仅适用于静态物体

**优化点：**
- 降低网格复杂度（减面）
- 使用凸包近似 (convex hull approximation)
- 优先使用复合原始碰撞体

**非凸面碰撞体：**
- 仅用于静态环境
- 无 Rigidbody
- 高细节地形/建筑物

### 触发器碰撞体 (Trigger Colliders)

使用触发器进行检测，而不产生物理响应：

```csharp
collider.isTrigger = true;
```

**好处：**
- 比完全碰撞更便宜
- 不施加物理力
- 适用于检测区域（拾取物、伤害区域、触发器）

**触发器优化：**
- 专用触发器层级
- 简单的碰撞体形状（球体、盒体）
- 不需要时禁用

```csharp
public class PickupTrigger : MonoBehaviour
{
    private SphereCollider triggerCollider;

    private void Awake()
    {
        triggerCollider = GetComponent<SphereCollider>();
        triggerCollider.isTrigger = true;
        gameObject.layer = LayerMask.NameToLayer("Triggers");
    }

    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            Collect();
            triggerCollider.enabled = false;  // 收集后禁用
        }
    }
}
```

## 射线检测优化 (Raycasting Optimization)

### 射线检测基础 (Raycasting Basics)

射线检测有开销 - 需优化使用：

```csharp
// 基础射线检测（检查所有层级）
bool hit = Physics.Raycast(origin, direction, out RaycastHit hitInfo);

// 优化后的射线检测（层级遮罩，最大距离）
int layerMask = 1 << LayerMask.NameToLayer("Enemy");
bool hit = Physics.Raycast(origin, direction, out RaycastHit hitInfo, maxDistance, layerMask);
```

**性能因素：**
1.  **Layer mask (层级遮罩)**：限制检查的对象
2.  **Max distance (最大距离)**：提前终止射线
3.  **Frequency (频率)**：每帧更少的射线检测
4.  **Query trigger interaction (查询触发器交互)**：如果不需要则跳过触发器

### 层级遮罩优化 (Layer Mask Optimization)

```csharp
// ❌ 坏 - 无层级遮罩（检查所有物体）
private void Update()
{
    Physics.Raycast(origin, direction, out RaycastHit hit);
}

// ✅ 好 - 缓存层级遮罩，限制距离
private int targetLayerMask;
private const float maxRayDistance = 100f;

private void Awake()
{
    targetLayerMask = LayerMask.GetMask("Enemy", "Environment");
}

private void Update()
{
    Physics.Raycast(origin, direction, out RaycastHit hit, maxRayDistance, targetLayerMask);
}
```

**层级遮罩创建：**
```csharp
// 单个层级
int layerMask = 1 << LayerMask.NameToLayer("Enemy");

// 多个层级
int layerMask = LayerMask.GetMask("Enemy", "Environment");

// 反向选择（排除特定层级）
int layerMask = ~(1 << LayerMask.NameToLayer("Ignore"));
```

### RaycastNonAlloc (无分配射线检测)

使用 NonAlloc 变体避免内存分配：

```csharp
// ❌ 坏 - RaycastAll 分配数组
private void Update()
{
    RaycastHit[] hits = Physics.RaycastAll(origin, direction);  // GC 分配！

    foreach (var hit in hits)
    {
        ProcessHit(hit);
    }
}

// ✅ 好 - RaycastNonAlloc 重用数组
private RaycastHit[] hits = new RaycastHit[10];

private void Update()
{
    int hitCount = Physics.RaycastNonAlloc(origin, direction, hits, maxDistance, layerMask);

    for (int i = 0; i < hitCount; i++)
    {
        ProcessHit(hits[i]);
    }
}
```

**NonAlloc 变体：**
- `Physics.RaycastNonAlloc()`
- `Physics.SphereCastNonAlloc()`
- `Physics.BoxCastNonAlloc()`
- `Physics.CapsuleCastNonAlloc()`
- `Physics.OverlapSphereNonAlloc()`

### 射线检测频率 (Raycast Frequency)

降低射线检测频率：

```csharp
// ❌ 坏 - 每帧都进行射线检测
private void Update()
{
    Physics.Raycast(origin, direction, out RaycastHit hit);
}

// ✅ 好 - 每 N 帧检测一次
private int frameCounter = 0;
private const int raycastInterval = 5;

private void Update()
{
    frameCounter++;
    if (frameCounter >= raycastInterval)
    {
        frameCounter = 0;
        Physics.Raycast(origin, direction, out RaycastHit hit);
    }
}

// ✅ 更好 - 使用带间隔的协程
private void Start()
{
    StartCoroutine(RaycastRoutine());
}

private IEnumerator RaycastRoutine()
{
    WaitForSeconds wait = new WaitForSeconds(0.1f);

    while (true)
    {
        Physics.Raycast(origin, direction, out RaycastHit hit);
        yield return wait;
    }
}
```

### 查询触发器交互 (Query Trigger Interaction)

控制射线检测是否命中触发器：

```csharp
// 忽略触发器（更快）
Physics.Raycast(origin, direction, out RaycastHit hit, maxDistance, layerMask, QueryTriggerInteraction.Ignore);

// 仅与触发器碰撞
Physics.Raycast(origin, direction, out RaycastHit hit, maxDistance, layerMask, QueryTriggerInteraction.Collide);

// 使用全局设置（默认）
Physics.Raycast(origin, direction, out RaycastHit hit, maxDistance, layerMask, QueryTriggerInteraction.UseGlobal);
```

**全局设置：**
```
Edit > Project Settings > Physics > Queries Hit Triggers
```

## 物理查询优化 (Physics Queries Optimization)

### OverlapSphere 优化 (OverlapSphere Optimization)

```csharp
// ❌ 坏 - OverlapSphere 分配内存
private void Update()
{
    Collider[] colliders = Physics.OverlapSphere(position, radius);  // GC 分配
}

// ✅ 好 - OverlapSphereNonAlloc
private Collider[] colliders = new Collider[20];

private void Update()
{
    int count = Physics.OverlapSphereNonAlloc(position, radius, colliders, layerMask);

    for (int i = 0; i < count; i++)
    {
        ProcessCollider(colliders[i]);
    }
}
```

### 触发区域与轮询 (Trigger Zones vs Polling)

**轮询方法（CPU 密集）：**
```csharp
// ❌ 坏 - 每帧检查所有敌人的距离
private void Update()
{
    foreach (var enemy in enemies)
    {
        if (Vector3.Distance(transform.position, enemy.position) < detectRange)
        {
            OnEnemyDetected(enemy);
        }
    }
}
```

**触发器方法（事件驱动，高效）：**
```csharp
// ✅ 好 - 使用触发器碰撞体
private SphereCollider detectionTrigger;

private void Awake()
{
    detectionTrigger = gameObject.AddComponent<SphereCollider>();
    detectionTrigger.isTrigger = true;
    detectionTrigger.radius = detectRange;
}

private void OnTriggerEnter(Collider other)
{
    if (other.CompareTag("Enemy"))
    {
        OnEnemyDetected(other.GetComponent<Enemy>());
    }
}
```

**好处：**
- 事件驱动（仅在需要时触发）
- 物理引擎处理检测
- 在物体数量多时扩展性更好

## 角色控制器与刚体 (Character Controller vs Rigidbody)

### 角色控制器 (Character Controller)

用于角色的自定义物理：

```csharp
private CharacterController controller;

private void Awake()
{
    controller = GetComponent<CharacterController>();
}

private void Update()
{
    Vector3 move = transform.forward * speed * Time.deltaTime;
    controller.Move(move);
}
```

**优点：**
- 比 Rigidbody 轻量
- 自定义碰撞响应
- 内置坡度处理
- 不需要连续施加力

**缺点：**
- 无自动物理（重力、力）
- 需要手动处理碰撞
- 不受其他 Rigidbody 影响

**适用场景：**
- 玩家角色（FPS，第三人称）
- 具有自定义移动的 NPC
- 不需要完整物理模拟时

### 刚体角色 (Rigidbody Character)

完整的物理模拟：

```csharp
private Rigidbody rb;

private void Awake()
{
    rb = GetComponent<Rigidbody>();
    rb.constraints = RigidbodyConstraints.FreezeRotation;
}

private void FixedUpdate()
{
    rb.MovePosition(rb.position + velocity * Time.fixedDeltaTime);
}
```

**优点：**
- 完整的物理集成
- 受力、碰撞影响
- 逼真的交互

**缺点：**
- 性能开销较大
- 难以精确控制
- 可能被物体推动

**适用场景：**
- 基于物理的游戏玩法（布娃娃、载具）
- 需要逼真物理的物体
- 需要与 Rigidbody 交互时

## 物理设置优化 (Physics Settings Optimization)

### 项目设置 (Project Settings)

**Edit > Project Settings > Physics**

**关键设置：**

**Gravity (重力)：**
```csharp
Physics.gravity = new Vector3(0, -9.81f, 0);  // 默认值
```
根据游戏手感调整（低重力 = 漂浮感，高重力 = 沉重感）。

**Default Material (默认材质)：**
- Friction (摩擦力)：0.4 (默认)
- Bounciness (弹性)：0

**Sleep Threshold (休眠阈值)**：0.005 (速度低于此值物体休眠)

**Default Contact Offset (默认接触偏移)**：0.01 (碰撞皮肤宽度)

**Queries Hit Backfaces (查询击中背面)**：false (优化射线检测)

**Queries Hit Triggers (查询击中触发器)**：true (在查询中包含触发器 - 如果不需要则禁用)

**Layer Collision Matrix (层级碰撞矩阵)**：配置逐层碰撞（最重要的设置）

### 自动模拟 (Auto Simulation)

禁用自动模拟以进行手动控制：

```csharp
// 禁用自动模拟
Physics.autoSimulation = false;

// 手动模拟
private void FixedUpdate()
{
    Physics.Simulate(Time.fixedDeltaTime);
}
```

**用例：**
- 自定义物理更新时机
- 独立暂停物理
- 高级控制流

### 解算器迭代次数 (Solver Iterations)

**Edit > Project Settings > Physics**

**Default Solver Iterations (默认解算器迭代次数)**：6 (位置)
**Default Solver Velocity Iterations (默认解算器速度迭代次数)**：1

**较高值：**
- 更精确的物理
- 更高的 CPU 开销
- 适合复杂的关节/布娃娃

**较低值：**
- 更快的物理
- 精度较低
- 适合简单的碰撞

**建议：**
- 大多数游戏使用默认值 (6, 1)
- 布娃娃/载具增加值 (8-10, 2-4)
- 移动端减少值 (4, 1)

## 物理性能分析 (Physics Profiling)

### 物理分析器 (Physics Profiler)

**Profiler Window > Physics**

**关键指标：**
- **Physics.Processing**：总物理时间
- **Physics.Contacts**：碰撞检测
- **Physics.Solver**：约束解算
- **Physics.Callbacks**：OnCollision/OnTrigger 回调

**目标：**
- 60 FPS (桌面端) < 2ms
- 30 FPS (移动端) < 4ms

### 物理调试器 (Physics Debugger)

**Window > Analysis > Physics Debugger**

**功能：**
- 可视化碰撞体
- 显示活动接触点
- 查看 Rigidbody 状态（休眠、唤醒）
- 识别碰撞对

**用法：**
1. 启用 Physics Debugger
2. 运行游戏
3. 查看活动碰撞
4. 识别意外的碰撞对
5. 优化层级碰撞矩阵

## 最佳实践总结 (Best Practices Summary)

✅ **DO (推荐)：**
- 配置层级碰撞矩阵 (Layer Collision Matrix)（最大的优化点）
- 尽可能使用原始碰撞体（球体、胶囊体、盒体）
- 让刚体在静止时休眠
- 缓存射线检测的层级遮罩
- 使用 NonAlloc 物理查询
- 降低射线检测频率（不要每帧都检测）
- 使用触发器碰撞体作为检测区域
- 冻结不必要的刚体约束
- 对快速物体使用 Continuous Speculative 碰撞检测
- 定期使用 Physics Profiler 进行分析

❌ **DON'T (不推荐)：**
- 启用所有层级碰撞（巨大的开销）
- 在动态物体上使用非凸面网格碰撞体
- 在没有层级遮罩的情况下每帧进行射线检测
- 使用 RaycastAll（会产生分配 - 应使用 NonAlloc）
- 将休眠阈值设为 0（阻止休眠）
- 不必要地使用高解算器迭代次数
- 忘记设置物体层级
- 在可以使用原始碰撞体时使用网格碰撞体
- 轮询附近的物体（应使用触发器代替）

**黄金法则**：首先配置层级碰撞矩阵。仅此一项优化就可以减少 50% 以上的物理开销。使用 Physics Profiler 分析并识别瓶颈。

遵循这些物理优化技巧，以在 Unity 游戏中实现响应迅速且高性能的物理效果。
