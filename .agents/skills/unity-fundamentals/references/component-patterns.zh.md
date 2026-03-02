# Unity 组件架构 - 最佳实践

Unity 中基于组件的架构、GetComponent 模式、通信策略和组合技术的完整指南。

## 基于组件的设计原则 (Component-Based Design Principles)

### 单一职责 (Single Responsibility)

每个组件处理 GameObject 行为的一个方面：

```csharp
// ❌ 糟糕 - 上帝组件做所有事情
public class Player : MonoBehaviour
{
    public int health;
    public float moveSpeed;
    public Rigidbody rb;

    void Update()
    {
        HandleInput();
        Move();
        Attack();
        UpdateUI();
        CheckHealth();
    }
}

// ✅ 优秀 - 分离的组件
public class PlayerHealth : MonoBehaviour { }
public class PlayerMovement : MonoBehaviour { }
public class PlayerCombat : MonoBehaviour { }
public class PlayerInput : MonoBehaviour { }
```

**益处：**
- 更易于测试
- 更好的可重用性
- 调试更简单
- 职责清晰

### 组合优于继承 (Composition Over Inheritance)

优先使用组件组合而不是深层的继承层次结构：

```csharp
// ❌ 糟糕 - 深层继承
public class Character : MonoBehaviour { }
public class Enemy : Character { }
public class FlyingEnemy : Enemy { }
public class FastFlyingEnemy : FlyingEnemy { }

// ✅ 优秀 - 组件组合
public class Character : MonoBehaviour
{
    private Health health;
    private Movement movement;
    private Combat combat;
}

// 通过组件配置创建变体
// FastFlyingEnemy = Character + FlyingMovement + FastSpeed + MeleeAttack
```

## GetComponent 模式 (GetComponent Patterns)

### 缓存组件引用 (Caching Component References)

**始终缓存 GetComponent 结果** - 永远不要重复调用：

```csharp
// ❌ 糟糕 - 重复调用 GetComponent
private void Update()
{
    GetComponent<Rigidbody>().velocity = Vector3.forward;  // 慢！
    GetComponent<Animator>().SetBool("walking", true);     // 慢！
}

// ✅ 优秀 - 在 Awake 中缓存一次
private Rigidbody rb;
private Animator animator;

private void Awake()
{
    rb = GetComponent<Rigidbody>();
    animator = GetComponent<Animator>();
}

private void Update()
{
    rb.velocity = Vector3.forward;         // 快
    animator.SetBool("walking", true);     // 快
}
```

**性能影响**：GetComponent 开销很大。缓存可提供 10-100 倍的加速。

### TryGetComponent 模式

对可选组件使用 `TryGetComponent`：

```csharp
// ❌ 糟糕 - GetComponent 后进行空检查
Renderer renderer = GetComponent<Renderer>();
if (renderer != null)
{
    renderer.enabled = false;
}

// ✅ 优秀 - TryGetComponent (Unity 2019.2+)
if (TryGetComponent<Renderer>(out var renderer))
{
    renderer.enabled = false;
}
```

**益处：**
- 单次调用（更快）
- 意图清晰
- 无空赋值

### RequireComponent 属性

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

**保证：**
- 如果缺少组件会自动添加
- 当依赖者存在时无法被移除
- 清晰的依赖文档

### GetComponentInChildren 模式

搜索此 GameObject 及其子对象：

```csharp
// 在层次结构中查找第一个 Animator
private Animator animator;

private void Awake()
{
    animator = GetComponentInChildren<Animator>();
}

// 在子对象中查找所有 Collider
private Collider[] colliders;

private void Awake()
{
    colliders = GetComponentsInChildren<Collider>();
}

// 包含非活动的 GameObjects
private Transform[] bones;

private void Awake()
{
    bones = GetComponentsInChildren<Transform>(includeInactive: true);
}
```

**用例：**
- 在子对象中查找组件
- 角色骨骼（Animator，骨头）
- UI 层次结构

**性能：**昂贵 - 缓存结果，不要在 Update 中调用。

### GetComponentInParent 模式

搜索此 GameObject 及其父对象：

```csharp
// 查找 Canvas 父对象
private Canvas canvas;

private void Awake()
{
    canvas = GetComponentInParent<Canvas>();
}

// 检查是否属于 UI 层次结构的一部分
private bool IsUIElement()
{
    return GetComponentInParent<Canvas>() != null;
}
```

**用例：**
- UI 元素查找 Canvas
- 嵌套系统查找根管理器
- 层次结构查询

## 组件通信模式 (Component Communication Patterns)

### 直接引用 (Inspector)

对于已知关系最常见的模式：

```csharp
public class HealthDisplay : MonoBehaviour
{
    [SerializeField] private PlayerHealth playerHealth;

    private void OnEnable()
    {
        playerHealth.OnHealthChanged += UpdateDisplay;
    }

    private void OnDisable()
    {
        playerHealth.OnHealthChanged -= UpdateDisplay;
    }

    private void UpdateDisplay(int health)
    {
        // 更新 UI
    }
}
```

**优点：**
- 快（直接引用）
- 在 Inspector 中清晰可见
- 编译时安全

**缺点：**
- 需要手动连线
- 紧耦合

**何时使用：**已知的、固定的关系

### GetComponent (同一 GameObject)

对于同一 GameObject 上的组件：

```csharp
public class CharacterController : MonoBehaviour
{
    private CharacterMovement movement;
    private CharacterCombat combat;
    private CharacterInventory inventory;

    private void Awake()
    {
        movement = GetComponent<CharacterMovement>();
        combat = GetComponent<CharacterCombat>();
        inventory = GetComponent<CharacterInventory>();
    }

    public void Attack()
    {
        if (combat.CanAttack())
        {
            combat.PerformAttack();
            movement.StopMovement();
        }
    }
}
```

**何时使用：**需要协调的同一 GameObject 上的组件

### 事件系统 (解耦)

用于松耦合和多对多关系：

```csharp
// 定义事件
public class PlayerHealth : MonoBehaviour
{
    public event Action<int> OnHealthChanged;
    public event Action OnDeath;

    private int health = 100;

    public void TakeDamage(int damage)
    {
        health -= damage;
        OnHealthChanged?.Invoke(health);

        if (health <= 0)
            OnDeath?.Invoke();
    }
}

// 从任何组件订阅
public class HealthBar : MonoBehaviour
{
    [SerializeField] private PlayerHealth playerHealth;

    private void OnEnable()
    {
        playerHealth.OnHealthChanged += UpdateBar;
        playerHealth.OnDeath += OnPlayerDeath;
    }

    private void OnDisable()
    {
        playerHealth.OnHealthChanged -= UpdateBar;
        playerHealth.OnDeath -= OnPlayerDeath;
    }

    private void UpdateBar(int health) { }
    private void OnPlayerDeath() { }
}
```

**优点：**
- 松耦合
- 多个订阅者
- 易于扩展

**缺点：**
- 内存泄漏风险（忘记取消订阅）
- 调试较难

**关键：**始终在 OnDisable/OnDestroy 中取消订阅

### 消息系统 (Unity 内置)

谨慎使用 - 性能不如直接调用：

```csharp
// 接收者
public class TargetComponent : MonoBehaviour
{
    private void TakeDamage(int damage)
    {
        // 处理伤害
    }
}

// 发送者
GameObject target = ...;
target.SendMessage("TakeDamage", 10);  // 在所有组件上调用 TakeDamage

// 变体
SendMessage("Method", value, SendMessageOptions.DontRequireReceiver);
SendMessageUpwards("Method", value);  // 此对象 + 父对象
BroadcastMessage("Method", value);    // 此对象 + 子对象
```

**优点：**
- 不需要引用
- 跨组件工作

**缺点：**
- 基于字符串（无编译时检查）
- 慢（基于反射）
- 难以追踪

**何时使用：**极少。优先使用事件或接口。

### 接口模式

类型安全的通信，没有紧耦合：

```csharp
// 定义接口
public interface IDamageable
{
    void TakeDamage(int damage);
    int CurrentHealth { get; }
}

// 实现接口
public class PlayerHealth : MonoBehaviour, IDamageable
{
    [SerializeField] private int maxHealth = 100;
    private int health;

    public int CurrentHealth => health;

    private void Awake()
    {
        health = maxHealth;
    }

    public void TakeDamage(int damage)
    {
        health -= damage;
        if (health <= 0)
            Die();
    }

    private void Die() { }
}

// 使用接口
public class Weapon : MonoBehaviour
{
    private void OnTriggerEnter(Collider other)
    {
        var damageable = other.GetComponent<IDamageable>();
        if (damageable != null)
        {
            damageable.TakeDamage(10);
        }
    }
}
```

**优点：**
- 类型安全
- 松耦合
- 清晰的契约

**缺点：**
- GetComponent<Interface> 较慢（缓存结果）
- 无 Inspector 支持

**性能模式：**
```csharp
// 缓存接口引用
private IDamageable cachedDamageable;

private void OnTriggerEnter(Collider other)
{
    if (cachedDamageable == null)
        cachedDamageable = other.GetComponent<IDamageable>();

    cachedDamageable?.TakeDamage(10);
}
```

### 单例模式 (Singleton Pattern)

用于全局管理器（谨慎使用）：

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

// 从任何地方访问
public class AnyScript : MonoBehaviour
{
    private void Start()
    {
        GameManager.Instance.DoSomething();
    }
}
```

**用于：**
- 游戏管理器
- 音频管理器
- 输入管理器
- 存档管理器

**避免用于：**
- 其他所有东西（创建全局状态）

## 组件组织模式 (Component Organization Patterns)

### 协调器模式 (Coordinator Pattern)

一个组件协调多个专用组件：

```csharp
public class Character : MonoBehaviour
{
    // 协调的组件
    private CharacterMovement movement;
    private CharacterCombat combat;
    private CharacterInventory inventory;
    private CharacterAnimation animation;

    private void Awake()
    {
        movement = GetComponent<CharacterMovement>();
        combat = GetComponent<CharacterCombat>();
        inventory = GetComponent<CharacterInventory>();
        animation = GetComponent<CharacterAnimation>();
    }

    public void Attack()
    {
        if (combat.CanAttack())
        {
            movement.StopMovement();
            combat.PerformAttack();
            animation.PlayAttackAnimation();
        }
    }

    public void UseItem(Item item)
    {
        inventory.RemoveItem(item);
        item.Use(this);
        animation.PlayUseItemAnimation();
    }
}
```

**益处：**
- 清晰的入口点
- 协调复杂的行为
- 每个专用组件保持专注

### 数据组件模式 (Data Component Pattern)

将数据与行为分离：

```csharp
// 数据组件
[System.Serializable]
public class CharacterStats
{
    public int maxHealth = 100;
    public float moveSpeed = 5f;
    public int attackDamage = 25;
}

// 行为组件引用数据
public class Character : MonoBehaviour
{
    [SerializeField] private CharacterStats stats;

    public int MaxHealth => stats.maxHealth;
    public float MoveSpeed => stats.moveSpeed;
}

public class CharacterMovement : MonoBehaviour
{
    private Character character;

    private void Awake()
    {
        character = GetComponent<Character>();
    }

    private void Update()
    {
        // 使用 character.MoveSpeed
    }
}
```

**替代方案：ScriptableObject 数据**
```csharp
[CreateAssetMenu]
public class CharacterData : ScriptableObject
{
    public int maxHealth = 100;
    public float moveSpeed = 5f;
}

public class Character : MonoBehaviour
{
    [SerializeField] private CharacterData data;
}
```

### 状态组件模式 (State Component Pattern)

每个状态是一个单独的组件：

```csharp
// 基础状态
public abstract class CharacterState : MonoBehaviour
{
    public abstract void Enter();
    public abstract void Exit();
    public abstract void UpdateState();
}

// 具体状态
public class IdleState : CharacterState
{
    public override void Enter() { }
    public override void Exit() { }
    public override void UpdateState() { }
}

public class MovingState : CharacterState
{
    public override void Enter() { }
    public override void Exit() { }
    public override void UpdateState() { }
}

// 状态管理器
public class CharacterStateMachine : MonoBehaviour
{
    private CharacterState currentState;

    private void Awake()
    {
        // 获取所有状态组件
        var states = GetComponents<CharacterState>();
    }

    public void ChangeState(CharacterState newState)
    {
        currentState?.Exit();
        currentState = newState;
        currentState.Enter();
    }

    private void Update()
    {
        currentState?.UpdateState();
    }
}
```

## 性能考量 (Performance Considerations)

### 组件缓存 (Component Caching)

缓存所有组件引用：

```csharp
// ❌ 糟糕 - 每帧查找
private void Update()
{
    var manager = FindObjectOfType<GameManager>();  // 极其缓慢
    var rb = GetComponent<Rigidbody>();             // 慢
}

// ✅ 优秀 - 缓存一次
private GameManager manager;
private Rigidbody rb;

private void Awake()
{
    manager = FindObjectOfType<GameManager>();
    rb = GetComponent<Rigidbody>();
}

private void Update()
{
    // 使用缓存的引用 - 快
}
```

### 避免重复搜索

缓存搜索结果：

```csharp
// ❌ 糟糕 - 重复搜索
private void Update()
{
    GameObject[] enemies = GameObject.FindGameObjectsWithTag("Enemy");
    foreach (var enemy in enemies)
    {
        // 处理
    }
}

// ✅ 优秀 - 维护列表
private List<Enemy> enemies = new List<Enemy>();

// 敌人注册/注销自己
public void RegisterEnemy(Enemy enemy)
{
    enemies.Add(enemy);
}

public void UnregisterEnemy(Enemy enemy)
{
    enemies.Remove(enemy);
}
```

### 组件池 (Component Pooling)

重用组件而不是销毁：

```csharp
public class BulletPool : MonoBehaviour
{
    [SerializeField] private GameObject bulletPrefab;
    [SerializeField] private int poolSize = 20;

    private Queue<GameObject> pool = new Queue<GameObject>();

    private void Awake()
    {
        for (int i = 0; i < poolSize; i++)
        {
            GameObject bullet = Instantiate(bulletPrefab);
            bullet.SetActive(false);
            pool.Enqueue(bullet);
        }
    }

    public GameObject GetBullet()
    {
        if (pool.Count > 0)
        {
            GameObject bullet = pool.Dequeue();
            bullet.SetActive(true);
            return bullet;
        }

        // 池已耗尽 - 创建新的
        return Instantiate(bulletPrefab);
    }

    public void ReturnBullet(GameObject bullet)
    {
        bullet.SetActive(false);
        pool.Enqueue(bullet);
    }
}
```

## 常见的反模式 (Common Anti-Patterns)

### 上帝组件 (God Component)

❌ **不要**创建做所有事情的组件：

```csharp
// 糟糕 - 500+ 行，做所有事情
public class Player : MonoBehaviour
{
    // 移动
    public float moveSpeed;

    // 战斗
    public int attackDamage;

    // 库存
    public List<Item> items;

    // UI
    public Text healthText;

    // 音频
    public AudioClip jumpSound;

    private void Update()
    {
        HandleMovement();
        HandleCombat();
        UpdateInventoryUI();
        CheckAudio();
        // ... 还有 20 个方法
    }
}
```

✅ **要**拆分成专注的组件

### 组件耦合 (Component Coupling)

❌ **不要**紧密耦合组件：

```csharp
// 糟糕 - PlayerMovement 直接访问 PlayerCombat 内部
public class PlayerMovement : MonoBehaviour
{
    private PlayerCombat combat;

    private void Update()
    {
        if (!combat.isAttacking && !combat.isBlocking && combat.stamina > 10)
        {
            Move();
        }
    }
}
```

✅ **要**使用清晰的接口：

```csharp
// 优秀 - 清晰的接口
public class PlayerCombat : MonoBehaviour
{
    public bool CanMove() => !isAttacking && !isBlocking && stamina > 10;
}

public class PlayerMovement : MonoBehaviour
{
    private PlayerCombat combat;

    private void Update()
    {
        if (combat.CanMove())
            Move();
    }
}
```

### 循环依赖 (Circular Dependencies)

❌ **不要**创建循环组件引用：

```csharp
// 糟糕 - A 需要 B，B 需要 A
public class ComponentA : MonoBehaviour
{
    private ComponentB b;
}

public class ComponentB : MonoBehaviour
{
    private ComponentA a;
}
```

✅ **要**使用事件或协调器：

```csharp
// 优秀 - 使用事件
public class ComponentA : MonoBehaviour
{
    public event Action OnSomethingHappened;
}

public class ComponentB : MonoBehaviour
{
    [SerializeField] private ComponentA a;

    private void OnEnable()
    {
        a.OnSomethingHappened += HandleEvent;
    }
}
```

## 最佳实践总结 (Best Practices Summary)

✅ **要 (DO):**
- 在 Awake 中缓存所有 GetComponent 调用
- 使用 RequireComponent 处理依赖关系
- 保持组件专注（单一职责）
- 优先使用组合而非继承
- 使用事件进行解耦
- 始终取消订阅事件
- 使用接口作为类型安全的契约
- 对频繁实例化的组件进行池化

❌ **不要 (DON'T):**
- 在 Update/FixedUpdate 中调用 GetComponent
- 在 Update 中使用 Find 方法
- 创建上帝组件
- 紧密耦合组件
- 创建循环依赖
- 使用 SendMessage（优先使用事件/接口）
- 忘记取消订阅事件
- 把所有东西都做成单例

遵循这些模式以实现整洁、高性能、可维护的 Unity 组件架构。
