# 移动开发框架参考

移动开发框架综合指南：React Native、Flutter 和原生开发。

## 框架概览 (2024-2025)

### React Native
- **语言**: JavaScript/TypeScript
- **GitHub Star 数**: 121,000+
- **采用率**: 35% 的移动开发者，67% 的熟悉度
- **性能**: 80-90% 原生性能
- **架构**: 基于桥接（旧版）→ 新架构（JSI, Fabric, Codegen）
- **渲染**: 原生组件
- **热重载**: 支持
- **社区**: 庞大（npm 生态系统，每周 300 万+ 下载量）

### Flutter
- **语言**: Dart
- **GitHub Star 数**: 170,000+（增长最快）
- **采用率**: 46% 的移动开发者
- **性能**: 85-95% 原生性能
- **架构**: "万物皆 Widget"
- **渲染**: 自定义 Impeller 渲染引擎（消除卡顿）
- **热重载**: 支持（业界最快）
- **社区**: 快速增长（pub.dev 上有 23,000+ 包）

### 原生 iOS (Swift/SwiftUI)
- **语言**: Swift
- **性能**: 100% 原生
- **UI 框架**: SwiftUI（声明式）或 UIKit（命令式）
- **最新**: Swift 6，具备编译时数据竞争检测
- **工具**: Xcode 16, Swift Package Manager
- **并发**: async/await, actors, @MainActor

### 原生 Android (Kotlin/Jetpack Compose)
- **语言**: Kotlin
- **性能**: 100% 原生
- **UI 框架**: Jetpack Compose（声明式）或 Views（命令式）
- **最新**: Kotlin 2.1, Compose 1.7
- **工具**: Android Studio Hedgehog+
- **协程**: 用于异步的 Kotlin 协程

## React Native 深度解析

### 核心概念

**新架构 (0.82+ 强制)**
- **JSI (JavaScript Interface)**: JS 与原生直接通信，消除桥接
- **Fabric**: 具有同步布局的新渲染系统
- **Codegen**: JS 和原生代码之间的静态类型安全
- **Turbo Modules**: 懒加载原生模块

**性能优化**
- **Hermes 引擎**: 启动速度提升 30-40%，内存占用减少
- **原生驱动动画**: 卸载到 UI 线程（60 FPS）
- **FlatList 虚拟化**: 仅渲染可见项
- **图片优化**: FastImage 库，渐进式加载

### 最佳实践

**项目结构（基于功能）**
```
src/
├── features/
│   ├── auth/
│   ├── profile/
│   └── dashboard/
├── shared/
│   ├── components/
│   ├── hooks/
│   └── utils/
├── navigation/
├── services/
└── stores/
```

**状态管理 (2024-2025)**
1. **Zustand** (新星): 极少样板代码，3KB，优秀的 TypeScript 支持
2. **Redux Toolkit**: 企业级应用，时间旅行调试，DevTools
3. **Recoil**: Meta 构建，基于原子，实验性
4. **Context API**: 简单应用，避免属性透传 (prop drilling)

**导航**
- **React Navigation**: 行业标准，80%+ 采用率
- TypeScript 类型安全导航
- 深度链接配置
- Tab、Stack、Drawer 导航器

**TypeScript 采用**
- 85%+ 的新 React Native 项目使用 TypeScript
- 类型安全可预防 15% 的运行时错误
- 更好的 IDE 支持和自动补全

### 测试策略

**单元测试**
- **Jest**: 默认测试运行器
- **React Native Testing Library**: 组件测试，最佳实践
- 目标：70-80%+ 代码覆盖率

**端到端 (E2E) 测试**
- **Detox**: 灰盒测试，快速，可靠（推荐）
- **Appium**: 跨平台，基于 WebDriver
- **Maestro**: 新秀，简单的基于 YAML 的测试

**示例 (React Native Testing Library)**
```javascript
import { render, fireEvent, waitFor } from '@testing-library/react-native';

test('login button should be enabled when form is valid', async () => {
  const { getByTestId } = render(<LoginScreen />);
  const emailInput = getByTestId('email-input');
  const passwordInput = getByTestId('password-input');
  const loginButton = getByTestId('login-button');

  fireEvent.changeText(emailInput, 'test@example.com');
  fireEvent.changeText(passwordInput, 'password123');

  await waitFor(() => {
    expect(loginButton).not.toBeDisabled();
  });
});
```

### 何时选择 React Native

**✅ 最适合：**
- 团队具备 JavaScript/TypeScript 专业知识
- 与 Web (React) 代码共享
- 快速原型设计和 MVP
- 需要强大的社区支持
- npm 生态系统集成
- 商业应用（12.57% 市场份额）

**❌ 不理想：**
- 重度图形/游戏（使用原生或 Unity）
- 极致性能至关重要
- 深度平台特定集成
- 团队不熟悉 JavaScript

## Flutter 深度解析

### 核心概念

**"万物皆 Widget"**
- UI 由可组合的 widget 构建
- 不可变 widget 树
- 使用 setState/状态管理进行响应式更新

**渲染引擎**
- **Impeller**: 新渲染引擎（iOS 稳定版，Android 预览版）
- 消除着色器卡顿
- 在支持的设备上可达 120 FPS
- 自定义 Skia 渲染（完全控制）

**性能特性**
- **Const widgets**: 编译时优化
- **RepaintBoundary**: 隔离昂贵的重绘
- **ListView.builder**: 长列表懒加载
- **Cached network images**: 图片优化

### 最佳实践

**项目结构（功能优先）**
```
lib/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   └── profile/
├── core/
│   ├── theme/
│   ├── utils/
│   └── widgets/
├── routing/
└── main.dart
```

**状态管理 (2024-2025)**
1. **Riverpod 3**: 现代，编译安全，Flutter 团队推荐
2. **Bloc**: 企业级应用，事件驱动，可预测状态
3. **Provider**: 初学者，简单应用
4. **GetX**: 一体化（状态 + 路由 + 依赖注入），观点鲜明

**导航**
- **GoRouter**: 官方推荐 (2024+)，声明式路由
- 代码生成的类型安全路由
- 内置深度链接
- 在大多数用例中取代 Navigator 2.0

**优先级（官方）**
1. **P0**: 立即修复（崩溃，数据丢失）
2. **P1**: 几天内修复（主要功能损坏）
3. **P2**: 几周内修复（烦恼）
4. **P3**: 锦上添花

### 测试策略

**单元测试**
- **flutter_test**: 内置测试包
- **Mockito**: 模拟依赖
- 目标：80%+ 代码覆盖率

**Widget 测试**
- **WidgetTester**: 测试 UI 和交互
- **Golden Tests**: 视觉回归测试

**集成测试**
- **integration_test**: 端到端测试
- 在真机或模拟器上运行

**示例 (Widget 测试)**
```dart
testWidgets('Counter increments', (WidgetTester tester) async {
  await tester.pumpWidget(MyApp());

  expect(find.text('0'), findsOneWidget);
  expect(find.text('1'), findsNothing);

  await tester.tap(find.byIcon(Icons.add));
  await tester.pump();

  expect(find.text('0'), findsNothing);
  expect(find.text('1'), findsOneWidget);
});
```

### 何时选择 Flutter

**✅ 最适合：**
- 性能关键型应用
- 复杂动画和自定义 UI
- 多平台（移动、Web、桌面）
- 跨平台一致的 UI
- 成长型团队/初创公司（开发最快）
- 视觉要求高的应用

**❌ 不理想：**
- 团队不熟悉 Dart
- 严重依赖原生平台功能
- 现有的庞大 JavaScript/原生代码库
- 应用体积至关重要（<20MB）

## 原生 iOS (Swift/SwiftUI)

### 核心概念

**Swift 6 (2024-2025)**
- 编译时数据竞争检测
- 增强的并发性：async/await, actors, @MainActor
- 强大的宏系统
- 移动语义以提升性能

**SwiftUI vs UIKit**
- **SwiftUI**: 声明式，代码量减少 40%，iOS 13+，现代方法
- **UIKit**: 命令式，细粒度控制，遗留支持，复杂定制
- 两者可在同一项目中协同工作

### 架构模式

**MVVM (最流行)**
```swift
// ViewModel (ObservableObject)
class LoginViewModel: ObservableObject {
    @Published var email = ""
    @Published var password = ""
    @Published var isLoading = false

    func login() async {
        isLoading = true
        // Login logic
        isLoading = false
    }
}

// View
struct LoginView: View {
    @StateObject private var viewModel = LoginViewModel()

    var body: some View {
        VStack {
            TextField("Email", text: $viewModel.email)
            SecureField("Password", text: $viewModel.password)
            Button("Login") {
                Task { await viewModel.login() }
            }
        }
    }
}
```

**TCA (The Composable Architecture)**
- 采用率增长 (v1.13+)
- 非常适合复杂应用
- 学习曲线较陡
- 可预测的状态管理

### 何时选择原生 iOS

**✅ 最适合：**
- 仅 iOS 应用
- 需要极致性能
- 最新 Apple 功能（WidgetKit, Live Activities, App Clips）
- 深度 iOS 生态系统集成
- 具备 Swift/iOS 专业知识的团队

## 原生 Android (Kotlin/Jetpack Compose)

### 核心概念

**Kotlin 2.1 (2024-2025)**
- 设计即空安全
- 用于异步的协程
- 类型安全状态的密封类
- 扩展函数

**Jetpack Compose**
- 声明式 UI（类似 SwiftUI/React）
- 前 1,000 个应用中采用率达 60%
- Material Design 3 集成
- Kotlin 2.0+ 的 Compose 编译器

### 架构模式

**MVVM + Clean Architecture**
```kotlin
// ViewModel
class LoginViewModel(
    private val loginUseCase: LoginUseCase
) : ViewModel() {
    private val _uiState = MutableStateFlow(LoginUiState())
    val uiState: StateFlow<LoginUiState> = _uiState.asStateFlow()

    fun login(email: String, password: String) {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            loginUseCase(email, password)
                .onSuccess { /* Navigate */ }
                .onFailure { /* Show error */ }
            _uiState.update { it.copy(isLoading = false) }
        }
    }
}

// Composable
@Composable
fun LoginScreen(viewModel: LoginViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsState()

    Column {
        TextField(
            value = uiState.email,
            onValueChange = { /* update */ }
        )
        Button(onClick = { viewModel.login() }) {
            Text("Login")
        }
    }
}
```

### 何时选择原生 Android

**✅ 最适合：**
- 仅 Android 应用
- 需要极致性能
- Material Design 3 实现
- 深度 Android 生态系统集成
- 具备 Kotlin/Android 专业知识的团队

## 框架比较矩阵

| 特性 | React Native | Flutter | 原生 iOS | 原生 Android |
|---------|--------------|---------|------------|----------------|
| **语言** | JavaScript/TS | Dart | Swift | Kotlin |
| **学习曲线** | 容易 | 中等 | 中等 | 中等 |
| **性能** | 80-90% | 85-95% | 100% | 100% |
| **热重载** | 支持 | 支持（最快） | 预览 | 实时编辑 |
| **代码共享** | Web (React) | Web/桌面 | 无 | 无 |
| **社区规模** | 庞大 | 增长中 | 仅 iOS | 仅 Android |
| **UI 范式** | 组件 | Widgets | 声明式 | 声明式 |
| **第三方库** | npm (300万+) | pub.dev (23,000+) | SPM | Maven |
| **应用体积** | 40-50MB | 15-20MB | 10-15MB | 10-15MB |
| **构建时间** | 中等 | 快 | 慢 (Xcode) | 中等 |
| **调试** | Chrome/Safari | DevTools | Xcode | Android Studio |
| **平台手感** | 需调整 | 需调整 | 原生 | 原生 |
| **启动时间** | 中等 | 快 | 最快 | 最快 |
| **最适合** | JS 团队 | 性能 | 仅 iOS | 仅 Android |

## 迁移路径

### React Native → Flutter
- **工作量**: 高（完全重写）
- **时间线**: 中型应用 3-6 个月
- **优势**: 更好的性能，更小的应用体积
- **挑战**: 新语言 (Dart)，不同的生态系统

### Flutter → React Native
- **工作量**: 高（完全重写）
- **时间线**: 中型应用 3-6 个月
- **优势**: 更大的社区，Web 代码共享
- **挑战**: 性能较低，应用体积较大

### 跨平台 → 原生
- **工作量**: 极高（分别开发 iOS 和 Android 应用）
- **时间线**: 中型应用 6-12 个月
- **优势**: 极致性能，平台特性
- **挑战**: 维护两个代码库，2倍团队规模

### 原生 → 跨平台
- **工作量**: 高（合并为一个代码库）
- **时间线**: 中型应用 4-8 个月
- **优势**: 单一代码库，开发更快
- **挑战**: 性能权衡，平台差异

## 决策框架

### 从这里开始：你需要原生性能吗？
- **否** → 跨平台 (React Native 或 Flutter)
- **是** → 原生 (Swift 或 Kotlin)

### 如果跨平台：团队懂 JavaScript 吗？
- **是** → React Native
- **否** → Flutter

### 如果原生：仅 iOS 还是仅 Android？
- **仅 iOS** → Swift/SwiftUI
- **仅 Android** → Kotlin/Compose
- **两者** → 重新考虑跨平台

### 其他因素：
- **现有代码库**: 使用相同技术
- **存在 Web 应用**: React Native (代码共享)
- **需要桌面端**: Flutter (多平台)
- **预算受限**: 跨平台
- **性能关键**: 原生
- **复杂动画**: Flutter 或原生
- **商业侧重**: React Native (更大的市场份额)

## 资源

**React Native:**
- 官方文档: https://reactnative.dev/
- 新架构: https://reactnative.dev/docs/the-new-architecture/landing-page
- Expo: https://expo.dev/ (推荐框架)
- 目录: https://reactnative.directory/

**Flutter:**
- 官方文档: https://flutter.dev/
- Pub.dev: https://pub.dev/
- Codelabs: https://flutter.dev/codelabs
- Widget 目录: https://flutter.dev/widgets

**原生 iOS:**
- Swift 文档: https://swift.org/documentation/
- SwiftUI 教程: https://developer.apple.com/tutorials/swiftui
- iOS 人机界面指南 (HIG): https://developer.apple.com/design/human-interface-guidelines/

**原生 Android:**
- Kotlin 文档: https://kotlinlang.org/docs/home.html
- Compose 文档: https://developer.android.com/jetpack/compose
- Material 3: https://m3.material.io/
- Android 指南: https://developer.android.com/guide
