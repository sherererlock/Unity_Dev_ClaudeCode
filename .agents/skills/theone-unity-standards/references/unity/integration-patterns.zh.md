# 服务/桥接/适配器集成模式

## 模式结构

在集成第三方系统或复杂功能时：

```
YourFeature/
├── Core/
│   ├── Services/          # 核心业务逻辑（框架无关）
│   ├── Models/            # 数据模型
│   └── Interfaces/        # 抽象接口
├── Integration/
│   ├── Adapters/          # 将系统输出转换为游戏参数
│   ├── Bridges/           # 连接游戏事件与系统
│   └── Services/          # 特定于游戏的服务实现
└── DI/
    └── VContainer/        # 依赖注入注册
```

## 完整示例：难度系统集成

### 1. 适配器（ADAPTER）：将系统值转换为游戏参数

```csharp
public class DifficultyGameAdapter
{
    private readonly IDifficultyService difficultyService;

    [Preserve]
    public DifficultyGameAdapter(IDifficultyService difficultyService)
    {
        this.difficultyService = difficultyService;
    }

    public GameDifficultyParameters GetGameParameters()
    {
        var systemDifficulty = this.difficultyService.CurrentDifficulty;

        return new GameDifficultyParameters
        {
            EnemyCount = this.CalculateEnemyCount(systemDifficulty),
            EnemySpeed = this.CalculateEnemySpeed(systemDifficulty),
            EnemyHealth = this.CalculateEnemyHealth(systemDifficulty),
        };
    }

    private int CalculateEnemyCount(float difficulty) =>
        Mathf.RoundToInt(5 + difficulty * 3);

    private float CalculateEnemySpeed(float difficulty) =>
        1f + difficulty * 0.5f;

    private float CalculateEnemyHealth(float difficulty) =>
        100f + difficulty * 50f;
}
```

### 2. 桥接器（BRIDGE）：连接游戏事件与系统

```csharp
public class DifficultyGameBridge : IInitializable, IDisposable
{
    private readonly IDifficultyService difficultyService;
    private readonly DifficultyGameAdapter adapter;
    private readonly IGameplayService gameplayService;
    private readonly SignalBus signalBus;

    [Preserve]
    public DifficultyGameBridge(
        IDifficultyService difficultyService,
        DifficultyGameAdapter adapter,
        IGameplayService gameplayService,
        SignalBus signalBus)
    {
        this.difficultyService = difficultyService;
        this.adapter = adapter;
        this.gameplayService = gameplayService;
        this.signalBus = signalBus;
    }

    void IInitializable.Initialize()
    {
        // 订阅游戏事件
        this.signalBus.Subscribe<LevelStartedSignal>(this.OnLevelStarted);
        this.signalBus.Subscribe<LevelCompletedSignal>(this.OnLevelCompleted);

        // 订阅系统事件
        this.difficultyService.OnDifficultyChanged += this.OnDifficultyChanged;
    }

    private void OnLevelStarted(LevelStartedSignal signal)
    {
        // 将当前难度应用到游戏
        var parameters = this.adapter.GetGameParameters();
        this.gameplayService.ApplyDifficulty(parameters);
    }

    private void OnLevelCompleted(LevelCompletedSignal signal)
    {
        // 根据表现更新难度系统
        var performance = new PerformanceData
        {
            CompletionTime = signal.CompletionTime,
            Score = signal.Score,
            Success = true
        };

        this.difficultyService.RecordPerformance(performance);
    }

    private void OnDifficultyChanged(float newDifficulty)
    {
        // 通知游戏难度已变更
        this.signalBus.Fire(new DifficultyChangedSignal
        {
            NewDifficulty = newDifficulty
        });
    }

    void IDisposable.Dispose()
    {
        this.signalBus.TryUnsubscribe<LevelStartedSignal>(this.OnLevelStarted);
        this.signalBus.TryUnsubscribe<LevelCompletedSignal>(this.OnLevelCompleted);
        this.difficultyService.OnDifficultyChanged -= this.OnDifficultyChanged;
    }
}
```

### 3. 注册（REGISTRATION）：用于简洁 DI 设置的扩展方法

```csharp
public static class DifficultyIntegrationVContainer
{
    public static void RegisterDifficultyIntegration(this IContainerBuilder builder)
    {
        // 注册核心难度系统
        builder.RegisterModule(new DifficultyModule());

        // 注册适配器
        builder.Register<DifficultyGameAdapter>(Lifetime.Singleton);

        // 注册桥接器为入口点（自动初始化）
        builder.RegisterEntryPoint<DifficultyGameBridge>();
    }
}
```

## 为什么使用此模式？

### 关注点分离
- **核心逻辑** 独立于 Unity/游戏细节
- **适配器** 处理系统间的转换
- **桥接器** 处理系统间的事件流

### 可测试性
- 无需 Unity 即可测试核心逻辑
- 模拟适配器以进行特定于游戏的测试
- 模拟桥接器以进行集成测试

### 可维护性
- 易于替换第三方系统
- 游戏的更改不会影响核心逻辑
- 核心逻辑的更改不会影响游戏

### 灵活性
- 相同的核心逻辑可用于不同的项目
- 针对不同游戏类型的多个适配器
- 针对不同事件系统的多个桥接器

## 高级集成示例

### 分析集成

```csharp
// 适配器：将游戏事件转换为分析格式
public class AnalyticsAdapter
{
    public AnalyticsEvent ConvertLevelCompleted(LevelCompletedSignal signal)
    {
        return new AnalyticsEvent
        {
            Name = "level_completed",
            Properties = new Dictionary<string, object>
            {
                ["level"] = signal.LevelNumber,
                ["time"] = signal.CompletionTime,
                ["score"] = signal.Score,
            }
        };
    }
}

// 桥接器：连接游戏事件与分析
public class AnalyticsBridge : IInitializable, IDisposable
{
    private readonly SignalBus signalBus;
    private readonly IAnalyticsService analyticsService;
    private readonly AnalyticsAdapter adapter;

    void IInitializable.Initialize()
    {
        this.signalBus.Subscribe<LevelCompletedSignal>(this.OnLevelCompleted);
    }

    private void OnLevelCompleted(LevelCompletedSignal signal)
    {
        var analyticsEvent = this.adapter.ConvertLevelCompleted(signal);
        this.analyticsService.Track(analyticsEvent);
    }

    void IDisposable.Dispose()
    {
        this.signalBus.TryUnsubscribe<LevelCompletedSignal>(this.OnLevelCompleted);
    }
}
```

### IAP（应用内购买）集成

```csharp
// 适配器：将 IAP 产品转换为游戏物品
public class IAPAdapter
{
    public GameItem ConvertProduct(IAPProduct product)
    {
        return new GameItem
        {
            Id = product.ProductId,
            Name = product.Title,
            Description = product.Description,
            Price = product.Price,
            Currency = product.CurrencyCode,
        };
    }
}

// 桥接器：连接 IAP 事件与游戏
public class IAPBridge : IInitializable, IDisposable
{
    private readonly IIAPService iapService;
    private readonly IAPAdapter adapter;
    private readonly SignalBus signalBus;
    private readonly CurrencyController currencyController;

    void IInitializable.Initialize()
    {
        this.iapService.OnPurchaseComplete += this.OnPurchaseComplete;
        this.signalBus.Subscribe<PurchaseRequestedSignal>(this.OnPurchaseRequested);
    }

    private void OnPurchaseRequested(PurchaseRequestedSignal signal)
    {
        this.iapService.Purchase(signal.ProductId);
    }

    private void OnPurchaseComplete(IAPPurchase purchase)
    {
        var gameItem = this.adapter.ConvertProduct(purchase.Product);
        this.currencyController.Earn(gameItem.Price);
        this.signalBus.Fire(new PurchaseCompletedSignal(gameItem));
    }

    void IDisposable.Dispose()
    {
        this.iapService.OnPurchaseComplete -= this.OnPurchaseComplete;
        this.signalBus.TryUnsubscribe<PurchaseRequestedSignal>(this.OnPurchaseRequested);
    }
}
```
