# Unity Rendering Optimization - Complete Guide

Comprehensive guide to GPU optimization, draw call reduction, batching, culling, and shader performance in Unity.

## Draw Call Fundamentals

### What Are Draw Calls?

A draw call is a CPU command to GPU to render a mesh with specific material settings.

**Cost breakdown:**
- **SetPass Calls**: Change material/shader (most expensive)
- **Draw Calls**: Render mesh (less expensive if same material)

**Performance target:**
- **Mobile**: <100 draw calls
- **PC/Console**: <500-1000 draw calls
- **VR**: <50-100 draw calls per eye

### Viewing Draw Call Stats

**Game View > Stats panel**:
- SetPass calls
- Batches
- Saved by batching
- Tris/Verts

**Frame Debugger**: Window > Analysis > Frame Debugger
- Shows each draw call
- Material changes
- Batching failures

## Static Batching

Combine static meshes into single draw call.

### Enabling Static Batching

**Mark GameObjects as Static:**
1. Select GameObject
2. Check "Static" checkbox (top-right of Inspector)
3. Or set specific flags: "Batching Static"

```csharp
// Set static flag from code
gameObject.isStatic = true;

// Or specific flag
GameObjectUtility.SetStaticEditorFlags(gameObject, StaticEditorFlags.BatchingStatic);
```

### Static Batching Requirements

**All meshes must share:**
- Same material instance
- Same lightmap
- Same additional vertex streams

**Not batched:**
- Different materials
- Different lightmap indices
- Skinned meshes
- Dynamic (moving) objects

### Static Batching Limitations

**Pros:**
- Reduces draw calls significantly
- No runtime overhead
- Works with lightmapping

**Cons:**
- Increased memory usage (combined meshes)
- Cannot move batched objects
- Breaks if object moves

**When to use:**
- Environment props
- Static buildings
- Background elements

## Dynamic Batching

Automatically batch small dynamic meshes at runtime.

### Dynamic Batching Requirements

**Meshes must be:**
- **Vertex count**: <300 vertices (Unity 2020+), <900 (older)
- **Same material**: Shared material instance
- **Same scale**: Uniform or non-uniform (but consistent)
- **No shadows**: Shadow casters not batched

```csharp
// Dynamic batching happens automatically
// Just ensure meshes meet criteria
```

### Dynamic Batching Limitations

**Pros:**
- Works with moving objects
- No setup required
- No memory overhead

**Cons:**
- Limited to small meshes
- CPU cost to rebuild batches
- Doesn't work with many shaders

**When to use:**
- Small particles
- Simple UI elements
- Duplicate small props (coins, pickups)

### Disabling Dynamic Batching

Sometimes dynamic batching adds overhead:

**Edit > Project Settings > Player > Other Settings**
- Uncheck "Dynamic Batching"

**Or per-material:**
- Shader must not support batching

## GPU Instancing

Render many identical meshes in one draw call.

### Enabling GPU Instancing

**Material setup:**
1. Select material
2. Check "Enable GPU Instancing"
3. Shader must support instancing

**Shader requirement:**
```hlsl
// Shader must have instancing variant
#pragma multi_compile_instancing

// Access instance ID
UNITY_VERTEX_INPUT_INSTANCE_ID
```

### GPU Instancing Requirements

**All instances must share:**
- Same mesh
- Same material (not instance properties)
- Compatible shader

**Material properties:**
```csharp
// ❌ BAD - Each object gets material instance, breaks instancing
private void Start()
{
    GetComponent<Renderer>().material.color = Color.red;  // Instantiates material!
}

// ✅ GOOD - Use MaterialPropertyBlock
private MaterialPropertyBlock props;

private void Start()
{
    props = new MaterialPropertyBlock();
    var renderer = GetComponent<Renderer>();

    props.SetColor("_Color", Color.red);
    renderer.SetPropertyBlock(props);  // Preserves instancing!
}
```

### GPU Instancing vs Static Batching

| Feature | GPU Instancing | Static Batching |
|---------|----------------|-----------------|
| Memory | Low (reuses mesh) | High (combines meshes) |
| Per-instance data | Yes (MPB) | No |
| Movement | Supported | Not supported |
| Mesh requirements | Identical meshes | Any static mesh |
| Draw calls | 1 per material | 1 per batch |

**Use GPU Instancing for:**
- Many identical objects (trees, rocks, grass)
- Objects with per-instance variations (color, scale)
- Objects that can move

**Use Static Batching for:**
- Mixed geometry
- Permanent static objects
- When memory isn't constrained

## SRP Batcher

Optimizes material property updates (URP/HDRP only).

### Enabling SRP Batcher

**URP/HDRP:**
- Enabled by default in URP/HDRP
- Check in Graphics Settings

**Requirements:**
- Shader Graph or compatible shader
- Shaders declare material properties in CBUFFER
- UnityPerMaterial constant buffer

### SRP Batcher Benefits

**Traditional rendering:**
- SetPass call for each material change
- Rebind all material properties

**SRP Batcher:**
- GPU buffers persist between draw calls
- Only upload changed properties
- Batches objects with different materials

**Performance gain**: 2-3x faster SetPass calls.

### SRP Batcher Compatibility

**Compatible:**
- Shader Graph shaders (automatic)
- Custom shaders with UnityPerMaterial CBUFFER

**Not compatible:**
- MaterialPropertyBlock (breaks SRP batching)
- Built-in pipeline shaders
- Some legacy shaders

**Check compatibility:**
```
Select material > Inspector > Shader > SRP Batcher Status
```

## Occlusion Culling

Don't render objects hidden behind other objects.

### Setting Up Occlusion Culling

**1. Mark static occluders and occludees:**
```
GameObject > Check "Occluder Static" for buildings/walls
GameObject > Check "Occludee Static" for objects to cull
```

**2. Bake occlusion data:**
```
Window > Rendering > Occlusion Culling
Click "Bake" button
```

**3. Configure settings:**
- **Smallest Occluder**: Minimum size to block view (default: 5)
- **Smallest Hole**: Small gaps visibility (default: 0.25)
- **Backface Threshold**: Backface culling tolerance (default: 100)

### Occlusion Culling Visualization

**Scene View:**
- Occlusion Culling > Visualization
- Shows occluder volumes
- Previews from camera

**Runtime:**
- Frame Stats shows culled objects
- Profile with Frame Debugger

### Occlusion Culling Best Practices

✅ **DO:**
- Use on large, complex scenes
- Mark large solid objects as occluders (buildings, walls)
- Bake after finalizing scene layout
- Test from key camera positions

❌ **DON'T:**
- Use on open outdoor scenes (little benefit)
- Mark small objects as occluders (overhead)
- Rely on for performance-critical scenes alone
- Forget to rebake after scene changes

**Performance**: Saves GPU rendering, adds CPU culling cost. Test impact with profiler.

## Frustum Culling

Unity's automatic culling of objects outside camera view.

### How Frustum Culling Works

Objects outside camera frustum aren't rendered:
- Culling happens automatically
- Based on renderer bounds
- CPU checks bounds vs frustum

### Optimizing Frustum Culling

**Accurate bounds:**
```csharp
// Update bounds if mesh changes at runtime
private void UpdateMesh()
{
    meshFilter.mesh = newMesh;
    meshFilter.mesh.RecalculateBounds();  // Update bounds
}
```

**Large bounds penalty:**
- Objects with huge bounds (skinned meshes, particles) culled less effectively
- Consider splitting large objects

**Renderer culling:**
```csharp
// Check if renderer is visible
if (myRenderer.isVisible)
{
    // Update only visible objects
}
```

## LOD (Level of Detail)

Use simpler meshes at distance.

### LOD Group Setup

**1. Create LOD Group:**
```
GameObject > 3D Object > LOD Group
```

**2. Add LOD levels:**
- LOD 0 (100% - 50%): High detail mesh
- LOD 1 (50% - 25%): Medium detail mesh
- LOD 2 (25% - 10%): Low detail mesh
- Culled (<10%): Don't render

**3. Assign renderers to each LOD level**

### LOD Best Practices

**Poly count reduction per LOD:**
- LOD 0: 100% (full detail)
- LOD 1: 50% (half polygons)
- LOD 2: 25% (quarter polygons)
- LOD 3: 10% (billboard or simple shape)

**LOD transitions:**
- **Fade Mode**: Cross-fade between LODs (smooth, slight overhead)
- **SpeedTree**: Special handling for foliage

**When to use LOD:**
- Objects visible at multiple distances
- High poly count models
- Many instances (trees, rocks, characters)

**When to skip LOD:**
- Objects always at fixed distance
- Already low-poly models
- Small scenes

## Material and Shader Optimization

### Material Instances

```csharp
// ❌ BAD - Creates material instance, breaks batching
private void Start()
{
    GetComponent<Renderer>().material.color = Color.red;  // New material instance!
}

// ✅ GOOD - Share material for batching
[SerializeField] private Material sharedMaterial;

private void Start()
{
    GetComponent<Renderer>().sharedMaterial = sharedMaterial;
}

// ✅ GOOD - Use MaterialPropertyBlock for variations
private MaterialPropertyBlock mpb;

private void Start()
{
    mpb = new MaterialPropertyBlock();
    var renderer = GetComponent<Renderer>();

    mpb.SetColor("_Color", Color.red);
    renderer.SetPropertyBlock(mpb);  // Preserves batching with GPU instancing
}
```

### Shader Variants

Limit shader keywords to reduce variants:

```csharp
// ❌ BAD - Many keywords create combinatorial explosion
#pragma shader_feature FEATURE_A
#pragma shader_feature FEATURE_B
#pragma shader_feature FEATURE_C
#pragma shader_feature FEATURE_D
// Creates 2^4 = 16 variants!

// ✅ GOOD - Use multi_compile for essential variants only
#pragma multi_compile _ FEATURE_A FEATURE_B
// Creates 3 variants
```

**Shader stripping:**
- Edit > Project Settings > Graphics > Shader Stripping
- Strip unused variants from build

### Shader Complexity

**Fragment shader cost** (most expensive):
- Complex lighting calculations
- Multiple texture samples
- Mathematical operations (sin, cos, pow)

**Optimize shaders:**
- Use simpler lighting models on mobile
- Reduce texture samples
- Bake lighting where possible
- Use lookup textures (LUTs) for complex math

**Mobile shader guidelines:**
- <5 texture samples
- Avoid complex math (pow, sin, cos)
- Use half precision when possible
- Limit vertex shader complexity

## Texture Optimization

### Texture Compression

**Desktop:**
- **DXT1/BC1**: RGB, no alpha (4:1 compression)
- **DXT5/BC3**: RGBA with alpha (4:1 compression)
- **BC7**: High quality (slower decode)

**Mobile:**
- **ASTC**: Adaptive, best quality/size (6×6 is good default)
- **ETC2**: Android, good quality
- **PVRTC**: iOS (older), lower quality

**Configuration:**
```
Select texture > Inspector
Platform: Android/iOS
Format: ASTC 6x6 / ETC2 / PVRTC
```

### Texture Resolution

Use smallest resolution that looks acceptable:
- **UI**: Power of 2, no mipmaps (256×256, 512×512)
- **3D Objects**: Power of 2, with mipmaps (1024×1024, 2048×2048)
- **Environment**: Can be larger (4096×4096), use compression

**Mipmaps:**
- Enable for 3D textures (reduce aliasing, improve performance)
- Disable for UI (saves memory, no benefit)

### Texture Atlasing

Combine multiple textures into one:

```csharp
// Multiple materials (many draw calls)
Material woodMat;
Material stoneMat;
Material metalMat;

// Single atlas material (one draw call)
Material atlasMaterial;  // Combined texture with all variants
```

**Atlas tools:**
- Sprite Atlas (2D)
- Texture Packer (external tool)
- Manual atlas in DCC tool

**Benefits:**
- Fewer draw calls (same material)
- Reduced memory (single texture)
- Better batching

**Drawbacks:**
- More complex UV mapping
- Harder to maintain
- Wasted space if sprites vary in size

## Lighting Optimization

### Baked Lighting

Precompute lighting at build time:

**Mark lights as Baked:**
```
Light > Mode: Baked
```

**Mark objects as Lightmap Static:**
```
GameObject > Lightmap Static checkbox
```

**Bake lighting:**
```
Window > Rendering > Lighting > Generate Lighting
```

**Benefits:**
- No runtime lighting cost
- High quality shadows/GI
- Best performance

**Limitations:**
- Static only
- Longer bake times
- Larger build size (lightmaps)

### Mixed Lighting

Combine baked and real-time:

**Light > Mode: Mixed**

**Modes:**
- **Baked Indirect**: Direct light real-time, indirect baked
- **Subtractive**: One real-time light shadows, rest baked
- **Shadowmask**: Baked shadows, real-time light

**Use for:**
- Dynamic objects in static environment
- Moving characters with realistic lighting

### Real-Time Lighting

Dynamic lights calculated each frame:

**Light > Mode: Realtime**

**Optimization:**
- Limit real-time lights to 1-2 per object
- Use small light range
- Disable shadows when possible
- Prefer Baked/Mixed when object is static

**Pixel Light limit:**
```
Edit > Project Settings > Quality > Pixel Light Count
```
Limit simultaneous pixel lights. Remaining lights use vertex lighting (lower quality, better performance).

## Particle System Optimization

### Particle Count

Reduce max particles:

```csharp
ParticleSystem.MainModule main = ps.main;
main.maxParticles = 50;  // Lower limit
```

**Guidelines:**
- Mobile: <200 particles per system
- PC: <500-1000 particles per system
- VR: <100 particles per system

### Particle Rendering

**Optimization settings:**
- **Rendering Mode**: Billboard (fastest), Stretched Billboard, Mesh (slowest)
- **Max Particle Size**: Lower values are faster
- **Disable features**: Collision, shadows if not needed

```csharp
// Disable collision
ParticleSystem.CollisionModule collision = ps.collision;
collision.enabled = false;

// Disable shadows
renderer.shadowCastingMode = ShadowCastingMode.Off;
renderer.receiveShadows = false;
```

### Particle Shader

Use simple mobile particle shaders:
- Particles/Standard Unlit
- Particles/Additive
- Custom simple shaders

Avoid:
- Particles/Standard Surface (expensive)
- Complex lighting in particle shaders

## UI Optimization

### Canvas Optimization

**Canvas render modes:**
- **Screen Space - Overlay**: No camera, fastest
- **Screen Space - Camera**: Camera-based, supports post-processing
- **World Space**: 3D canvas, most expensive

**Nested canvases:**
```
Main Canvas (Screen Space - Overlay)
└── Static UI Canvas (nested, won't rebuild)
└── Dynamic UI Canvas (nested, rebuilds frequently)
```

Separate static/dynamic elements to different canvases - only dynamic canvas rebuilds.

### Canvas Batching

**Batch requirements:**
- Same material
- Same texture (atlas)
- Same Z-depth
- No overlapping with different materials

**Break batching:**
- Different materials
- Mask components
- Z-depth changes (overlapping different materials)

### Raycast Targets

Disable raycast on non-interactive UI:

```csharp
// ❌ BAD - All Images are raycast targets by default
Image backgroundImage;  // Raycast Target = true (unnecessary)

// ✅ GOOD - Disable raycast on non-interactive elements
Image backgroundImage;
backgroundImage.raycastTarget = false;  // Not clickable, skip raycast
```

**Optimization**: Reduces raycast overhead, especially on complex UI.

### Canvas Rebuilds

Minimize canvas rebuilds:

**Triggers:**
- Position/size changes
- Text changes
- Color changes
- Enable/disable

**Optimization:**
```csharp
// ❌ BAD - Updates text every frame
private void Update()
{
    scoreText.text = "Score: " + score;  // Rebuilds canvas every frame!
}

// ✅ GOOD - Update only when changed
private int lastScore = -1;

private void UpdateScore(int newScore)
{
    if (score != lastScore)
    {
        lastScore = newScore;
        scoreText.text = "Score: " + newScore;  // Rebuild only on change
    }
}
```

## Profiling Rendering

### Frame Debugger

**Window > Analysis > Frame Debugger**

**Features:**
- Step through each draw call
- View batching/instancing
- Inspect material properties
- Identify batching breaks

**Usage:**
1. Enable Frame Debugger
2. Run game
3. Inspect specific frame
4. Find expensive draw calls
5. Identify batching issues

### GPU Profiler

**Profiler Window > GPU**

**Metrics:**
- Total GPU time
- Rendering.Camera.Render (main rendering cost)
- Shadows.RenderShadowMap
- Opaque/Transparent geometry

**Optimization targets:**
- GPU time <16.67ms for 60 FPS
- Identify spikes (shadow rendering, post-processing)

### Graphics Jobs

Enable multi-threaded rendering:

**Edit > Project Settings > Player > Other Settings**
- **Graphics Jobs**: Distribute rendering across CPU cores

**Benefits:**
- Reduces main thread rendering overhead
- Better CPU utilization
- Higher FPS with same draw calls

**Limitations:**
- Not all platforms support
- Slight overhead on simple scenes

## Best Practices Summary

✅ **DO:**
- Enable GPU Instancing for identical meshes
- Use static batching for static environment
- Implement LOD for distant objects
- Bake lighting when possible
- Compress textures appropriately
- Use Material Property Blocks for variations
- Profile with Frame Debugger regularly
- Separate static/dynamic UI canvases
- Disable raycast targets on non-interactive UI
- Use occlusion culling on complex scenes

❌ **DON'T:**
- Access `.material` property repeatedly (instantiates)
- Create many unique materials (breaks batching)
- Use real-time lights unnecessarily
- Forget to enable GPU instancing on materials
- Use high-res uncompressed textures
- Enable shadows on all lights
- Have massive particle counts
- Rebuild UI canvas every frame
- Use dynamic batching for everything (CPU cost)

**Golden rule**: Profile rendering with Frame Debugger and GPU Profiler. Measure draw calls, batches, and GPU time. Optimize based on data, not assumptions.

Follow these rendering optimization techniques for smooth, high-framerate Unity games across all platforms.
