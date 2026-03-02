# Unity 预制件工作流 - 完整指南

Unity 预制件综合指南：涵盖创建、变体、嵌套、覆盖、编程使用以及可维护游戏开发的最佳实践。

## 预制件基础

### 什么是预制件 (Prefabs)？

预制件是存储为资产的可重用游戏对象 (GameObject) 模板。对预制件资产的更改会传播到所有实例。

**关键概念：**
- **预制件资产 (Prefab Asset)**：存储在项目窗口中的模板（.prefab 文件）
- **预制件实例 (Prefab Instance)**：场景中预制件的副本（层级视图中显示为蓝色文本）
- **实例覆盖 (Instance Override)**：对特定实例的修改（显示为粗体蓝色文本）
- **预制件变体 (Prefab Variant)**：继承自另一个预制件的预制件

### 为什么要使用预制件？

1.  **可重用性**：一次创建，多次使用
2.  **一致性**：更改会传播到所有实例
3.  **效率**：比复制游戏对象更快
4.  **组织性**：模块化、可维护的资产
5.  **运行时生成**：在运行时实例化

## 创建预制件

### 基本预制件创建

将游戏对象从层级视图 (Hierarchy) 拖到项目窗口 (Project window)：

```
层级视图:                 项目窗口:
Enemy GameObject -> Assets/Prefabs/Enemy.prefab
```

**视觉指示：**
- **原始游戏对象**：在层级视图中变为蓝色
- **预制件资产**：在项目窗口中显示为蓝色立方体图标

### 通过代码创建预制件

```csharp
#if UNITY_EDITOR
using UnityEditor;

public class PrefabCreator
{
    [MenuItem("Tools/Create Prefab")]
    private static void CreatePrefab()
    {
        GameObject selected = Selection.activeGameObject;
        if (selected == null)
            return;

        string path = "Assets/Prefabs/" + selected.name + ".prefab";
        PrefabUtility.SaveAsPrefabAsset(selected, path);
    }
}
#endif
```

### 解包预制件 (Unpacking Prefabs)

断开预制件连接以独立编辑：

**右键点击预制件实例：**
- **Unpack Prefab (解包预制件)**：断开此实例的连接（保留在层级视图中）
- **Unpack Prefab Completely (完全解包预制件)**：解包包括嵌套预制件在内的所有内容

**通过代码：**
```csharp
#if UNITY_EDITOR
GameObject instance = PrefabUtility.InstantiatePrefab(prefab) as GameObject;
PrefabUtility.UnpackPrefabInstance(instance, PrefabUnpackMode.Completely, InteractionMode.AutomatedAction);
#endif
```

## 预制件实例

### 实例覆盖 (Instance Overrides)

修改预制件实例时，值会显示为 **粗体蓝色**：

**覆盖类型：**
- 变换更改（位置、旋转、缩放）
- 组件属性更改
- 添加的组件
- 移除的组件
- 游戏对象层级更改

**管理覆盖：**
- **Revert (还原)**：丢弃覆盖，匹配预制件
- **Apply (应用)**：将覆盖推送到预制件资产（影响所有实例）
- **Apply to Prefab Variant (应用到预制件变体)**：使用覆盖创建变体

### 实例覆盖工作流

```
1. 在场景中选择预制件实例
2. 修改属性（变为粗体蓝色）
3. 右键点击属性或使用 Overrides 下拉菜单
4. 选择：
   - Revert: 丢弃更改
   - Apply to Prefab: 更新资产
   - Apply as Override in Variant: 创建变体
```

**查看所有覆盖：**
- 打开检视面板 (Inspector) 顶部的 Overrides 下拉菜单
- 在一处查看所有覆盖
- 选择性地或一次性全部应用/还原

### 自动保存与手动应用

Unity 2020+ 在进入预制件模式时会自动保存预制件更改。对于实例：

**需要手动应用：**
- 实例覆盖不会自动应用到预制件
- 防止意外更改所有实例
- 需要显式的应用操作

## 预制件变体 (Prefab Variants)

### 创建变体

变体继承自基础预制件，并具有特定的覆盖：

**方法 1：从现有预制件**
1. 在项目窗口中右键点击预制件
2. Create > Prefab Variant
3. 按需修改变体

**方法 2：从实例覆盖**
1. 修改预制件实例
2. Overrides 下拉菜单 > Apply as Override in Variant
3. 使用该覆盖创建变体

**层级示例：**
```
Enemy (基础预制件)
├── FastEnemy (变体: moveSpeed = 10)
├── TankEnemy (变体: health = 200)
└── FlyingEnemy (变体: 添加了 FlyingMovement 组件)
```

### 变体继承

对基础预制件的更改会传播到变体：

```
基础 Enemy:
- maxHealth = 100
- moveSpeed = 5

FastEnemy 变体:
- 继承 maxHealth = 100 (来自基础)
- 覆盖 moveSpeed = 10

如果基础将 maxHealth 更改为 150:
- FastEnemy 自动获得 maxHealth = 150
- FastEnemy 保持 moveSpeed = 10 的覆盖
```

**断开继承：**
- 在变体中覆盖属性
- 属性不再随基础更新
- 可以还原覆盖以重新继承

### 变体用例

**角色变化：**
```
Character (基础)
├── PlayerCharacter (变体: 玩家脚本)
├── EnemyCharacter (变体: AI 脚本)
│   ├── MeleeEnemy (变体: 近战组件)
│   └── RangedEnemy (变体: 远程组件)
```

**UI 预制件：**
```
Button (基础)
├── PrimaryButton (变体: 颜色, 更大)
├── SecondaryButton (变体: 不同颜色)
└── DangerButton (变体: 红色, 警告图标)
```

## 嵌套预制件 (Nested Prefabs)

### 预制件组合

预制件可以包含其他预制件：

```
Car (预制件)
├── Wheel (预制件) x4
├── Engine (预制件)
├── Door (预制件) x4
└── Seat (预制件) x4
```

**好处：**
- 模块化设计
- 重用子组件
- 独立编辑嵌套预制件
- 更改正确传播

### 编辑嵌套预制件

**选项 1：直接编辑嵌套预制件**
1. 在项目窗口中双击嵌套预制件
2. 在预制件模式下打开
3. 更改影响该预制件的所有使用

**选项 2：在父预制件内编辑**
1. 在预制件模式下打开父预制件
2. 选择嵌套预制件实例
3. 正常使用 Overrides 系统

**选项 3：选择并打开**
1. 在场景/父级中选择嵌套预制件实例
2. 点击检视面板中的 "Open"
3. 在预制件模式下打开嵌套预制件

### 嵌套预制件覆盖

覆盖递归工作：

```
House (预制件)
└── Door (嵌套预制件)
    └── Handle (嵌套预制件)
        └── material (属性)

场景实例:
House Instance
└── Door Instance (覆盖: color = red)
    └── Handle Instance (覆盖: scale = 1.2)
```

两个覆盖都独立维护。

## 预制件模式 (Prefab Mode)

### 进入预制件模式

**打开预制件进行编辑：**
- 在项目窗口中双击预制件
- 选择实例，点击检视面板中的 "Open Prefab"
- 右键点击实例 > Open Prefab

**视觉指示：**
- 顶部有蓝色面包屑导航栏
- 场景仅显示预制件内容
- 隔离的编辑环境

### 预制件模式的好处

1.  **隔离编辑**：只看预制件，没有场景杂乱
2.  **上下文保留**：预制件位置匹配场景用法
3.  **安全更改**：应用前预览
4.  **自动保存**：退出时自动保存更改

### 退出预制件模式

- 点击面包屑中的场景名称
- 点击面包屑中的箭头
- 关闭标签页（自动保存）

## 编程使用预制件

### 实例化预制件

```csharp
public class PrefabSpawner : MonoBehaviour
{
    [SerializeField] private GameObject prefab;

    private void Start()
    {
        // 基本实例化
        GameObject instance = Instantiate(prefab);

        // 指定位置和旋转
        GameObject instance2 = Instantiate(prefab, transform.position, Quaternion.identity);

        // 指定父级
        GameObject instance3 = Instantiate(prefab, transform);

        // 指定所有参数
        GameObject instance4 = Instantiate(prefab, new Vector3(0, 5, 0), Quaternion.Euler(0, 90, 0), transform);
    }
}
```

### 实例化后配置

```csharp
public class EnemySpawner : MonoBehaviour
{
    [SerializeField] private GameObject enemyPrefab;
    [SerializeField] private Transform spawnPoint;

    private void SpawnEnemy()
    {
        // 实例化
        GameObject enemy = Instantiate(enemyPrefab, spawnPoint.position, Quaternion.identity);

        // 配置实例
        enemy.name = "Enemy_" + Time.time;  // 唯一名称
        enemy.transform.SetParent(transform);  // 父级设为生成器

        // 获取并配置组件
        var enemyAI = enemy.GetComponent<EnemyAI>();
        if (enemyAI != null)
        {
            enemyAI.SetTarget(FindPlayerTransform());
            enemyAI.SetDifficulty(currentDifficulty);
        }

        var health = enemy.GetComponent<Health>();
        if (health != null)
        {
            health.SetMaxHealth(100);
        }
    }

    private Transform FindPlayerTransform()
    {
        return GameObject.FindWithTag("Player")?.transform;
    }

    private int currentDifficulty = 1;
}
```

### 销毁预制件实例

```csharp
// 立即销毁（慎用）
Destroy(enemyInstance);

// 延迟销毁（更适合清理）
Destroy(enemyInstance, 2f);  // 2秒后销毁

// 立即销毁（编辑器和运行时）
DestroyImmediate(enemyInstance);  // 仅在编辑器代码中使用
```

### 预制件池 (Prefab Pooling)

重用实例而不是销毁：

```csharp
public class PrefabPool : MonoBehaviour
{
    [SerializeField] private GameObject prefab;
    [SerializeField] private int initialPoolSize = 10;

    private Queue<GameObject> availablePool = new Queue<GameObject>();
    private List<GameObject> allInstances = new List<GameObject>();

    private void Awake()
    {
        // 预实例化池
        for (int i = 0; i < initialPoolSize; i++)
        {
            CreateNewInstance();
        }
    }

    private GameObject CreateNewInstance()
    {
        GameObject instance = Instantiate(prefab, transform);
        instance.SetActive(false);
        availablePool.Enqueue(instance);
        allInstances.Add(instance);
        return instance;
    }

    public GameObject Get()
    {
        GameObject instance;

        if (availablePool.Count > 0)
        {
            instance = availablePool.Dequeue();
        }
        else
        {
            // 池已耗尽 - 创建新的
            instance = CreateNewInstance();
        }

        instance.SetActive(true);
        return instance;
    }

    public void Return(GameObject instance)
    {
        instance.SetActive(false);
        availablePool.Enqueue(instance);
    }

    private void OnDestroy()
    {
        // 清理所有实例
        foreach (var instance in allInstances)
        {
            if (instance != null)
                Destroy(instance);
        }
    }
}
```

**用法：**
```csharp
public class BulletSpawner : MonoBehaviour
{
    [SerializeField] private PrefabPool bulletPool;

    private void Fire()
    {
        GameObject bullet = bulletPool.Get();
        bullet.transform.position = firePoint.position;

        // 3秒后返回池
        StartCoroutine(ReturnAfterDelay(bullet, 3f));
    }

    private IEnumerator ReturnAfterDelay(GameObject obj, float delay)
    {
        yield return new WaitForSeconds(delay);
        bulletPool.Return(obj);
    }
}
```

## 预制件工作流最佳实践

### 组织结构

**文件夹结构：**
```
Assets/
└── Prefabs/
    ├── Characters/
    │   ├── Player/
    │   └── Enemies/
    ├── Environment/
    │   ├── Props/
    │   └── Buildings/
    ├── UI/
    │   ├── Panels/
    │   └── Buttons/
    └── Effects/
        ├── Particles/
        └── Audio/
```

**命名约定：**
- 使用描述性名称：`EnemyOrc`，而不是 `Enemy1`
- 变体包含基础名称：`ButtonPrimary`，`ButtonSecondary`
- 用于生成的预制件：`BulletProjectile`，`ExplosionEffect`

### 版本控制

**良好实践：**
- 将预制件更改与场景分开提交
- 在提交信息中记录破坏性更改
- 使用预制件变体而不是复制
- 提交前测试预制件更改

**合并冲突：**
- 预制件文件是 YAML（可合并）
- 使用 Unity 的 Smart Merge 工具
- 发生冲突时，预制件优先选择 "theirs"（他们的），场景优先选择 "yours"（你的）

### 测试更改

**应用到预制件前：**
1. 在场景中测试覆盖
2. 验证所有系统仍正常工作
3. 检查依赖的预制件/场景
4. 应用到预制件
5. 测试所有实例

**应用后：**
1. 检查使用该预制件的其他场景
2. 验证预制件变体仍正常工作
3. 测试运行时实例化

### 预制件变体 vs 复制

✅ **使用变体的情况：**
- 具有细微差别的相似对象
- 保持与基础的连接
- 更改应该传播

❌ **不要使用变体的情况：**
- 完全不同的对象
- 临时测试预制件
- 一次性修改

### 预制件引用

**场景引用：**
- 预制件在预制件模式下**可以**引用场景对象
- 当预制件在不同场景中使用时，引用会断开
- 解决方案：在运行时初始化引用

**预制件到预制件引用：**
```csharp
public class Weapon : MonoBehaviour
{
    [SerializeField] private GameObject bulletPrefab;  // ✅ 好的 - 预制件引用

    private void Fire()
    {
        Instantiate(bulletPrefab, firePoint.position, firePoint.rotation);
    }
}
```

**ScriptableObject 引用：**
```csharp
[CreateAssetMenu]
public class WeaponData : ScriptableObject
{
    public int damage;
    public float fireRate;
    public GameObject projectilePrefab;
}

public class Weapon : MonoBehaviour
{
    [SerializeField] private WeaponData data;  // ✅ 好的 - ScriptableObject 引用
}
```

## 高级预制件技术

### 预制件特定脚本

仅在预制件实例中运行的脚本：

```csharp
public class PrefabInitializer : MonoBehaviour
{
    private void Awake()
    {
#if UNITY_EDITOR
        // 检查这是否是预制件资产（不是实例）
        if (UnityEditor.PrefabUtility.IsPartOfPrefabAsset(gameObject))
        {
            // 不在预制件资产中运行初始化
            return;
        }
#endif

        // 初始化预制件实例
        InitializeComponents();
    }

    private void InitializeComponents()
    {
        // 运行时初始化
    }
}
```

### 仅编辑器使用的预制件工具

```csharp
#if UNITY_EDITOR
using UnityEditor;

public class PrefabUtilities
{
    [MenuItem("Tools/Prefabs/Select All Instances")]
    private static void SelectAllInstances()
    {
        GameObject selected = Selection.activeGameObject;
        if (selected == null)
            return;

        GameObject prefabAsset = PrefabUtility.GetCorrespondingObjectFromSource(selected);
        if (prefabAsset == null)
            return;

        // 查找此预制件的所有实例
        var instances = new List<GameObject>();
        foreach (var go in GameObject.FindObjectsOfType<GameObject>())
        {
            if (PrefabUtility.GetCorrespondingObjectFromSource(go) == prefabAsset)
            {
                instances.Add(go);
            }
        }

        Selection.objects = instances.ToArray();
    }

    [MenuItem("Tools/Prefabs/Update All Instances")]
    private static void UpdateAllInstances()
    {
        GameObject prefabAsset = Selection.activeGameObject;
        if (prefabAsset == null || !PrefabUtility.IsPartOfPrefabAsset(prefabAsset))
            return;

        // 查找所有实例并还原覆盖
        foreach (var go in GameObject.FindObjectsOfType<GameObject>())
        {
            if (PrefabUtility.GetCorrespondingObjectFromSource(go) == prefabAsset)
            {
                PrefabUtility.RevertPrefabInstance(go, InteractionMode.AutomatedAction);
            }
        }
    }
}
#endif
```

### 预制件验证

在编辑器中验证预制件结构：

```csharp
#if UNITY_EDITOR
using UnityEditor;

public class PrefabValidator
{
    [MenuItem("Tools/Prefabs/Validate Selected")]
    private static void ValidateSelectedPrefab()
    {
        GameObject selected = Selection.activeGameObject;
        if (selected == null)
        {
            Debug.LogError("No prefab selected");
            return;
        }

        // 检查是否为预制件
        if (!PrefabUtility.IsPartOfPrefabAsset(selected) &&
            PrefabUtility.GetPrefabInstanceStatus(selected) == PrefabInstanceStatus.NotAPrefab)
        {
            Debug.LogError("Selected object is not a prefab");
            return;
        }

        // 验证所需组件
        bool valid = true;

        if (selected.GetComponent<Renderer>() == null)
        {
            Debug.LogWarning("Prefab missing Renderer component");
            valid = false;
        }

        if (selected.GetComponent<Collider>() == null)
        {
            Debug.LogWarning("Prefab missing Collider component");
            valid = false;
        }

        if (valid)
            Debug.Log("Prefab validation passed");
    }
}
#endif
```

## 常见预制件模式

### 工厂模式 (Factory Pattern)

```csharp
public class EnemyFactory : MonoBehaviour
{
    [System.Serializable]
    public class EnemyPrefabData
    {
        public string enemyType;
        public GameObject prefab;
        public int poolSize;
    }

    [SerializeField] private List<EnemyPrefabData> enemyPrefabs;

    private Dictionary<string, PrefabPool> pools = new Dictionary<string, PrefabPool>();

    private void Awake()
    {
        // 为每种敌人类型创建池
        foreach (var data in enemyPrefabs)
        {
            var poolObj = new GameObject($"{data.enemyType}_Pool");
            poolObj.transform.SetParent(transform);

            var pool = poolObj.AddComponent<PrefabPool>();
            // 配置带有预制件和大小的池
            pools[data.enemyType] = pool;
        }
    }

    public GameObject SpawnEnemy(string enemyType, Vector3 position)
    {
        if (pools.TryGetValue(enemyType, out var pool))
        {
            GameObject enemy = pool.Get();
            enemy.transform.position = position;
            return enemy;
        }

        Debug.LogError($"Enemy type {enemyType} not found");
        return null;
    }
}
```

### 预制件注册表 (Prefab Registry)

```csharp
[CreateAssetMenu]
public class PrefabRegistry : ScriptableObject
{
    [System.Serializable]
    public class PrefabEntry
    {
        public string id;
        public GameObject prefab;
    }

    [SerializeField] private List<PrefabEntry> prefabs;

    private Dictionary<string, GameObject> prefabDict;

    public void Initialize()
    {
        prefabDict = new Dictionary<string, GameObject>();
        foreach (var entry in prefabs)
        {
            prefabDict[entry.id] = entry.prefab;
        }
    }

    public GameObject GetPrefab(string id)
    {
        if (prefabDict == null)
            Initialize();

        return prefabDict.TryGetValue(id, out var prefab) ? prefab : null;
    }
}
```

## 故障排除

### 缺少预制件引用

**症状**：预制件引用显示 "None" 或 "Missing"

**原因：**
1. 预制件被删除
2. 预制件被移动/重命名但未更新引用
3. GUID 改变（罕见）

**解决方案：**
1. 找到原始预制件或替代品
2. 重新分配引用
3. 对于多个实例，使用查找和替换引用

### 预制件连接断开

**症状**：实例不再链接到预制件（黑色文本而不是蓝色）

**原因**：预制件资产被删除或移动

**解决方案：**
1. 找到预制件资产并恢复它
2. 或：从实例创建新预制件
3. 或：使用 PrefabUtility.ReconnectToLastPrefab（编辑器）重新连接

### 覆盖未应用

**症状**：应用的覆盖不影响其他实例

**原因：**
1. 应用到了变体而不是基础
2. 其他实例在同一属性上有覆盖
3. 预制件文件未保存

**解决方案：**
1. 检查是否应用到了正确的预制件
2. 还原其他实例上的覆盖
3. 保存项目/场景

### 预制件实例损坏

**症状**：实例行为奇怪，属性重置

**原因**：合并冲突或文件损坏

**解决方案：**
1. 删除实例
2. 从预制件重新实例化
3. 重新应用必要的覆盖

## 最佳实践总结

✅ **建议 (DO):**
- 对所有可重用游戏对象使用预制件
- 为相似对象创建变体
- 在应用到预制件前测试覆盖
- 在逻辑文件夹中组织预制件
- 对频繁生成的对象使用预制件池
- 记录破坏性更改
- 在提交前验证预制件

❌ **不建议 (DON'T):**
- 复制预制件而不是使用变体
- 从预制件引用场景对象
- 将未测试的覆盖应用到预制件
- 创建深层嵌套的预制件层级（>3 层）
- 更改后忘记测试所有实例
- 混合使用预制件和常规游戏对象用于同一目的

遵循这些工作流，在 Unity 项目中实现可维护、高效的预制件使用。
