# 性能优化

## Unity 特有的简洁模式

### 组件访问
```csharp
// 推荐：结合 TryGetComponent 使用空条件运算符
if (gameObject.TryGetComponent<Enemy>(out var enemy))
{
    enemy.TakeDamage(10);
}

// 推荐：结合空条件运算符使用 GetComponentInChildren
transform.GetComponentInChildren<Weapon>()?.Fire();

// 推荐：结合 GetComponentsInChildren 使用 LINQ
var allWeapons = GetComponentsInChildren<Weapon>()
    .Where(w => w.IsActive)
    .ToList();
```

### 向量操作
```csharp
// 推荐：简洁的向量计算
var direction = (target.position - transform.position).normalized;
var distance = Vector3.Distance(a, b);
var midpoint = (a + b) / 2f;

// 推荐：对可空 Transform 使用空条件运算符
var position = targetTransform?.position ?? Vector3.zero;
```

## 何时优先考虑可读性而非简洁性

有时冗长的代码更具可读性。在以下情况下优先选择冗长的写法：

1. **复杂逻辑**：使用中间变量的多步计算更清晰
2. **调试**：临时变量有助于单步调试
3. **团队熟悉度**：初级团队成员可能需要更明确的代码
4. **性能关键**：在热路径中，手动循环可能比 LINQ 更快

```csharp
// 允许：为了复杂逻辑的清晰度而使用冗长写法
var baseScore = player.Kills * 100;
var bonusScore = player.Assists * 50;
var timeBonus = CalculateTimeBonus(player.CompletionTime);
var finalScore = (baseScore + bonusScore + timeBonus) * player.Multiplier;

// 替代难以阅读的单行代码：
var finalScore = (player.Kills * 100 + player.Assists * 50 +
    CalculateTimeBonus(player.CompletionTime)) * player.Multiplier;
```

## LINQ 性能优化

### 使用 .ToArray() vs .ToList()

```csharp
// ✅ 推荐：不进行修改时使用 ToArray()
public IReadOnlyList<Enemy> GetActiveEnemies()
{
    return enemies.Where(e => e.IsActive).ToArray(); // 只读，无需修改
}

// ✅ 推荐：需要修改时使用 ToList()
public List<Enemy> GetModifiableEnemies()
{
    var list = enemies.Where(e => e.IsActive).ToList();
    list.Add(newEnemy); // 将被修改
    return list;
}
```

### 使用只读集合接口

```csharp
// ✅ 推荐：无需修改时使用只读接口
public IReadOnlyList<string> Names { get; }
public IReadOnlyCollection<Player> Players { get; }
public IReadOnlyDictionary<string, int> Scores { get; }

// ❌ 不推荐：只读即足够时使用可变接口
public List<string> Names { get; } // 允许外部修改
public Dictionary<string, int> Scores { get; } // 暴露可变性
```

### 避免不必要的枚举

```csharp
// ✅ 推荐：如果不需要，不要枚举
var query = items.Where(i => i.IsValid); // IEnumerable，尚未求值
if (needsList)
{
    return query.ToList(); // 仅在需要时枚举
}
return query;

// ❌ 不推荐：过早枚举
var list = items.Where(i => i.IsValid).ToList(); // 总是分配内存
if (needsList)
{
    return list;
}
return list; // 如果不需要枚举则浪费分配
```
