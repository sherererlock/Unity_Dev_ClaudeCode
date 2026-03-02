# Unity 管理器模式 - 参考

用于在 Unity 中实现和组织管理器系统的详细模式。

## 单例模式变体

### 线程安全单例

```csharp
public class ThreadSafeSingleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T instance;
    private static readonly object lockObject = new object();

    public static T Instance
    {
        get
        {
            lock (lockObject)
            {
                if (instance == null)
                {
                    instance = FindObjectOfType<T>();
                    if (instance == null)
                    {
                        GameObject obj = new GameObject(typeof(T).Name);
                        instance = obj.AddComponent<T>();
                        DontDestroyOnLoad(obj);
                    }
                }
                return instance;
            }
        }
    }
}
```

### 懒加载单例

```csharp
public class LazySingleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static readonly Lazy<T> lazyInstance = new Lazy<T>(() =>
    {
        var obj = new GameObject(typeof(T).Name);
        var instance = obj.AddComponent<T>();
        DontDestroyOnLoad(obj);
        return instance;
    });

    public static T Instance => lazyInstance.Value;
}
```

## 管理器初始化

### 引导场景

```csharp
[DefaultExecutionOrder(-100)]
public class GameBootstrap : MonoBehaviour
{
    [SerializeField] private bool initializeOnAwake = true;

    private void Awake()
    {
        if (initializeOnAwake)
            Initialize();
    }

    private void Initialize()
    {
        // 按特定顺序初始化管理器
        InitializeCore();
        InitializeSystems();
        InitializeGameplay();
    }

    private void InitializeCore()
    {
        var settings = GameSettings.Instance;
        var save = SaveManager.Instance;
    }

    private void InitializeSystems()
    {
        var audio = AudioManager.Instance;
        var input = InputManager.Instance;
    }

    private void InitializeGameplay()
    {
        var game = GameManager.Instance;
    }
}
```

## 服务定位器

### 高级服务定位器

```csharp
public interface IService { }

public class ServiceLocator
{
    private static ServiceLocator instance;
    public static ServiceLocator Instance => instance ?? (instance = new ServiceLocator());

    private readonly Dictionary<Type, IService> services = new Dictionary<Type, IService>();

    public void Register<T>(T service) where T : IService
    {
        var type = typeof(T);
        if (services.ContainsKey(type))
        {
            Debug.LogWarning($"Service {type} already registered. Replacing.");
        }
        services[type] = service;
    }

    public T Get<T>() where T : IService
    {
        var type = typeof(T);
        if (services.TryGetValue(type, out var service))
        {
            return (T)service;
        }
        throw new InvalidOperationException($"Service {type} not found");
    }

    public bool TryGet<T>(out T service) where T : IService
    {
        var type = typeof(T);
        if (services.TryGetValue(type, out var foundService))
        {
            service = (T)foundService;
            return true;
        }
        service = default;
        return false;
    }

    public void Unregister<T>() where T : IService
    {
        services.Remove(typeof(T));
    }

    public void Clear()
    {
        services.Clear();
    }
}
```

## 管理器通信

### 基于事件的管理器通信

```csharp
public class GameManager : Singleton<GameManager>
{
    public event Action OnGameStarted;
    public event Action OnGamePaused;
    public event Action OnGameEnded;

    public void StartGame()
    {
        OnGameStarted?.Invoke();
    }
}

public class AudioManager : Singleton<AudioManager>
{
    protected override void Awake()
    {
        base.Awake();
        GameManager.Instance.OnGameStarted += OnGameStarted;
        GameManager.Instance.OnGameEnded += OnGameEnded;
    }

    private void OnDestroy()
    {
        if (GameManager.Instance != null)
        {
            GameManager.Instance.OnGameStarted -= OnGameStarted;
            GameManager.Instance.OnGameEnded -= OnGameEnded;
        }
    }

    private void OnGameStarted() => PlayMusic("GameMusic");
    private void OnGameEnded() => StopMusic();
}
```

## 管理器最佳实践

✅ 建议做：
- 对真正的全局、单实例系统使用单例
- 按受控顺序初始化管理器
- 管理器之间通过事件进行通信
- 保持管理器专注于单一职责

❌ 不建议做：
- 为所有事物创建单例
- 允许管理器直接调用彼此的方法
- 创建形成循环的管理器依赖关系
- 将游戏玩法逻辑放入管理器中（应使用控制器代替）
