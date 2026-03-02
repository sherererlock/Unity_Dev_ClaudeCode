# Unity 特有代码审查

## 组件访问

### 检查清单
- [ ] 是否使用 `TryGetComponent` 代替 `GetComponent` 加空值检查？
- [ ] 是否在 Unity 方法中使用了空值条件运算符？

### 常见违规

#### ❌ 带空值检查的 GetComponent
```csharp
// 错误：带空值检查的 GetComponent
var enemy = gameObject.GetComponent<Enemy>();
if (enemy != null)
{
    enemy.TakeDamage(10);
}

// 错误：方法调用前的空值检查
var weapon = GetComponentInChildren<Weapon>();
if (weapon != null)
{
    weapon.Fire();
}
```

### ✅ 简洁的组件访问
```csharp
// 正确：TryGetComponent
if (gameObject.TryGetComponent<Enemy>(out var enemy))
{
    enemy.TakeDamage(10);
}

// 正确：空值条件运算符
GetComponentInChildren<Weapon>()?.Fire();
```

## MonoBehaviour 生命周期

### 检查清单
- [ ] Unity 生命周期方法的顺序是否正确？
- [ ] 是否在 OnDestroy() 中进行了适当的清理？

### ✅ 正确顺序
```csharp
public class MyBehaviour : MonoBehaviour
{
    // Awake → OnEnable → Start → Update → LateUpdate → OnDisable → OnDestroy

    private void Awake()
    {
        // 初始化引用
    }

    private void OnEnable()
    {
        // 订阅事件
    }

    private void Start()
    {
        // 初始化游戏逻辑
    }

    private void Update()
    {
        // 每帧逻辑
    }

    private void LateUpdate()
    {
        // 所有 Update 调用之后
    }

    private void OnDisable()
    {
        // 暂停/禁用逻辑
    }

    private void OnDestroy()
    {
        // 清理订阅，取消事件订阅
    }
}
```

## 组件缓存

### 常见违规

#### ❌ Update 中的 GetComponent
```csharp
// 错误：每帧调用 GetComponent
private void Update()
{
    var rigidbody = GetComponent<Rigidbody>();
    rigidbody.AddForce(Vector3.up);
}
```

### ✅ 缓存组件
```csharp
// 正确：在 Awake 中缓存
private Rigidbody rb;

private void Awake()
{
    this.rb = GetComponent<Rigidbody>();
}

private void Update()
{
    this.rb.AddForce(Vector3.up);
}
```

## SerializeField 与 Public

### 常见违规

#### ❌ Inspector 的 Public 字段
```csharp
// 错误：Public 字段
public float speed = 5f;
public GameObject prefab;
```

### ✅ 带 Private 的 SerializeField
```csharp
// 正确：带 Private 的 SerializeField
[SerializeField] private float speed = 5f;
[SerializeField] private GameObject prefab;
```

## 协程与 UniTask

### 常见违规

#### ❌ 用于异步的协程
```csharp
// 错误：协程
private IEnumerator LoadAsync()
{
    yield return new WaitForSeconds(1f);
    Debug.Log("Loaded");
}

private void Start()
{
    StartCoroutine(LoadAsync());
}
```

### ✅ UniTask
```csharp
// 正确：UniTask
private async UniTask LoadAsync(CancellationToken cancellationToken)
{
    await UniTask.WaitForSeconds(1f, cancellationToken: cancellationToken);
    Debug.Log("Loaded");
}

private async void Start()
{
    await LoadAsync(this.GetCancellationTokenOnDestroy());
}
```

## Find 和 FindObjectOfType

### 常见违规

#### ❌ Update 中的 Find
```csharp
// 错误：每帧调用 Find
private void Update()
{
    var player = GameObject.Find("Player");
    var enemy = FindObjectOfType<Enemy>();
}
```

### ✅ 查找一次或使用依赖注入
```csharp
// 正确：在 Awake 中查找一次
private GameObject player;
private Enemy enemy;

private void Awake()
{
    this.player = GameObject.Find("Player");
    this.enemy = FindObjectOfType<Enemy>();
}

// 更好：使用依赖注入
private readonly Player player;

[Preserve]
public GameController(Player player)
{
    this.player = player;
}
```

## 销毁与释放

### 检查清单
- [ ] Unity 对象是否被正确销毁？
- [ ] 托管资源是否被释放？

### 常见违规

#### ❌ 未销毁 GameObject
```csharp
// 错误：未销毁
private void RemoveEnemy()
{
    this.enemy = null; // ❌ GameObject 仍然存在于场景中
}
```

### ✅ 适当的清理
```csharp
// 正确：销毁 GameObject
private void RemoveEnemy()
{
    if (this.enemy != null)
    {
        Destroy(this.enemy.gameObject);
        this.enemy = null;
    }
}

// 正确：释放托管资源
public void Dispose()
{
    this.subscription?.Dispose();
    this.cts?.Cancel();
    this.cts?.Dispose();
}
```

## Vector3 和 Quaternion

### 常见违规

#### ❌ 创建新向量
```csharp
// 错误：创建新向量
transform.position = new Vector3(0, 0, 0);
transform.rotation = new Quaternion(0, 0, 0, 1);
```

### ✅ 使用静态属性
```csharp
// 正确：使用静态属性
transform.position = Vector3.zero;
transform.rotation = Quaternion.identity;
```

## Layer 和 Tag 比较

### 常见违规

#### ❌ 字符串比较
```csharp
// 错误：字符串比较
if (gameObject.tag == "Player")
{
    // ...
}
```

### ✅ CompareTag
```csharp
// 正确：CompareTag
if (gameObject.CompareTag("Player"))
{
    // ...
}
```

## 完整示例

### ❌ 糟糕的 Unity 代码
```csharp
public class EnemyController : MonoBehaviour
{
    // ❌ Public 字段
    public float speed = 5f;
    public GameObject projectilePrefab;

    private void Update()
    {
        // ❌ 每帧调用 GetComponent
        var rb = GetComponent<Rigidbody>();

        // ❌ 每帧调用 Find
        var player = GameObject.Find("Player");

        // ❌ 字符串 Tag 比较
        if (player.tag == "Player")
        {
            // ❌ 创建新的 Vector3
            var direction = new Vector3(1, 0, 0);
            rb.AddForce(direction * this.speed);
        }

        // ❌ 带空值检查的 GetComponent
        var weapon = GetComponent<Weapon>();
        if (weapon != null)
        {
            weapon.Fire();
        }
    }
}
```

### ✅ 优秀的 Unity 代码
```csharp
public class EnemyController : MonoBehaviour
{
    // ✅ SerializeField
    [SerializeField] private float speed = 5f;
    [SerializeField] private GameObject projectilePrefab;

    // ✅ 缓存组件
    private Rigidbody rb;
    private Transform playerTransform;
    private Weapon weapon;

    private void Awake()
    {
        // ✅ 缓存组件一次
        this.rb = GetComponent<Rigidbody>();

        // ✅ 查找一次
        var player = GameObject.Find("Player");
        this.playerTransform = player?.transform;

        // ✅ TryGetComponent
        TryGetComponent<Weapon>(out this.weapon);
    }

    private void Update()
    {
        if (this.playerTransform == null) return;

        // ✅ CompareTag
        if (this.playerTransform.CompareTag("Player"))
        {
            // ✅ 使用 Vector3.right
            this.rb.AddForce(Vector3.right * this.speed);
        }

        // ✅ 空值条件运算符
        this.weapon?.Fire();
    }
}
```

## 审查严重程度

### 🔴 严重问题
- Update 中的 GetComponent/Find (性能问题)
- 未销毁 GameObject (内存泄漏)
- 未释放托管资源 (内存泄漏)

### 🟡 重要问题
- 使用 Public 字段代替 SerializeField
- 使用带空值检查的 GetComponent 代替 TryGetComponent
- 创建新的 Vector3/Quaternion 代替使用静态属性
- 字符串 Tag 比较代替 CompareTag
- 未缓存组件

### 🟢 建议
- 可以使用空值条件运算符
- 可以使用 UniTask 代替协程
- 可以使用依赖注入代替 Find
