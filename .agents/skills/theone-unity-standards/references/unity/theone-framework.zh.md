# TheOne.DI + 发布者/订阅者框架

部分 TheOne Studio 项目使用替代框架栈，而不是 VContainer+SignalBus。

## 框架栈

**使用：**
- **TheOne.DI** - DI 容器包装器（替代 VContainer）
- **IPublisher<T> / ISubscriber<T>** - 事件消息传递（替代 SignalBus）
- **`[Inject]` 属性** - 构造函数注入标记
- **TheOne.Extensions** - 扩展方法
- **TheOne.Logging.ILogger** - 日志记录（运行时，无守卫，无前缀，无构造函数日志），Debug.Log（仅编辑器）
- **TheOne.ResourceManagement** - 资产加载/卸载
- **TheOne.Data** - 玩家数据 (JSON)，蓝图 (CSV)
- **TheOne.Pooling** - 对象池
- **TheOne.Lifecycle** - 异步加载屏幕

## 依赖注入模式

```csharp
public sealed class ExampleService : IAsyncEarlyLoadable, IDisposable
{
    private readonly IAssetsManager assetsManager;
    private readonly IPublisher<ExampleSignal> publisher;
    private IDisposable? subscription;
    private GameObject? loadedPrefab;

    [Inject] // ✅ 使用 [Inject] 属性配合 TheOne.DI
    public ExampleService(
        IAssetsManager assetsManager,
        IPublisher<ExampleSignal> publisher,
        ISubscriber<ExampleSignal> subscriber
    )
    {
        this.assetsManager = assetsManager;
        this.publisher = publisher;
        this.subscription = subscriber.Subscribe(this.OnSignal);
    }

    private void OnSignal(ExampleSignal signal)
    {
        if (signal.Value < 0)
        {
            throw new ArgumentException($"Invalid value: {signal.Value}", nameof(signal));
        }

        this.publisher.Publish(new ExampleSignal(signal.Value * 2));
    }

    public void Dispose()
    {
        this.subscription?.Dispose();
        this.assetsManager.Unload(this.loadedPrefab);
    }
}
```

## 发布者/订阅者事件模式

### 信号定义

```csharp
// 将信号定义为 record（与 SignalBus 相同）
public sealed record ExampleSignal(int Value);
public sealed record LevelCompletedSignal(int Level, float Time);
```

### 发布事件

```csharp
public sealed class GameService
{
    private readonly IPublisher<LevelCompletedSignal> levelCompletedPublisher;

    [Inject]
    public GameService(IPublisher<LevelCompletedSignal> publisher)
    {
        this.levelCompletedPublisher = publisher;
    }

    public void CompleteLevel(int level, float time)
    {
        // 发布事件
        this.levelCompletedPublisher.Publish(new LevelCompletedSignal(level, time));
    }
}
```

### 订阅事件

```csharp
public sealed class AnalyticsService : IDisposable
{
    private IDisposable? subscription;

    [Inject]
    public AnalyticsService(ISubscriber<LevelCompletedSignal> subscriber)
    {
        // Subscribe 返回 IDisposable
        this.subscription = subscriber.Subscribe(this.OnLevelCompleted);
    }

    private void OnLevelCompleted(LevelCompletedSignal signal)
    {
        // 处理事件
        this.TrackEvent("level_completed", ("level", signal.Level));
    }

    public void Dispose()
    {
        // 销毁订阅以取消订阅
        this.subscription?.Dispose();
    }
}
```

## 资源管理

**关键规则：**
- 在 `Initialize`/`OnInstantiate` 中加载的资产必须在 `Dispose`/`OnCleanup` 中卸载
- 避免在运行时（`Update`/`Tick`）使用 `Find`/`GetComponent`，仅在初始化（`Awake`/`Start`/`OnSpawn`）中使用

```csharp
public sealed class PrefabService : IAsyncEarlyLoadable, IDisposable
{
    private readonly IAssetsManager assetsManager;
    private GameObject? loadedPrefab;

    public async UniTask LoadAsync(IProgress<float> progress, CancellationToken cancellationToken)
    {
        // 加载资产
        this.loadedPrefab = await this.assetsManager.LoadAsync<GameObject>(
            "path/to/prefab",
            cancellationToken
        );
    }

    public void Dispose()
    {
        // 卸载资产
        this.assetsManager.Unload(this.loadedPrefab);
    }
}
```

## 何时使用此栈

**检查项目依赖：**
- 如果看到 `TheOne.DI` 包 → 使用此模式
- 如果看到 `IPublisher<T>`/`ISubscriber<T>` → 使用发布者/订阅者（而非 SignalBus）
- 如果在现有代码中看到 `[Inject]` 属性 → 遵循该模式

## 与 VContainer+SignalBus 的主要区别

| 方面 | VContainer+SignalBus | TheOne.DI+Publisher/Subscriber |
|--------|---------------------|-------------------------------|
| DI 容器 | VContainer | TheOne.DI |
| 构造函数标记 | `[Preserve]` | `[Inject]` |
| 事件 | SignalBus | IPublisher<T>/ISubscriber<T> |
| 订阅 | `signalBus.Subscribe<T>(handler)` | `subscriber.Subscribe(handler)` |
| 发布 | `signalBus.Fire(signal)` | `publisher.Publish(signal)` |
| 取消订阅 | `signalBus.TryUnsubscribe<T>(handler)` | `subscription.Dispose()` |
| 初始化 | `IInitializable.Initialize()` | `IAsyncEarlyLoadable.LoadAsync()` |

## 完整示例

```csharp
public sealed class DifficultyService : IAsyncEarlyLoadable, IDisposable
{
    private readonly IAssetsManager assetsManager;
    private readonly IPublisher<DifficultyChangedSignal> difficultyPublisher;
    private IDisposable? levelCompletedSubscription;
    private DifficultyConfig? config;

    [Inject]
    public DifficultyService(
        IAssetsManager assetsManager,
        IPublisher<DifficultyChangedSignal> difficultyPublisher,
        ISubscriber<LevelCompletedSignal> levelCompletedSubscriber
    )
    {
        this.assetsManager = assetsManager;
        this.difficultyPublisher = difficultyPublisher;
        this.levelCompletedSubscription = levelCompletedSubscriber.Subscribe(this.OnLevelCompleted);
    }

    public async UniTask LoadAsync(IProgress<float> progress, CancellationToken cancellationToken)
    {
        // 加载配置
        this.config = await this.assetsManager.LoadAsync<DifficultyConfig>(
            "Configs/DifficultyConfig",
            cancellationToken
        );
    }

    private void OnLevelCompleted(LevelCompletedSignal signal)
    {
        // 计算新难度
        var newDifficulty = this.CalculateDifficulty(signal);

        // 发布难度变更
        this.difficultyPublisher.Publish(new DifficultyChangedSignal(newDifficulty));
    }

    public void Dispose()
    {
        // 取消订阅事件
        this.levelCompletedSubscription?.Dispose();

        // 卸载资产
        this.assetsManager.Unload(this.config);
    }
}
```
