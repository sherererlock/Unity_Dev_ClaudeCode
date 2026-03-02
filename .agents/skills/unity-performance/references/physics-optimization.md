# Unity Physics Optimization - Complete Guide

Complete guide to optimizing Unity's physics system for better performance: collision optimization, Rigidbody best practices, raycasting, and physics settings.

## Physics Update Fundamentals

### FixedUpdate Timing

Physics updates at fixed timestep (default: 0.02s = 50 Hz):

```csharp
// Configure timestep
// Edit > Project Settings > Time > Fixed Timestep
Time.fixedDeltaTime = 0.02f;  // 50 FPS physics
```

**Performance impact:**
- Lower timestep (0.01 = 100 Hz): More accurate, more CPU cost
- Higher timestep (0.04 = 25 Hz): Less accurate, less CPU cost

**Recommended:**
- **Desktop**: 0.02s (50 Hz)
- **Mobile**: 0.025-0.033s (30-40 Hz)
- **VR**: 0.0111s (90 Hz) to match display

### FixedUpdate Cost

FixedUpdate runs independent of framerate:

**60 FPS game, 50 Hz physics:**
- Some frames: 0 FixedUpdates
- Some frames: 1 FixedUpdate
- Some frames: 2 FixedUpdates (catch-up)

**Profile with:**
```
Profiler > CPU > FixedUpdate.PhysicsFixedUpdate
```

High cost indicates physics optimization needed.

## Layer-Based Collision

Most impactful physics optimization.

### Physics Layer Collision Matrix

**Configure: Edit > Project Settings > Physics > Layer Collision Matrix**

**Example setup:**
```
Layer 8:  Player
Layer 9:  Enemies
Layer 10: Projectiles
Layer 11: Environment
Layer 12: Triggers
Layer 13: Ragdolls
```

**Disable unnecessary collisions:**
- Player vs Player ❌
- Enemies vs Enemies ❌
- Projectiles vs Projectiles ❌
- Triggers vs Triggers ❌
- Ragdolls vs Ragdolls ❌ (unless needed)

**Enable only needed collisions:**
- Player vs Environment ✅
- Player vs Enemies ✅
- Projectiles vs Enemies ✅
- Projectiles vs Environment ✅

**Performance gain**: 30-70% reduction in collision checks.

### Setting Layers

```csharp
// Set layer in code
gameObject.layer = LayerMask.NameToLayer("Player");

// Set layer recursively
private void SetLayerRecursive(GameObject obj, int layer)
{
    obj.layer = layer;
    foreach (Transform child in obj.transform)
    {
        SetLayerRecursive(child.gameObject, layer);
    }
}

// Usage
SetLayerRecursive(character, LayerMask.NameToLayer("Player"));
```

### Trigger Layers

Separate trigger colliders to dedicated layer:

```csharp
// Trigger layer setup
Layer 12: Triggers

// Disable Triggers vs Triggers in collision matrix
// Enable Triggers vs Player, Enemies, etc.
```

**Benefits:**
- Explicit control over trigger interactions
- Prevents unintended trigger-trigger collisions

## Rigidbody Optimization

### Rigidbody Sleep

Rigidbodies sleep when velocity falls below threshold:

**Configure: Edit > Project Settings > Physics**
- **Sleep Threshold**: Velocity below which Rigidbody sleeps (default: 0.005)
- **Default Contact Offset**: Surface collision tolerance (default: 0.01)

**Sleep behavior:**
```csharp
// Check if sleeping
if (rb.IsSleeping())
{
    // Rigidbody not simulated - free performance
}

// Force sleep
rb.Sleep();

// Wake up
rb.WakeUp();

// Prevent sleep
rb.sleepThreshold = 0;  // Never sleeps
```

**Optimization:**
- Sleeping Rigidbodies don't consume physics CPU
- Let objects sleep when stationary
- Wake only when needed

**Manual wake example:**
```csharp
public class Projectile : MonoBehaviour
{
    private Rigidbody rb;

    private void Awake()
    {
        rb = GetComponent<Rigidbody>();
    }

    public void Launch(Vector3 force)
    {
        rb.WakeUp();  // Ensure awake before applying force
        rb.AddForce(force, ForceMode.Impulse);
    }
}
```

### Rigidbody Constraints

Freeze unnecessary axes:

```csharp
// Freeze rotation on XZ axes (character controller)
rb.constraints = RigidbodyConstraints.FreezeRotationX | RigidbodyConstraints.FreezeRotationZ;

// Freeze position Y (top-down game)
rb.constraints = RigidbodyConstraints.FreezePositionY;

// Freeze all rotation (prevent tipping)
rb.constraints = RigidbodyConstraints.FreezeRotation;
```

**Performance**: Constraints reduce degrees of freedom, simplifying physics calculations.

### Continuous vs Discrete Collision

**Collision Detection modes:**

**Discrete** (default, fastest):
- Checks collision at current position
- Fast moving objects can tunnel through thin obstacles
- Best for slow-moving or large objects

**Continuous** (moderate cost):
- Uses swept collision detection
- Prevents tunneling for fast projectiles
- Best for player/important objects

**Continuous Dynamic** (expensive):
- Continuous against both static and dynamic objects
- Highest accuracy, highest cost
- Use sparingly (critical fast objects only)

**Continuous Speculative** (good balance):
- Unity 2020.2+, recommended for most uses
- Good tunneling prevention, lower cost than Continuous
- Works with triggers

```csharp
// Set collision detection
rb.collisionDetectionMode = CollisionDetectionMode.ContinuousSpeculative;
```

**Recommendation:**
- Static/slow objects: Discrete
- Player/projectiles: Continuous Speculative
- Critical fast objects: Continuous Dynamic (sparingly)

### Interpolation

Smooth Rigidbody movement between FixedUpdate calls:

```csharp
// None (default) - No smoothing, can appear choppy
rb.interpolation = RigidbodyInterpolation.None;

// Interpolate - Smooth based on previous frame
rb.interpolation = RigidbodyInterpolation.Interpolate;

// Extrapolate - Predict next frame (higher latency risk)
rb.interpolation = RigidbodyInterpolation.Extrapolate;
```

**Use Interpolate for:**
- Player characters
- Camera-following objects
- Visible physics objects

**Use None for:**
- Effects/particles
- Objects far from camera
- When performance critical

**Cost**: Minimal CPU overhead, visual smoothness benefit.

## Collider Optimization

### Collider Complexity

Collider types ranked by performance (fastest to slowest):

1. **Sphere Collider** - Fastest
2. **Capsule Collider** - Very fast
3. **Box Collider** - Fast
4. **Mesh Collider (convex)** - Moderate
5. **Mesh Collider (non-convex)** - Slow

**Recommendations:**
- Use primitives (sphere, capsule, box) when possible
- Combine multiple primitives instead of mesh collider
- Use convex mesh colliders only when necessary
- Avoid non-convex mesh colliders (static objects only)

### Compound Colliders

Multiple simple colliders instead of complex mesh collider:

```csharp
// ❌ BAD - Complex mesh collider (slow)
MeshCollider complexCollider;

// ✅ GOOD - Compound primitive colliders (fast)
Character (GameObject)
├── Body (Capsule Collider)
├── Head (Sphere Collider)
└── Weapon (Box Collider)
```

**Benefits:**
- Faster collision detection
- Better performance with many objects
- More control over collision layers

### Mesh Collider Best Practices

**Convex mesh colliders:**
```csharp
meshCollider.convex = true;
```

**Requirements for dynamic objects:**
- Rigidbody + Mesh Collider must be convex
- Non-convex colliders only for static objects

**Optimization:**
- Reduce mesh complexity (decimation)
- Use convex hull approximation
- Prefer compound primitives

**Non-convex colliders:**
- Static environment only
- No Rigidbody
- High detail terrain/buildings

### Trigger Colliders

Use triggers for detection without physics response:

```csharp
collider.isTrigger = true;
```

**Benefits:**
- Cheaper than full collision
- No physics forces applied
- Good for detection zones (pickups, damage areas, triggers)

**Trigger optimization:**
- Dedicated trigger layer
- Simpler collider shapes (sphere, box)
- Disable when not needed

```csharp
public class PickupTrigger : MonoBehaviour
{
    private SphereCollider triggerCollider;

    private void Awake()
    {
        triggerCollider = GetComponent<SphereCollider>();
        triggerCollider.isTrigger = true;
        gameObject.layer = LayerMask.NameToLayer("Triggers");
    }

    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            Collect();
            triggerCollider.enabled = false;  // Disable after collection
        }
    }
}
```

## Raycasting Optimization

### Raycasting Basics

Raycasts have cost - optimize usage:

```csharp
// Basic raycast (checks all layers)
bool hit = Physics.Raycast(origin, direction, out RaycastHit hitInfo);

// Optimized raycast (layer mask, max distance)
int layerMask = 1 << LayerMask.NameToLayer("Enemy");
bool hit = Physics.Raycast(origin, direction, out RaycastHit hitInfo, maxDistance, layerMask);
```

**Performance factors:**
1. **Layer mask**: Limits objects checked
2. **Max distance**: Early ray termination
3. **Frequency**: Fewer raycasts per frame
4. **Query trigger interaction**: Skip triggers if not needed

### Layer Mask Optimization

```csharp
// ❌ BAD - No layer mask (checks everything)
private void Update()
{
    Physics.Raycast(origin, direction, out RaycastHit hit);
}

// ✅ GOOD - Cache layer mask, limit distance
private int targetLayerMask;
private const float maxRayDistance = 100f;

private void Awake()
{
    targetLayerMask = LayerMask.GetMask("Enemy", "Environment");
}

private void Update()
{
    Physics.Raycast(origin, direction, out RaycastHit hit, maxRayDistance, targetLayerMask);
}
```

**Layer mask creation:**
```csharp
// Single layer
int layerMask = 1 << LayerMask.NameToLayer("Enemy");

// Multiple layers
int layerMask = LayerMask.GetMask("Enemy", "Environment");

// Inverse (everything except)
int layerMask = ~(1 << LayerMask.NameToLayer("Ignore"));
```

### RaycastNonAlloc

Avoid allocation with NonAlloc variants:

```csharp
// ❌ BAD - RaycastAll allocates array
private void Update()
{
    RaycastHit[] hits = Physics.RaycastAll(origin, direction);  // GC allocation!

    foreach (var hit in hits)
    {
        ProcessHit(hit);
    }
}

// ✅ GOOD - RaycastNonAlloc reuses array
private RaycastHit[] hits = new RaycastHit[10];

private void Update()
{
    int hitCount = Physics.RaycastNonAlloc(origin, direction, hits, maxDistance, layerMask);

    for (int i = 0; i < hitCount; i++)
    {
        ProcessHit(hits[i]);
    }
}
```

**NonAlloc variants:**
- `Physics.RaycastNonAlloc()`
- `Physics.SphereCastNonAlloc()`
- `Physics.BoxCastNonAlloc()`
- `Physics.CapsuleCastNonAlloc()`
- `Physics.OverlapSphereNonAlloc()`

### Raycast Frequency

Reduce raycast frequency:

```csharp
// ❌ BAD - Raycast every frame
private void Update()
{
    Physics.Raycast(origin, direction, out RaycastHit hit);
}

// ✅ GOOD - Raycast every N frames
private int frameCounter = 0;
private const int raycastInterval = 5;

private void Update()
{
    frameCounter++;
    if (frameCounter >= raycastInterval)
    {
        frameCounter = 0;
        Physics.Raycast(origin, direction, out RaycastHit hit);
    }
}

// ✅ BETTER - Coroutine with interval
private void Start()
{
    StartCoroutine(RaycastRoutine());
}

private IEnumerator RaycastRoutine()
{
    WaitForSeconds wait = new WaitForSeconds(0.1f);

    while (true)
    {
        Physics.Raycast(origin, direction, out RaycastHit hit);
        yield return wait;
    }
}
```

### Query Trigger Interaction

Control whether raycasts hit triggers:

```csharp
// Ignore triggers (faster)
Physics.Raycast(origin, direction, out RaycastHit hit, maxDistance, layerMask, QueryTriggerInteraction.Ignore);

// Collide with triggers only
Physics.Raycast(origin, direction, out RaycastHit hit, maxDistance, layerMask, QueryTriggerInteraction.Collide);

// Use global setting (default)
Physics.Raycast(origin, direction, out RaycastHit hit, maxDistance, layerMask, QueryTriggerInteraction.UseGlobal);
```

**Global setting:**
```
Edit > Project Settings > Physics > Queries Hit Triggers
```

## Physics Queries Optimization

### OverlapSphere Optimization

```csharp
// ❌ BAD - OverlapSphere allocates
private void Update()
{
    Collider[] colliders = Physics.OverlapSphere(position, radius);  // GC allocation
}

// ✅ GOOD - OverlapSphereNonAlloc
private Collider[] colliders = new Collider[20];

private void Update()
{
    int count = Physics.OverlapSphereNonAlloc(position, radius, colliders, layerMask);

    for (int i = 0; i < count; i++)
    {
        ProcessCollider(colliders[i]);
    }
}
```

### Trigger Zones vs Polling

**Polling approach (CPU intensive):**
```csharp
// ❌ BAD - Check distance every frame for all enemies
private void Update()
{
    foreach (var enemy in enemies)
    {
        if (Vector3.Distance(transform.position, enemy.position) < detectRange)
        {
            OnEnemyDetected(enemy);
        }
    }
}
```

**Trigger approach (event-driven, efficient):**
```csharp
// ✅ GOOD - Use trigger collider
private SphereCollider detectionTrigger;

private void Awake()
{
    detectionTrigger = gameObject.AddComponent<SphereCollider>();
    detectionTrigger.isTrigger = true;
    detectionTrigger.radius = detectRange;
}

private void OnTriggerEnter(Collider other)
{
    if (other.CompareTag("Enemy"))
    {
        OnEnemyDetected(other.GetComponent<Enemy>());
    }
}
```

**Benefits:**
- Event-driven (only fires when needed)
- Physics engine handles detection
- Scales better with many objects

## Character Controller vs Rigidbody

### Character Controller

Custom physics for characters:

```csharp
private CharacterController controller;

private void Awake()
{
    controller = GetComponent<CharacterController>();
}

private void Update()
{
    Vector3 move = transform.forward * speed * Time.deltaTime;
    controller.Move(move);
}
```

**Pros:**
- Lighter than Rigidbody
- Custom collision response
- Built-in slope handling
- No continuous force application needed

**Cons:**
- No automatic physics (gravity, forces)
- Manual collision handling
- Not affected by other Rigidbodies

**Use for:**
- Player characters (FPS, third-person)
- NPCs with custom movement
- When full physics not needed

### Rigidbody Character

Full physics simulation:

```csharp
private Rigidbody rb;

private void Awake()
{
    rb = GetComponent<Rigidbody>();
    rb.constraints = RigidbodyConstraints.FreezeRotation;
}

private void FixedUpdate()
{
    rb.MovePosition(rb.position + velocity * Time.fixedDeltaTime);
}
```

**Pros:**
- Full physics integration
- Affected by forces, collisions
- Realistic interactions

**Cons:**
- Heavier performance
- Harder to control precisely
- Can be pushed by objects

**Use for:**
- Physics-based gameplay (ragdolls, vehicles)
- Objects that need realistic physics
- When interaction with Rigidbodies required

## Physics Settings Optimization

### Project Settings

**Edit > Project Settings > Physics**

**Key settings:**

**Gravity:**
```csharp
Physics.gravity = new Vector3(0, -9.81f, 0);  // Default
```
Adjust for game feel (lower gravity = floaty, higher = heavy).

**Default Material:**
- Friction: 0.4 (default)
- Bounciness: 0

**Sleep Threshold**: 0.005 (objects sleep below this velocity)

**Default Contact Offset**: 0.01 (collision skin width)

**Queries Hit Backfaces**: false (optimize raycasts)

**Queries Hit Triggers**: true (include triggers in queries - disable if not needed)

**Layer Collision Matrix**: Configure per-layer collision (most important setting)

### Auto Simulation

Disable auto-simulation for manual control:

```csharp
// Disable auto-simulation
Physics.autoSimulation = false;

// Manual simulation
private void FixedUpdate()
{
    Physics.Simulate(Time.fixedDeltaTime);
}
```

**Use cases:**
- Custom physics update timing
- Pausing physics independently
- Advanced control flow

### Solver Iterations

**Edit > Project Settings > Physics**

**Default Solver Iterations**: 6 (position)
**Default Solver Velocity Iterations**: 1

**Higher values:**
- More accurate physics
- Higher CPU cost
- Better for complex joints/ragdolls

**Lower values:**
- Faster physics
- Less accurate
- OK for simple collision

**Recommendation:**
- Default (6, 1) for most games
- Increase for ragdolls/vehicles (8-10, 2-4)
- Decrease for mobile (4, 1)

## Physics Profiling

### Physics Profiler

**Profiler Window > Physics**

**Key metrics:**
- **Physics.Processing**: Total physics time
- **Physics.Contacts**: Collision detection
- **Physics.Solver**: Constraint solving
- **Physics.Callbacks**: OnCollision/OnTrigger callbacks

**Targets:**
- <2ms for 60 FPS (desktop)
- <4ms for 30 FPS (mobile)

### Physics Debugger

**Window > Analysis > Physics Debugger**

**Features:**
- Visualize colliders
- Show active contacts
- View Rigidbody states (sleeping, awake)
- Identify collision pairs

**Usage:**
1. Enable Physics Debugger
2. Run game
3. View active collisions
4. Identify unexpected collision pairs
5. Optimize layer collision matrix

## Best Practices Summary

✅ **DO:**
- Configure Layer Collision Matrix (biggest optimization)
- Use primitive colliders (sphere, capsule, box) when possible
- Let Rigidbodies sleep when stationary
- Cache layer masks for raycasts
- Use NonAlloc physics queries
- Reduce raycast frequency (not every frame)
- Use trigger colliders for detection zones
- Freeze unnecessary Rigidbody constraints
- Use Continuous Speculative collision for fast objects
- Profile with Physics Profiler regularly

❌ **DON'T:**
- Enable all layer collisions (massive overhead)
- Use non-convex mesh colliders on dynamic objects
- Raycast every frame without layer mask
- Use RaycastAll (allocates - use NonAlloc)
- Set sleep threshold to 0 (prevents sleeping)
- Use high solver iterations unnecessarily
- Forget to set object layers
- Use Mesh Colliders when primitives work
- Poll for nearby objects (use triggers instead)

**Golden rule**: Configure Layer Collision Matrix first. This single optimization can reduce physics cost by 50%+. Profile with Physics Profiler to identify bottlenecks.

Follow these physics optimization techniques for responsive, performant physics in Unity games.
