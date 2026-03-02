# VContainer 依赖注入模式

## 完整服务注册模式

```csharp
using VContainer;
using VContainer.Unity;

public class GameLifetimeScope : LifetimeScope
{
    protected override void Configure(IContainerBuilder builder)
    {
        // 注册单例服务
        builder.Register<AnalyticService>(Lifetime.Singleton)
            .AsImplementedInterfaces();

        // 注册为接口和自身
        builder.Register<GameplayService>(Lifetime.Singleton)
            .AsInterfacesAndSelf();

        // 注册并自动初始化
        builder.RegisterEntryPoint<LevelService>();

        // 注册控制器
        builder.Register<UITemplateLevelDataController>(Lifetime.Singleton);

        // 注册场景中的组件
        builder.RegisterComponentInHierarchy<ToastController>();

        // 在新预制体中注册组件
        builder.RegisterComponentInNewPrefabResource<PlayerController>(
            nameof(PlayerController),
            Lifetime.Singleton
        ).UnderTransform(this.transform);
    }
}
```

## 完整服务实现模式

```csharp
namespace TheOneStudio.YourFeature.Services
{
    using GameFoundation.Signals;
    using VContainer.Unity;

    public sealed class YourService : IInitializable, IDisposable
    {
        #region Dependency Injection

        private readonly SignalBus signalBus;
        private readonly IAnalyticServices analyticService;
        private readonly UITemplateDataController dataController;

        [Preserve]
        public YourService(
            SignalBus signalBus,
            IAnalyticServices analyticService,
            UITemplateDataController dataController)
        {
            this.signalBus = signalBus;
            this.analyticService = analyticService;
            this.dataController = dataController;
        }

        #endregion

        #region IInitializable Implementation

        void IInitializable.Initialize()
        {
            // 订阅信号
            this.signalBus.Subscribe<WonSignal>(this.OnWon);
            this.signalBus.Subscribe<LostSignal>(this.OnLost);

            // 初始化服务逻辑
            this.LoadConfiguration();
        }

        #endregion

        #region Signal Handlers

        private void OnWon(WonSignal signal)
        {
            // 处理胜利事件
            var currentLevel = this.dataController.CurrentLevel;
            this.analyticService.Track("level_won", ("level", currentLevel));
        }

        private void OnLost(LostSignal signal)
        {
            // 处理失败事件
        }

        #endregion

        #region IDisposable Implementation

        void IDisposable.Dispose()
        {
            // 始终取消订阅信号
            this.signalBus.TryUnsubscribe<WonSignal>(this.OnWon);
            this.signalBus.TryUnsubscribe<LostSignal>(this.OnLost);
        }

        #endregion

        private void LoadConfiguration()
        {
            // 服务初始化逻辑
        }
    }
}
```

## 分离 DI 程序集模式

**✅ 推荐结构：**

```
YourFeature/
├── Runtime/
│   ├── YourFeature.asmdef           # 核心逻辑（无 DI 框架）
│   │   References: ["TheOne.Logging"]
│   │
│   └── DI/
│       └── YourFeature.VContainer.asmdef  # DI 集成
│           References: [
│               "YourFeature",
│               "VContainer",
│               "VContainer.Unity"
│           ]
```

**核心程序集（框架无关）：**
```json
{
    "name": "DynamicUserDifficulty",
    "rootNamespace": "TheOneStudio.DynamicUserDifficulty",
    "references": [
        "TheOne.Logging"
    ]
}
```

**DI 程序集（VContainer 集成）：**
```json
{
    "name": "DynamicUserDifficulty.VContainer",
    "rootNamespace": "TheOneStudio.DynamicUserDifficulty.DI",
    "references": [
        "DynamicUserDifficulty",
        "VContainer",
        "VContainer.Unity"
    ]
}
```

**优势：**
- 核心逻辑与 DI 框架无关
- 可在不触及核心代码的情况下更换 DI 框架
- 关注点分离清晰
- 更好的可测试性

## 条件功能注册

```csharp
public class GameLifetimeScope : LifetimeScope
{
    protected override void Configure(IContainerBuilder builder)
    {
        // 核心系统（始终注册）
        builder.RegisterGameFoundation(this.transform);
        builder.RegisterGameplay();

        // 条件功能
        #if DIFFICULTY_SYSTEM
        this.RegisterDifficultySystem(builder);
        #endif

        #if ANALYTICS_ENABLED
        this.RegisterAnalytics(builder);
        #endif
    }

    #if DIFFICULTY_SYSTEM
    private void RegisterDifficultySystem(IContainerBuilder builder)
    {
        var config = Resources.Load<DifficultyConfig>("Configs/DifficultyConfig");
        if (config == null)
        {
            Debug.LogWarning("[Difficulty] Config not found, skipping registration");
            return;
        }

        builder.RegisterDifficultyIntegration();

        #if UNITY_EDITOR || DEVELOPMENT_BUILD
        // 仅在编辑器/开发构建中显示调试 UI
        builder.RegisterComponentOnNewGameObject<DifficultyDebugUI>(Lifetime.Singleton)
            .DontDestroyOnLoad();
        #endif
    }
    #endif
}
```
