# Unity MonoBehaviour 生命周期 - 完整参考

MonoBehaviour 所有生命周期方法、执行顺序和使用模式的完整文档。

## 完整生命周期顺序

### 初始化阶段 (Initialization Phase)

1. **Awake()**
   - 在脚本实例加载时调用
   - 在场景加载之前
   - 每个组件生命周期仅调用一次
   - 同一个 GameObject 内的执行顺序：未定义
   - **用于：** 自我初始化、组件缓存

2. **OnEnable()**
   - 在 Awake() 之后以及每当 GameObject 激活时调用
   - 可以被调用多次
   - **用于：** 事件订阅、启用功能

3. **Start()**
   - 在首次 Update() 之前，所有 Awake() 调用之后调用
   - 每个组件生命周期仅调用一次（如果已启用）
   - 仅在组件启用时调用
   - **用于：** 查找其他对象、最终设置

### 物理阶段 (Physics Phase)

4. **FixedUpdate()**
   - 以固定的时间步长调用（默认：0.02秒 = 50 FPS）
   - 独立于帧率
   - **用于：** 所有物理操作（Rigidbody、力）

### 更新阶段 (Update Phase)

5. **Update()**
   - 每帧调用一次
   - 基于帧率的可变时间
   - **用于：** 输入、非物理移动、帧更新

6. **LateUpdate()**
   - 在所有 Update() 方法之后调用
   - **用于：** 相机跟随、最终位置调整

### 渲染阶段 (Rendering Phase)

7. **OnPreCull()** - 在相机剔除之前
8. **OnBecameVisible()** - 当渲染器对任何相机可见时
9. **OnBecameInvisible()** - 当渲染器不再可见时
10. **OnWillRenderObject()** - 为每个渲染该对象的相机调用
11. **OnPreRender()** - 在相机渲染场景之前
12. **OnRenderObject()** - 在所有常规渲染之后
13. **OnPostRender()** - 在相机完成渲染之后
14. **OnRenderImage()** - 图像后处理

### GUI 阶段 (GUI Phase)

15. **OnGUI()** - 每帧调用多次用于 GUI 事件
    - 传统 GUI 系统（在新项目中避免使用）

### Gizmo 阶段 (Gizmo Phase)

16. **OnDrawGizmos()** - 在 Scene 视图中绘制 Gizmos
17. **OnDrawGizmosSelected()** - 当 GameObject 被选中时绘制 Gizmos

### 停用阶段 (Deactivation Phase)

18. **OnDisable()**
    - 当 GameObject 停用时
    - 在 OnDestroy() 之前
    - 可以被调用多次
    - **用于：** 取消事件订阅、清理

19. **OnDestroy()**
    - 当 GameObject 被销毁时
    - 每个组件生命周期仅调用一次
    - **用于：** 最终清理、资源释放

### 应用程序生命周期 (Application Lifecycle)

20. **OnApplicationPause()** - 当应用程序暂停/恢复时
21. **OnApplicationFocus()** - 当应用程序失去/获得焦点时
22. **OnApplicationQuit()** - 在应用程序退出之前

## 方法详情

### Awake()

```csharp
private void Awake()
{
    // 自我初始化
    health = maxHealth;
    inventory = new List<Item>();

    // 组件缓存（仅限此 GameObject）
    rb = GetComponent<Rigidbody>();
    animator = GetComponent<Animator>();

    // 单例设置
    if (Instance == null)
        Instance = this;
    else
        Destroy(gameObject);
}
```

**执行：**
- 总是被调用，即使组件被禁用
- 同一个 GameObject 上的多个脚本调用顺序是随机的
- 发生在场景完全加载之前

**常见用途：**
- 初始化字段
- 缓存 GetComponent 结果
- 设置单例
- 创建对象池

**避免：**
- 引用其他 GameObject（可能还不存在）
- 调用其他脚本的方法（可能尚未初始化）

### Start()

```csharp
private void Start()
{
    // 安全引用其他对象
    player = GameObject.FindWithTag("Player");
    manager = FindObjectOfType<GameManager>();

    // 调用其他组件的方法
    weapon.Initialize(this);
    ui.SetPlayer(this);

    // 向系统注册
    GameManager.Instance.RegisterEnemy(this);
}
```

**执行：**
- 在首次 Update() 之前调用
- 在所有 Awake() 调用完成之后
- 仅当组件启用时
- 每个生命周期调用一次

**常见用途：**
- 查找其他 GameObject
- 引用其他脚本
- 最终初始化
- 向管理器注册

### OnEnable() / OnDisable()

```csharp
private void OnEnable()
{
    // 订阅事件
    GameEvents.OnLevelComplete += HandleLevelComplete;
    InputActions.Jump.performed += OnJump;

    // 启用系统
    StartCoroutine(SpawnEnemies());
    particleSystem.Play();
}

private void OnDisable()
{
    // 取消订阅事件（至关重要！）
    GameEvents.OnLevelComplete -= HandleLevelComplete;
    InputActions.Jump.performed -= OnJump;

    // 禁用系统
    StopAllCoroutines();
    particleSystem.Stop();
}
```

**执行：**
- OnEnable: 在 Awake() 之后，每当 GameObject 激活时
- OnDisable: 当 GameObject 停用时，在 OnDestroy() 之前
- 可以被调用多次

**常见用途：**
- 事件订阅/取消订阅
- 启用/禁用协程
- 激活/停用系统
- 对象池激活

**关键模式：**
```csharp
// 始终成对订阅/取消订阅
private void OnEnable() => Event += Handler;
private void OnDisable() => Event -= Handler;
```

### Update() vs FixedUpdate()

#### Update()

```csharp
private void Update()
{
    // 输入处理（依赖帧）
    if (Input.GetKeyDown(KeyCode.Space))
        Jump();

    // 非物理移动
    transform.position += velocity * Time.deltaTime;

    // 基于帧的逻辑
    UpdateAnimation();
    CheckForEnemies();
}
```

**时机：** 可变，取决于帧率（60 FPS = 16.67ms，30 FPS = 33.33ms）

**用于：**
- 输入处理
- 非物理移动
- 动画更新
- UI 更新
- 依赖帧的逻辑

**避免：**
- 物理操作（使用 FixedUpdate）
- 空的 Update() 方法（移除它们）

#### FixedUpdate()

```csharp
private void FixedUpdate()
{
    // 物理操作
    rb.AddForce(input * moveSpeed);

    // 基于物理的移动
    rb.MovePosition(rb.position + velocity * Time.fixedDeltaTime);

    // 应用扭矩/旋转
    rb.AddTorque(torque);
}
```

**时机：** 固定间隔，默认 0.02秒 (50 FPS)，独立于帧率

**用于：**
- Rigidbody 操作 (AddForce, velocity, MovePosition)
- 物理计算
- 依赖碰撞的逻辑

**避免：**
- 输入处理（可能会错过固定更新之间的输入）
- 渲染/动画（可能会卡顿）

**时间常量：**
- `Time.deltaTime` 在 Update() 中 - 自上一帧以来的时间
- `Time.fixedDeltaTime` 在 FixedUpdate() 中 - 固定物理时间步长

### LateUpdate()

```csharp
private void LateUpdate()
{
    // 相机跟随（在玩家移动之后）
    Vector3 targetPosition = player.position + offset;
    transform.position = Vector3.Lerp(transform.position, targetPosition, followSpeed * Time.deltaTime);

    // 公告板效果（面向相机）
    transform.LookAt(Camera.main.transform);

    // 最终位置钳制
    transform.position = ClampToBounds(transform.position);
}
```

**时机：** 在所有 Update() 调用完成之后

**用于：**
- 相机跟随（在对象移动后追踪）
- IK（反向运动学）
- 最终位置调整
- 公告板效果
- 程序化动画

### OnDestroy()

```csharp
private void OnDestroy()
{
    // 取消订阅静态事件
    GameEvents.OnLevelComplete -= HandleLevelComplete;

    // 释放资源
    if (customTexture != null)
        Destroy(customTexture);

    // 通知系统
    GameManager.Instance?.UnregisterEnemy(this);

    // 保存数据
    SaveProgress();
}
```

**执行：**
- 当 GameObject 被销毁时
- 在场景卸载时
- 在应用程序退出时
- 每个生命周期一次

**用于：**
- 最终清理
- 取消订阅静态事件
- 资源释放
- 保存数据

## 执行顺序配置

### 脚本执行顺序

在 **Edit > Project Settings > Script Execution Order** 中配置：

```
-1000: GameManager (最先初始化)
     0: Default scripts (默认脚本)
  1000: CameraController (最后执行)
```

**何时使用：**
- 管理器必须在其他脚本之前初始化
- 相机必须在所有移动之后更新
- 需要特定的初始化顺序

**避免：**
- 过度使用（会产生隐藏的依赖关系）
- 负值/正值表示顺序的反模式

### 最佳实践

与其依赖执行顺序，不如：

```csharp
// ❌ 坏 - 依赖执行顺序
void Start()
{
    manager.DoSomething();  // 仅当 manager.Start() 已经运行时才有效
}

// ✅ 好 - 显式初始化
void Start()
{
    manager = GameManager.Instance;  // 单例在 Awake 中初始化
    manager.DoSomething();  // 安全
}
```

## 常见模式

### 初始化模式

```csharp
public class Character : MonoBehaviour
{
    [SerializeField] private int maxHealth = 100;

    private int health;
    private Rigidbody rb;
    private Animator animator;
    private Transform target;

    // 1. 在自身上缓存组件
    private void Awake()
    {
        health = maxHealth;
        rb = GetComponent<Rigidbody>();
        animator = GetComponent<Animator>();
    }

    // 2. 查找外部引用
    private void Start()
    {
        target = GameObject.FindWithTag("Player")?.transform;
        GameManager.Instance.RegisterCharacter(this);
    }

    // 3. 订阅事件
    private void OnEnable()
    {
        GameEvents.OnGameOver += HandleGameOver;
    }

    // 4. 取消订阅事件
    private void OnDisable()
    {
        GameEvents.OnGameOver -= HandleGameOver;
    }

    // 5. 最终清理
    private void OnDestroy()
    {
        GameManager.Instance?.UnregisterCharacter(this);
    }
}
```

### 协程模式

```csharp
private void OnEnable()
{
    StartCoroutine(SpawnEnemies());
}

private void OnDisable()
{
    StopAllCoroutines();  // 禁用时清理
}

private IEnumerator SpawnEnemies()
{
    while (true)
    {
        SpawnEnemy();
        yield return new WaitForSeconds(spawnDelay);
    }
}
```

### 事件订阅模式

```csharp
// 正确 - 成对订阅/取消订阅
private void OnEnable()
{
    PlayerEvents.OnDeath += HandlePlayerDeath;
    InputManager.OnJump += HandleJump;
}

private void OnDisable()
{
    PlayerEvents.OnDeath -= HandlePlayerDeath;
    InputManager.OnJump -= HandleJump;
}

// 错误 - 在 Start 中订阅，忘记取消订阅
private void Start()
{
    PlayerEvents.OnDeath += HandlePlayerDeath;  // 内存泄漏！
}
```

## 性能考量

### 空方法

```csharp
// ❌ 坏 - 空方法仍然有开销
private void Update() { }
private void FixedUpdate() { }

// ✅ 好 - 移除未使用的方法
// (如果不需要，则没有 Update 或 FixedUpdate)
```

**影响：** Unity 会调用所有生命周期方法，即使是空的。移除未使用的。

### 频繁调用

每帧调用的方法：
- Update() - 每帧 (~60 FPS = 3600 次/分钟)
- FixedUpdate() - 每个物理步长 (~50 FPS = 3000 次/分钟)
- LateUpdate() - 每帧

**优化：** 尽量减少这些方法中的工作量。尽可能使用事件、协程或 Invoke。

### GC 分配 (GC Allocation)

```csharp
// ❌ 坏 - 每帧分配
private void Update()
{
    string message = "Health: " + health;  // 字符串分配
    GetComponent<Renderer>().material.color = Color.red;  // 材质分配
}

// ✅ 好 - 缓存并避免分配
private Renderer rend;
private readonly StringBuilder messageBuilder = new StringBuilder();

private void Awake()
{
    rend = GetComponent<Renderer>();
}

private void Update()
{
    messageBuilder.Clear();
    messageBuilder.Append("Health: ").Append(health);  // 无分配
}
```

## 故障排除

### Start() 未被调用

**症状：** Start() 方法从未执行

**原因：**
1. 组件在检视面板 (Inspector) 中被禁用
2. GameObject 处于非活动状态
3. 组件在 Awake() 中被禁用

**解决方案：** 启用组件/GameObject

### Start() 中的 NullReferenceException

**症状：** Start() 中的变量为 null

**原因：**
1. Awake() 尚未运行（不应该发生）
2. 引用的对象不存在
3. Find() 方法返回 null

**解决方案：**
```csharp
private void Start()
{
    target = GameObject.FindWithTag("Player")?.transform;
    if (target == null)
        Debug.LogError("Player not found!");
}
```

### OnDisable() 之后事件被调用

**症状：** 即使 GameObject 被销毁后，事件处理程序仍被调用

**原因：** 忘记取消订阅

**解决方案：** 始终在 OnDisable() 或 OnDestroy() 中取消订阅

```csharp
private void OnDestroy()
{
    // 取消订阅静态事件
    GameEvents.OnLevelComplete -= HandleLevelComplete;
}
```

## 总结

**初始化顺序：**
1. Awake() - 在自身上缓存组件
2. OnEnable() - 订阅事件
3. Start() - 查找其他对象，最终设置

**更新循环：**
1. FixedUpdate() - 物理
2. Update() - 输入，逻辑
3. LateUpdate() - 相机，最终调整

**清理：**
1. OnDisable() - 取消订阅事件
2. OnDestroy() - 最终清理

**关键规则：**
- 在 Awake() 中缓存，在 Start() 中查找
- 物理在 FixedUpdate() 中，输入在 Update() 中
- 在 OnEnable() 中订阅，在 OnDisable() 中取消订阅
- 移除空的生命周期方法
- 尽可能使用事件代替 Update()

遵循此生命周期模式以获得可靠、高性能的 Unity 代码。
