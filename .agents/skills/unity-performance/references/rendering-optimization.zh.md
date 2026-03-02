# Unity 渲染优化 - 完整指南

本指南全面介绍了 Unity 中的 GPU 优化、Draw Call 减少、批处理（Batching）、剔除（Culling）以及着色器（Shader）性能优化。

## Draw Call 基础

### 什么是 Draw Call？

Draw Call 是 CPU 向 GPU 发出的命令，用于渲染具有特定材质设置的网格（Mesh）。

**开销细分：**
- **SetPass Calls**：更改材质/着色器（开销最大）
- **Draw Calls**：渲染网格（如果材质相同，开销较小）

**性能目标：**
- **移动端**：<100 Draw Calls
- **PC/主机**：<500-1000 Draw Calls
- **VR**：每只眼睛 <50-100 Draw Calls

### 查看 Draw Call 统计信息

**Game View（游戏视图） > Stats 面板**：
- SetPass calls
- Batches（批次）
- Saved by batching（通过批处理节省的数量）
- Tris/Verts（三角形/顶点数）

**Frame Debugger（帧调试器）**：Window > Analysis > Frame Debugger
- 显示每个 Draw Call
- 材质更改
- 批处理失败原因

## 静态批处理 (Static Batching)

将静态网格合并为单个 Draw Call。

### 启用静态批处理

**将 GameObject 标记为 Static（静态）：**
1. 选择 GameObject
2. 勾选 "Static" 复选框（Inspector 面板右上角）
3. 或者设置特定的标志："Batching Static"

```csharp
// 通过代码设置静态标志
gameObject.isStatic = true;

// 或者设置特定标志
GameObjectUtility.SetStaticEditorFlags(gameObject, StaticEditorFlags.BatchingStatic);
```

### 静态批处理要求

**所有网格必须共享：**
- 相同的材质实例
- 相同的光照贴图（Lightmap）
- 相同的额外顶点流

**不会被批处理的情况：**
- 不同的材质
- 不同的光照贴图索引
- 蒙皮网格（Skinned meshes）
- 动态（移动）对象

### 静态批处理的局限性

**优点：**
- 显著减少 Draw Calls
- 无运行时开销
- 适用于光照贴图

**缺点：**
- 内存使用量增加（合并后的网格）
- 无法移动已批处理的对象
- 如果对象移动会破坏批处理

**适用场景：**
- 环境道具
- 静态建筑物
- 背景元素

## 动态批处理 (Dynamic Batching)

在运行时自动批处理小型动态网格。

### 动态批处理要求

**网格必须满足：**
- **顶点数**：<300 顶点（Unity 2020+），<900（旧版本）
- **相同材质**：共享材质实例
- **相同缩放**：均匀或非均匀（但必须一致）
- **无阴影**：投射阴影的物体不会被批处理

```csharp
// 动态批处理自动发生
// 只需确保网格符合标准
```

### 动态批处理的局限性

**优点：**
- 适用于移动对象
- 无需设置
- 无内存开销

**缺点：**
- 仅限于小型网格
- 重建批次有 CPU 开销
- 不适用于许多着色器

**适用场景：**
- 小型粒子
- 简单的 UI 元素
- 重复的小型道具（金币、拾取物）

### 禁用动态批处理

有时动态批处理会增加开销：

**Edit > Project Settings > Player > Other Settings**
- 取消勾选 "Dynamic Batching"

**或者针对每个材质：**
- 着色器必须不支持批处理

## GPU 实例化 (GPU Instancing)

在一次 Draw Call 中渲染许多相同的网格。

### 启用 GPU 实例化

**材质设置：**
1. 选择材质
2. 勾选 "Enable GPU Instancing"
3. 着色器必须支持实例化

**着色器要求：**
```hlsl
// 着色器必须有实例化变体
#pragma multi_compile_instancing

// 访问实例 ID
UNITY_VERTEX_INPUT_INSTANCE_ID
```

### GPU 实例化要求

**所有实例必须共享：**
- 相同的网格
- 相同的材质（非实例属性）
- 兼容的着色器

**材质属性：**
```csharp
// ❌ 错误 - 每个对象获取材质实例，破坏实例化
private void Start()
{
    GetComponent<Renderer>().material.color = Color.red;  // 实例化材质！
}

// ✅ 正确 - 使用 MaterialPropertyBlock
private MaterialPropertyBlock props;

private void Start()
{
    props = new MaterialPropertyBlock();
    var renderer = GetComponent<Renderer>();

    props.SetColor("_Color", Color.red);
    renderer.SetPropertyBlock(props);  // 保留实例化！
}
```

### GPU 实例化 vs 静态批处理

| 特性 | GPU 实例化 | 静态批处理 |
|---------|----------------|-----------------|
| 内存 | 低（重用网格） | 高（合并网格） |
| 逐实例数据 | 支持 (MPB) | 不支持 |
| 移动 | 支持 | 不支持 |
| 网格要求 | 相同的网格 | 任意静态网格 |
| Draw Calls | 每个材质 1 次 | 每个批次 1 次 |

**使用 GPU 实例化用于：**
- 许多相同的对象（树木、岩石、草）
- 具有逐实例变化的对象（颜色、缩放）
- 可以移动的对象

**使用静态批处理用于：**
- 混合几何体
- 永久静态对象
- 内存不受限时

## SRP Batcher

优化材质属性更新（仅限 URP/HDRP）。

### 启用 SRP Batcher

**URP/HDRP：**
- URP/HDRP 中默认启用
- 在 Graphics Settings 中检查

**要求：**
- Shader Graph 或兼容的着色器
- 着色器在 CBUFFER 中声明材质属性
- UnityPerMaterial 常量缓冲区

### SRP Batcher 优势

**传统渲染：**
- 每次材质更改都需要 SetPass call
- 重新绑定所有材质属性

**SRP Batcher：**
- GPU 缓冲区在 Draw Calls 之间持久存在
- 仅上传更改的属性
- 批处理具有不同材质的对象

**性能提升**：SetPass calls 快 2-3 倍。

### SRP Batcher 兼容性

**兼容：**
- Shader Graph 着色器（自动）
- 具有 UnityPerMaterial CBUFFER 的自定义着色器

**不兼容：**
- MaterialPropertyBlock（破坏 SRP batching）
- 内置管线着色器
- 一些旧版着色器

**检查兼容性：**
```
选择材质 > Inspector > Shader > SRP Batcher Status
```

## 遮挡剔除 (Occlusion Culling)

不渲染被其他对象遮挡的对象。

### 设置遮挡剔除

**1. 标记静态遮挡者（Occluders）和被遮挡者（Occludees）：**
```
GameObject > 勾选 "Occluder Static"（针对建筑物/墙壁）
GameObject > 勾选 "Occludee Static"（针对要剔除的对象）
```

**2. 烘焙遮挡数据：**
```
Window > Rendering > Occlusion Culling
点击 "Bake" 按钮
```

**3. 配置设置：**
- **Smallest Occluder**：阻挡视线的最小尺寸（默认：5）
- **Smallest Hole**：可视的小缝隙（默认：0.25）
- **Backface Threshold**：背面剔除容差（默认：100）

### 遮挡剔除可视化

**Scene View（场景视图）：**
- Occlusion Culling > Visualization
- 显示遮挡体积
- 从相机预览

**运行时：**
- Frame Stats 显示被剔除的对象
- 使用 Frame Debugger 进行分析

### 遮挡剔除最佳实践

✅ **建议：**
- 在大型、复杂的场景中使用
- 将大型实体对象标记为遮挡者（建筑物、墙壁）
- 在最终确定场景布局后烘焙
- 从关键相机位置进行测试

❌ **不建议：**
- 在开放的户外场景中使用（收益甚微）
- 将小物体标记为遮挡者（增加开销）
- 仅依赖此功能进行性能关键场景的优化
- 场景更改后忘记重新烘焙

**性能**：节省 GPU 渲染，增加 CPU 剔除成本。使用 Profiler 测试影响。

## 视锥体剔除 (Frustum Culling)

Unity 自动剔除相机视图之外的对象。

### 视锥体剔除的工作原理

相机视锥体之外的对象不会被渲染：
- 剔除自动发生
- 基于渲染器边界（Bounds）
- CPU 检查边界与视锥体的关系

### 优化视锥体剔除

**准确的边界：**
```csharp
// 如果网格在运行时更改，更新边界
private void UpdateMesh()
{
    meshFilter.mesh = newMesh;
    meshFilter.mesh.RecalculateBounds();  // 更新边界
}
```

**大边界惩罚：**
- 具有巨大边界的对象（蒙皮网格、粒子）剔除效果较差
- 考虑拆分大型对象

**渲染器剔除：**
```csharp
// 检查渲染器是否可见
if (myRenderer.isVisible)
{
    // 仅更新可见对象
}
```

## LOD (多细节层次)

在远距离使用更简单的网格。

### LOD Group 设置

**1. 创建 LOD Group：**
```
GameObject > 3D Object > LOD Group
```

**2. 添加 LOD 级别：**
- LOD 0 (100% - 50%)：高细节网格
- LOD 1 (50% - 25%)：中等细节网格
- LOD 2 (25% - 10%)：低细节网格
- Culled (<10%)：不渲染

**3. 将渲染器分配给每个 LOD 级别**

### LOD 最佳实践

**每个 LOD 的多边形计数减少：**
- LOD 0：100%（全细节）
- LOD 1：50%（一半多边形）
- LOD 2：25%（四分之一多边形）
- LOD 3：10%（公告板或简单形状）

**LOD 过渡：**
- **Fade Mode**：在 LOD 之间交叉淡入淡出（平滑，轻微开销）
- **SpeedTree**：植物的特殊处理

**何时使用 LOD：**
- 在多个距离可见的对象
- 高多边形计数的模型
- 许多实例（树木、岩石、角色）

**何时跳过 LOD：**
- 始终处于固定距离的对象
- 已经是低多边形的模型
- 小型场景

## 材质和着色器优化

### 材质实例

```csharp
// ❌ 错误 - 创建材质实例，破坏批处理
private void Start()
{
    GetComponent<Renderer>().material.color = Color.red;  // 新的材质实例！
}

// ✅ 正确 - 共享材质以进行批处理
[SerializeField] private Material sharedMaterial;

private void Start()
{
    GetComponent<Renderer>().sharedMaterial = sharedMaterial;
}

// ✅ 正确 - 使用 MaterialPropertyBlock 进行变化
private MaterialPropertyBlock mpb;

private void Start()
{
    mpb = new MaterialPropertyBlock();
    var renderer = GetComponent<Renderer>();

    mpb.SetColor("_Color", Color.red);
    renderer.SetPropertyBlock(mpb);  // 通过 GPU 实例化保留批处理
}
```

### 着色器变体

限制着色器关键字以减少变体：

```csharp
// ❌ 错误 - 许多关键字导致组合爆炸
#pragma shader_feature FEATURE_A
#pragma shader_feature FEATURE_B
#pragma shader_feature FEATURE_C
#pragma shader_feature FEATURE_D
// 创建 2^4 = 16 个变体！

// ✅ 正确 - 仅对基本变体使用 multi_compile
#pragma multi_compile _ FEATURE_A FEATURE_B
// 创建 3 个变体
```

**着色器剥离 (Shader Stripping)：**
- Edit > Project Settings > Graphics > Shader Stripping
- 从构建中剥离未使用的变体

### 着色器复杂度

**片元着色器成本**（最昂贵）：
- 复杂的光照计算
- 多个纹理采样
- 数学运算（sin, cos, pow）

**优化着色器：**
- 在移动设备上使用更简单的光照模型
- 减少纹理采样
- 尽可能烘焙光照
- 使用查找纹理（LUT）进行复杂的数学运算

**移动端着色器指南：**
- <5 个纹理采样
- 避免复杂的数学运算（pow, sin, cos）
- 尽可能使用半精度（half precision）
- 限制顶点着色器复杂度

## 纹理优化

### 纹理压缩

**桌面端：**
- **DXT1/BC1**：RGB，无 Alpha（4:1 压缩）
- **DXT5/BC3**：RGBA 带 Alpha（4:1 压缩）
- **BC7**：高质量（解码较慢）

**移动端：**
- **ASTC**：自适应，最佳质量/大小（6x6 是不错的默认值）
- **ETC2**：Android，质量好
- **PVRTC**：iOS（旧版），质量较低

**配置：**
```
选择纹理 > Inspector
Platform: Android/iOS
Format: ASTC 6x6 / ETC2 / PVRTC
```

### 纹理分辨率

使用看起来可接受的最小分辨率：
- **UI**：2 的幂，无 Mipmap（256x256, 512x512）
- **3D 对象**：2 的幂，带 Mipmap（1024x1024, 2048x2048）
- **环境**：可以更大（4096x4096），使用压缩

**Mipmaps：**
- 对 3D 纹理启用（减少锯齿，提高性能）
- 对 UI 禁用（节省内存，无益处）

### 纹理图集 (Texture Atlasing)

将多个纹理合并为一个：

```csharp
// 多个材质（许多 Draw Calls）
Material woodMat;
Material stoneMat;
Material metalMat;

// 单个图集材质（一个 Draw Call）
Material atlasMaterial;  // 包含所有变体的组合纹理
```

**图集工具：**
- Sprite Atlas (2D)
- Texture Packer (外部工具)
- DCC 工具中的手动图集

**优点：**
- 更少的 Draw Calls（相同的材质）
- 减少内存（单个纹理）
- 更好的批处理

**缺点：**
- 更复杂的 UV 映射
- 更难维护
- 如果 Sprite 大小不一，会浪费空间

## 光照优化

### 烘焙光照 (Baked Lighting)

在构建时预计算光照：

**将灯光标记为 Baked：**
```
Light > Mode: Baked
```

**将对象标记为 Lightmap Static：**
```
GameObject > Lightmap Static checkbox
```

**烘焙光照：**
```
Window > Rendering > Lighting > Generate Lighting
```

**优点：**
- 无运行时光照成本
- 高质量阴影/GI（全局光照）
- 最佳性能

**局限性：**
- 仅限静态对象
- 烘焙时间较长
- 构建尺寸较大（光照贴图）

### 混合光照 (Mixed Lighting)

结合烘焙和实时光照：

**Light > Mode: Mixed**

**模式：**
- **Baked Indirect**：直接光实时，间接光烘焙
- **Subtractive**：一个实时光阴影，其余烘焙
- **Shadowmask**：烘焙阴影，实时光

**用于：**
- 静态环境中的动态对象
- 具有逼真光照的移动角色

### 实时光照 (Real-Time Lighting)

每帧计算动态光照：

**Light > Mode: Realtime**

**优化：**
- 将实时灯光限制为每个对象 1-2 个
- 使用小范围的光照
- 尽可能禁用阴影
- 当对象是静态时，首选 Baked/Mixed

**像素光限制：**
```
Edit > Project Settings > Quality > Pixel Light Count
```
限制同时像素光的数量。其余灯光使用顶点光照（质量较低，性能较好）。

## 粒子系统优化

### 粒子数量

减少最大粒子数：

```csharp
ParticleSystem.MainModule main = ps.main;
main.maxParticles = 50;  // 降低限制
```

**指南：**
- 移动端：每个系统 <200 个粒子
- PC：每个系统 <500-1000 个粒子
- VR：每个系统 <100 个粒子

### 粒子渲染

**优化设置：**
- **Rendering Mode**：Billboard（最快），Stretched Billboard，Mesh（最慢）
- **Max Particle Size**：较低的值更快
- **禁用功能**：如果不需要，禁用碰撞、阴影

```csharp
// 禁用碰撞
ParticleSystem.CollisionModule collision = ps.collision;
collision.enabled = false;

// 禁用阴影
renderer.shadowCastingMode = ShadowCastingMode.Off;
renderer.receiveShadows = false;
```

### 粒子着色器

使用简单的移动端粒子着色器：
- Particles/Standard Unlit
- Particles/Additive
- 自定义简单着色器

避免：
- Particles/Standard Surface（昂贵）
- 粒子着色器中的复杂光照

## UI 优化

### Canvas 优化

**Canvas 渲染模式：**
- **Screen Space - Overlay**：无相机，最快
- **Screen Space - Camera**：基于相机，支持后处理
- **World Space**：3D Canvas，最昂贵

**嵌套 Canvas：**
```
Main Canvas (Screen Space - Overlay)
└── Static UI Canvas (嵌套，不会重建)
└── Dynamic UI Canvas (嵌套，频繁重建)
```

将静态/动态元素分离到不同的 Canvas - 只有动态 Canvas 会重建。

### Canvas 批处理

**批处理要求：**
- 相同的材质
- 相同的纹理（图集）
- 相同的 Z 深度
- 无不同材质的重叠

**破坏批处理：**
- 不同的材质
- Mask 组件
- Z 深度变化（重叠不同材质）

### Raycast Targets (射线检测目标)

对非交互式 UI 禁用 Raycast：

```csharp
// ❌ 错误 - 默认情况下所有 Image 都是 Raycast Target
Image backgroundImage;  // Raycast Target = true (不必要)

// ✅ 正确 - 对非交互元素禁用 Raycast
Image backgroundImage;
backgroundImage.raycastTarget = false;  // 不可点击，跳过 Raycast
```

**优化**：减少 Raycast 开销，特别是在复杂 UI 上。

### Canvas 重建

尽量减少 Canvas 重建：

**触发器：**
- 位置/大小更改
- 文本更改
- 颜色更改
- 启用/禁用

**优化：**
```csharp
// ❌ 错误 - 每帧更新文本
private void Update()
{
    scoreText.text = "Score: " + score;  // 每帧重建 Canvas！
}

// ✅ 正确 - 仅在更改时更新
private int lastScore = -1;

private void UpdateScore(int newScore)
{
    if (score != lastScore)
    {
        lastScore = newScore;
        scoreText.text = "Score: " + newScore;  // 仅在更改时重建
    }
}
```

## 分析渲染

### Frame Debugger (帧调试器)

**Window > Analysis > Frame Debugger**

**功能：**
- 逐步执行每个 Draw Call
- 查看批处理/实例化
- 检查材质属性
- 识别批处理中断原因

**用法：**
1. 启用 Frame Debugger
2. 运行游戏
3. 检查特定帧
4. 查找昂贵的 Draw Calls
5. 识别批处理问题

### GPU Profiler (GPU 分析器)

**Profiler Window > GPU**

**指标：**
- 总 GPU 时间
- Rendering.Camera.Render（主要渲染成本）
- Shadows.RenderShadowMap
- Opaque/Transparent geometry（不透明/透明几何体）

**优化目标：**
- GPU 时间 <16.67ms（对于 60 FPS）
- 识别峰值（阴影渲染、后处理）

### Graphics Jobs (图形作业)

启用多线程渲染：

**Edit > Project Settings > Player > Other Settings**
- **Graphics Jobs**：在 CPU 核心之间分配渲染任务

**优点：**
- 减少主线程渲染开销
- 更好的 CPU 利用率
- 在相同 Draw Calls 下获得更高 FPS

**局限性：**
- 并非所有平台都支持
- 简单场景可能有轻微开销

## 最佳实践总结

✅ **建议：**
- 对相同的网格启用 GPU 实例化
- 对静态环境使用静态批处理
- 对远处对象实施 LOD
- 尽可能烘焙光照
- 适当地压缩纹理
- 使用 Material Property Blocks 进行变化
- 定期使用 Frame Debugger 进行分析
- 分离静态/动态 UI Canvas
- 在非交互式 UI 上禁用 Raycast Targets
- 在复杂场景上使用遮挡剔除

❌ **不建议：**
- 重复访问 `.material` 属性（会实例化）
- 创建许多独特的材质（破坏批处理）
- 不必要地使用实时光照
- 忘记在材质上启用 GPU 实例化
- 使用高分辨率未压缩纹理
- 在所有灯光上启用阴影
- 拥有大量粒子数
- 每帧重建 UI Canvas
- 对所有内容使用动态批处理（CPU 成本）

**黄金法则**：使用 Frame Debugger 和 GPU Profiler 分析渲染。测量 Draw Calls、批次和 GPU 时间。基于数据而非假设进行优化。

遵循这些渲染优化技术，在所有平台上实现流畅、高帧率的 Unity 游戏。
