---
name: Unity Architecture
description: 当用户询问“游戏架构”、“设计模式”、“管理器模式”、“单例模式”、“ScriptableObject”、“ScriptableObject 架构”、“事件系统”、“观察者模式”、“发布-订阅”、“Unity 中的 MVC”、“依赖注入”、“服务定位器”，或需要关于构建 Unity 项目和游戏系统的指导时，应使用此技能。
version: 0.1.0
---

# Unity 游戏架构

针对可扩展、可维护的 Unity 项目的基本架构模式和设计原则。

## 概述

良好的架构可以分离关注点，减少耦合，并使代码易于测试和维护。本技能涵盖了 Unity 游戏开发中经过验证的模式。

**核心架构概念：**
- 管理器模式和全局系统
- 基于 ScriptableObject 的数据架构
- 事件驱动通信
- 组件组合模式
- 依赖管理

## 管理器模式 (Manager Pattern)

协调整个游戏功能的中心化系统。

### 单例管理器 (Singleton Manager)

全局管理器最常见的模式：

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

    public void StartGame() { }
    public void PauseGame() { }
    public void EndGame() { }
}

// 从任何地方访问
public class Player : MonoBehaviour
{
    private void Start()
    {
        GameManager.Instance.StartGame();
    }
}
```

**何时使用：**
- 游戏状态管理 (GameManager)
- 音频管理 (AudioManager)
- 输入管理 (InputManager)
- 保存/加载系统 (SaveManager)
- UI 管理 (UIManager)

**何时不使用：**
- 所有情况（避免“单例地狱”）
- 临时系统
- 需要多个实例的系统

### 通用单例基类 (Generic Singleton Base)

可重用的单例模式：

```csharp
public class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T instance;

    public static T Instance
    {
        get
        {
            if (instance == null)
            {
                instance = FindObjectOfType<T>();

                if (instance == null)
                {
                    GameObject singleton = new GameObject(typeof(T).Name);
                    instance = singleton.AddComponent<T>();
                    DontDestroyOnLoad(singleton);
                }
            }

            return instance;
        }
    }

    protected virtual void Awake()
    {
        if (instance == null)
        {
            instance = this as T;
            DontDestroyOnLoad(gameObject);
        }
        else if (instance != this)
        {
            Destroy(gameObject);
        }
    }
}

// 用法
public class GameManager : Singleton<GameManager>
{
    protected override void Awake()
    {
        base.Awake();
        // 额外的初始化
    }
}
```

### 管理器初始化顺序 (Manager Initialization Order)

控制管理器的初始化：

```csharp
// 使用脚本执行顺序 (Script Execution Order):
// Edit > Project Settings > Script Execution Order

// 或者显式初始化
public class GameBootstrap : MonoBehaviour
{
    private void Awake()
    {
        InitializeManagers();
    }

    private void InitializeManagers()
    {
        // 按特定顺序初始化
        var saveManager = SaveManager.Instance;
        var audioManager = AudioManager.Instance;
        var gameManager = GameManager.Instance;

        // 管理器在 Awake 中初始化，但在此处访问可确保顺序
    }
}
```

**最佳实践**：使用显式初始化场景或引导程序 (bootstrapper)。

### 服务定位器模式 (Service Locator Pattern)

单例模式的替代方案，用于依赖注入：

```csharp
public class ServiceLocator
{
    private static ServiceLocator instance;
    public static ServiceLocator Instance => instance ?? (instance = new ServiceLocator());

    private readonly Dictionary<Type, object> services = new Dictionary<Type, object>();

    public void RegisterService<T>(T service)
    {
        services[typeof(T)] = service;
    }

    public T GetService<T>()
    {
        if (services.TryGetValue(typeof(T), out var service))
        {
            return (T)service;
        }

        throw new Exception($"Service {typeof(T)} not found");
    }
}

// 注册服务
public class GameBootstrap : MonoBehaviour
{
    private void Awake()
    {
        var audioManager = new AudioManager();
        ServiceLocator.Instance.RegisterService(audioManager);

        var saveManager = new SaveManager();
        ServiceLocator.Instance.RegisterService(saveManager);
    }
}

// 访问服务
public class Player : MonoBehaviour
{
    private void Start()
    {
        var audio = ServiceLocator.Instance.GetService<AudioManager>();
        audio.PlaySound("Jump");
    }
}
```

**相对于单例的优点：**
- 可测试（注入模拟服务）
- 无静态依赖
- 依赖关系清晰

**缺点：**
- 更多设置代码
- 运行时字典查找
- 可发现性较差

## ScriptableObject 架构

使用 ScriptableObject 的数据驱动设计。

### ScriptableObject 数据容器

将数据与行为分离：

```csharp
[CreateAssetMenu(fileName = "WeaponData", menuName = "Game/Weapon Data")]
public class WeaponData : ScriptableObject
{
    public string weaponName;
    public int damage;
    public float fireRate;
    public Sprite icon;
    public GameObject projectilePrefab;
}

public class Weapon : MonoBehaviour
{
    [SerializeField] private WeaponData data;

    public void Fire()
    {
        Instantiate(data.projectilePrefab, firePoint.position, firePoint.rotation);
    }

    public int GetDamage() => data.damage;
}
```

**优点：**
- 数据与代码分离
- 跨场景共享数据
- 编辑无需更改代码
- 对设计师友好

**用于：**
- 物品/武器属性
- 角色数据
- 游戏平衡数值
- 配置

### ScriptableObject 事件

使用 ScriptableObject 的事件系统：

```csharp
[CreateAssetMenu(fileName = "GameEvent", menuName = "Events/Game Event")]
public class GameEvent : ScriptableObject
{
    private readonly List<GameEventListener> listeners = new List<GameEventListener>();

    public void Raise()
    {
        for (int i = listeners.Count - 1; i >= 0; i--)
        {
            listeners[i].OnEventRaised();
        }
    }

    public void RegisterListener(GameEventListener listener)
    {
        if (!listeners.Contains(listener))
            listeners.Add(listener);
    }

    public void UnregisterListener(GameEventListener listener)
    {
        listeners.Remove(listener);
    }
}

public class GameEventListener : MonoBehaviour
{
    [SerializeField] private GameEvent gameEvent;
    [SerializeField] private UnityEvent response;

    private void OnEnable()
    {
        gameEvent.RegisterListener(this);
    }

    private void OnDisable()
    {
        gameEvent.UnregisterListener(this);
    }

    public void OnEventRaised()
    {
        response.Invoke();
    }
}
```

**用法：**
```
创建 GameEvent 资产: "OnPlayerDeath"
将 GameEventListener 附加到 UI
配置响应: 显示死亡屏幕
当玩家死亡时引发事件
```

**优点：**
- 设计师可访问
- Inspector 中的可视化连线
- 解耦系统
- 可重用事件

### ScriptableObject 变量

跨场景共享变量：

```csharp
public abstract class ScriptableVariable<T> : ScriptableObject
{
    [SerializeField] private T value;

    public T Value
    {
        get => value;
        set
        {
            this.value = value;
            OnValueChanged?.Invoke(value);
        }
    }

    public event Action<T> OnValueChanged;
}

[CreateAssetMenu(fileName = "IntVariable", menuName = "Variables/Int")]
public class IntVariable : ScriptableVariable<int> { }

[CreateAssetMenu(fileName = "FloatVariable", menuName = "Variables/Float")]
public class FloatVariable : ScriptableVariable<float> { }
```

**用法：**
```csharp
public class Player : MonoBehaviour
{
    [SerializeField] private IntVariable playerHealth;

    public void TakeDamage(int damage)
    {
        playerHealth.Value -= damage;  // 更新所有订阅者
    }
}

public class HealthUI : MonoBehaviour
{
    [SerializeField] private IntVariable playerHealth;
    [SerializeField] private Text healthText;

    private void OnEnable()
    {
        playerHealth.OnValueChanged += UpdateUI;
        UpdateUI(playerHealth.Value);
    }

    private void OnDisable()
    {
        playerHealth.OnValueChanged -= UpdateUI;
    }

    private void UpdateUI(int health)
    {
        healthText.text = $"Health: {health}";
    }
}
```

**优点：**
- 跨场景持久化
- 多个监听器
- Inspector 可编辑
- 运行时更改随处反映

## 事件系统

组件之间的解耦通信。

### C# 事件

标准 C# 事件模式：

```csharp
public class Health : MonoBehaviour
{
    public event Action<int> OnHealthChanged;
    public event Action OnDeath;

    private int health = 100;

    public void TakeDamage(int damage)
    {
        health -= damage;
        OnHealthChanged?.Invoke(health);

        if (health <= 0)
        {
            OnDeath?.Invoke();
        }
    }
}

public class HealthUI : MonoBehaviour
{
    [SerializeField] private Health playerHealth;

    private void OnEnable()
    {
        playerHealth.OnHealthChanged += UpdateHealthBar;
        playerHealth.OnDeath += ShowDeathScreen;
    }

    private void OnDisable()
    {
        playerHealth.OnHealthChanged -= UpdateHealthBar;
        playerHealth.OnDeath -= ShowDeathScreen;
    }

    private void UpdateHealthBar(int health) { }
    private void ShowDeathScreen() { }
}
```

**关键**：始终在 `OnDisable` 中取消订阅以防止内存泄漏。

### UnityEvent

Inspector 可分配事件：

```csharp
public class Interactable : MonoBehaviour
{
    [SerializeField] private UnityEvent onInteract;
    [SerializeField] private UnityEvent<int> onScoreChanged;

    public void Interact()
    {
        onInteract?.Invoke();
    }

    public void AddScore(int points)
    {
        onScoreChanged?.Invoke(points);
    }
}
```

**优点：**
- 设计师可在 Inspector 中访问
- 简单交互无需代码
- 可视化连线

**缺点：**
- 比 C# 事件慢
- 无编译时检查
- 较难调试

**用于：**
- 简单交互
- 设计师驱动的事件
- 原型制作

### 全局事件总线 (Global Event Bus)

中心化事件系统：

```csharp
public static class EventBus
{
    private static readonly Dictionary<Type, Delegate> eventTable = new Dictionary<Type, Delegate>();

    public static void Subscribe<T>(Action<T> handler)
    {
        if (eventTable.TryGetValue(typeof(T), out var existingHandler))
        {
            eventTable[typeof(T)] = Delegate.Combine(existingHandler, handler);
        }
        else
        {
            eventTable[typeof(T)] = handler;
        }
    }

    public static void Unsubscribe<T>(Action<T> handler)
    {
        if (eventTable.TryGetValue(typeof(T), out var existingHandler))
        {
            var newHandler = Delegate.Remove(existingHandler, handler);
            if (newHandler == null)
                eventTable.Remove(typeof(T));
            else
                eventTable[typeof(T)] = newHandler;
        }
    }

    public static void Publish<T>(T eventData)
    {
        if (eventTable.TryGetValue(typeof(T), out var handler))
        {
            (handler as Action<T>)?.Invoke(eventData);
        }
    }
}

// 事件数据类型
public struct PlayerDiedEvent
{
    public Vector3 position;
    public string killedBy;
}

// 订阅
public class DeathUI : MonoBehaviour
{
    private void OnEnable()
    {
        EventBus.Subscribe<PlayerDiedEvent>(OnPlayerDied);
    }

    private void OnDisable()
    {
        EventBus.Unsubscribe<PlayerDiedEvent>(OnPlayerDied);
    }

    private void OnPlayerDied(PlayerDiedEvent data)
    {
        ShowDeathScreen(data.position, data.killedBy);
    }
}

// 发布
public class Player : MonoBehaviour
{
    private void Die()
    {
        EventBus.Publish(new PlayerDiedEvent
        {
            position = transform.position,
            killedBy = "Enemy"
        });
    }
}
```

**优点：**
- 完全解耦
- 无需直接引用
- 结构体类型安全

**缺点：**
- 运行时开销（字典查找）
- 较难追踪事件流
- 如果忘记取消订阅会有内存泄漏风险

## 组件组合模式

### 策略模式 (Strategy Pattern)

可互换的行为：

```csharp
public interface IMovementStrategy
{
    void Move(Transform transform, float speed);
}

public class GroundMovement : MonoBehaviour, IMovementStrategy
{
    public void Move(Transform transform, float speed)
    {
        // 地面移动
    }
}

public class FlyingMovement : MonoBehaviour, IMovementStrategy
{
    public void Move(Transform transform, float speed)
    {
        // 飞行移动
    }
}

public class Character : MonoBehaviour
{
    [SerializeField] private float speed = 5f;
    private IMovementStrategy movementStrategy;

    private void Awake()
    {
        movementStrategy = GetComponent<IMovementStrategy>();
    }

    private void Update()
    {
        movementStrategy.Move(transform, speed);
    }
}
```

**优点：**
- 运行时切换行为
- 可重用策略
- 开闭原则

### 状态机模式 (State Machine Pattern)

管理对象状态：

```csharp
public interface IState
{
    void Enter();
    void Execute();
    void Exit();
}

public class IdleState : MonoBehaviour, IState
{
    public void Enter() => Debug.Log("Entering Idle");
    public void Execute() { }
    public void Exit() => Debug.Log("Exiting Idle");
}

public class StateMachine : MonoBehaviour
{
    private IState currentState;

    public void ChangeState(IState newState)
    {
        currentState?.Exit();
        currentState = newState;
        currentState?.Enter();
    }

    private void Update()
    {
        currentState?.Execute();
    }
}

public class Enemy : MonoBehaviour
{
    private StateMachine stateMachine;
    private IdleState idleState;
    private ChaseState chaseState;

    private void Awake()
    {
        stateMachine = GetComponent<StateMachine>();
        idleState = GetComponent<IdleState>();
        chaseState = GetComponent<ChaseState>();

        stateMachine.ChangeState(idleState);
    }

    public void OnPlayerSpotted()
    {
        stateMachine.ChangeState(chaseState);
    }
}
```

### 观察者模式 (Observer Pattern)

一对多通知：

```csharp
public interface IObserver
{
    void OnNotify(string eventType);
}

public class Subject : MonoBehaviour
{
    private readonly List<IObserver> observers = new List<IObserver>();

    public void Attach(IObserver observer)
    {
        if (!observers.Contains(observer))
            observers.Add(observer);
    }

    public void Detach(IObserver observer)
    {
        observers.Remove(observer);
    }

    protected void Notify(string eventType)
    {
        for (int i = observers.Count - 1; i >= 0; i--)
        {
            observers[i].OnNotify(eventType);
        }
    }
}

public class Player : Subject
{
    public void Jump()
    {
        Notify("PlayerJumped");
    }
}

public class AudioObserver : MonoBehaviour, IObserver
{
    [SerializeField] private Player player;

    private void OnEnable()
    {
        player.Attach(this);
    }

    private void OnDisable()
    {
        player.Detach(this);
    }

    public void OnNotify(string eventType)
    {
        if (eventType == "PlayerJumped")
            PlayJumpSound();
    }
}
```

## Unity 中的 MVC/MVP

### 模型-视图-控制器 (Model-View-Controller)

分离数据、表现和逻辑：

```csharp
// Model - 数据
public class PlayerModel
{
    public int Health { get; private set; } = 100;
    public int Score { get; private set; } = 0;

    public event Action<int> OnHealthChanged;
    public event Action<int> OnScoreChanged;

    public void TakeDamage(int damage)
    {
        Health -= damage;
        OnHealthChanged?.Invoke(Health);
    }

    public void AddScore(int points)
    {
        Score += points;
        OnScoreChanged?.Invoke(Score);
    }
}

// View - 表现
public class PlayerView : MonoBehaviour
{
    [SerializeField] private Text healthText;
    [SerializeField] private Text scoreText;

    public void UpdateHealth(int health)
    {
        healthText.text = $"Health: {health}";
    }

    public void UpdateScore(int score)
    {
        scoreText.text = $"Score: {score}";
    }
}

// Controller - 逻辑
public class PlayerController : MonoBehaviour
{
    private PlayerModel model;
    private PlayerView view;

    private void Awake()
    {
        model = new PlayerModel();
        view = GetComponent<PlayerView>();

        model.OnHealthChanged += view.UpdateHealth;
        model.OnScoreChanged += view.UpdateScore;
    }

    private void OnDestroy()
    {
        model.OnHealthChanged -= view.UpdateHealth;
        model.OnScoreChanged -= view.UpdateScore;
    }

    public void TakeDamage(int damage)
    {
        model.TakeDamage(damage);
    }

    public void AddScore(int points)
    {
        model.AddScore(points);
    }
}
```

**优点：**
- 可测试（模拟 model/view）
- 可重用组件
- 清晰的分离

**何时使用：**
- 复杂的 UI
- 需要可测试的代码
- 同一数据的多个视图

## 附加资源

### 参考文件

有关详细的架构模式，请查阅：
- **`references/manager-patterns.md`** - 管理器实现、初始化、通信
- **`references/scriptableobject-architecture.md`** - 高级 SO 模式、运行时集合、变量
- **`references/event-systems.md`** - 事件模式、消息总线、发布-订阅系统
- **`references/design-patterns.md`** - Unity 的工厂、对象池、命令、策略模式

## 最佳实践

✅ **要做：**
- 使用管理器处理全局系统（音频、输入、保存）
- 利用 ScriptableObject 存储数据
- 实现事件驱动通信
- 优先组合而非继承
- 始终在 OnDisable 中取消订阅事件
- 使用依赖注入以提高可测试性
- 保持管理器专注（单一职责）

❌ **不要：**
- 把所有东西都做成单例
- 创建上帝管理器（应只处理一个关注点）
- 忘记取消订阅事件
- 硬编码依赖
- 任何事都用单例
- 混合数据和表现
- 创建深层继承层次结构

**黄金法则**：为变化而设计。松耦合、高内聚的系统更容易维护、测试和扩展。

---

应用这些架构模式，以构建经得起时间考验的可扩展、可维护的 Unity 项目。
