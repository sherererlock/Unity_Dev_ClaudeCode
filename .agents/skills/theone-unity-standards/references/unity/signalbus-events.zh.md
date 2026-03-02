# SignalBus 事件模式

## 信号定义最佳实践

### ✅ 使用：类（Classes）或记录（Records）

```csharp
// 推荐：用于简单信号的记录（Record）
public sealed record WonSignal;
public sealed record LostSignal;
public sealed record RestartSignal;

// 推荐：用于包含数据的信号的类（Class）
public class DifficultyChangedSignal
{
    public float OldDifficulty { get; set; }
    public float NewDifficulty { get; set; }
    public string Reason { get; set; }
}

// 推荐：包含属性的记录（Record）
public sealed record LevelCompletedSignal(int LevelNumber, float CompletionTime, int Score);
```

### ❌ 绝不使用：结构体（Structs）

```csharp
// 糟糕：结构体信号会导致问题
public struct WonSignal { } // ❌ 错误

// 为什么结构体会有问题：
// 1. SignalBus 使用引用相等性来跟踪订阅
// 2. 结构体与泛型类型一起使用时会导致装箱/拆箱开销
// 3. 每次 Fire() 都会创建一个新副本（值语义），破坏了引用标识
// 4. TryUnsubscribe() 将无法正常工作 - 无法匹配值类型的引用
// 5. 装箱产生的内存分配消除了结构体的性能优势
```

## 完整的信号使用模式

```csharp
public sealed class GameplayService : IInitializable, IDisposable
{
    private readonly SignalBus signalBus;

    [Preserve]
    public GameplayService(SignalBus signalBus)
    {
        this.signalBus = signalBus;
    }

    void IInitializable.Initialize()
    {
        // ✅ 订阅信号
        this.signalBus.Subscribe<RestartSignal>(this.OnRestart);
        this.signalBus.Subscribe<DifficultyChangedSignal>(this.OnDifficultyChanged);
    }

    private void OnRestart(RestartSignal signal)
    {
        // 处理重新开始
        this.ResetGame();
    }

    private void OnDifficultyChanged(DifficultyChangedSignal signal)
    {
        // 处理难度变更
        this.ApplyDifficulty(signal.NewDifficulty);
    }

    public void CompleteLevel(int level, float time, int score)
    {
        // ✅ 触发信号
        this.signalBus.Fire(new LevelCompletedSignal(level, time, score));
    }

    void IDisposable.Dispose()
    {
        // ✅ 始终取消订阅（使用 TryUnsubscribe 以避免异常）
        this.signalBus.TryUnsubscribe<RestartSignal>(this.OnRestart);
        this.signalBus.TryUnsubscribe<DifficultyChangedSignal>(this.OnDifficultyChanged);
    }
}
```

## 信号命名约定

- **动作信号（Action Signals）**: `VerbSignal`（动词+Signal） (例如：`WonSignal`, `LostSignal`, `RestartSignal`)
- **状态变更（State Change）**: `StateChangedSignal`（状态+ChangedSignal） (例如：`DifficultyChangedSignal`, `LevelChangedSignal`)
- **用户操作（User Action）**: `UserActionSignal`（用户+ActionSignal） (例如：`UserZoomSignal`, `UserClickSignal`)

## 高级信号模式

### 具有多个订阅者的信号

```csharp
// 多个服务可以订阅同一个信号
public sealed class AnalyticsService : IInitializable, IDisposable
{
    private readonly SignalBus signalBus;

    void IInitializable.Initialize()
    {
        this.signalBus.Subscribe<LevelCompletedSignal>(this.OnLevelCompleted);
    }

    private void OnLevelCompleted(LevelCompletedSignal signal)
    {
        // 追踪分析数据
        this.TrackEvent("level_completed",
            ("level", signal.LevelNumber),
            ("time", signal.CompletionTime));
    }

    void IDisposable.Dispose()
    {
        this.signalBus.TryUnsubscribe<LevelCompletedSignal>(this.OnLevelCompleted);
    }
}

public sealed class UIService : IInitializable, IDisposable
{
    private readonly SignalBus signalBus;

    void IInitializable.Initialize()
    {
        this.signalBus.Subscribe<LevelCompletedSignal>(this.OnLevelCompleted);
    }

    private void OnLevelCompleted(LevelCompletedSignal signal)
    {
        // 显示完成 UI
        this.ShowVictoryScreen(signal);
    }

    void IDisposable.Dispose()
    {
        this.signalBus.TryUnsubscribe<LevelCompletedSignal>(this.OnLevelCompleted);
    }
}
```

### 信号链

```csharp
public sealed class GameStateService : IInitializable, IDisposable
{
    private readonly SignalBus signalBus;

    void IInitializable.Initialize()
    {
        this.signalBus.Subscribe<LevelCompletedSignal>(this.OnLevelCompleted);
    }

    private void OnLevelCompleted(LevelCompletedSignal signal)
    {
        // 处理关卡完成
        this.SaveProgress(signal);

        // 触发链中的下一个信号
        if (this.IsLastLevel(signal.LevelNumber))
        {
            this.signalBus.Fire(new GameCompletedSignal());
        }
        else
        {
            this.signalBus.Fire(new ShowNextLevelSignal(signal.LevelNumber + 1));
        }
    }

    void IDisposable.Dispose()
    {
        this.signalBus.TryUnsubscribe<LevelCompletedSignal>(this.OnLevelCompleted);
    }
}
```
