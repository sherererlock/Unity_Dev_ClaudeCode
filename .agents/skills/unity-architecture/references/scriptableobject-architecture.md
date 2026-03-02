# ScriptableObject Architecture - Advanced Patterns

Advanced ScriptableObject patterns for data-driven Unity architecture.

## Runtime Sets

Track active instances of a type:

```csharp
[CreateAssetMenu(menuName = "Sets/Enemy Set")]
public class EnemyRuntimeSet : ScriptableObject
{
    private readonly List<Enemy> items = new List<Enemy>();

    public IReadOnlyList<Enemy> Items => items;

    public void Add(Enemy enemy)
    {
        if (!items.Contains(enemy))
            items.Add(enemy);
    }

    public void Remove(Enemy enemy)
    {
        items.Remove(enemy);
    }

    public void Clear()
    {
        items.Clear();
    }
}

public class Enemy : MonoBehaviour
{
    [SerializeField] private EnemyRuntimeSet runtimeSet;

    private void OnEnable()
    {
        runtimeSet.Add(this);
    }

    private void OnDisable()
    {
        runtimeSet.Remove(this);
    }
}

// Access all enemies
public class TargetingSystem : MonoBehaviour
{
    [SerializeField] private EnemyRuntimeSet allEnemies;

    private Enemy FindNearestEnemy()
    {
        Enemy nearest = null;
        float minDist = float.MaxValue;

        foreach (var enemy in allEnemies.Items)
        {
            float dist = Vector3.Distance(transform.position, enemy.transform.position);
            if (dist < minDist)
            {
                minDist = dist;
                nearest = enemy;
            }
        }

        return nearest;
    }
}
```

## Scriptable Object Variables with History

```csharp
public abstract class VariableWithHistory<T> : ScriptableObject
{
    [SerializeField] private T initialValue;
    private T runtimeValue;
    private Stack<T> history = new Stack<T>();

    public T Value
    {
        get => runtimeValue;
        set
        {
            history.Push(runtimeValue);
            runtimeValue = value;
            OnValueChanged?.Invoke(runtimeValue);
        }
    }

    public event Action<T> OnValueChanged;

    public void ResetToInitial()
    {
        Value = initialValue;
        history.Clear();
    }

    public bool CanUndo => history.Count > 0;

    public void Undo()
    {
        if (CanUndo)
        {
            runtimeValue = history.Pop();
            OnValueChanged?.Invoke(runtimeValue);
        }
    }

    private void OnEnable()
    {
        runtimeValue = initialValue;
        history.Clear();
    }
}
```

## Scriptable Object Events with Parameters

```csharp
public abstract class GameEventBase<T> : ScriptableObject
{
    private readonly List<IGameEventListener<T>> listeners = new List<IGameEventListener<T>>();

    public void Raise(T param)
    {
        for (int i = listeners.Count - 1; i >= 0; i--)
        {
            listeners[i].OnEventRaised(param);
        }
    }

    public void RegisterListener(IGameEventListener<T> listener)
    {
        if (!listeners.Contains(listener))
            listeners.Add(listener);
    }

    public void UnregisterListener(IGameEventListener<T> listener)
    {
        listeners.Remove(listener);
    }
}

[CreateAssetMenu(menuName = "Events/Int Event")]
public class IntGameEvent : GameEventBase<int> { }

public interface IGameEventListener<T>
{
    void OnEventRaised(T param);
}

public class IntGameEventListener : MonoBehaviour, IGameEventListener<int>
{
    [SerializeField] private IntGameEvent gameEvent;
    [SerializeField] private UnityEvent<int> response;

    private void OnEnable() => gameEvent.RegisterListener(this);
    private void OnDisable() => gameEvent.UnregisterListener(this);

    public void OnEventRaised(int param)
    {
        response.Invoke(param);
    }
}
```

## Configuration ScriptableObjects

```csharp
[CreateAssetMenu(menuName = "Config/Game Config")]
public class GameConfig : ScriptableObject
{
    [Header("Player")]
    public float playerMoveSpeed = 5f;
    public int playerMaxHealth = 100;

    [Header("Enemies")]
    public float enemySpawnRate = 2f;
    public int maxEnemies = 20;

    [Header("Balance")]
    public AnimationCurve difficultyCurve;
    public int[] levelScoreThresholds;

    public int GetScoreThreshold(int level)
    {
        return level < levelScoreThresholds.Length ? levelScoreThresholds[level] : int.MaxValue;
    }
}

// Access globally
public class ConfigManager : MonoBehaviour
{
    [SerializeField] private GameConfig config;
    public static GameConfig Config { get; private set; }

    private void Awake()
    {
        Config = config;
    }
}

// Use anywhere
public class Player : MonoBehaviour
{
    private float moveSpeed;

    private void Start()
    {
        moveSpeed = ConfigManager.Config.playerMoveSpeed;
    }
}
```

## Best Practices

✅ DO:
- Use runtime sets for tracking active instances
- Store configuration in ScriptableObjects
- Implement events with ScriptableObjects for designer control
- Reset ScriptableObject runtime values on scene load

❌ DON'T:
- Reference scene objects from ScriptableObjects
- Modify ScriptableObject values in builds (runtime changes lost)
- Create circular SO references
- Store large amounts of runtime data in SOs
