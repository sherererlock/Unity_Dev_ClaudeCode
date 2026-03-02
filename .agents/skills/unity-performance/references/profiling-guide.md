# Unity Profiling - Complete Guide

Comprehensive guide to Unity's profiling tools, performance measurement, bottleneck identification, and optimization workflow.

## Profiler Window

### Opening the Profiler

**Window > Analysis > Profiler** (Ctrl+7 / Cmd+7)

**Profiler modules:**
- CPU Usage
- GPU Usage
- Rendering
- Memory
- Audio
- Physics
- Physics (2D)

### Profiler Connection

**Playmode profiling:**
- Automatically profiles running game
- Click Record button to start/stop
- Most accurate for development

**Build profiling:**
1. Build with "Development Build" + "Autoconnect Profiler"
2. Run build
3. Profiler connects automatically
4. More accurate for final performance

**Manual connection:**
- Profiler Window > dropdown > Select target (IP or device)
- Connect to running build

## CPU Profiler

### Reading CPU Profiler

**Timeline view:**
- Each spike = expensive frame
- Click spike to see detailed breakdown
- Color-coded by category

**Hierarchy view:**
- Call tree showing execution time
- Self %: Time in this method only
- Total %: Time in method + children
- GC Alloc: Garbage collection allocations
- Calls: Number of invocations

**Categories:**
- **Rendering**: DrawCalls, SetPass, camera rendering
- **Scripts**: Update, FixedUpdate, Coroutines
- **Physics**: Physics simulation
- **Animation**: Animator updates
- **GC.Alloc**: Garbage collection allocations
- **Others**: Loading, VSync, etc.

### CPU Usage Analysis

**Target: <16.67ms for 60 FPS**

**Budget breakdown:**
- Rendering: 6-8ms
- Scripts: 4-6ms
- Physics: 2-3ms
- Other: 2-3ms

**Common bottlenecks:**

1. **Rendering.Camera.Render**
   - Too many draw calls
   - Complex shaders
   - Expensive post-processing

2. **PlayerLoop > Update**
   - Too many Update methods
   - Expensive Update logic
   - Find methods in Update

3. **FixedUpdate.PhysicsFixedUpdate**
   - Too many physics objects
   - Complex collision detection
   - Layer collision matrix not configured

4. **GC.Alloc**
   - String allocations
   - LINQ operations
   - Repeated GetComponent calls
   - Collection allocations

### Deep Profile Mode

Enable deep profiling for complete call stack:

**Profiler > Deep Profile checkbox**

**Benefits:**
- Shows all method calls (even non-MonoBehaviour)
- Complete call hierarchy
- Identifies exact bottleneck methods

**Drawbacks:**
- Significantly slows game (5-10x overhead)
- Memory intensive
- Use on small scenes or specific areas

**Usage:**
1. Disable deep profile by default
2. Enable when investigating specific bottleneck
3. Profile small scene or isolated system
4. Disable after identifying issue

### Call Stacks

Enable call stacks for allocation sources:

**Profiler > Call Stacks dropdown > Enable for specific categories**

**Options:**
- All
- GC.Alloc only (most useful)
- Script methods only

**Shows:**
- Exact line causing allocation
- Full call stack to allocation
- Easy to identify allocation sources

**Example:**
```
GC.Alloc: 2.5 KB
  PlayerController.Update() Line 45
    string message = "Health: " + health;  // Allocation source!
```

### Timeline View

**Profiler > Timeline module**

**Features:**
- Visual timeline of frame execution
- Thread activity (Main, Render, Worker threads)
- Identify parallelization opportunities
- See frame timing breakdown

**Usage:**
- Spot long-running methods (wide bars)
- Check thread utilization
- Identify blocking waits

## GPU Profiler

### GPU Usage Module

**Profiler > GPU module**

**Metrics:**
- Total GPU time
- Rendering.Camera.Render
- Shadows.RenderShadowMap
- Opaque/Transparent geometry
- Post-processing effects

**Target: <16.67ms for 60 FPS**

**Common GPU bottlenecks:**

1. **Too many draw calls**
   - Enable batching (static, dynamic, GPU instancing)
   - Use SRP Batcher (URP/HDRP)
   - Reduce unique materials

2. **Expensive shaders**
   - Simplify fragment shaders
   - Reduce texture samples
   - Use simpler lighting models

3. **Shadow rendering**
   - Reduce shadow distance
   - Lower shadow resolution
   - Fewer shadow-casting lights

4. **Overdraw (transparency)**
   - Minimize transparent objects
   - Reduce particle count
   - Optimize UI overlays

### Frame Debugger

**Window > Analysis > Frame Debugger**

**Features:**
- Step through each draw call
- View rendered output per call
- Inspect material properties
- Identify batching issues

**Usage:**
1. Enable Frame Debugger
2. Run game, pause on expensive frame
3. Step through draw calls
4. Find:
   - Redundant draws
   - Batching breaks
   - Expensive shader passes
   - Overdraw sources

**Batching analysis:**
- Look for consecutive draws with same material
- Check why batching failed (tooltip shows reason)
- Fix batching issues (material instances, different textures)

## Memory Profiler

### Memory Module

**Profiler > Memory module**

**Simple mode metrics:**
- **Total Allocated**: Current heap size
- **Reserved**: Memory reserved from OS
- **Texture Memory**: All loaded textures
- **Mesh Memory**: All loaded meshes
- **Audio Memory**: All loaded audio clips

**Detailed mode:**
- Click "Take Sample: Playmode"
- Shows detailed allocation breakdown
- Objects by type
- Reference tree

### Memory Profiler Package

**More advanced memory analysis:**

**Install: Window > Package Manager > Memory Profiler**

**Features:**
- Detailed heap snapshots
- Memory leak detection
- Compare snapshots
- Object reference chains

**Workflow:**
1. Take snapshot at start
2. Play game for a while
3. Take second snapshot
4. Compare snapshots
5. Find growing allocations
6. Identify leaks

**Leak indicators:**
- Growing managed heap
- Increasing object count
- Textures/meshes not released
- Event handlers not unsubscribed

### Managed Memory

**Managed heap:**
- C# objects
- Unity managed objects
- Grows until GC triggered

**Monitor:**
```csharp
private void LogMemory()
{
    long totalMemory = GC.GetTotalMemory(false);
    Debug.Log($"Managed memory: {totalMemory / (1024f * 1024f):F2} MB");
}
```

**Optimization:**
- Reduce allocations (see memory-optimization.md)
- Object pooling
- Reuse collections
- Cache references

### Native Memory

**Unmanaged memory:**
- Textures
- Meshes
- Audio clips
- Shaders
- Native plugins

**Optimization:**
- Compress textures
- Reduce texture resolution
- Unload unused assets
- Use AssetBundles for streaming

## Physics Profiler

### Physics Module

**Profiler > Physics module**

**Metrics:**
- **Physics.Processing**: Total physics time
- **Physics.Contacts**: Collision detection
- **Physics.Solver**: Constraint solving
- **Physics.Callbacks**: OnCollision/OnTrigger events
- **Active Rigidbodies**: Awake Rigidbody count
- **Active Colliders**: Active collider count

**Targets:**
- Desktop: <2ms
- Mobile: <4ms

**Optimization:**
- Configure Layer Collision Matrix
- Let Rigidbodies sleep
- Use primitive colliders
- Reduce active Rigidbody count

### Physics Debugger

**Window > Analysis > Physics Debugger**

**Features:**
- Visualize colliders in scene
- Show active contacts (collision pairs)
- View Rigidbody states (sleeping/awake)
- Identify unexpected collisions

**Usage:**
1. Enable Physics Debugger
2. Run game
3. View collision visualization
4. Identify:
   - Unnecessary collision pairs
   - Objects not sleeping
   - Excessive contacts

## Rendering Stats

### Game View Stats

**Game View > Stats button**

**Key metrics:**
- **FPS**: Frames per second
- **Frame time**: ms per frame
- **SetPass calls**: Material changes (minimize!)
- **Draw calls**: Rendering commands
- **Batches**: Batched draw calls
- **Tris/Verts**: Geometry complexity
- **Screen**: Resolution

**Targets:**
- SetPass calls: <100 (mobile), <300 (PC)
- Draw calls: <500 (mobile), <2000 (PC)
- Batches: Lower is better (matches draw calls when batched)

### Profiler Rendering Module

**Profiler > Rendering module**

**Metrics:**
- Batches
- SetPass calls
- Triangles/Vertices
- Used Textures
- Render Textures
- Shadow casters

## Audio Profiler

### Audio Module

**Profiler > Audio module**

**Metrics:**
- Playing sources
- Audio CPU usage
- Audio memory
- DSP buffer usage

**Optimization:**
- Limit simultaneous audio sources
- Use audio compression
- Reduce sample rate for effects
- Pool AudioSource components

## Custom Profiler Markers

### Creating Custom Markers

Profile specific code sections:

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

**Naming convention:**
```
"Category.Subcategory.Method"
"AI.Pathfinding.AStar"
"Rendering.UI.Canvas"
"Physics.Raycasting.LineOfSight"
```

**Shows in Profiler:**
- Custom category in hierarchy
- Detailed timing for specific code sections
- Easy identification in Profiler

### Auto Markers

Automatic method profiling:

```csharp
using Unity.Profiling;

[ProfilerMarker]
private void ExpensiveMethod()
{
    // Automatically profiled
}
```

Unity 2021.2+ only, experimental feature.

## Profiling Workflow

### Step 1: Baseline Measurement

Record current performance:

1. Build with Development Build + Profiler
2. Run target scene
3. Record typical gameplay (30-60 seconds)
4. Note key metrics:
   - Average FPS
   - Frame time spikes
   - GC.Alloc per frame
   - Physics time
   - Rendering time

**Document baseline:**
```
Baseline Performance (Scene: MainLevel, Platform: PC)
- Average FPS: 45
- Frame time: 22ms
- Rendering: 10ms
- Scripts: 8ms
- Physics: 4ms
- GC.Alloc: 2.5 KB/frame
```

### Step 2: Identify Bottleneck

Find most expensive system:

1. **CPU profiling:**
   - Find tallest spike in CPU Usage
   - Click spike, view Hierarchy
   - Sort by Total % or Self %
   - Identify top method

2. **GPU profiling:**
   - Check GPU Usage module
   - Find longest GPU time
   - Use Frame Debugger to drill down

3. **Memory profiling:**
   - Check GC.Alloc in CPU profiler
   - Identify allocation sources with Call Stacks
   - Find growing memory in Memory Profiler

**Bottleneck priority:**
1. Fix rendering if GPU-bound (Frame Debugger shows long frames)
2. Fix scripts if CPU-bound (Update/FixedUpdate expensive)
3. Fix physics if Physics.Processing high
4. Fix GC if frequent allocation spikes

### Step 3: Targeted Optimization

Apply specific optimization:

**Examples:**

**High rendering cost:**
- Enable GPU Instancing
- Reduce draw calls (batching)
- Simplify shaders

**Expensive scripts:**
- Cache GetComponent calls
- Reduce Update frequency
- Replace polling with events

**Physics overhead:**
- Configure Layer Collision Matrix
- Use primitive colliders
- Reduce active Rigidbodies

**GC allocations:**
- Cache strings (StringBuilder)
- Reuse collections (List.Clear)
- Avoid LINQ in Update

### Step 4: Measure Improvement

Compare before/after:

1. Apply single optimization
2. Re-profile with same scene
3. Measure delta:
   - Frame time reduction
   - FPS increase
   - GC.Alloc reduction

**Document improvement:**
```
Optimization: Enabled GPU Instancing on trees (500 instances)
Before: 22ms frame time (45 FPS)
After: 16ms frame time (60 FPS)
Improvement: 6ms (27% faster)
```

### Step 5: Repeat

Continue optimization cycle:
1. Profile
2. Identify next bottleneck
3. Optimize
4. Measure
5. Repeat until performance target met

## Platform-Specific Profiling

### Mobile Profiling

**Connect device:**
1. Build with Development + Autoconnect Profiler
2. Install on device
3. Run app
4. Profiler connects via USB/WiFi

**Mobile-specific metrics:**
- **GPU**: Check for overdraw, shader complexity
- **Memory**: Monitor closely (limited RAM)
- **Battery**: Profile power consumption
- **Thermal**: Watch for throttling

**Tools:**
- Xcode Instruments (iOS)
- Android Studio Profiler (Android)
- Unity Profiler (cross-platform)

### VR Profiling

**VR-specific considerations:**
- Target 90 FPS minimum (11.1ms frame time)
- Profile each eye separately
- Minimize draw calls (<100 per eye)

**VR Profiler (Oculus/Meta):**
- Oculus Performance HUD
- Ovr Metrics Tool
- Meta Quest Developer Hub

### Console Profiling

**Platform-specific tools:**
- **PlayStation**: SN-DBS debugger
- **Xbox**: PIX
- **Nintendo Switch**: Profiler tool

Connect Unity Profiler + platform tools for comprehensive analysis.

## Profiler Best Practices

✅ **DO:**
- Profile on target hardware (not just editor)
- Build with Development Build for accurate profiling
- Record typical gameplay scenarios
- Use Deep Profile sparingly (heavy overhead)
- Enable Call Stacks for GC.Alloc investigation
- Take Memory snapshots periodically
- Document baseline before optimizing
- Measure after each optimization
- Use Frame Debugger for rendering issues
- Create custom ProfilerMarkers for critical code

❌ **DON'T:**
- Profile only in Editor (not accurate)
- Rely on FPS counter alone (use Profiler)
- Optimize without measuring first (profile-driven optimization only)
- Enable Deep Profile for large scenes (crashes/hangs)
- Make multiple changes before measuring
- Ignore GC.Alloc spikes (causes stuttering)
- Skip platform-specific profiling (mobile, console)

## Quick Profiling Checklist

**Before optimizing:**
- [ ] Build with Development Build + Profiler enabled
- [ ] Profile on target hardware
- [ ] Record baseline metrics (FPS, frame time, allocations)
- [ ] Identify bottleneck (CPU, GPU, Physics, Memory)

**During optimization:**
- [ ] Apply single optimization
- [ ] Re-profile immediately
- [ ] Measure improvement (quantify delta)
- [ ] Document change and impact

**After optimization:**
- [ ] Verify FPS target met (60/30 FPS)
- [ ] Check for new bottlenecks (optimization can shift bottleneck)
- [ ] Test on min-spec hardware
- [ ] Profile different scenes/scenarios

## Profiling Tools Summary

| Tool | Purpose | Use For |
|------|---------|---------|
| CPU Profiler | Script performance | Update loops, GC allocations |
| GPU Profiler | Rendering performance | Draw calls, shader cost |
| Frame Debugger | Draw call analysis | Batching issues, overdraw |
| Memory Profiler | Heap analysis | Memory leaks, texture memory |
| Physics Profiler | Physics performance | Collision cost, Rigidbody count |
| Stats Panel | Quick metrics | FPS, draw calls, tris |
| Timeline View | Thread analysis | Parallelization, waits |

**Golden rule**: Always profile before optimizing. Measure, don't guess. Profile-driven optimization is the only reliable path to performance.

Follow this profiling workflow to systematically identify and eliminate performance bottlenecks in Unity games.
