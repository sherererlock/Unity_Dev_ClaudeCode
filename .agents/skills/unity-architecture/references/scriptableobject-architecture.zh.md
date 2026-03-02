# ScriptableObject 架构 - 高级模式

用于数据驱动 Unity 架构的高级 ScriptableObject 模式。

## 运行时集合 (Runtime Sets)

追踪某一类型的活动实例：

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

// 访问所有敌人
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

## 带历史记录的 ScriptableObject 变量

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

## 带参数的 ScriptableObject 事件

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

## 配置类 ScriptableObjects

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

// 全局访问
public class ConfigManager : MonoBehaviour
{
    [SerializeField] private GameConfig config;
    public static GameConfig Config { get; private set; }

    private void Awake()
    {
        Config = config;
    }
}

// 随处使用
public class Player : MonoBehaviour
{
    private float moveSpeed;

    private void Start()
    {
        moveSpeed = ConfigManager.Config.playerMoveSpeed;
    }
}
```

## 最佳实践

✅ 建议 (DO)：
- 使用运行时集合来追踪活动实例
- 将配置存储在 ScriptableObject 中
- 使用 ScriptableObject 实现事件，以便设计师进行控制
- 在场景加载时重置 ScriptableObject 的运行时数值

❌ 禁止 (DON'T)：
- 从 ScriptableObject 引用场景对象
- 在构建版本中修改 ScriptableObject 的值（运行时更改会丢失）
- 创建循环的 SO 引用
- 在 SO 中存储大量运行时数据
