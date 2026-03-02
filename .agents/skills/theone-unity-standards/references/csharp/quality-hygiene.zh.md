# C# 质量与代码卫生

## 启用可空引用类型

```csharp
// ✅ GOOD: 在 .csproj 中启用可空注解
<PropertyGroup>
    <Nullable>enable</Nullable>
</PropertyGroup>

// 显式声明可空
public string? OptionalName { get; set; } // 可以为 null
public string RequiredName { get; set; } = string.Empty; // 绝不为 null

// ❌ BAD: 忽略可空性
public string Name { get; set; } // 警告：非空属性必须包含非空值
```

## 使用最小访问权限修饰符

```csharp
// ✅ GOOD: 最严格的访问权限
private readonly IService service; // 仅在此类中可访问
internal sealed class Helper { } // 仅在此程序集中可访问
public interface IPublicApi { } // 仅在必要时公开

// ❌ BAD: 全部公开
public IService service; // 不必要的暴露
public class Helper { } // 应该是 internal
```

## 修复所有警告

**规则：** 将警告视为错误。切勿忽略编译器警告。

```csharp
// ✅ GOOD: 修复警告
#pragma warning disable CS0649 // 字段从未赋值 - 修复代码而不是禁用警告

// 更好：实际修复问题
private readonly string name = string.Empty;
```

## 针对错误抛出异常（+ 正确的日志记录）

**关键规则**：抛出异常，而不是记录错误或返回默认值。

**日志记录准则：**
- **TheOne.Logging.ILogger**：用于运行时脚本（信息性日志）
  - ✅ ILogger 内部处理条件编译（不需要 #if 保护）
  - ✅ ILogger 自动处理前缀（不需要 [prefix]）
  - ❌ 切勿在构造函数中记录日志（保持构造函数快速且无副作用）
  - ❌ 移除冗长的日志（只保留必要的日志）
  - ❌ 禁止使用空值条件运算符（DI 保证非空：使用 `this.logger.Debug()` 而不是 `this.logger?.Debug()`）
- **Debug.Log**：仅用于编辑器脚本（#if UNITY_EDITOR）
- **Exceptions**：用于错误（切勿记录错误 - 抛出异常！）

```csharp
// ✅ EXCELLENT: 运行时使用 TheOne.Logging.ILogger
public sealed class GameService
{
    private readonly TheOne.Logging.ILogger logger;

    public GameService(TheOne.Logging.ILogger logger)
    {
        this.logger = logger;
    }

    public Player GetPlayer(string id)
    {
        return players.TryGetValue(id, out var player)
            ? player
            : throw new KeyNotFoundException($"Player not found: {id}");
    }

    public void StartGame()
    {
        this.logger.Info("Game started");
        this.LoadLevel(1);
    }

    private void ProcessGameData()
    {
        this.logger.Debug("Processing critical game data");
    }
}

// ✅ GOOD: 仅在编辑器脚本中使用 Debug.Log
#if UNITY_EDITOR
public class EditorTool
{
    public void ProcessAssets()
    {
        Debug.Log("Processing assets...");
    }
}
#endif

// ❌ WRONG: 条件编译保护（ILogger 会处理这个）
public void StartGame()
{
#if UNITY_EDITOR || DEVELOPMENT_BUILD
    this.logger.Info("Game started");
#endif
}

// ❌ WRONG: 手动前缀（ILogger 会处理这个）
public void StartGame()
{
    this.logger.Info("[GameService] Game started");
}

// ❌ WRONG: 在构造函数中记录日志
public GameService(TheOne.Logging.ILogger logger)
{
    this.logger = logger;
    this.logger.Info("GameService created");
}

// ❌ WRONG: 空值条件运算符（DI 保证非空）
public void StartGame()
{
    this.logger?.Info("Game started");
}

// ❌ WRONG: 冗长且不必要的日志
public void ProcessItem(Item item)
{
    this.logger.Debug("Entering ProcessItem");
    this.logger.Debug($"Item: {item}");
    var result = item.Calculate();
    this.logger.Debug($"Result: {result}");
    return result;
}

// ❌ WRONG: 返回默认值或记录错误
public Player GetPlayer(string id)
{
    if (!players.TryGetValue(id, out var player))
    {
        this.logger.Error("Player not found");
        return null;
    }
    return player;
}

// ❌ WRONG: 在运行时代码中使用 Debug.Log
public void StartGame()
{
    Debug.Log("Game started");
}
```

**为何这很重要：**
- **Exceptions**：强制调用者正确处理错误
- **TheOne.Logging.ILogger**：用于生产环境的结构化日志，具有自动前缀/保护处理
- **Debug.Log**：仅用于编辑器开发，在构建中会被剥离
- **No Constructor Logging**：构造函数应快速且无副作用
- **No Null Checks**：DI 容器保证依赖项非空

## 使用 nameof 代替字符串字面量

```csharp
// ✅ GOOD: 使用 nameof
public void SetName(string value)
{
    ArgumentNullException.ThrowIfNull(value, nameof(value));
    this.name = value;
}

// ❌ BAD: 字符串字面量
public void SetName(string value)
{
    if (value == null) throw new ArgumentNullException("value"); // 重构时可能会破坏
}
```

## 在最深作用域中使用 Using 指令

```csharp
// ✅ GOOD: 在最深作用域中使用 using（方法级别）
public void ProcessPlayerData()
{
    using System.Text.Json;

    var json = JsonSerializer.Serialize(this.playerData);
    File.WriteAllText("player.json", json);
}

public void ProcessEnemyData()
{
    // 这里不需要 JsonSerializer，没有 using 指令
    var data = this.enemyData.ToString();
}

// ❌ BAD: 仅在一个方法中使用时却在文件级别使用 using
using System.Text.Json; // 文件级别 - 仅在 ProcessPlayerData 中使用

namespace MyGame
{
    public class DataProcessor
    {
        public void ProcessPlayerData()
        {
            var json = JsonSerializer.Serialize(this.playerData);
        }
    }
}
```

**好处：**
- 减少命名空间污染
- 使每个作用域的依赖关系明确
- 更容易重构和移动代码

## 使用 readonly 和 const

```csharp
// ✅ GOOD: 对不重新赋值的字段使用 readonly
private readonly IService service;
private readonly List<string> names = new();

// ✅ GOOD: 对常量使用 const
private const int MaxPlayers = 10;
private const string DefaultName = "Player";

// ❌ BAD: 不需要时可变
private IService service; // 应该是 readonly
private int maxPlayers = 10; // 应该是 const
```

## 添加尾随逗号

```csharp
// ✅ GOOD: 尾随逗号（更容易进行 diff）
var player = new Player
{
    Name = "Alice",
    Score = 100,
    Level = 5, // ← 尾随逗号
};

var items = new[]
{
    "sword",
    "shield",
    "potion", // ← 尾随逗号
};

// ❌ BAD: 没有尾随逗号
var player = new Player
{
    Name = "Alice",
    Score = 100,
    Level = 5 // 很难添加新属性
};
```

## 禁止行内注释

```csharp
// ✅ GOOD: 描述性名称，无行内注释
var activeEnemiesInRange = enemies
    .Where(e => e.IsActive)
    .Where(e => Vector3.Distance(e.Position, playerPosition) < attackRange)
    .ToList();

// ❌ BAD: 行内注释
var enemies = allEnemies.Where(e => e.IsActive) // 过滤活跃敌人
    .Where(e => Vector3.Distance(e.Position, playerPosition) < 10f) // 检查范围
    .ToList(); // 转换为列表
```
