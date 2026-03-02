# C# 代码质量审查

## LINQ 与手动循环

### 检查清单
- [ ] 在适当的情况下，是否已将手动循环替换为 LINQ？
- [ ] LINQ 是否易读且不过于复杂？

### 常见违规

#### ❌ 冗长的手动循环
```csharp
// 差：手动循环
List<Enemy> activeEnemies = new List<Enemy>();
foreach (var enemy in allEnemies)
{
    if (enemy.IsActive && enemy.Health > 0)
    {
        activeEnemies.Add(enemy);
    }
}

// 差：手动计数
int count = 0;
foreach (var player in players)
{
    if (player.Score > 100)
    {
        count++;
    }
}
```

### ✅ 简洁的 LINQ
```csharp
// 好：LINQ
var activeEnemies = allEnemies
    .Where(e => e.IsActive && e.Health > 0)
    .ToList();

// 好：计数
var count = players.Count(p => p.Score > 100);
```

## 表达式主体成员

### 检查清单
- [ ] 简单方法是否使用了 `=>`？
- [ ] 简单属性是否使用了 `=>`？

### 常见违规

#### ❌ 冗长的完整主体
```csharp
// 差：单行代码使用完整方法主体
public int GetHealth()
{
    return this.currentHealth;
}

public string Name
{
    get { return this.name; }
}
```

### ✅ 简洁的表达式主体
```csharp
// 好：表达式主体
public int GetHealth() => this.currentHealth;

public string Name => this.name;
```

## 空值处理

### 检查清单
- [ ] 是否使用 `??` 代替了冗长的空值检查？
- [ ] 是否使用 `?.` 进行空值条件访问？
- [ ] 是否使用 `??=` 进行延迟初始化？

### 常见违规

#### ❌ 冗长的空值检查
```csharp
// 差：冗长的空值检查
string result;
if (playerName != null)
    result = playerName;
else
    result = "Unknown";

// 差：嵌套的空值检查
if (player != null)
{
    if (player.Weapon != null)
    {
        damage = player.Weapon.Damage;
    }
}

// 差：延迟初始化
if (this.cache == null)
{
    this.cache = new Dictionary<string, int>();
}
```

### ✅ 简洁的空值处理
```csharp
// 好：空值合并
var result = playerName ?? "Unknown";

// 好：空值条件
var damage = player?.Weapon?.Damage ?? 0;

// 好：空值合并赋值
this.cache ??= new Dictionary<string, int>();
```

## 模式匹配

### 检查清单
- [ ] 是否使用模式匹配代替了类型检查？
- [ ] 在适当的情况下是否使用了 switch 表达式？

### 常见违规

#### ❌ 旧式类型检查
```csharp
// 差：带转换的类型检查
if (obj is Player)
{
    Player player = (Player)obj;
    player.TakeDamage(10);
}

// 差：带类型的 Switch
switch (obj.GetType().Name)
{
    case "Player":
        ((Player)obj).Attack();
        break;
}
```

### ✅ 现代模式匹配
```csharp
// 好：模式匹配
if (obj is Player player)
{
    player.TakeDamage(10);
}

// 好：Switch 表达式
var action = obj switch
{
    Player p => p.Attack(),
    Enemy e => e.Attack(),
    _ => 0
};
```

## 集合初始化

### 常见违规

#### ❌ 冗长的初始化
```csharp
// 差：冗长的集合初始化
var list = new List<string>();
list.Add("Item1");
list.Add("Item2");
list.Add("Item3");

// 差：冗长的字典初始化
var dict = new Dictionary<string, int>();
dict.Add("Health", 100);
dict.Add("Mana", 50);
```

### ✅ 简洁的初始化
```csharp
// 好：集合初始化器
var list = new List<string> { "Item1", "Item2", "Item3" };

// 好：字典初始化器
var dict = new Dictionary<string, int>
{
    ["Health"] = 100,
    ["Mana"] = 50
};
```

## 字符串插值

### 常见违规

#### ❌ 字符串连接
```csharp
// 差：字符串连接
var message = "Player " + playerName + " scored " + score + " points";

// 差：String.Format
var message = string.Format("Player {0} scored {1} points", playerName, score);
```

### ✅ 字符串插值
```csharp
// 好：字符串插值
var message = $"Player {playerName} scored {score} points";

// 好：带格式化
var message = $"Time: {time:F2}s, Score: {score:N0}";
```

## var 用法

### 检查清单
- [ ] 当类型从右侧显而易见时，是否使用了 `var`？

### 常见违规

#### ❌ 显而易见时使用显式类型
```csharp
// 差：类型显而易见
List<Enemy> enemies = new List<Enemy>();
Dictionary<string, int> scores = new Dictionary<string, int>();
PlayerController controller = GetComponent<PlayerController>();
```

### ✅ 显而易见时使用 var
```csharp
// 好：使用 var
var enemies = new List<Enemy>();
var scores = new Dictionary<string, int>();
var controller = GetComponent<PlayerController>();
```

## 扩展方法

### 常见违规

#### ❌ 工具类
```csharp
// 差：静态工具类
public static class VectorUtils
{
    public static bool IsNearZero(Vector3 vector)
    {
        return vector.magnitude < 0.01f;
    }
}

// 用法
if (VectorUtils.IsNearZero(velocity))
```

### ✅ 扩展方法
```csharp
// 好：扩展方法
public static class VectorExtensions
{
    public static bool IsNearZero(this Vector3 vector)
    {
        return vector.magnitude < 0.01f;
    }
}

// 用法
if (velocity.IsNearZero())
```

## 完整示例

### ❌ 冗长的代码（多个问题）
```csharp
public class PlayerManager
{
    private Dictionary<string, Player> players;

    public List<Player> GetActivePlayers()
    {
        List<Player> result = new List<Player>();
        foreach (var kvp in this.players)
        {
            Player player = kvp.Value;
            if (player != null && player.IsActive)
            {
                result.Add(player);
            }
        }
        return result;
    }

    public string GetPlayerName(string id)
    {
        Player player = null;
        if (this.players.ContainsKey(id))
        {
            player = this.players[id];
        }

        if (player != null)
        {
            return player.Name;
        }
        else
        {
            return "Unknown";
        }
    }
}
```

### ✅ 简洁的代码（已修复）
```csharp
public class PlayerManager
{
    private Dictionary<string, Player> players = new();

    public List<Player> GetActivePlayers() =>
        this.players.Values
            .Where(p => p.IsActive)
            .ToList();

    public string GetPlayerName(string id) =>
        this.players.TryGetValue(id, out var player)
            ? player.Name
            : "Unknown";
}
```

## 审查严重性

### 🟡 重要问题
- 使用手动循环而不是 LINQ
- 使用冗长的空值检查而不是空值运算符
- 简单成员未使用表达式主体
- 未使用模式匹配
- 类型显而易见时未使用 var

### 🟢 建议
- 可以使用集合初始化器
- 可以使用字符串插值
- 可以使用扩展方法而不是工具类
- 可以使用 switch 表达式
