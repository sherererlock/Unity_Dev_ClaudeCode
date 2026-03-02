# Unity MonoBehaviour Lifecycle - Complete Reference

Complete documentation of all MonoBehaviour lifecycle methods, execution order, and usage patterns.

## Complete Lifecycle Order

### Initialization Phase

1. **Awake()**
   - Called when script instance loads
   - Before scene loads
   - Once per component lifetime
   - Order within same GameObject: undefined
   - **Use for:** Self-initialization, component caching

2. **OnEnable()**
   - Called after Awake() and whenever GameObject activates
   - Can be called multiple times
   - **Use for:** Event subscription, enabling functionality

3. **Start()**
   - Called before first Update(), after all Awake() calls
   - Once per component lifetime (if enabled)
   - Only called if component is enabled
   - **Use for:** Finding other objects, final setup

### Physics Phase

4. **FixedUpdate()**
   - Called at fixed timestep (default: 0.02s = 50 FPS)
   - Independent of frame rate
   - **Use for:** All physics operations (Rigidbody, forces)

### Update Phase

5. **Update()**
   - Called once per frame
   - Variable timing based on frame rate
   - **Use for:** Input, non-physics movement, frame updates

6. **LateUpdate()**
   - Called after all Update() methods
   - **Use for:** Camera following, final position adjustments

### Rendering Phase

7. **OnPreCull()** - Before camera culling
8. **OnBecameVisible()** - When renderer becomes visible to any camera
9. **OnBecameInvisible()** - When renderer no longer visible
10. **OnWillRenderObject()** - For each camera rendering the object
11. **OnPreRender()** - Before camera renders scene
12. **OnRenderObject()** - After all regular rendering
13. **OnPostRender()** - After camera finishes rendering
14. **OnRenderImage()** - Image post-processing

### GUI Phase

15. **OnGUI()** - Called multiple times per frame for GUI events
   - Legacy GUI system (avoid in new projects)

### Gizmo Phase

16. **OnDrawGizmos()** - Draw gizmos in Scene view
17. **OnDrawGizmosSelected()** - Draw gizmos when GameObject selected

### Deactivation Phase

18. **OnDisable()**
   - When GameObject deactivated
   - Before OnDestroy()
   - Can be called multiple times
   - **Use for:** Event unsubscription, cleanup

19. **OnDestroy()**
   - When GameObject destroyed
   - Once per component lifetime
   - **Use for:** Final cleanup, resource release

### Application Lifecycle

20. **OnApplicationPause()** - When application pauses/resumes
21. **OnApplicationFocus()** - When application loses/gains focus
22. **OnApplicationQuit()** - Before application quits

## Method Details

### Awake()

```csharp
private void Awake()
{
    // Self-initialization
    health = maxHealth;
    inventory = new List<Item>();

    // Component caching (THIS GameObject only)
    rb = GetComponent<Rigidbody>();
    animator = GetComponent<Animator>();

    // Singleton setup
    if (Instance == null)
        Instance = this;
    else
        Destroy(gameObject);
}
```

**Execution:**
- Always called, even if component disabled
- Called in random order for multiple scripts on same GameObject
- Happens before scene fully loads

**Common Uses:**
- Initialize fields
- Cache GetComponent results
- Setup singletons
- Create object pools

**Avoid:**
- Referencing other GameObjects (might not exist yet)
- Calling methods on other scripts (may not be initialized)

### Start()

```csharp
private void Start()
{
    // Safe to reference other objects
    player = GameObject.FindWithTag("Player");
    manager = FindObjectOfType<GameManager>();

    // Call methods on other components
    weapon.Initialize(this);
    ui.SetPlayer(this);

    // Register with systems
    GameManager.Instance.RegisterEnemy(this);
}
```

**Execution:**
- Called before first Update()
- After all Awake() calls complete
- Only if component enabled
- Called once per lifetime

**Common Uses:**
- Find other GameObjects
- Reference other scripts
- Final initialization
- Register with managers

### OnEnable() / OnDisable()

```csharp
private void OnEnable()
{
    // Subscribe to events
    GameEvents.OnLevelComplete += HandleLevelComplete;
    InputActions.Jump.performed += OnJump;

    // Enable systems
    StartCoroutine(SpawnEnemies());
    particleSystem.Play();
}

private void OnDisable()
{
    // Unsubscribe from events (CRITICAL!)
    GameEvents.OnLevelComplete -= HandleLevelComplete;
    InputActions.Jump.performed -= OnJump;

    // Disable systems
    StopAllCoroutines();
    particleSystem.Stop();
}
```

**Execution:**
- OnEnable: After Awake(), whenever GameObject activates
- OnDisable: When GameObject deactivates, before OnDestroy()
- Can be called multiple times

**Common Uses:**
- Event subscription/unsubscription
- Enable/disable coroutines
- Activate/deactivate systems
- Pool object activation

**Critical Pattern:**
```csharp
// ALWAYS pair subscribe/unsubscribe
private void OnEnable() => Event += Handler;
private void OnDisable() => Event -= Handler;
```

### Update() vs FixedUpdate()

#### Update()

```csharp
private void Update()
{
    // Input handling (frame-dependent)
    if (Input.GetKeyDown(KeyCode.Space))
        Jump();

    // Non-physics movement
    transform.position += velocity * Time.deltaTime;

    // Frame-based logic
    UpdateAnimation();
    CheckForEnemies();
}
```

**Timing:** Variable, depends on frame rate (60 FPS = 16.67ms, 30 FPS = 33.33ms)

**Use for:**
- Input handling
- Non-physics movement
- Animation updates
- UI updates
- Frame-dependent logic

**Avoid:**
- Physics operations (use FixedUpdate)
- Empty Update() methods (remove them)

#### FixedUpdate()

```csharp
private void FixedUpdate()
{
    // Physics operations
    rb.AddForce(input * moveSpeed);

    // Physics-based movement
    rb.MovePosition(rb.position + velocity * Time.fixedDeltaTime);

    // Apply torque/rotation
    rb.AddTorque(torque);
}
```

**Timing:** Fixed interval, default 0.02s (50 FPS), independent of frame rate

**Use for:**
- Rigidbody operations (AddForce, velocity, MovePosition)
- Physics calculations
- Collision-dependent logic

**Avoid:**
- Input handling (can miss inputs between fixed updates)
- Rendering/animation (can stutter)

**Time Constants:**
- `Time.deltaTime` in Update() - Time since last frame
- `Time.fixedDeltaTime` in FixedUpdate() - Fixed physics timestep

### LateUpdate()

```csharp
private void LateUpdate()
{
    // Camera follow (after player moved)
    Vector3 targetPosition = player.position + offset;
    transform.position = Vector3.Lerp(transform.position, targetPosition, followSpeed * Time.deltaTime);

    // Billboard effect (face camera)
    transform.LookAt(Camera.main.transform);

    // Final position clamping
    transform.position = ClampToBounds(transform.position);
}
```

**Timing:** After all Update() calls complete

**Use for:**
- Camera following (track after objects moved)
- IK (inverse kinematics)
- Final position adjustments
- Billboard effects
- Procedural animation

### OnDestroy()

```csharp
private void OnDestroy()
{
    // Unsubscribe from static events
    GameEvents.OnLevelComplete -= HandleLevelComplete;

    // Release resources
    if (customTexture != null)
        Destroy(customTexture);

    // Notify systems
    GameManager.Instance?.UnregisterEnemy(this);

    // Save data
    SaveProgress();
}
```

**Execution:**
- When GameObject destroyed
- On scene unload
- On application quit
- Once per lifetime

**Use for:**
- Final cleanup
- Unsubscribe from static events
- Resource release
- Save data

## Execution Order Configuration

### Script Execution Order

Configure in **Edit > Project Settings > Script Execution Order**:

```
-1000: GameManager (initialize first)
     0: Default scripts
  1000: CameraController (execute last)
```

**Use when:**
- Manager must initialize before others
- Camera must update after all movement
- Specific initialization order required

**Avoid:**
- Overusing (creates hidden dependencies)
- Negative/positive values indicate order anti-pattern

### Best Practice

Instead of relying on execution order:

```csharp
// ❌ BAD - Depends on execution order
void Start()
{
    manager.DoSomething();  // Works only if manager.Start() already ran
}

// ✅ GOOD - Explicit initialization
void Start()
{
    manager = GameManager.Instance;  // Singleton initialized in Awake
    manager.DoSomething();  // Safe
}
```

## Common Patterns

### Initialization Pattern

```csharp
public class Character : MonoBehaviour
{
    [SerializeField] private int maxHealth = 100;

    private int health;
    private Rigidbody rb;
    private Animator animator;
    private Transform target;

    // 1. Cache components on self
    private void Awake()
    {
        health = maxHealth;
        rb = GetComponent<Rigidbody>();
        animator = GetComponent<Animator>();
    }

    // 2. Find external references
    private void Start()
    {
        target = GameObject.FindWithTag("Player")?.transform;
        GameManager.Instance.RegisterCharacter(this);
    }

    // 3. Subscribe to events
    private void OnEnable()
    {
        GameEvents.OnGameOver += HandleGameOver;
    }

    // 4. Unsubscribe from events
    private void OnDisable()
    {
        GameEvents.OnGameOver -= HandleGameOver;
    }

    // 5. Final cleanup
    private void OnDestroy()
    {
        GameManager.Instance?.UnregisterCharacter(this);
    }
}
```

### Coroutine Pattern

```csharp
private void OnEnable()
{
    StartCoroutine(SpawnEnemies());
}

private void OnDisable()
{
    StopAllCoroutines();  // Clean up when disabled
}

private IEnumerator SpawnEnemies()
{
    while (true)
    {
        SpawnEnemy();
        yield return new WaitForSeconds(spawnDelay);
    }
}
```

### Event Subscription Pattern

```csharp
// CORRECT - Paired subscribe/unsubscribe
private void OnEnable()
{
    PlayerEvents.OnDeath += HandlePlayerDeath;
    InputManager.OnJump += HandleJump;
}

private void OnDisable()
{
    PlayerEvents.OnDeath -= HandlePlayerDeath;
    InputManager.OnJump -= HandleJump;
}

// WRONG - Subscribe in Start, forget to unsubscribe
private void Start()
{
    PlayerEvents.OnDeath += HandlePlayerDeath;  // Memory leak!
}
```

## Performance Considerations

### Empty Methods

```csharp
// ❌ BAD - Empty methods still have overhead
private void Update() { }
private void FixedUpdate() { }

// ✅ GOOD - Remove unused methods
// (No Update or FixedUpdate if not needed)
```

**Impact:** Unity calls all lifecycle methods, even if empty. Remove unused ones.

### Frequent Calls

Methods called every frame:
- Update() - Every frame (~60 FPS = 3600 calls/minute)
- FixedUpdate() - Every physics step (~50 FPS = 3000 calls/minute)
- LateUpdate() - Every frame

**Optimization:** Minimize work in these methods. Use events, coroutines, or invoke when possible.

### GC Allocation

```csharp
// ❌ BAD - Allocates every frame
private void Update()
{
    string message = "Health: " + health;  // String allocation
    GetComponent<Renderer>().material.color = Color.red;  // Material allocation
}

// ✅ GOOD - Cache and avoid allocation
private Renderer rend;
private readonly StringBuilder messageBuilder = new StringBuilder();

private void Awake()
{
    rend = GetComponent<Renderer>();
}

private void Update()
{
    messageBuilder.Clear();
    messageBuilder.Append("Health: ").Append(health);  // No allocation
}
```

## Troubleshooting

### Start() Not Called

**Symptom:** Start() method never executes

**Causes:**
1. Component is disabled in Inspector
2. GameObject is inactive
3. Component disabled in Awake()

**Solution:** Enable component/GameObject

### NullReferenceException in Start()

**Symptom:** Variables are null in Start()

**Causes:**
1. Awake() hasn't run (shouldn't happen)
2. Referenced object doesn't exist
3. Find() methods returning null

**Solution:**
```csharp
private void Start()
{
    target = GameObject.FindWithTag("Player")?.transform;
    if (target == null)
        Debug.LogError("Player not found!");
}
```

### Event Called After OnDisable()

**Symptom:** Event handler called even after GameObject destroyed

**Cause:** Forgot to unsubscribe

**Solution:** Always unsubscribe in OnDisable() or OnDestroy()

```csharp
private void OnDestroy()
{
    // Unsubscribe from static events
    GameEvents.OnLevelComplete -= HandleLevelComplete;
}
```

## Summary

**Initialization Order:**
1. Awake() - Cache components on self
2. OnEnable() - Subscribe to events
3. Start() - Find other objects, final setup

**Update Loop:**
1. FixedUpdate() - Physics
2. Update() - Input, logic
3. LateUpdate() - Camera, final adjustments

**Cleanup:**
1. OnDisable() - Unsubscribe from events
2. OnDestroy() - Final cleanup

**Key Rules:**
- Cache in Awake(), find in Start()
- Physics in FixedUpdate(), input in Update()
- Subscribe in OnEnable(), unsubscribe in OnDisable()
- Remove empty lifecycle methods
- Use events instead of Update() when possible

Follow this lifecycle pattern for reliable, performant Unity code.
