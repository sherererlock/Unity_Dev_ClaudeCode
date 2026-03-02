# UniTask 异步/等待模式

VContainer 和 TheOne.DI 项目都使用 UniTask 进行异步操作。

## 必需模式

- 使用 `async UniTaskVoid` 或 `UniTask.Void(async () => {})` 代替 `async void`
- 如果不 `await` `UniTask`，请使用 `.Forget()`
- 为异步方法使用 `CancellationToken`
- 为异步方法添加 `Async` 后缀
- 使用 `UniTask.WaitForSeconds` 代替 `UniTask.Delay`

## 异步方法模式

### ✅ 推荐：async UniTask

```csharp
public async UniTask DoWorkAsync(CancellationToken cancellationToken)
{
    await UniTask.WaitForSeconds(1f, cancellationToken: cancellationToken);

    var workResult = await this.ProcessDataAsync(cancellationToken);
    return workResult;
}
```

### ✅ 推荐：用于“射后不理”的 async UniTaskVoid

```csharp
public async UniTaskVoid FireAndForgetAsync()
{
    await UniTask.Delay(100);

    // 执行不需要等待的工作
    Debug.Log("Background work");
}
```

### ✅ 推荐：不等待时使用 Forget()

```csharp
public void StartWork()
{
    // “射后不理”且不创建 async void
    this.DoWorkAsync(default).Forget();
}
```

### ❌ 错误：async void

```csharp
// 糟糕：async void
public async void DoWork() // ❌ 不要使用 async void
{
    await UniTask.Delay(100);
}
```

## UniTask 计时

### ✅ 推荐：WaitForSeconds

```csharp
// 使用 WaitForSeconds 实现 Unity 友好的延迟
await UniTask.WaitForSeconds(1f, cancellationToken: token);
```

### ❌ 错误：Delay（Unity 友好度较低）

```csharp
// 糟糕：Delay 的 Unity 友好度较低
await UniTask.Delay(1000, cancellationToken: token);
```

## 完整服务示例

```csharp
public sealed class LoadingService : IAsyncEarlyLoadable, IDisposable
{
    private readonly IAssetsManager assetsManager;
    private CancellationTokenSource? cts;
    private GameObject? loadedPrefab;

    [Inject] // 或者 VContainer 使用 [Preserve]
    public LoadingService(IAssetsManager assetsManager)
    {
        this.assetsManager = assetsManager;
    }

    public async UniTask LoadAsync(IProgress<float> progress, CancellationToken cancellationToken)
    {
        this.cts = CancellationTokenSource.CreateLinkedTokenSource(cancellationToken);

        try
        {
            // 加载资产并报告进度
            this.loadedPrefab = await this.assetsManager.LoadAsync<GameObject>(
                "path/to/prefab",
                this.cts.Token
            );

            progress?.Report(0.5f);

            // 等待初始化
            await UniTask.WaitForSeconds(1f, cancellationToken: this.cts.Token);

            progress?.Report(1f);
        }
        catch (OperationCanceledException)
        {
            // 处理取消
            Debug.Log("Loading cancelled");
        }
    }

    public void Dispose()
    {
        // 取消正在进行的操作
        this.cts?.Cancel();
        this.cts?.Dispose();

        // 卸载资产
        this.assetsManager.Unload(this.loadedPrefab);
    }
}
```

## 高级 UniTask 模式

### 超时模式

```csharp
public async UniTask<bool> TryLoadWithTimeoutAsync(float timeoutSeconds)
{
    var cts = new CancellationTokenSource();

    try
    {
        // 创建超时任务
        var timeoutTask = UniTask.Delay(
            TimeSpan.FromSeconds(timeoutSeconds),
            cancellationToken: cts.Token
        );

        // 创建加载任务
        var loadTask = this.LoadAsync(cts.Token);

        // 等待第一个完成
        var result = await UniTask.WhenAny(loadTask, timeoutTask);

        if (result == 0)
        {
            // 加载完成
            return true;
        }
        else
        {
            // 超时
            Debug.LogWarning("Load timeout");
            return false;
        }
    }
    finally
    {
        cts.Cancel();
        cts.Dispose();
    }
}
```

### 并行加载模式

```csharp
public async UniTask LoadMultipleAsync(CancellationToken cancellationToken)
{
    // 并行开始多个加载
    var task1 = this.assetsManager.LoadAsync<GameObject>("prefab1", cancellationToken);
    var task2 = this.assetsManager.LoadAsync<Texture2D>("texture1", cancellationToken);
    var task3 = this.assetsManager.LoadAsync<AudioClip>("audio1", cancellationToken);

    // 等待所有完成
    await UniTask.WhenAll(task1, task2, task3);

    // 所有资产已加载
    Debug.Log("All assets loaded");
}
```

### 重试模式

```csharp
public async UniTask<T> LoadWithRetryAsync<T>(string path, int maxRetries, CancellationToken cancellationToken)
    where T : UnityEngine.Object
{
    for (int i = 0; i < maxRetries; i++)
    {
        try
        {
            return await this.assetsManager.LoadAsync<T>(path, cancellationToken);
        }
        catch (Exception ex)
        {
            Debug.LogWarning($"Load failed (attempt {i + 1}/{maxRetries}): {ex.Message}");

            if (i < maxRetries - 1)
            {
                // 重试前等待
                await UniTask.WaitForSeconds(1f, cancellationToken: cancellationToken);
            }
            else
            {
                // 最后一次重试失败，重新抛出
                throw;
            }
        }
    }

    throw new InvalidOperationException("This should never be reached");
}
```

## 反射用的 [Preserve]

两个框架都要求通过反射实例化的类使用 `[Preserve]`：

```csharp
// 将 [Preserve] 用于：
// - DI 容器注册
// - JSON 反序列化
// - CSV 反序列化
// - 任何基于反射的实例化

public sealed class PlayerData
{
    [Preserve]
    public PlayerData() { } // JSON 反序列化器使用的构造函数
}
```

## MonoBehaviour UniTask 集成

```csharp
public class GameManager : MonoBehaviour
{
    private CancellationTokenSource? cts;

    private async void Start()
    {
        // 创建绑定到 GameObject 生命周期的取消令牌
        this.cts = new CancellationTokenSource();

        // 使用 GetCancellationTokenOnDestroy() 进行自动清理
        await this.InitializeAsync(this.GetCancellationTokenOnDestroy());
    }

    private async UniTask InitializeAsync(CancellationToken cancellationToken)
    {
        // 异步初始化
        await UniTask.WaitForSeconds(1f, cancellationToken: cancellationToken);

        Debug.Log("Initialized");
    }

    private void OnDestroy()
    {
        // 取消正在进行的操作
        this.cts?.Cancel();
        this.cts?.Dispose();
    }
}
```

## 错误处理

```csharp
public async UniTask LoadSafelyAsync(CancellationToken cancellationToken)
{
    try
    {
        await this.LoadAsync(cancellationToken);
    }
    catch (OperationCanceledException)
    {
        // 优雅地处理取消
        Debug.Log("Load cancelled");
    }
    catch (Exception ex)
    {
        // 处理其他错误
        Debug.LogError($"Load failed: {ex.Message}");
        throw; // 如果需要则重新抛出
    }
}
```
