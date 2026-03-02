# 架构审查：VContainer、SignalBus、控制器

## VContainer 依赖注入审查

### 检查清单
- [ ] 所有依赖项是否都通过构造函数注入？
- [ ] 注入的字段是否标记为 `readonly`？
- [ ] 构造函数是否添加了 `[Preserve]` 属性？
- [ ] 服务是否在 LifetimeScope 中正确注册？
- [ ] 是否避免了字段/属性注入？

### 常见违规

#### ❌ 字段注入
```csharp
// 错误：字段注入
public class MyService
{
    [Inject] private SignalBus signalBus; // ❌ 错误
}
```

#### ❌ 缺少 readonly
```csharp
// 错误：缺少 readonly
public class MyService
{
    private SignalBus signalBus; // ❌ 应为 readonly

    public MyService(SignalBus signalBus)
    {
        this.signalBus = signalBus;
    }
}
```

#### ❌ 缺少 [Preserve]
```csharp
// 错误：缺少 [Preserve]
public MyService(SignalBus signalBus) // ❌ 缺少属性
{
    this.signalBus = signalBus;
}
```

### ✅ 正确模式
```csharp
public sealed class MyService
{
    private readonly SignalBus signalBus;
    private readonly IAnalyticServices analyticService;

    [Preserve]
    public MyService(
        SignalBus signalBus,
        IAnalyticServices analyticService)
    {
        this.signalBus = signalBus;
        this.analyticService = analyticService;
    }
}
```

## SignalBus 事件系统审查

### 检查清单
- [ ] Signal 是否使用 class/record（而非 struct）？
- [ ] Signal 订阅是否在 Initialize() 中进行？
- [ ] Signal 取消订阅是否在 Dispose() 中进行？
- [ ] 是否使用了 TryUnsubscribe()（而非 Unsubscribe()）？
- [ ] 是否避免使用 MessagePipe（应使用 SignalBus）？

### 常见违规

#### ❌ Struct Signal
```csharp
// 错误：Struct signal
public struct WonSignal { } // ❌ 应使用 class 或 record
```

#### ❌ 在构造函数中订阅
```csharp
// 错误：在构造函数中订阅
public MyService(SignalBus signalBus)
{
    signalBus.Subscribe<WonSignal>(OnWon); // ❌ 应在 Initialize 中进行
}
```

#### ❌ 使用 Unsubscribe 而非 TryUnsubscribe
```csharp
// 错误：使用 Unsubscribe 而非 TryUnsubscribe
void IDisposable.Dispose()
{
    this.signalBus.Unsubscribe<WonSignal>(OnWon); // ❌ 应使用 TryUnsubscribe
}
```

#### ❌ 未取消订阅
```csharp
// 错误：未取消订阅
public class MyService : IInitializable
{
    void IInitializable.Initialize()
    {
        this.signalBus.Subscribe<WonSignal>(OnWon);
    }
    // ❌ 缺少 IDisposable 实现！
}
```

### ✅ 正确模式
```csharp
// 正确：Record signal
public sealed record WonSignal;

public sealed class MyService : IInitializable, IDisposable
{
    private readonly SignalBus signalBus;

    [Preserve]
    public MyService(SignalBus signalBus)
    {
        this.signalBus = signalBus;
    }

    void IInitializable.Initialize()
    {
        this.signalBus.Subscribe<WonSignal>(this.OnWon);
    }

    private void OnWon(WonSignal signal)
    {
        // 处理事件
    }

    void IDisposable.Dispose()
    {
        this.signalBus.TryUnsubscribe<WonSignal>(this.OnWon);
    }
}
```

## 数据控制器使用审查

### 检查清单
- [ ] 数据是否通过控制器访问（而非直接访问）？
- [ ] 业务规则是否在控制器中（而非分散各地）？
- [ ] 控制器在数据变更时是否发送 Signal？

### 常见违规

#### ❌ 直接访问数据
```csharp
// 错误：直接访问数据
public class GameService
{
    private readonly UITemplateUserLevelData levelData;

    public void CompleteLevel()
    {
        this.levelData.CurrentLevel++; // ❌ 绝不要直接访问
        this.levelData.TotalWins++;
    }
}
```

### ✅ 正确模式
```csharp
// 正确：使用控制器
public class GameService
{
    private readonly UITemplateLevelDataController levelController;

    public void CompleteLevel()
    {
        this.levelController.PassCurrentLevel(); // ✅ 使用控制器
    }
}
```

## 完整架构审查示例

### ❌ 糟糕的代码（多处违规）
```csharp
public class GameService
{
    // ❌ 字段注入
    [Inject] private SignalBus signalBus;

    // ❌ 非 readonly
    private UITemplateUserLevelData levelData;

    // ❌ 缺少 [Preserve]
    public GameService(UITemplateUserLevelData levelData)
    {
        this.levelData = levelData;

        // ❌ 在构造函数中订阅
        this.signalBus.Subscribe<WonSignal>(OnWon);
    }

    private void OnWon(WonSignal signal)
    {
        // ❌ 直接访问数据
        this.levelData.CurrentLevel++;
    }

    // ❌ 缺少 IDisposable - 内存泄漏！
}
```

### ✅ 优秀的代码（正确模式）
```csharp
public sealed class GameService : IInitializable, IDisposable
{
    // ✅ readonly 字段
    private readonly SignalBus signalBus;
    private readonly UITemplateLevelDataController levelController;

    // ✅ 带有 [Preserve] 的构造函数注入
    [Preserve]
    public GameService(
        SignalBus signalBus,
        UITemplateLevelDataController levelController)
    {
        this.signalBus = signalBus;
        this.levelController = levelController;
    }

    // ✅ 在 Initialize 中订阅
    void IInitializable.Initialize()
    {
        this.signalBus.Subscribe<WonSignal>(this.OnWon);
    }

    private void OnWon(WonSignal signal)
    {
        // ✅ 使用控制器
        this.levelController.PassCurrentLevel();
    }

    // ✅ 在 Dispose 中取消订阅
    void IDisposable.Dispose()
    {
        this.signalBus.TryUnsubscribe<WonSignal>(this.OnWon);
    }
}
```

## 审查严重程度

### 🔴 严重问题
- 使用字段注入代替构造函数注入
- 直接访问数据（未使用控制器）
- 缺少 IDisposable 实现（内存泄漏）
- 未取消订阅 Signal
- 使用 struct 作为 Signal

### 🟡 重要问题
- 缺少 `[Preserve]` 属性
- 注入字段缺少 `readonly`
- 在构造函数中订阅而非在 `Initialize` 中
- 使用 `Unsubscribe` 而非 `TryUnsubscribe`

### 🟢 建议
- 可以密封类（sealed）
- 可以使用显式接口实现
- 可以添加 XML 文档
