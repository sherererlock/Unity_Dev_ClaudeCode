---
name: Unity 基础
description: 当用户询问 "Unity 生命周期", "MonoBehaviour 方法", "Awake vs Start", "Update vs FixedUpdate", "Unity 序列化", "[SerializeField]", "GetComponent", "组件引用", "Unity 预制件", "预制件工作流", 或需要关于 Unity 基础模式和最佳实践的指导时，应使用此技能。
version: 0.1.0
---

# Unity 基础

关于 Unity 核心系统、MonoBehaviour 生命周期、序列化、组件架构和预制件工作流的全面指南。

## 概述

Unity 开发围绕着 MonoBehaviour 组件、组件模式以及特定的回调方法生命周期展开。理解这些基础知识对于编写正确、高性能的 Unity 代码至关重要。本技能涵盖：

- MonoBehaviour 生命周期方法及其正确用法
- 序列化系统和检视面板 (Inspector) 集成
- 基于组件的架构和 GetComponent 模式
- 预制件 (Prefab) 工作流和最佳实践

## MonoBehaviour 生命周期

Unity 按照预定的顺序在 MonoBehaviour 脚本上调用特定方法。使用错误的方法会导致 bug、空引用和性能低下。

### 初始化方法

#### Awake()

在脚本实例加载时调用，在任何 Start() 调用之前。用于**初始化该对象自身的状态**。

```csharp
private void Awake()
{
    // 在此 GameObject 上缓存组件引用
    rigidbody = GetComponent<Rigidbody>();
    animator = GetComponent<Animator>();

    // 初始化私有状态
    currentHealth = maxHealth;
    inventory = new List<Item>();

    // 不要引用其他对象 - 它们可能尚未初始化
}
```

**使用 Awake() 进行：**
- 缓存组件引用
- 初始化私有字段
- 设置内部状态
- 创建单例/管理器

**不要使用 Awake() 进行：**
- 引用其他 GameObject（请改用 Start）
- 执行依赖于其他脚本已准备就绪的操作

#### Start()

在第一次 Update() 之前调用，在所有 Awake() 调用完成后。用于**引用其他对象的设置**。

```csharp
private void Start()
{
    // 可以安全地引用其他 GameObject - 它们已初始化
    playerTransform = GameObject.FindWithTag("Player").transform;
    gameManager = FindObjectOfType<GameManager>();

    // 向管理器或系统注册
    GameManager.Instance.RegisterEnemy(this);

    // 执行依赖于其他组件的初始化
    SetupWeapon(gameManager.GetStartingWeapon());
}
```

**使用 Start() 进行：**
- 查找并引用其他 GameObject
- 调用其他组件上的初始化方法
- 向管理器或系统注册
- 依赖于场景完全初始化的设置

**常用模式：**
```csharp
private Camera mainCamera;  // 在 Awake 中缓存
private Transform target;    // 在 Start 中查找

private void Awake()
{
    mainCamera = Camera.main;  // 缓存昂贵的查找操作
}

private void Start()
{
    target = GameObject.FindWithTag("Target").transform;  // 场景加载后查找
}
```

#### OnEnable() / OnDisable()

当 GameObject 变为激活/非激活状态时调用。用于**订阅/取消订阅事件**。

```csharp
private void OnEnable()
{
    // 订阅事件
    GameEvents.OnPlayerDied += HandlePlayerDeath;
    InputManager.OnJumpPressed += HandleJump;

    // 重新启用功能
    StartCoroutine(SpawnEnemies());
}

private void OnDisable()
{
    // 取消订阅事件（防止内存泄漏！）
    GameEvents.OnPlayerDied -= HandlePlayerDeath;
    InputManager.OnJumpPressed -= HandleJump;

    // 清理临时状态
    StopAllCoroutines();
}
```

**关键：** 务必在 OnDisable() 中取消订阅，以防止内存泄漏。

### 更新方法

#### Update()

每一帧调用。请谨慎使用 - 性能关键。

```csharp
private void Update()
{
    // 输入处理
    if (Input.GetKeyDown(KeyCode.Space))
        Jump();

    // 帧相关逻辑
    UpdateUI();

    // 避免在此处进行昂贵的操作
    // 不要每帧调用 GetComponent
    // 不要使用 Find 方法
}
```

**尽可能避免使用 Update()。** 优先选择事件驱动的方法。

#### FixedUpdate()

以固定的时间步长调用（默认 50 FPS）。**仅用于物理操作**。

```csharp
private void FixedUpdate()
{
    // 物理操作
    rigidbody.AddForce(moveDirection * moveSpeed);

    // 基于物理的移动
    rigidbody.MovePosition(transform.position + velocity * Time.fixedDeltaTime);

    // 不要在此处处理输入（使用 Update）
    // 不要在此处更新 UI（使用 Update 或 LateUpdate）
}
```

**规则：** 如果涉及 Rigidbody 或物理，请使用 FixedUpdate()。其他所有内容使用 Update() 或事件驱动方法。

#### LateUpdate()

在所有 Update() 调用之后调用。用于**摄像机跟随和最终调整**。

```csharp
private void LateUpdate()
{
    // 摄像机跟随（在 Update 中玩家移动之后）
    transform.position = target.position + offset;

    // 最终位置调整
    ClampToBounds();

    // 依赖于世界状态的 UI 更新
    UpdateHealthBar();
}
```

### 销毁方法

#### OnDestroy()

当 GameObject 被销毁时调用。用于**最终清理**。

```csharp
private void OnDestroy()
{
    // 取消订阅静态事件
    GameEvents.OnPlayerDied -= HandlePlayerDeath;

    // 释放资源
    if (texture != null)
        Destroy(texture);

    // 通知其他系统
    GameManager.Instance.UnregisterEnemy(this);
}
```

### 生命周期顺序总结

1. **Awake()** - 初始化自身
2. **OnEnable()** - 订阅事件
3. **Start()** - 引用其他对象，最终设置
4. **FixedUpdate()** - 物理 (50 FPS)
5. **Update()** - 每帧逻辑 (可变 FPS)
6. **LateUpdate()** - 摄像机，最终调整
7. **OnDisable()** - 取消订阅事件
8. **OnDestroy()** - 最终清理

## 序列化系统

Unity 的序列化系统控制在检视面板 (Inspector) 中显示的内容以及数据的持久化方式。

### 序列化字段

使私有字段在 Inspector 中可编辑：

```csharp
[SerializeField] private int maxHealth = 100;
[SerializeField] private float moveSpeed = 5f;
[SerializeField] private GameObject prefab;
```

**好处：**
- 保持封装（私有访问权限）
- Inspector 编辑
- 预制件/场景数据持久化

**最佳实践：** 优先使用 `[SerializeField] private` 而不是 `public` 字段。

### Header 和 Tooltip

组织 Inspector 部分：

```csharp
[Header("Movement Settings")]
[Tooltip("Maximum movement speed in units per second")]
[SerializeField] private float moveSpeed = 5f;

[Tooltip("Rotation speed in degrees per second")]
[SerializeField] private float rotationSpeed = 180f;

[Header("Combat Settings")]
[SerializeField] private int attackDamage = 25;
```

### 序列化规则

**Unity 序列化什么：**
- 公共字段（除非使用 [HideInInspector]）
- 带有 [SerializeField] 的私有字段
- 支持的类型：基元类型、Unity 对象、结构体、数组、List

**Unity 不序列化什么：**
- 属性 (Properties)
- 没有 [SerializeField] 的私有字段
- 字典 (Dictionaries)
- 接口 (Interfaces)
- 静态字段 (Static fields)

### 属性 vs 序列化字段

```csharp
// 不要直接暴露内部状态
public int health;  // 坏：公共字段

// 要使用属性进行受控访问
[SerializeField] private int health = 100;
public int Health
{
    get => health;
    set
    {
        health = Mathf.Clamp(value, 0, maxHealth);
        OnHealthChanged?.Invoke(health);
    }
}
```

## 组件架构

Unity 使用组件模式 - 功能由多个组件组合而成。

### GetComponent 模式

**在 Awake() 中缓存引用**以避免重复查找：

```csharp
// ❌ 坏 - 重复调用 GetComponent
private void Update()
{
    GetComponent<Rigidbody>().velocity = Vector3.forward;  // 昂贵！
}

// ✅ 好 - 缓存一次
private Rigidbody rb;

private void Awake()
{
    rb = GetComponent<Rigidbody>();
}

private void Update()
{
    rb.velocity = Vector3.forward;  // 快速
}
```

### 组件变体

```csharp
// GetComponent<T>() - 在此 GameObject 上
rb = GetComponent<Rigidbody>();

// GetComponentInChildren<T>() - 此 GameObject 及其子对象
animator = GetComponentInChildren<Animator>();

// GetComponentInParent<T>() - 此 GameObject 及其父对象
canvas = GetComponentInParent<Canvas>();

// GetComponents<T>() - 多个组件
Collider[] colliders = GetComponents<Collider>();
```

**性能：** 缓存所有 GetComponent 结果。永远不要在 Update/FixedUpdate 中调用。

### 必需组件 (Required Components)

声明组件依赖关系：

```csharp
[RequireComponent(typeof(Rigidbody))]
[RequireComponent(typeof(Collider))]
public class PlayerMovement : MonoBehaviour
{
    private Rigidbody rb;

    private void Awake()
    {
        rb = GetComponent<Rigidbody>();  // 保证存在
    }
}
```

### 组件通信

**直接引用 (Inspector):**
```csharp
[SerializeField] private HealthDisplay healthDisplay;

private void TakeDamage(int amount)
{
    health -= amount;
    healthDisplay.UpdateHealth(health);  // 直接调用
}
```

**GetComponent (运行时):**
```csharp
private void OnTriggerEnter(Collider other)
{
    var health = other.GetComponent<Health>();
    if (health != null)
        health.TakeDamage(10);
}
```

**事件 (解耦):**
```csharp
public event Action<int> OnHealthChanged;

private void TakeDamage(int amount)
{
    health -= amount;
    OnHealthChanged?.Invoke(health);  // 任何订阅者都会收到通知
}
```

## 预制件 (Prefab) 工作流

预制件是可重用的 GameObject 模板。理解预制件工作流可防止常见问题。

### 创建预制件

将 GameObject 从层级 (Hierarchy) 窗口拖到项目 (Project) 窗口。层级窗口中的蓝色文本表示预制件实例。

### 预制件实例

对预制件实例的更改：
- **Override (覆盖)** - 仅更改此实例（粗体蓝色）
- **Apply (应用)** - 将更改推送到预制件资产（影响所有实例）
- **Revert (还原)** - 丢弃实例更改，匹配预制件

### 预制件变体

创建基础预制件的变体：

```
Base Prefab: Enemy
├── Variant: FastEnemy (速度增加)
├── Variant: TankEnemy (生命值增加)
└── Variant: FlyingEnemy (增加飞行能力)
```

对基础预制件的更改会传播到变体。

### 嵌套预制件

预制件可以包含其他预制件：

```
Car Prefab
├── Wheel Prefab (x4)
├── Engine Prefab
└── Door Prefab (x4)
```

独立编辑嵌套预制件。

### 编程方式使用预制件

```csharp
[SerializeField] private GameObject enemyPrefab;
[SerializeField] private Transform spawnPoint;

private void SpawnEnemy()
{
    // 实例化预制件
    GameObject enemy = Instantiate(enemyPrefab, spawnPoint.position, Quaternion.identity);

    // 配置实例
    enemy.GetComponent<Enemy>().SetTarget(player);

    // 设置父级到容器（可选）
    enemy.transform.SetParent(enemyContainer);
}
```

### 预制件最佳实践

1.  **对任何在运行时生成的物体使用预制件**（敌人、抛射物、UI 面板）
2.  **创建预制件变体**而不是复制预制件
3.  **小心应用更改** - 会影响所有实例
4.  **将预制件保存在有组织的文件夹中** - Prefabs/Characters, Prefabs/UI 等
5.  **在应用到所有实例之前测试预制件更改**

## 常用模式

### 单例管理器

```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
        DontDestroyOnLoad(gameObject);
    }
}
```

### 对象初始化

```csharp
public class Enemy : MonoBehaviour
{
    [SerializeField] private int health = 100;

    private Rigidbody rb;
    private Transform target;

    // 1. 在自身缓存组件
    private void Awake()
    {
        rb = GetComponent<Rigidbody>();
    }

    // 2. 查找外部引用
    private void Start()
    {
        target = GameObject.FindWithTag("Player").transform;
    }

    // 3. 订阅事件
    private void OnEnable()
    {
        GameEvents.OnWaveComplete += HandleWaveComplete;
    }

    // 4. 取消订阅事件
    private void OnDisable()
    {
        GameEvents.OnWaveComplete -= HandleWaveComplete;
    }
}
```

## 额外资源

### 参考文件

有关特定主题的详细指导：

- **`references/lifecycle-detailed.md`** - 完整的生命周期方法参考
- **`references/serialization-guide.md`** - 高级序列化模式
- **`references/component-patterns.md`** - 组件架构最佳实践
- **`references/prefab-workflows.md`** - 全面的预制件使用指南

### 快速参考

**生命周期顺序：** Awake → OnEnable → Start → FixedUpdate → Update → LateUpdate → OnDisable → OnDestroy

**序列化：** `[SerializeField] private` 优于 `public`

**组件：** 在 Awake() 中缓存，永远不要在 Update() 中调用 GetComponent

**预制件：** 用于可重用对象，在应用更改前进行测试

---

遵循这些基础知识来构建稳固、高性能、可维护且无错误的 Unity 项目。
