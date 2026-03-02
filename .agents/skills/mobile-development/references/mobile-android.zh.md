# Android 原生开发

Kotlin 和 Jetpack Compose Android 开发完整指南 (2024-2025)。

## Kotlin 2.1 概览

### 主要特性
- **空安全 (Null safety)**：不再有 NullPointerExceptions
- **协程 (Coroutines)**：结构化并发
- **扩展函数 (Extension functions)**：无需继承即可扩展类
- **密封类 (Sealed classes)**：类型安全的状态管理
- **数据类 (Data classes)**：自动生成 equals/hashCode/toString

### 现代 Kotlin 模式

**协程 (Coroutines):**
```kotlin
// 挂起函数
suspend fun fetchUser(id: String): User {
    return withContext(Dispatchers.IO) {
        api.getUser(id)
    }
}

// 在 ViewModel 中使用
viewModelScope.launch {
    try {
        val user = fetchUser("123")
        _uiState.update { it.copy(user = user) }
    } catch (e: Exception) {
        _uiState.update { it.copy(error = e.message) }
    }
}
```

**Flow (响应式流):**
```kotlin
class UserRepository {
    fun observeUsers(): Flow<List<User>> = flow {
        while (true) {
            emit(database.getUsers())
            delay(5000)  // 每 5 秒轮询一次
        }
    }.flowOn(Dispatchers.IO)
}

// 在 ViewModel 中收集
init {
    viewModelScope.launch {
        repository.observeUsers().collect { users ->
            _uiState.update { it.copy(users = users) }
        }
    }
}
```

**密封类 (类型安全状态):**
```kotlin
sealed class UiState {
    object Loading : UiState()
    data class Success(val data: List<User>) : UiState()
    data class Error(val message: String) : UiState()
}

// 模式匹配
when (uiState) {
    is UiState.Loading -> ShowLoader()
    is UiState.Success -> ShowData(uiState.data)
    is UiState.Error -> ShowError(uiState.message)
}
```

## Jetpack Compose

### 为什么要用 Compose?
- **声明式 (Declarative)**：描述 UI 状态，而非命令式指令
- **60% 采用率**：在前 1,000 个应用中 (2024)
- **更少代码**：相比 Views 减少 40%
- **现代**：专为 Kotlin 和协程构建
- **Material 3**：一流支持

### Compose 基础

```kotlin
@Composable
fun UserListScreen(viewModel: UserViewModel = viewModel()) {
    val uiState by viewModel.uiState.collectAsState()

    Column(modifier = Modifier.fillMaxSize()) {
        when (val state = uiState) {
            is UiState.Loading -> {
                CircularProgressIndicator(
                    modifier = Modifier.align(Alignment.CenterHorizontally)
                )
            }
            is UiState.Success -> {
                LazyColumn {
                    items(state.data) { user ->
                        UserItem(user)
                    }
                }
            }
            is UiState.Error -> {
                Text(
                    text = state.message,
                    color = MaterialTheme.colorScheme.error
                )
            }
        }
    }
}

@Composable
fun UserItem(user: User) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    ) {
        Text(
            text = user.name,
            style = MaterialTheme.typography.bodyLarge
        )
    }
}
```

**关键 Composables:**
- `Column/Row/Box`：布局
- `LazyColumn/LazyRow`：Recycler 等价物 (虚拟化)
- `Text/Image/Icon`：内容
- `Button/TextField`：输入
- `Card/Surface`：容器

## 架构模式

### MVVM 配合整洁架构 (Clean Architecture)

```kotlin
// 领域层 (Domain Layer) - 用例 (Use Case)
class GetUsersUseCase @Inject constructor(
    private val repository: UserRepository
) {
    operator fun invoke(): Flow<Result<List<User>>> =
        repository.getUsers()
}

// 数据层 (Data Layer) - 仓库 (Repository)
interface UserRepository {
    fun getUsers(): Flow<Result<List<User>>>
}

class UserRepositoryImpl @Inject constructor(
    private val api: UserApi,
    private val dao: UserDao
) : UserRepository {
    override fun getUsers(): Flow<Result<List<User>>> = flow {
        // 先读取本地缓存
        val cachedUsers = dao.getUsers()
        emit(Result.success(cachedUsers))

        // 然后从网络获取
        try {
            val networkUsers = api.getUsers()
            dao.insertUsers(networkUsers)
            emit(Result.success(networkUsers))
        } catch (e: Exception) {
            emit(Result.failure(e))
        }
    }.flowOn(Dispatchers.IO)
}

// 表现层 (Presentation Layer) - ViewModel
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUsersUseCase: GetUsersUseCase
) : ViewModel() {

    private val _uiState = MutableStateFlow(UserUiState())
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()

    init {
        loadUsers()
    }

    private fun loadUsers() {
        viewModelScope.launch {
            getUsersUseCase().collect { result ->
                result.onSuccess { users ->
                    _uiState.update { it.copy(users = users, isLoading = false) }
                }.onFailure { error ->
                    _uiState.update { it.copy(error = error.message, isLoading = false) }
                }
            }
        }
    }
}

// UI 状态
data class UserUiState(
    val users: List<User> = emptyList(),
    val isLoading: Boolean = true,
    val error: String? = null
)
```

### MVI (Model-View-Intent)

**何时使用:**
- 需要单向数据流
- 复杂的状态管理
- 时间旅行调试 (Time-travel debugging)
- 可预测的状态更新

```kotlin
// 状态 (State)
data class UserScreenState(
    val users: List<User> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)

// 事件 (用户意图)
sealed class UserEvent {
    object LoadUsers : UserEvent()
    data class DeleteUser(val id: String) : UserEvent()
    object RetryLoad : UserEvent()
}

// ViewModel
class UserViewModel : ViewModel() {
    private val _state = MutableStateFlow(UserScreenState())
    val state: StateFlow<UserScreenState> = _state.asStateFlow()

    fun onEvent(event: UserEvent) {
        when (event) {
            is UserEvent.LoadUsers -> loadUsers()
            is UserEvent.DeleteUser -> deleteUser(event.id)
            is UserEvent.RetryLoad -> loadUsers()
        }
    }
}
```

## 依赖注入 (Dependency Injection)

### Hilt (推荐用于大型应用)

**设置:**
```kotlin
// App 类
@HiltAndroidApp
class MyApplication : Application()

// Activity
@AndroidEntryPoint
class MainActivity : ComponentActivity()

// ViewModel
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository,
    private val analytics: Analytics
) : ViewModel()

// 模块 (Module)
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit = Retrofit.Builder()
        .baseUrl("https://api.example.com")
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    @Provides
    @Singleton
    fun provideUserApi(retrofit: Retrofit): UserApi =
        retrofit.create(UserApi::class.java)
}
```

### Koin (轻量级替代方案)

**设置:**
```kotlin
// 模块定义
val appModule = module {
    single { UserRepository(get()) }
    viewModel { UserViewModel(get()) }
}

// Application
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidContext(this@MyApp)
            modules(appModule)
        }
    }
}

// 使用
class UserViewModel(
    private val repository: UserRepository
) : ViewModel()
```

**Hilt vs Koin:**
- **Hilt**：编译时，类型安全，Google 支持，设置复杂
- **Koin**：运行时，简单 DSL，设置快 50%，基于反射

## 性能优化

### R8 优化

**自动优化:**
- 代码缩减 (移除未使用的代码)
- 混淆 (重命名类/方法)
- 优化 (方法内联)

```groovy
// build.gradle
android {
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt')
        }
    }
}
```

**影响:**
- 应用体积减少 10-20%
- 启动速度加快 20%
- 更难被逆向工程

### 基准配置文件 (Baseline Profiles)

**性能提升:**
- 启动速度加快 10-20%
- 减少关键路径的卡顿
- 热点代码的 AOT 编译

```gradle
// build.gradle
dependencies {
    implementation "androidx.profileinstaller:profileinstaller:1.3.1"
}
```

### Compose 性能

**1. 稳定性注解:**
```kotlin
// 标记稳定的类
@Stable
data class User(val name: String, val age: Int)

// 不可变集合
@Immutable
data class UserList(val users: List<User>)
```

**2. 避免重组 (Recomposition):**
```kotlin
// ❌ 坏: 每次渲染都重组
@Composable
fun UserList(users: List<User>) {
    LazyColumn {
        items(users) { user ->
            Text(user.name)  // 每次都会重新创建
        }
    }
}

// ✅ 好: 使用键 (Keys)
@Composable
fun UserList(users: List<User>) {
    LazyColumn {
        items(users, key = { it.id }) { user ->
            Text(user.name)
        }
    }
}
```

**3. 记住昂贵的计算:**
```kotlin
@Composable
fun ExpensiveList(items: List<Item>) {
    val sortedItems = remember(items) {
        items.sortedBy { it.priority }
    }

    LazyColumn {
        items(sortedItems) { item ->
            ItemCard(item)
        }
    }
}
```

## 测试

### 单元测试 (JUnit + MockK)

```kotlin
class UserViewModelTest {
    private lateinit var viewModel: UserViewModel
    private val mockRepository = mockk<UserRepository>()

    @Before
    fun setup() {
        viewModel = UserViewModel(mockRepository)
    }

    @Test
    fun `loadUsers should update state with users`() = runTest {
        // 给定 (Given)
        val users = listOf(User("1", "Test", "test@example.com"))
        coEvery { mockRepository.getUsers() } returns flowOf(Result.success(users))

        // 当 (When)
        viewModel.loadUsers()

        // 那么 (Then)
        val state = viewModel.uiState.value
        assertEquals(users, state.users)
        assertFalse(state.isLoading)
    }
}
```

### Compose 测试

```kotlin
class UserListScreenTest {
    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun displayUsers() {
        val users = listOf(User("1", "John", "john@example.com"))

        composeTestRule.setContent {
            UserListScreen(
                users = users,
                onUserClick = {}
            )
        }

        composeTestRule.onNodeWithText("John").assertIsDisplayed()
    }
}
```

### 插桩测试 (Espresso)

```kotlin
@RunWith(AndroidJUnit4::class)
class LoginActivityTest {
    @get:Rule
    val activityRule = ActivityScenarioRule(LoginActivity::class.java)

    @Test
    fun loginFlow() {
        onView(withId(R.id.emailField))
            .perform(typeText("test@example.com"))

        onView(withId(R.id.passwordField))
            .perform(typeText("password123"))

        onView(withId(R.id.loginButton))
            .perform(click())

        onView(withText("Welcome"))
            .check(matches(isDisplayed()))
    }
}
```

## Material Design 3

### 主题设置

```kotlin
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context)
            else dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}
```

### Material 组件

```kotlin
// 卡片 (Cards)
Card(
    modifier = Modifier.fillMaxWidth(),
    elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
) {
    Text("Content")
}

// 浮动操作按钮 (FAB)
FloatingActionButton(onClick = { /* 做些什么 */ }) {
    Icon(Icons.Default.Add, contentDescription = "Add")
}

// 导航 (Navigation)
NavigationBar {
    items.forEach { item ->
        NavigationBarItem(
            icon = { Icon(item.icon, contentDescription = null) },
            label = { Text(item.label) },
            selected = selectedItem == item,
            onClick = { selectedItem = item }
        )
    }
}
```

## Google Play 要求 (2024-2025)

### SDK 要求
- **当前**: 目标 Android 14 (API 34)
- **强制 (2025年8月31日)**: 目标 Android 15 (API 35)

### 隐私与安全
- **隐私政策**: 收集数据的应用必须提供
- **数据安全**: Play 控制台中的表单
- **权限**: 仅请求所需权限，证明危险权限的合理性
- **加密**: 网络使用 HTTPS，敏感数据使用 KeyStore

### AAB (Android App Bundle)
```gradle
android {
    bundle {
        density {
            enableSplit true
        }
        abi {
            enableSplit true
        }
        language {
            enableSplit true
        }
    }
}
```

**优势:**
- 下载体积减小 15-30%
- 动态功能模块
- 支持即时应用 (Instant apps)

## 常见陷阱

1.  **阻塞主线程**: 使用带有 Dispatchers.IO 的协程
2.  **内存泄漏**: 注销监听器，取消协程
3.  **配置更改**: 使用 ViewModel，避免 Activity 引用
4.  **大图片**: 使用 Coil/Glide 进行缓存和调整大小
5.  **忘记权限**: 运行时权限请求
6.  **忽略 Android 版本**: 在多个 API 级别上测试
7.  **未处理返回按键**: OnBackPressedDispatcher
8.  **硬编码字符串**: 使用 strings.xml 进行本地化
9.  **不使用 Proguard/R8**: 在发布版本中启用
10. **忽略电池**: 使用 WorkManager 处理后台任务

## 资源

**官方:**
- Kotlin 文档: https://kotlinlang.org/docs/home.html
- Compose 文档: https://developer.android.com/jetpack/compose
- Material 3: https://m3.material.io/
- Android 指南: https://developer.android.com/guide

**社区:**
- Android Weekly: https://androidweekly.net/
- Kt.Academy: https://kt.academy/
- Coding in Flow: https://codinginflow.com/
- Philipp Lackner: https://pl-coding.com/
