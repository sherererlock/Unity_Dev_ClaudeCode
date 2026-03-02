# Unity 性能分析 - 完整指南

全面介绍 Unity 的性能分析工具、性能测量、瓶颈识别和优化工作流程的指南。

## Profiler 窗口

### 打开 Profiler

**Window > Analysis > Profiler** (Ctrl+7 / Cmd+7)

**Profiler 模块：**
- CPU Usage (CPU 使用率)
- GPU Usage (GPU 使用率)
- Rendering (渲染)
- Memory (内存)
- Audio (音频)
- Physics (物理)
- Physics (2D) (2D 物理)

### Profiler 连接

**播放模式分析：**
- 自动分析正在运行的游戏
- 点击录制按钮开始/停止
- 开发阶段最准确

**构建版本分析：**
1. 使用 "Development Build" + "Autoconnect Profiler" 进行构建
2. 运行构建版本
3. Profiler 自动连接
4. 最终性能数据更准确

**手动连接：**
- Profiler 窗口 > 下拉菜单 > 选择目标（IP 或设备）
- 连接到正在运行的构建版本

## CPU Profiler（CPU 分析器）

### 解读 CPU Profiler

**时间轴视图：**
- 每个峰值 = 高耗时帧
- 点击峰值查看详细分类
- 按类别颜色编码

**层级视图：**
- 显示执行时间的调用树
- Self %: 自身占比（仅在该方法中花费的时间）
- Total %: 总占比（该方法及其子方法花费的时间）
- GC Alloc: 垃圾回收内存分配
- Calls: 调用次数

**类别：**
- **Rendering**: DrawCalls, SetPass, 相机渲染
- **Scripts**: Update, FixedUpdate, 协程
- **Physics**: 物理模拟
- **Animation**: Animator 更新
- **GC.Alloc**: 垃圾回收分配
- **Others**: 加载, VSync 等

### CPU 使用率分析

**目标：60 FPS 时 <16.67ms**

**预算细分：**
- Rendering (渲染): 6-8ms
- Scripts (脚本): 4-6ms
- Physics (物理): 2-3ms
- Other (其他): 2-3ms

**常见瓶颈：**

1. **Rendering.Camera.Render**
   - Draw calls 太多
   - 着色器复杂
   - 后处理开销大

2. **PlayerLoop > Update**
   - Update 方法太多
   - Update 逻辑开销大
   - 在 Update 中使用了 Find 方法

3. **FixedUpdate.PhysicsFixedUpdate**
   - 物理对象太多
   - 碰撞检测复杂
   - 未配置层级碰撞矩阵

4. **GC.Alloc**
   - 字符串分配
   - LINQ 操作
   - 重复调用 GetComponent
   - 集合分配

### 深度分析模式 (Deep Profile Mode)

启用深度分析以获取完整的调用堆栈：

**Profiler > Deep Profile 复选框**

**优点：**
- 显示所有方法调用（即使是非 MonoBehaviour）
- 完整的调用层级
- 识别确切的瓶颈方法

**缺点：**
- 显著拖慢游戏速度（5-10 倍开销）
- 内存占用大
- 仅在小场景或特定区域使用

**用法：**
1. 默认禁用深度分析
2. 调查特定瓶颈时启用
3. 分析小场景或隔离系统
4. 识别问题后禁用

### 调用堆栈 (Call Stacks)

为分配源启用调用堆栈：

**Profiler > Call Stacks 下拉菜单 > 为特定类别启用**

**选项：**
- All (全部)
- GC.Alloc only (仅 GC.Alloc，最有用)
- Script methods only (仅脚本方法)

**显示内容：**
- 导致分配的确切行
- 分配的完整调用堆栈
- 易于识别分配源

**示例：**
```
GC.Alloc: 2.5 KB
  PlayerController.Update() Line 45
    string message = "Health: " + health;  // Allocation source! (分配源！)
```

### 时间轴视图 (Timeline View)

**Profiler > Timeline 模块**

**特性：**
- 帧执行的可视化时间轴
- 线程活动（主线程、渲染线程、工作线程）
- 识别并行化机会
- 查看帧时间细分

**用法：**
- 发现长耗时方法（宽条）
- 检查线程利用率
- 识别阻塞等待

## GPU Profiler（GPU 分析器）

### GPU 使用率模块

**Profiler > GPU 模块**

**指标：**
- Total GPU time (总 GPU 时间)
- Rendering.Camera.Render
- Shadows.RenderShadowMap
- Opaque/Transparent geometry (不透明/透明几何体)
- Post-processing effects (后处理效果)

**目标：60 FPS 时 <16.67ms**

**常见 GPU 瓶颈：**

1. **Draw calls 太多**
   - 启用批处理（静态、动态、GPU Instancing）
   - 使用 SRP Batcher (URP/HDRP)
   - 减少唯一材质

2. **昂贵的着色器**
   - 简化片元着色器
   - 减少纹理采样
   - 使用更简单的光照模型

3. **阴影渲染**
   - 减小阴影距离
   - 降低阴影分辨率
   - 减少投射阴影的灯光

4. **过度绘制（透明度）**
   - 最小化透明对象
   - 减少粒子数量
   - 优化 UI 覆盖

### 帧调试器 (Frame Debugger)

**Window > Analysis > Frame Debugger**

**特性：**
- 逐步查看每个 draw call
- 查看每次调用的渲染输出
- 检查材质属性
- 识别批处理问题

**用法：**
1. 启用帧调试器
2. 运行游戏，在耗时帧暂停
3. 逐步查看 draw calls
4. 查找：
   - 冗余绘制
   - 批处理中断
   - 昂贵的着色器 pass
   - 过度绘制来源

**批处理分析：**
- 寻找使用相同材质的连续绘制
- 检查批处理失败原因（提示框显示原因）
- 修复批处理问题（材质实例、不同纹理）

## 内存分析器 (Memory Profiler)

### 内存模块

**Profiler > Memory 模块**

**简单模式指标：**
- **Total Allocated**: 当前堆大小
- **Reserved**: 从操作系统保留的内存
- **Texture Memory**: 所有加载的纹理
- **Mesh Memory**: 所有加载的网格
- **Audio Memory**: 所有加载的音频片段

**详细模式：**
- 点击 "Take Sample: Playmode"
- 显示详细的分配细分
- 按类型分类的对象
- 引用树

### Memory Profiler 包

**更高级的内存分析：**

**安装：Window > Package Manager > Memory Profiler**

**特性：**
- 详细的堆快照
- 内存泄漏检测
- 比较快照
- 对象引用链

**工作流程：**
1. 开始时拍摄快照
2. 运行游戏一段时间
3. 拍摄第二个快照
4. 比较快照
5. 查找增长的分配
6. 识别泄漏

**泄漏指标：**
- 增长的托管堆
- 增加的对象计数
- 纹理/网格未释放
- 事件处理程序未取消订阅

### 托管内存

**托管堆：**
- C# 对象
- Unity 托管对象
- 增长直到触发 GC

**监控：**
```csharp
private void LogMemory()
{
    long totalMemory = GC.GetTotalMemory(false);
    Debug.Log($"Managed memory: {totalMemory / (1024f * 1024f):F2} MB");
}
```

**优化：**
- 减少分配（见 memory-optimization.md）
- 对象池
- 重用集合
- 缓存引用

### 原生内存 (Native Memory)

**非托管内存：**
- 纹理
- 网格
- 音频片段
- 着色器
- 原生插件

**优化：**
- 压缩纹理
- 降低纹理分辨率
- 卸载未使用的资源
- 使用 AssetBundles 进行流式传输

## 物理分析器 (Physics Profiler)

### 物理模块

**Profiler > Physics 模块**

**指标：**
- **Physics.Processing**: 总物理时间
- **Physics.Contacts**: 碰撞检测
- **Physics.Solver**: 约束求解
- **Physics.Callbacks**: OnCollision/OnTrigger 事件
- **Active Rigidbodies**: 唤醒的刚体计数
- **Active Colliders**: 活跃碰撞体计数

**目标：**
- 桌面端：<2ms
- 移动端：<4ms

**优化：**
- 配置层级碰撞矩阵
- 让刚体休眠
- 使用原始碰撞体
- 减少活跃刚体数量

### 物理调试器 (Physics Debugger)

**Window > Analysis > Physics Debugger**

**特性：**
- 在场景中可视化碰撞体
- 显示活跃接触（碰撞对）
- 查看刚体状态（休眠/唤醒）
- 识别意外碰撞

**用法：**
1. 启用物理调试器
2. 运行游戏
3. 查看碰撞可视化
4. 识别：
   - 不必要的碰撞对
   - 未休眠的对象
   - 过多的接触

## 渲染统计 (Rendering Stats)

### 游戏视图统计

**Game View > Stats 按钮**

**关键指标：**
- **FPS**: 每秒帧数
- **Frame time**: 帧时间（每帧毫秒数）
- **SetPass calls**: 材质切换（最小化！）
- **Draw calls**: 渲染命令
- **Batches**: 批处理后的 draw calls
- **Tris/Verts**: 几何体复杂度
- **Screen**: 分辨率

**目标：**
- SetPass calls: <100 (移动端), <300 (PC)
- Draw calls: <500 (移动端), <2000 (PC)
- Batches: 越低越好（批处理时与 draw calls 匹配）

### Profiler 渲染模块

**Profiler > Rendering 模块**

**指标：**
- Batches (批次)
- SetPass calls (SetPass 调用)
- Triangles/Vertices (三角形/顶点)
- Used Textures (使用的纹理)
- Render Textures (渲染纹理)
- Shadow casters (阴影投射者)

## 音频分析器 (Audio Profiler)

### 音频模块

**Profiler > Audio 模块**

**指标：**
- Playing sources (播放源)
- Audio CPU usage (音频 CPU 使用率)
- Audio memory (音频内存)
- DSP buffer usage (DSP 缓冲区使用率)

**优化：**
- 限制同时播放的音频源
- 使用音频压缩
- 降低音效采样率
- 池化 AudioSource 组件

## 自定义 Profiler 标记

### 创建自定义标记

分析特定代码段：

```csharp
using Unity.Profiling;

public class AIController : MonoBehaviour
{
    private static readonly ProfilerMarker s_PathfindingMarker = new ProfilerMarker("AI.Pathfinding");
    private static readonly ProfilerMarker s_BehaviorMarker = new ProfilerMarker("AI.Behavior");

    private void Update()
    {
        s_PathfindingMarker.Begin();
        CalculatePath();
        s_PathfindingMarker.End();

        s_BehaviorMarker.Begin();
        UpdateBehavior();
        s_BehaviorMarker.End();
    }

    private void CalculatePath() { }
    private void UpdateBehavior() { }
}
```

**命名约定：**
```
"Category.Subcategory.Method"
"AI.Pathfinding.AStar"
"Rendering.UI.Canvas"
"Physics.Raycasting.LineOfSight"
```

**在 Profiler 中显示：**
- 层级视图中的自定义类别
- 特定代码段的详细计时
- 在 Profiler 中易于识别

### 自动标记

自动方法分析：

```csharp
using Unity.Profiling;

[ProfilerMarker]
private void ExpensiveMethod()
{
    // 自动分析
}
```

仅限 Unity 2021.2+，实验性功能。

## 性能分析工作流程

### 第 1 步：基线测量

记录当前性能：

1. 使用 Development Build + Profiler 构建
2. 运行目标场景
3. 录制典型游戏过程（30-60 秒）
4. 记录关键指标：
   - 平均 FPS
   - 帧时间峰值
   - 每帧 GC.Alloc
   - 物理时间
   - 渲染时间

**记录基线：**
```
基线性能 (场景: MainLevel, 平台: PC)
- 平均 FPS: 45
- 帧时间: 22ms
- 渲染: 10ms
- 脚本: 8ms
- 物理: 4ms
- GC.Alloc: 2.5 KB/帧
```

### 第 2 步：识别瓶颈

找出开销最大的系统：

1. **CPU 分析：**
   - 在 CPU Usage 中找到最高峰值
   - 点击峰值，查看层级视图
   - 按 Total % 或 Self % 排序
   - 识别排名第一的方法

2. **GPU 分析：**
   - 检查 GPU Usage 模块
   - 找到最长的 GPU 时间
   - 使用 Frame Debugger 深入分析

3. **内存分析：**
   - 在 CPU profiler 中检查 GC.Alloc
   - 使用 Call Stacks 识别分配源
   - 在 Memory Profiler 中查找增长的内存

**瓶颈优先级：**
1. 如果是 GPU 受限，修复渲染（Frame Debugger 显示长帧）
2. 如果是 CPU 受限，修复脚本（Update/FixedUpdate 开销大）
3. 如果 Physics.Processing 高，修复物理
4. 如果分配峰值频繁，修复 GC

### 第 3 步：针对性优化

应用特定优化：

**示例：**

**高渲染成本：**
- 启用 GPU Instancing
- 减少 draw calls（批处理）
- 简化着色器

**昂贵的脚本：**
- 缓存 GetComponent 调用
- 降低 Update 频率
- 用事件替换轮询

**物理开销：**
- 配置层级碰撞矩阵
- 使用原始碰撞体
- 减少活跃刚体

**GC 分配：**
- 缓存字符串 (StringBuilder)
- 重用集合 (List.Clear)
- 避免在 Update 中使用 LINQ

### 第 4 步：测量改进

比较前/后：

1. 应用单个优化
2. 使用相同场景重新分析
3. 测量变化：
   - 帧时间减少
   - FPS 增加
   - GC.Alloc 减少

**记录改进：**
```
优化：在树上启用 GPU Instancing（500 个实例）
之前: 22ms 帧时间 (45 FPS)
之后: 16ms 帧时间 (60 FPS)
改进: 6ms (快了 27%)
```

### 第 5 步：重复

继续优化循环：
1. 分析
2. 识别下一个瓶颈
3. 优化
4. 测量
5. 重复直到达到性能目标

## 平台特定分析

### 移动端分析

**连接设备：**
1. 使用 Development + Autoconnect Profiler 构建
2. 安装到设备
3. 运行应用
4. Profiler 通过 USB/WiFi 连接

**移动端特定指标：**
- **GPU**: 检查过度绘制，着色器复杂度
- **Memory**: 密切监控（RAM 有限）
- **Battery**: 分析功耗
- **Thermal**: 注意降频

**工具：**
- Xcode Instruments (iOS)
- Android Studio Profiler (Android)
- Unity Profiler (跨平台)

### VR 分析

**VR 特定注意事项：**
- 目标最低 90 FPS（11.1ms 帧时间）
- 分别分析每只眼睛
- 最小化 draw calls（每眼 <100）

**VR Profiler (Oculus/Meta):**
- Oculus Performance HUD
- Ovr Metrics Tool
- Meta Quest Developer Hub

### 主机分析

**平台特定工具：**
- **PlayStation**: SN-DBS debugger
- **Xbox**: PIX
- **Nintendo Switch**: Profiler tool

连接 Unity Profiler + 平台工具进行综合分析。

## Profiler 最佳实践

✅ **要做：**
- 在目标硬件上分析（不仅仅是编辑器）
- 使用 Development Build 构建以获得准确分析
- 录制典型游戏场景
- 谨慎使用 Deep Profile（开销大）
- 启用 Call Stacks 调查 GC.Alloc
- 定期拍摄内存快照
- 优化前记录基线
- 每次优化后测量
- 使用 Frame Debugger 处理渲染问题
- 为关键代码创建自定义 ProfilerMarkers

❌ **不要：**
- 仅在编辑器中分析（不准确）
- 仅依赖 FPS 计数器（使用 Profiler）
- 未测量先优化（仅进行数据驱动的优化）
- 在大场景中启用 Deep Profile（崩溃/卡死）
- 测量前进行多次更改
- 忽略 GC.Alloc 峰值（导致卡顿）
- 跳过平台特定分析（移动端、主机）

## 快速分析清单

**优化前：**
- [ ] 启用 Development Build + Profiler 构建
- [ ] 在目标硬件上分析
- [ ] 记录基线指标（FPS、帧时间、分配）
- [ ] 识别瓶颈（CPU、GPU、物理、内存）

**优化期间：**
- [ ] 应用单个优化
- [ ] 立即重新分析
- [ ] 测量改进（量化变化）
- [ ] 记录更改和影响

**优化后：**
- [ ] 验证是否达到 FPS 目标（60/30 FPS）
- [ ] 检查新瓶颈（优化可能会转移瓶颈）
- [ ] 在最低配置硬件上测试
- [ ] 分析不同场景/方案

## 分析工具总结

| 工具 | 目的 | 用于 |
|------|---------|---------|
| CPU Profiler | 脚本性能 | Update 循环, GC 分配 |
| GPU Profiler | 渲染性能 | Draw calls, 着色器开销 |
| Frame Debugger | Draw call 分析 | 批处理问题, 过度绘制 |
| Memory Profiler | 堆分析 | 内存泄漏, 纹理内存 |
| Physics Profiler | 物理性能 | 碰撞开销, 刚体计数 |
| Stats Panel | 快速指标 | FPS, draw calls, 三角形面数 |
| Timeline View | 线程分析 | 并行化, 等待 |

**黄金法则**：永远在优化前进行分析。测量，不要猜测。数据驱动的优化是通往高性能的唯一可靠途径。

遵循此分析工作流程，系统地识别并消除 Unity 游戏中的性能瓶颈。
