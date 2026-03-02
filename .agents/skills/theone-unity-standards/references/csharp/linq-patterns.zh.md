# LINQ 模式

## 1. 使用 LINQ 代替冗长的循环

### ❌ 避免：冗长的循环
```csharp
// 糟糕：带有临时列表的手动循环
List<Enemy> activeEnemies = new List<Enemy>();
foreach (var enemy in allEnemies)
{
    if (enemy.IsActive)
    {
        activeEnemies.Add(enemy);
    }
}

// 糟糕：用于计数的循环
int count = 0;
foreach (var item in items)
{
    if (item.IsValid)
    {
        count++;
    }
}

// 糟糕：用于转换的循环
List<string> names = new List<string>();
foreach (var player in players)
{
    names.Add(player.Name);
}
```

### ✅ 推荐：LINQ
```csharp
// 良好：使用 Where 过滤
var activeEnemies = allEnemies.Where(e => e.IsActive).ToList();

// 良好：使用 Count 计数
var count = items.Count(item => item.IsValid);

// 良好：使用 Select 转换
var names = players.Select(p => p.Name).ToList();

// 良好：复杂查询
var topScorers = players
    .Where(p => p.Score > 1000)
    .OrderByDescending(p => p.Score)
    .Take(10)
    .Select(p => new { p.Name, p.Score })
    .ToList();
```

## 2. 使用扩展方法代替工具类

### ❌ 避免：静态工具类
```csharp
// 糟糕：带有静态方法的工具类
public static class StringUtility
{
    public static bool IsNullOrEmpty(string value)
    {
        return string.IsNullOrEmpty(value);
    }

    public static string Capitalize(string value)
    {
        if (string.IsNullOrEmpty(value)) return value;
        return char.ToUpper(value[0]) + value.Substring(1);
    }
}

// 用法（糟糕）：
var result = StringUtility.Capitalize(text);
```

### ✅ 推荐：扩展方法
```csharp
// 良好：扩展方法
public static class StringExtensions
{
    public static bool IsNullOrEmpty(this string value)
    {
        return string.IsNullOrEmpty(value);
    }

    public static string Capitalize(this string value)
    {
        return string.IsNullOrEmpty(value)
            ? value
            : char.ToUpper(value[0]) + value.Substring(1);
    }

    public static string OrDefault(this string value, string defaultValue = "")
    {
        return value.IsNullOrEmpty() ? defaultValue : value;
    }
}

// 用法（良好）：
var result = text.Capitalize();
var name = playerName.OrDefault("Unknown");
```

## 7. 使用 var 进行类型推断

### ❌ 避免：冗余的类型声明
```csharp
// 糟糕：两侧都有冗余类型
Dictionary<string, List<Player>> playerGroups = new Dictionary<string, List<Player>>();
List<Enemy> enemies = new List<Enemy>();
Player player = new Player();
```

### ✅ 推荐：使用 var 提高清晰度
```csharp
// 良好：当类型明显时使用 var
var playerGroups = new Dictionary<string, List<Player>>();
var enemies = new List<Enemy>();
var player = new Player();

// 良好：var 与 LINQ 配合使用
var activeEnemies = allEnemies.Where(e => e.IsActive).ToList();

// 当类型不明显时，明确指定仍然是可以的
IEnumerable<Player> query = GetPlayers(); // 如果接口类型需要，这样是可以的
```

## LINQ 性能优化

### 使用 .ToArray() 对比 .ToList()

```csharp
// ✅ 良好：不修改时使用 ToArray()
public IReadOnlyList<Enemy> GetActiveEnemies()
{
    return enemies.Where(e => e.IsActive).ToArray(); // 只读，无需修改
}

// ✅ 良好：修改时使用 ToList()
public List<Enemy> GetModifiableEnemies()
{
    var list = enemies.Where(e => e.IsActive).ToList();
    list.Add(newEnemy); // 将被修改
    return list;
}
```

### 使用只读集合接口

```csharp
// ✅ 良好：无需修改时使用只读接口
public IReadOnlyList<string> Names { get; }
public IReadOnlyCollection<Player> Players { get; }
public IReadOnlyDictionary<string, int> Scores { get; }

// ❌ 糟糕：只读足够时使用可变接口
public List<string> Names { get; } // 允许外部修改
public Dictionary<string, int> Scores { get; } // 暴露可变性
```

### 避免不必要的枚举

```csharp
// ✅ 良好：如果不需要则不要枚举
var query = items.Where(i => i.IsValid); // IEnumerable，尚未求值
if (needsList)
{
    return query.ToList(); // 仅在需要时枚举
}
return query;

// ❌ 糟糕：过早枚举
var list = items.Where(i => i.IsValid).ToList(); // 总是分配
if (needsList)
{
    return list;
}
return list; // 如果不需要枚举则浪费分配
```
