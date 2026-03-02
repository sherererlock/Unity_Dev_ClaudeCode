---
name: unity-ecs-patterns
description: 掌握 Unity ECS（实体组件系统），结合 DOTS、Jobs 和 Burst 进行高性能游戏开发。适用于构建面向数据的游戏、优化性能或处理大量实体时使用。
---

# Unity ECS 模式

Unity 面向数据技术栈 (DOTS) 的生产级模式，包括实体组件系统 (ECS)、作业系统 (Job System) 和 Burst 编译器。

## 何时使用此技能

- 构建高性能 Unity 游戏
- 高效管理数千个实体
- 实现面向数据的游戏系统
- 优化 CPU 密集型游戏逻辑
- 将 OOP 游戏代码转换为 ECS
- 使用 Jobs 和 Burst 进行并行化

## 核心概念

### 1. ECS 与 OOP

| 方面 | 传统 OOP | ECS/DOTS |
| :--- | :--- | :--- |
| 数据布局 | 面向对象 | 面向数据 |
| 内存 | 分散 | 连续 |
| 处理方式 | 逐对象 | 批量处理 |
| 扩展性 | 随数量增加性能差 | 线性扩展 |
| 最适合 | 复杂行为 | 大规模模拟 |

### 2. DOTS 组件

```
Entity（实体）: 轻量级 ID（无数据）
Component（组件）: 纯数据（无行为）
System（系统）: 处理组件的逻辑
World（世界）: 实体的容器
Archetype（原型）: 组件的唯一组合
Chunk（块）: 相同原型实体的内存块
```

## 模式

### 模式 1：基础 ECS 设置

```csharp
using Unity.Entities;
using Unity.Mathematics;
using Unity.Transforms;
using Unity.Burst;
using Unity.Collections;

// Component: 纯数据，无方法
public struct Speed : IComponentData
{
    public float Value;
}

public struct Health : IComponentData
{
    public float Current;
    public float Max;
}

public struct Target : IComponentData
{
    public Entity Value;
}

// Tag component (零大小标记)
public struct EnemyTag : IComponentData { }
public struct PlayerTag : IComponentData { }

// Buffer component (可变大小数组)
[InternalBufferCapacity(8)]
public struct InventoryItem : IBufferElementData
{
    public int ItemId;
    public int Quantity;
}

// Shared component (分组实体)
public struct TeamId : ISharedComponentData
{
    public int Value;
}
```

### 模式 2：使用 ISystem 的系统（推荐）

```csharp
using Unity.Entities;
using Unity.Transforms;
using Unity.Mathematics;
using Unity.Burst;

// ISystem: 非托管，兼容 Burst，最高性能
[BurstCompile]
public partial struct MovementSystem : ISystem
{
    [BurstCompile]
    public void OnCreate(ref SystemState state)
    {
        // 系统运行前需要组件存在
        state.RequireForUpdate<Speed>();
    }

    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        float deltaTime = SystemAPI.Time.DeltaTime;

        // 简单 foreach - 自动生成作业
        foreach (var (transform, speed) in
            SystemAPI.Query<RefRW<LocalTransform>, RefRO<Speed>>())
        {
            transform.ValueRW.Position +=
                new float3(0, 0, speed.ValueRO.Value * deltaTime);
        }
    }

    [BurstCompile]
    public void OnDestroy(ref SystemState state) { }
}

// 使用显式作业以获得更多控制
[BurstCompile]
public partial struct MovementJobSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        var job = new MoveJob
        {
            DeltaTime = SystemAPI.Time.DeltaTime
        };

        state.Dependency = job.ScheduleParallel(state.Dependency);
    }
}

[BurstCompile]
public partial struct MoveJob : IJobEntity
{
    public float DeltaTime;

    void Execute(ref LocalTransform transform, in Speed speed)
    {
        transform.Position += new float3(0, 0, speed.Value * DeltaTime);
    }
}
```

### 模式 3：实体查询

```csharp
[BurstCompile]
public partial struct QueryExamplesSystem : ISystem
{
    private EntityQuery _enemyQuery;

    public void OnCreate(ref SystemState state)
    {
        // 为复杂情况手动构建查询
        _enemyQuery = new EntityQueryBuilder(Allocator.Temp)
            .WithAll<EnemyTag, Health, LocalTransform>()
            .WithNone<Dead>()
            .WithOptions(EntityQueryOptions.FilterWriteGroup)
            .Build(ref state);
    }

    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        // SystemAPI.Query - 最简单的方法
        foreach (var (health, entity) in
            SystemAPI.Query<RefRW<Health>>()
                .WithAll<EnemyTag>()
                .WithEntityAccess())
        {
            if (health.ValueRO.Current <= 0)
            {
                // 标记为销毁
                SystemAPI.GetSingleton<EndSimulationEntityCommandBufferSystem.Singleton>()
                    .CreateCommandBuffer(state.WorldUnmanaged)
                    .DestroyEntity(entity);
            }
        }

        // 获取数量
        int enemyCount = _enemyQuery.CalculateEntityCount();

        // 获取所有实体
        var enemies = _enemyQuery.ToEntityArray(Allocator.Temp);

        // 获取组件数组
        var healths = _enemyQuery.ToComponentDataArray<Health>(Allocator.Temp);
    }
}
```

### 模式 4：实体命令缓冲区 (结构性更改)

```csharp
// 结构性更改（创建/销毁/添加/移除）需要命令缓冲区
[BurstCompile]
[UpdateInGroup(typeof(SimulationSystemGroup))]
public partial struct SpawnSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        var ecbSingleton = SystemAPI.GetSingleton<BeginSimulationEntityCommandBufferSystem.Singleton>();
        var ecb = ecbSingleton.CreateCommandBuffer(state.WorldUnmanaged);

        foreach (var (spawner, transform) in
            SystemAPI.Query<RefRW<Spawner>, RefRO<LocalTransform>>())
        {
            spawner.ValueRW.Timer -= SystemAPI.Time.DeltaTime;

            if (spawner.ValueRO.Timer <= 0)
            {
                spawner.ValueRW.Timer = spawner.ValueRO.Interval;

                // 创建实体（推迟到同步点）
                Entity newEntity = ecb.Instantiate(spawner.ValueRO.Prefab);

                // 设置组件值
                ecb.SetComponent(newEntity, new LocalTransform
                {
                    Position = transform.ValueRO.Position,
                    Rotation = quaternion.identity,
                    Scale = 1f
                });

                // 添加组件
                ecb.AddComponent(newEntity, new Speed { Value = 5f });
            }
        }
    }
}

// 并行 ECB 使用
[BurstCompile]
public partial struct ParallelSpawnJob : IJobEntity
{
    public EntityCommandBuffer.ParallelWriter ECB;

    void Execute([EntityIndexInQuery] int index, in Spawner spawner)
    {
        Entity e = ECB.Instantiate(index, spawner.Prefab);
        ECB.AddComponent(index, e, new Speed { Value = 5f });
    }
}
```

### 模式 5：Aspect (组合组件)

```csharp
using Unity.Entities;
using Unity.Transforms;
using Unity.Mathematics;

// Aspect: 将相关组件组合在一起，使代码更整洁
public readonly partial struct CharacterAspect : IAspect
{
    public readonly Entity Entity;

    private readonly RefRW<LocalTransform> _transform;
    private readonly RefRO<Speed> _speed;
    private readonly RefRW<Health> _health;

    // 可选组件
    [Optional]
    private readonly RefRO<Shield> _shield;

    // 缓冲区
    private readonly DynamicBuffer<InventoryItem> _inventory;

    public float3 Position
    {
        get => _transform.ValueRO.Position;
        set => _transform.ValueRW.Position = value;
    }

    public float CurrentHealth => _health.ValueRO.Current;
    public float MaxHealth => _health.ValueRO.Max;
    public float MoveSpeed => _speed.ValueRO.Value;

    public bool HasShield => _shield.IsValid;
    public float ShieldAmount => HasShield ? _shield.ValueRO.Amount : 0f;

    public void TakeDamage(float amount)
    {
        float remaining = amount;

        if (HasShield && _shield.ValueRO.Amount > 0)
        {
            // 护盾首先吸收伤害
            remaining = math.max(0, amount - _shield.ValueRO.Amount);
        }

        _health.ValueRW.Current = math.max(0, _health.ValueRO.Current - remaining);
    }

    public void Move(float3 direction, float deltaTime)
    {
        _transform.ValueRW.Position += direction * _speed.ValueRO.Value * deltaTime;
    }

    public void AddItem(int itemId, int quantity)
    {
        _inventory.Add(new InventoryItem { ItemId = itemId, Quantity = quantity });
    }
}

// 在系统中使用 Aspect
[BurstCompile]
public partial struct CharacterSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        float dt = SystemAPI.Time.DeltaTime;

        foreach (var character in SystemAPI.Query<CharacterAspect>())
        {
            character.Move(new float3(1, 0, 0), dt);

            if (character.CurrentHealth < character.MaxHealth * 0.5f)
            {
                // 低生命值逻辑
            }
        }
    }
}
```

### 模式 6：单例组件

```csharp
// Singleton: 只有一个实体拥有此组件
public struct GameConfig : IComponentData
{
    public float DifficultyMultiplier;
    public int MaxEnemies;
    public float SpawnRate;
}

public struct GameState : IComponentData
{
    public int Score;
    public int Wave;
    public float TimeRemaining;
}

// 在世界创建时创建单例
public partial struct GameInitSystem : ISystem
{
    public void OnCreate(ref SystemState state)
    {
        var entity = state.EntityManager.CreateEntity();
        state.EntityManager.AddComponentData(entity, new GameConfig
        {
            DifficultyMultiplier = 1.0f,
            MaxEnemies = 100,
            SpawnRate = 2.0f
        });
        state.EntityManager.AddComponentData(entity, new GameState
        {
            Score = 0,
            Wave = 1,
            TimeRemaining = 120f
        });
    }
}

// 在系统中访问单例
[BurstCompile]
public partial struct ScoreSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        // 读取单例
        var config = SystemAPI.GetSingleton<GameConfig>();

        // 写入单例
        ref var gameState = ref SystemAPI.GetSingletonRW<GameState>().ValueRW;
        gameState.TimeRemaining -= SystemAPI.Time.DeltaTime;

        // 检查是否存在
        if (SystemAPI.HasSingleton<GameConfig>())
        {
            // ...
        }
    }
}
```

### 模式 7：Baking (转换 GameObject)

```csharp
using Unity.Entities;
using UnityEngine;

// Authoring component (编辑器中的 MonoBehaviour)
public class EnemyAuthoring : MonoBehaviour
{
    public float Speed = 5f;
    public float Health = 100f;
    public GameObject ProjectilePrefab;

    class Baker : Baker<EnemyAuthoring>
    {
        public override void Bake(EnemyAuthoring authoring)
        {
            var entity = GetEntity(TransformUsageFlags.Dynamic);

            AddComponent(entity, new Speed { Value = authoring.Speed });
            AddComponent(entity, new Health
            {
                Current = authoring.Health,
                Max = authoring.Health
            });
            AddComponent(entity, new EnemyTag());

            if (authoring.ProjectilePrefab != null)
            {
                AddComponent(entity, new ProjectilePrefab
                {
                    Value = GetEntity(authoring.ProjectilePrefab, TransformUsageFlags.Dynamic)
                });
            }
        }
    }
}

// 具有依赖关系的复杂 Baking
public class SpawnerAuthoring : MonoBehaviour
{
    public GameObject[] Prefabs;
    public float Interval = 1f;

    class Baker : Baker<SpawnerAuthoring>
    {
        public override void Bake(SpawnerAuthoring authoring)
        {
            var entity = GetEntity(TransformUsageFlags.Dynamic);

            AddComponent(entity, new Spawner
            {
                Interval = authoring.Interval,
                Timer = 0f
            });

            // Bake 预制件缓冲区
            var buffer = AddBuffer<SpawnPrefabElement>(entity);
            foreach (var prefab in authoring.Prefabs)
            {
                buffer.Add(new SpawnPrefabElement
                {
                    Prefab = GetEntity(prefab, TransformUsageFlags.Dynamic)
                });
            }

            // 声明依赖关系
            DependsOn(authoring.Prefabs);
        }
    }
}
```

### 模式 8：使用原生集合的 Jobs

```csharp
using Unity.Jobs;
using Unity.Collections;
using Unity.Burst;
using Unity.Mathematics;

[BurstCompile]
public struct SpatialHashJob : IJobParallelFor
{
    [ReadOnly]
    public NativeArray<float3> Positions;

    // 线程安全的哈希映射写入
    public NativeParallelMultiHashMap<int, int>.ParallelWriter HashMap;

    public float CellSize;

    public void Execute(int index)
    {
        float3 pos = Positions[index];
        int hash = GetHash(pos);
        HashMap.Add(hash, index);
    }

    int GetHash(float3 pos)
    {
        int x = (int)math.floor(pos.x / CellSize);
        int y = (int)math.floor(pos.y / CellSize);
        int z = (int)math.floor(pos.z / CellSize);
        return x * 73856093 ^ y * 19349663 ^ z * 83492791;
    }
}

[BurstCompile]
public partial struct SpatialHashSystem : ISystem
{
    private NativeParallelMultiHashMap<int, int> _hashMap;

    public void OnCreate(ref SystemState state)
    {
        _hashMap = new NativeParallelMultiHashMap<int, int>(10000, Allocator.Persistent);
    }

    public void OnDestroy(ref SystemState state)
    {
        _hashMap.Dispose();
    }

    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        var query = SystemAPI.QueryBuilder()
            .WithAll<LocalTransform>()
            .Build();

        int count = query.CalculateEntityCount();

        // 需要时调整大小
        if (_hashMap.Capacity < count)
        {
            _hashMap.Capacity = count * 2;
        }

        _hashMap.Clear();

        // 获取位置
        var positions = query.ToComponentDataArray<LocalTransform>(Allocator.TempJob);
        var posFloat3 = new NativeArray<float3>(count, Allocator.TempJob);

        for (int i = 0; i < count; i++)
        {
            posFloat3[i] = positions[i].Position;
        }

        // 构建哈希映射
        var hashJob = new SpatialHashJob
        {
            Positions = posFloat3,
            HashMap = _hashMap.AsParallelWriter(),
            CellSize = 10f
        };

        state.Dependency = hashJob.Schedule(count, 64, state.Dependency);

        // 清理
        positions.Dispose(state.Dependency);
        posFloat3.Dispose(state.Dependency);
    }
}
```

## 性能提示

```csharp
// 1. 随处使用 Burst
[BurstCompile]
public partial struct MySystem : ISystem { }

// 2. 优先使用 IJobEntity 而非手动迭代
[BurstCompile]
partial struct OptimizedJob : IJobEntity
{
    void Execute(ref LocalTransform transform) { }
}

// 3. 尽可能并行调度
state.Dependency = job.ScheduleParallel(state.Dependency);

// 4. 使用 ScheduleParallel 进行块迭代
[BurstCompile]
partial struct ChunkJob : IJobChunk
{
    public ComponentTypeHandle<Health> HealthHandle;

    public void Execute(in ArchetypeChunk chunk, int unfilteredChunkIndex,
        bool useEnabledMask, in v128 chunkEnabledMask)
    {
        var healths = chunk.GetNativeArray(ref HealthHandle);
        for (int i = 0; i < chunk.Count; i++)
        {
            // 处理
        }
    }
}

// 5. 避免在热路径中进行结构性更改
// 使用可启用组件代替添加/移除
public struct Disabled : IComponentData, IEnableableComponent { }
```

## 最佳实践

### 宜 (Do's)

- **使用 ISystem 而非 SystemBase** - 性能更好
- **Burst 编译所有内容** - 巨大的速度提升
- **批量处理结构性更改** - 使用 ECB (实体命令缓冲区)
- **使用 Profiler 分析** - 识别瓶颈
- **使用 Aspects** - 清晰的组件分组

### 忌 (Don'ts)

- **不要使用托管类型** - 会破坏 Burst 兼容性
- **不要在作业中进行结构性更改** - 使用 ECB
- **不要过度架构** - 从简单开始
- **不要忽略块利用率** - 对相似实体进行分组
- **不要忘记释放** - 原生集合会内存泄漏

## 资源

- [Unity DOTS 文档](https://docs.unity3d.com/Packages/com.unity.entities@latest)
- [Unity DOTS 示例](https://github.com/Unity-Technologies/EntityComponentSystemSamples)
- [Burst 用户指南](https://docs.unity3d.com/Packages/com.unity.burst@latest)
