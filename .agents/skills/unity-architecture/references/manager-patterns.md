# Unity Manager Patterns - Reference

Detailed patterns for implementing and organizing manager systems in Unity.

## Singleton Pattern Variants

### Thread-Safe Singleton

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

### Lazy Singleton

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

## Manager Initialization

### Bootstrap Scene

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
        // Initialize managers in specific order
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

## Service Locator

### Advanced Service Locator

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

## Manager Communication

### Event-Based Manager Communication

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

## Manager Best Practices

✅ DO:
- Use singletons for truly global, single-instance systems
- Initialize managers in controlled order
- Communicate between managers via events
- Keep managers focused on single responsibility

❌ DON'T:
- Create singletons for everything
- Allow managers to directly call each other's methods
- Create manager dependencies that form cycles
- Put gameplay logic in managers (use controllers instead)
