# Unity Component Architecture - Best Practices

Complete guide to component-based architecture in Unity, GetComponent patterns, communication strategies, and composition techniques.

## Component-Based Design Principles

### Single Responsibility

Each component handles one aspect of GameObject behavior:

```csharp
// ❌ BAD - God component doing everything
public class Player : MonoBehaviour
{
    public int health;
    public float moveSpeed;
    public Rigidbody rb;

    void Update()
    {
        HandleInput();
        Move();
        Attack();
        UpdateUI();
        CheckHealth();
    }
}

// ✅ GOOD - Separate components
public class PlayerHealth : MonoBehaviour { }
public class PlayerMovement : MonoBehaviour { }
public class PlayerCombat : MonoBehaviour { }
public class PlayerInput : MonoBehaviour { }
```

**Benefits:**
- Easier testing
- Better reusability
- Simpler debugging
- Clear responsibilities

### Composition Over Inheritance

Favor component composition instead of deep inheritance hierarchies:

```csharp
// ❌ BAD - Deep inheritance
public class Character : MonoBehaviour { }
public class Enemy : Character { }
public class FlyingEnemy : Enemy { }
public class FastFlyingEnemy : FlyingEnemy { }

// ✅ GOOD - Component composition
public class Character : MonoBehaviour
{
    private Health health;
    private Movement movement;
    private Combat combat;
}

// Create variations through component configuration
// FastFlyingEnemy = Character + FlyingMovement + FastSpeed + MeleeAttack
```

## GetComponent Patterns

### Caching Component References

**Always cache GetComponent results** - never call repeatedly:

```csharp
// ❌ BAD - Repeated GetComponent calls
private void Update()
{
    GetComponent<Rigidbody>().velocity = Vector3.forward;  // SLOW!
    GetComponent<Animator>().SetBool("walking", true);     // SLOW!
}

// ✅ GOOD - Cache once in Awake
private Rigidbody rb;
private Animator animator;

private void Awake()
{
    rb = GetComponent<Rigidbody>();
    animator = GetComponent<Animator>();
}

private void Update()
{
    rb.velocity = Vector3.forward;         // FAST
    animator.SetBool("walking", true);     // FAST
}
```

**Performance impact**: GetComponent is expensive. Caching provides 10-100x speedup.

### TryGetComponent Pattern

Use `TryGetComponent` for optional components:

```csharp
// ❌ BAD - Null check after GetComponent
Renderer renderer = GetComponent<Renderer>();
if (renderer != null)
{
    renderer.enabled = false;
}

// ✅ GOOD - TryGetComponent (Unity 2019.2+)
if (TryGetComponent<Renderer>(out var renderer))
{
    renderer.enabled = false;
}
```

**Benefits:**
- Single call (faster)
- Clear intent
- No null assignment

### RequireComponent Attribute

Declare component dependencies:

```csharp
[RequireComponent(typeof(Rigidbody))]
[RequireComponent(typeof(Collider))]
public class PlayerMovement : MonoBehaviour
{
    private Rigidbody rb;

    private void Awake()
    {
        rb = GetComponent<Rigidbody>();  // Guaranteed to exist
    }
}
```

**Guarantees:**
- Component automatically added if missing
- Cannot be removed while dependent exists
- Clear dependency documentation

### GetComponentInChildren Pattern

Search this GameObject and children:

```csharp
// Find first Animator in hierarchy
private Animator animator;

private void Awake()
{
    animator = GetComponentInChildren<Animator>();
}

// Find all Colliders in children
private Collider[] colliders;

private void Awake()
{
    colliders = GetComponentsInChildren<Collider>();
}

// Include inactive GameObjects
private Transform[] bones;

private void Awake()
{
    bones = GetComponentsInChildren<Transform>(includeInactive: true);
}
```

**Use cases:**
- Finding components in child objects
- Character rigs (Animator, bones)
- UI hierarchies

**Performance:** Expensive - cache results, don't call in Update.

### GetComponentInParent Pattern

Search this GameObject and parents:

```csharp
// Find Canvas parent
private Canvas canvas;

private void Awake()
{
    canvas = GetComponentInParent<Canvas>();
}

// Check if part of UI hierarchy
private bool IsUIElement()
{
    return GetComponentInParent<Canvas>() != null;
}
```

**Use cases:**
- UI elements finding Canvas
- Nested systems finding root manager
- Hierarchical queries

## Component Communication Patterns

### Direct Reference (Inspector)

Most common pattern for known relationships:

```csharp
public class HealthDisplay : MonoBehaviour
{
    [SerializeField] private PlayerHealth playerHealth;

    private void OnEnable()
    {
        playerHealth.OnHealthChanged += UpdateDisplay;
    }

    private void OnDisable()
    {
        playerHealth.OnHealthChanged -= UpdateDisplay;
    }

    private void UpdateDisplay(int health)
    {
        // Update UI
    }
}
```

**Pros:**
- Fast (direct reference)
- Clear in Inspector
- Compile-time safety

**Cons:**
- Manual wiring required
- Tight coupling

**When to use:** Known, fixed relationships

### GetComponent (Same GameObject)

For components on same GameObject:

```csharp
public class CharacterController : MonoBehaviour
{
    private CharacterMovement movement;
    private CharacterCombat combat;
    private CharacterInventory inventory;

    private void Awake()
    {
        movement = GetComponent<CharacterMovement>();
        combat = GetComponent<CharacterCombat>();
        inventory = GetComponent<CharacterInventory>();
    }

    public void Attack()
    {
        if (combat.CanAttack())
        {
            combat.PerformAttack();
            movement.StopMovement();
        }
    }
}
```

**When to use:** Components on same GameObject that need to coordinate

### Event System (Decoupled)

For loose coupling and many-to-many relationships:

```csharp
// Define events
public class PlayerHealth : MonoBehaviour
{
    public event Action<int> OnHealthChanged;
    public event Action OnDeath;

    private int health = 100;

    public void TakeDamage(int damage)
    {
        health -= damage;
        OnHealthChanged?.Invoke(health);

        if (health <= 0)
            OnDeath?.Invoke();
    }
}

// Subscribe from any component
public class HealthBar : MonoBehaviour
{
    [SerializeField] private PlayerHealth playerHealth;

    private void OnEnable()
    {
        playerHealth.OnHealthChanged += UpdateBar;
        playerHealth.OnDeath += OnPlayerDeath;
    }

    private void OnDisable()
    {
        playerHealth.OnHealthChanged -= UpdateBar;
        playerHealth.OnDeath -= OnPlayerDeath;
    }

    private void UpdateBar(int health) { }
    private void OnPlayerDeath() { }
}
```

**Pros:**
- Loose coupling
- Multiple subscribers
- Easy to extend

**Cons:**
- Memory leak risk (forgot unsubscribe)
- Harder to debug

**Critical:** Always unsubscribe in OnDisable/OnDestroy

### Message System (Unity Built-in)

Use sparingly - less performant than direct calls:

```csharp
// Receiver
public class TargetComponent : MonoBehaviour
{
    private void TakeDamage(int damage)
    {
        // Handle damage
    }
}

// Sender
GameObject target = ...;
target.SendMessage("TakeDamage", 10);  // Calls TakeDamage on all components

// Variants
SendMessage("Method", value, SendMessageOptions.DontRequireReceiver);
SendMessageUpwards("Method", value);  // This + parents
BroadcastMessage("Method", value);    // This + children
```

**Pros:**
- No reference needed
- Works across components

**Cons:**
- String-based (no compile-time check)
- Slow (reflection-based)
- Hard to track

**When to use:** Rarely. Prefer events or interfaces.

### Interface Pattern

Type-safe communication without tight coupling:

```csharp
// Define interface
public interface IDamageable
{
    void TakeDamage(int damage);
    int CurrentHealth { get; }
}

// Implement interface
public class PlayerHealth : MonoBehaviour, IDamageable
{
    [SerializeField] private int maxHealth = 100;
    private int health;

    public int CurrentHealth => health;

    private void Awake()
    {
        health = maxHealth;
    }

    public void TakeDamage(int damage)
    {
        health -= damage;
        if (health <= 0)
            Die();
    }

    private void Die() { }
}

// Use interface
public class Weapon : MonoBehaviour
{
    private void OnTriggerEnter(Collider other)
    {
        var damageable = other.GetComponent<IDamageable>();
        if (damageable != null)
        {
            damageable.TakeDamage(10);
        }
    }
}
```

**Pros:**
- Type-safe
- Loose coupling
- Clear contracts

**Cons:**
- GetComponent<Interface> is slow (cache results)
- No Inspector support

**Pattern for performance:**
```csharp
// Cache interface references
private IDamageable cachedDamageable;

private void OnTriggerEnter(Collider other)
{
    if (cachedDamageable == null)
        cachedDamageable = other.GetComponent<IDamageable>();

    cachedDamageable?.TakeDamage(10);
}
```

### Singleton Pattern

For global managers (use sparingly):

```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
        DontDestroyOnLoad(gameObject);
    }
}

// Access from anywhere
public class AnyScript : MonoBehaviour
{
    private void Start()
    {
        GameManager.Instance.DoSomething();
    }
}
```

**Use for:**
- Game managers
- Audio managers
- Input managers
- Save managers

**Avoid for:**
- Everything else (creates global state)

## Component Organization Patterns

### Coordinator Pattern

One component coordinates multiple specialized components:

```csharp
public class Character : MonoBehaviour
{
    // Coordinated components
    private CharacterMovement movement;
    private CharacterCombat combat;
    private CharacterInventory inventory;
    private CharacterAnimation animation;

    private void Awake()
    {
        movement = GetComponent<CharacterMovement>();
        combat = GetComponent<CharacterCombat>();
        inventory = GetComponent<CharacterInventory>();
        animation = GetComponent<CharacterAnimation>();
    }

    public void Attack()
    {
        if (combat.CanAttack())
        {
            movement.StopMovement();
            combat.PerformAttack();
            animation.PlayAttackAnimation();
        }
    }

    public void UseItem(Item item)
    {
        inventory.RemoveItem(item);
        item.Use(this);
        animation.PlayUseItemAnimation();
    }
}
```

**Benefits:**
- Clear entry point
- Coordinates complex behaviors
- Each specialized component stays focused

### Data Component Pattern

Separate data from behavior:

```csharp
// Data component
[System.Serializable]
public class CharacterStats
{
    public int maxHealth = 100;
    public float moveSpeed = 5f;
    public int attackDamage = 25;
}

// Behavior components reference data
public class Character : MonoBehaviour
{
    [SerializeField] private CharacterStats stats;

    public int MaxHealth => stats.maxHealth;
    public float MoveSpeed => stats.moveSpeed;
}

public class CharacterMovement : MonoBehaviour
{
    private Character character;

    private void Awake()
    {
        character = GetComponent<Character>();
    }

    private void Update()
    {
        // Use character.MoveSpeed
    }
}
```

**Alternative: ScriptableObject data**
```csharp
[CreateAssetMenu]
public class CharacterData : ScriptableObject
{
    public int maxHealth = 100;
    public float moveSpeed = 5f;
}

public class Character : MonoBehaviour
{
    [SerializeField] private CharacterData data;
}
```

### State Component Pattern

Each state is a separate component:

```csharp
// Base state
public abstract class CharacterState : MonoBehaviour
{
    public abstract void Enter();
    public abstract void Exit();
    public abstract void UpdateState();
}

// Concrete states
public class IdleState : CharacterState
{
    public override void Enter() { }
    public override void Exit() { }
    public override void UpdateState() { }
}

public class MovingState : CharacterState
{
    public override void Enter() { }
    public override void Exit() { }
    public override void UpdateState() { }
}

// State manager
public class CharacterStateMachine : MonoBehaviour
{
    private CharacterState currentState;

    private void Awake()
    {
        // Get all state components
        var states = GetComponents<CharacterState>();
    }

    public void ChangeState(CharacterState newState)
    {
        currentState?.Exit();
        currentState = newState;
        currentState.Enter();
    }

    private void Update()
    {
        currentState?.UpdateState();
    }
}
```

## Performance Considerations

### Component Caching

Cache all component references:

```csharp
// ❌ BAD - Find every frame
private void Update()
{
    var manager = FindObjectOfType<GameManager>();  // EXTREMELY SLOW
    var rb = GetComponent<Rigidbody>();             // SLOW
}

// ✅ GOOD - Cache once
private GameManager manager;
private Rigidbody rb;

private void Awake()
{
    manager = FindObjectOfType<GameManager>();
    rb = GetComponent<Rigidbody>();
}

private void Update()
{
    // Use cached references - FAST
}
```

### Avoid Repeated Searches

Cache search results:

```csharp
// ❌ BAD - Search repeatedly
private void Update()
{
    GameObject[] enemies = GameObject.FindGameObjectsWithTag("Enemy");
    foreach (var enemy in enemies)
    {
        // Process
    }
}

// ✅ GOOD - Maintain list
private List<Enemy> enemies = new List<Enemy>();

// Enemies register/unregister themselves
public void RegisterEnemy(Enemy enemy)
{
    enemies.Add(enemy);
}

public void UnregisterEnemy(Enemy enemy)
{
    enemies.Remove(enemy);
}
```

### Component Pooling

Reuse components instead of destroying:

```csharp
public class BulletPool : MonoBehaviour
{
    [SerializeField] private GameObject bulletPrefab;
    [SerializeField] private int poolSize = 20;

    private Queue<GameObject> pool = new Queue<GameObject>();

    private void Awake()
    {
        for (int i = 0; i < poolSize; i++)
        {
            GameObject bullet = Instantiate(bulletPrefab);
            bullet.SetActive(false);
            pool.Enqueue(bullet);
        }
    }

    public GameObject GetBullet()
    {
        if (pool.Count > 0)
        {
            GameObject bullet = pool.Dequeue();
            bullet.SetActive(true);
            return bullet;
        }

        // Pool exhausted - create new
        return Instantiate(bulletPrefab);
    }

    public void ReturnBullet(GameObject bullet)
    {
        bullet.SetActive(false);
        pool.Enqueue(bullet);
    }
}
```

## Common Anti-Patterns

### God Component

❌ **Don't** create components that do everything:

```csharp
// BAD - 500+ lines, does everything
public class Player : MonoBehaviour
{
    // Movement
    public float moveSpeed;

    // Combat
    public int attackDamage;

    // Inventory
    public List<Item> items;

    // UI
    public Text healthText;

    // Audio
    public AudioClip jumpSound;

    private void Update()
    {
        HandleMovement();
        HandleCombat();
        UpdateInventoryUI();
        CheckAudio();
        // ... 20 more methods
    }
}
```

✅ **Do** split into focused components

### Component Coupling

❌ **Don't** tightly couple components:

```csharp
// BAD - PlayerMovement directly accesses PlayerCombat internals
public class PlayerMovement : MonoBehaviour
{
    private PlayerCombat combat;

    private void Update()
    {
        if (!combat.isAttacking && !combat.isBlocking && combat.stamina > 10)
        {
            Move();
        }
    }
}
```

✅ **Do** use clear interfaces:

```csharp
// GOOD - Clear interface
public class PlayerCombat : MonoBehaviour
{
    public bool CanMove() => !isAttacking && !isBlocking && stamina > 10;
}

public class PlayerMovement : MonoBehaviour
{
    private PlayerCombat combat;

    private void Update()
    {
        if (combat.CanMove())
            Move();
    }
}
```

### Circular Dependencies

❌ **Don't** create circular component references:

```csharp
// BAD - A needs B, B needs A
public class ComponentA : MonoBehaviour
{
    private ComponentB b;
}

public class ComponentB : MonoBehaviour
{
    private ComponentA a;
}
```

✅ **Do** use events or coordinator:

```csharp
// GOOD - Use events
public class ComponentA : MonoBehaviour
{
    public event Action OnSomethingHappened;
}

public class ComponentB : MonoBehaviour
{
    [SerializeField] private ComponentA a;

    private void OnEnable()
    {
        a.OnSomethingHappened += HandleEvent;
    }
}
```

## Best Practices Summary

✅ **DO:**
- Cache all GetComponent calls in Awake
- Use RequireComponent for dependencies
- Keep components focused (single responsibility)
- Prefer composition over inheritance
- Use events for decoupling
- Always unsubscribe from events
- Use interfaces for type-safe contracts
- Pool frequently instantiated components

❌ **DON'T:**
- Call GetComponent in Update/FixedUpdate
- Use Find methods in Update
- Create god components
- Tightly couple components
- Create circular dependencies
- Use SendMessage (prefer events/interfaces)
- Forget to unsubscribe from events
- Make everything a singleton

Follow these patterns for clean, performant, maintainable Unity component architecture.
