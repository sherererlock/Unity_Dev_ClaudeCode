# 数据控制器模式

## ⚠️ 关键规则：切勿直接访问数据

**❌ 错误：直接数据访问**
```csharp
// ❌ 切勿这样做
var level = uiTemplateUserLevelData.CurrentLevel;
var sessionTime = gameSessionData.OpenTime;
var gold = playerData.Gold;
```

**✅ 正确：始终使用控制器**
```csharp
// ✅ 务必这样做
var level = levelDataController.CurrentLevel;
var sessionTime = sessionDataController.OpenTime;
var gold = playerDataController.Gold;
```

## 完整控制器实现

```csharp
namespace TheOneStudio.YourGame.Models.Controllers
{
    using GameFoundation.Signals;
    using TheOneStudio.YourGame.Models.Data;
    using TheOneStudio.YourGame.Blueprints;

    public class UITemplateLevelDataController : IUITemplateControllerData
    {
        #region Dependency Injection

        private readonly UITemplateLevelBlueprint levelBlueprint;
        private readonly UITemplateUserLevelData levelData;
        private readonly SignalBus signalBus;

        [Preserve]
        public UITemplateLevelDataController(
            UITemplateLevelBlueprint levelBlueprint,
            UITemplateUserLevelData levelData,
            SignalBus signalBus)
        {
            this.levelBlueprint = levelBlueprint;
            this.levelData = levelData;
            this.signalBus = signalBus;
        }

        #endregion

        #region Public Properties (Controlled Access)

        public int CurrentLevel => this.levelData.CurrentLevel;

        public int TotalWins => this.levelData.TotalWinCount;

        public float WinRate => this.CalculateWinRate();

        #endregion

        #region Public Methods (Business Logic)

        public void PassCurrentLevel()
        {
            // 更新数据
            this.levelData.CurrentLevel++;
            this.levelData.TotalWinCount++;

            // 向其他系统发送信号
            this.signalBus.Fire(new LevelPassedSignal(this.CurrentLevel));

            // 追踪分析数据
            this.TrackLevelCompletion();
        }

        public void FailCurrentLevel()
        {
            this.levelData.FailCount++;
            this.signalBus.Fire(new LevelFailedSignal(this.CurrentLevel));
        }

        public LevelConfig GetLevelConfig(int levelNumber)
        {
            return this.levelBlueprint.GetConfig(levelNumber);
        }

        #endregion

        #region Private Methods

        private float CalculateWinRate()
        {
            var totalGames = this.levelData.TotalWinCount + this.levelData.FailCount;
            return totalGames > 0
                ? (float)this.levelData.TotalWinCount / totalGames
                : 0f;
        }

        private void TrackLevelCompletion()
        {
            // 分析追踪逻辑
        }

        #endregion
    }
}
```

## 控制器优势

1. **封装性**：数据访问逻辑集中化
   - 所有数据访问都经过一个地方
   - 易于查找和修改业务逻辑

2. **验证**：在一处强制执行业务规则
   - 防止无效的数据状态
   - 强制执行约束（例如：等级不能为负数）

3. **一致性**：所有代码使用相同的访问模式
   - 团队成员使用相同的 API
   - 对如何访问数据没有困惑

4. **事件**：数据变更时控制器发送信号
   - 其他系统可以对数据变更做出反应
   - 系统之间松耦合

5. **测试**：易于为测试模拟控制器
   - 无需真实数据即可测试业务逻辑
   - 为 UI 测试模拟控制器响应

## 分析集成模式

```csharp
public void TrackEvent(string eventName, params (string Key, object Value)[] properties)
{
    this.analyticService.Track(
        new CustomEvent
        {
            EventName = eventName,
            EventProperties = properties
                .Append(("level", this.levelDataController.CurrentLevel))
                .Append(("session_id", this.sessionDataController.SessionId))
                .Append(("timestamp", DateTime.UtcNow))
                .ToDictionary(),
        }
    );
}
```

## 高级控制器模式

### 带缓存的控制器

```csharp
public class PlayerDataController
{
    private readonly PlayerData playerData;
    private readonly SignalBus signalBus;
    private float? cachedPowerLevel; // 缓存的计算值

    public float PowerLevel
    {
        get
        {
            // 如果可用，返回缓存值
            if (this.cachedPowerLevel.HasValue)
                return this.cachedPowerLevel.Value;

            // 计算并缓存
            this.cachedPowerLevel = this.CalculatePowerLevel();
            return this.cachedPowerLevel.Value;
        }
    }

    public void AddExperience(int amount)
    {
        this.playerData.Experience += amount;

        // 数据变更时使缓存失效
        this.cachedPowerLevel = null;

        this.signalBus.Fire(new ExperienceGainedSignal(amount));
    }
}
```

### 带验证的控制器

```csharp
public class CurrencyController
{
    private readonly CurrencyData currencyData;
    private readonly SignalBus signalBus;

    public bool CanAfford(int amount) => this.currencyData.Gold >= amount;

    public bool TrySpend(int amount)
    {
        if (!this.CanAfford(amount))
        {
            this.signalBus.Fire(new InsufficientFundsSignal());
            return false;
        }

        this.currencyData.Gold -= amount;
        this.signalBus.Fire(new CurrencySpentSignal(amount, this.currencyData.Gold));
        return true;
    }

    public void Earn(int amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive", nameof(amount));

        this.currencyData.Gold += amount;
        this.signalBus.Fire(new CurrencyEarnedSignal(amount, this.currencyData.Gold));
    }
}
```
