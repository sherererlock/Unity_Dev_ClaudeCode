# 现代 C# 特性

⚠️ **Unity 6 兼容性：** 本指南涵盖 Unity 6 中可用的 C# 9 特性。所有示例均已在 Unity 6 (C# 9) 中通过测试并兼容。

**C# 版本支持：**
- Unity 2020.2+: C# 8
- Unity 2021.2+: C# 9
- **Unity 6 (2023.2): C# 9** ✅ (当前版本)

## 3. 使用表达式主体成员 (Expression-Bodied Members)

### ❌ 避免：冗长的方法体
```csharp
// 差：具有完整方法体的单行方法
public int GetHealth()
{
    return this.currentHealth;
}

public bool IsAlive()
{
    return this.currentHealth > 0;
}

// 差：简单的属性获取器
private string name;
public string Name
{
    get { return this.name; }
}
```

### ✅ 推荐：表达式主体成员
```csharp
// 好：表达式主体方法
public int GetHealth() => this.currentHealth;

public bool IsAlive() => this.currentHealth > 0;

// 好：表达式主体属性
public string Name => this.name;

public string FullName => $"{this.firstName} {this.lastName}";

// 好：带有设置器的表达式主体属性
public string DisplayName
{
    get => this.displayName ?? this.name;
    set => this.displayName = value;
}
```

## 4. 使用空合并运算符 (Null-Coalescing Operators)

### ❌ 避免：冗长的空值检查
```csharp
// 差：冗长的空值检查
string result;
if (playerName != null)
{
    result = playerName;
}
else
{
    result = "Unknown";
}

// 差：嵌套的空值检查
if (player != null)
{
    if (player.Weapon != null)
    {
        var damage = player.Weapon.Damage;
    }
}

// 差：手动的空值赋值
if (this.cache == null)
{
    this.cache = new Dictionary<string, object>();
}
```

### ✅ 推荐：空合并运算符
```csharp
// 好：空合并 (??)
var result = playerName ?? "Unknown";

// 好：空条件 (?.)
var damage = player?.Weapon?.Damage ?? 0;

// 好：空合并赋值 (??=)
this.cache ??= new Dictionary<string, object>();

// 好：带方法调用的空合并
var position = transform?.position ?? Vector3.zero;
var count = items?.Count ?? 0;
```

## 5. 使用模式匹配代替类型检查

### ❌ 避免：旧式类型检查
```csharp
// 差：带强制转换的类型检查
if (obj is Player)
{
    Player player = (Player)obj;
    player.TakeDamage(10);
}

// 差：使用 as 运算符的类型检查
Player player = obj as Player;
if (player != null)
{
    player.TakeDamage(10);
}

// 差：带类型检查的 switch 语句
switch (obj.GetType().Name)
{
    case "Player":
        ((Player)obj).TakeDamage(10);
        break;
    case "Enemy":
        ((Enemy)obj).TakeDamage(20);
        break;
}
```

### ✅ 推荐：模式匹配
```csharp
// 好：使用 is 的模式匹配
if (obj is Player player)
{
    player.TakeDamage(10);
}

// 好：带模式匹配的 switch 表达式
var damage = obj switch
{
    Player player => player.TakeDamage(10),
    Enemy enemy => enemy.TakeDamage(20),
    Boss boss => boss.TakeDamage(50),
    _ => 0
};

// 好：属性模式匹配
if (weapon is { Damage: > 100, Rarity: Rarity.Legendary })
{
    ApplyBonusDamage(weapon);
}
```

## 6. 使用集合初始化器 (Unity 6 中的 C# 9)

⚠️ **Unity 6 使用 C# 9：** Unity 6 支持 C# 9，不支持 C# 12。集合表达式 (Collection expressions，C# 12) 不可用。

**Unity 版本对应 C# 版本：**
- Unity 2020.2+: C# 8
- Unity 2021.2+: C# 9
- Unity 6 (2023.2): C# 9 ✅ (当前版本)

请使用在 Unity 6 中完全支持的集合初始化器 (C# 3+)。

### ❌ 避免：冗长的集合初始化
```csharp
// 差：显式集合初始化
List<string> names = new List<string>();
names.Add("Alice");
names.Add("Bob");
names.Add("Charlie");

// 差：数组初始化
int[] numbers = new int[] { 1, 2, 3, 4, 5 };

// 差：字典初始化
Dictionary<string, int> scores = new Dictionary<string, int>();
scores.Add("Alice", 100);
scores.Add("Bob", 200);
```

### ✅ 推荐：集合初始化器 (兼容 C# 9)
```csharp
// 好：集合初始化器
var names = new List<string> { "Alice", "Bob", "Charlie" };

// 好：数组初始化 (简洁)
int[] numbers = { 1, 2, 3, 4, 5 };

// 好：字典初始化器
var scores = new Dictionary<string, int>
{
    { "Alice", 100 },
    { "Bob", 200 },
    { "Charlie", 300 }
};

// 好：目标类型 new (C# 9)
List<string> otherNames = new() { "David", "Eve" };
Dictionary<string, int> otherScores = new()
{
    { "David", 150 },
    { "Eve", 250 }
};
```

## 8. 使用现代 C# 特性

### 数据类记录 (Records)
```csharp
// 好：使用 record 作为不可变数据 (C# 9)
public sealed record PlayerData(string Name, int Score, int Level);

// 好：带验证的记录 (C# 9)
public sealed record WeaponData(string Name, int Damage)
{
    public Rarity Rarity { get; init; } = Rarity.Common;

    public WeaponData(string Name, int Damage) : this(Name, Damage)
    {
        if (Damage < 0) throw new ArgumentException("Damage cannot be negative");
    }
}

// 注意：'required' 关键字是 C# 11+ 特性 (在 Unity 6 中不可用)
// 请使用位置记录或构造函数来强制要求字段
```

### 仅初始化属性 (Init-Only Properties) (C# 9)
```csharp
// 好：用于不可变性的 init-only
public class GameConfig
{
    public string GameName { get; init; } = string.Empty; // 默认值以防止可空警告
    public int MaxPlayers { get; init; }
    public float TimeLimit { get; init; } = 300f;

    // 构造函数强制要求字段 (兼容 C# 9)
    public GameConfig(string gameName, int maxPlayers)
    {
        this.GameName = gameName;
        this.MaxPlayers = maxPlayers;
    }
}

// 用法：
var config = new GameConfig("Battle Arena", 8)
{
    TimeLimit = 600f // 如果需要，覆盖默认 TimeLimit
};
```

### With 表达式 (记录复制)
```csharp
// 好：使用 'with' 进行非破坏性变更
var originalPlayer = new PlayerData("Alice", 100, 5);
var leveledUpPlayer = originalPlayer with { Level = 6, Score = 150 };
```

## 9. 避免不必要的变量

### ❌ 避免：临时变量
```csharp
// 差：不必要的临时变量
var temp = player.GetHealth();
return temp;

// 差：简单转换的中间变量
var enemies = GetAllEnemies();
var activeEnemies = enemies.Where(e => e.IsActive);
return activeEnemies.ToList();
```

### ✅ 推荐：直接返回
```csharp
// 好：直接返回
return player.GetHealth();

// 好：链式操作
return GetAllEnemies()
    .Where(e => e.IsActive)
    .ToList();
```

## 10. 使用解构函数和元组 (Deconstructors and Tuples)

### 用于多重返回值的元组
```csharp
// 好：使用元组返回多个值
public (int health, int mana) GetStats()
{
    return (this.currentHealth, this.currentMana);
}

// 使用解构的用法：
var (health, mana) = player.GetStats();

// 好：命名元组成员
public (int Health, int Mana, int Stamina) GetFullStats()
{
    return (Health: this.health, Mana: this.mana, Stamina: this.stamina);
}
```
