# iOS 原生开发

使用 Swift 和 SwiftUI 进行 iOS 开发的完整指南 (2024-2025)。

## Swift 6 概览

### 关键特性
- **数据竞争安全 (Data race safety)**：编译时检测（Swift 6 中默认启用）
- **并发 (Concurrency)**：async/await, actors, @MainActor
- **宏系统 (Macro system)**：编译时代码生成
- **移动语义 (Move semantics)**：所有权优化
- **增强的泛型 (Enhanced generics)**：更强大的类型系统

### 现代 Swift 模式

**Async/Await:**
```swift
func fetchUser(id: String) async throws -> User {
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}

// 用法
Task {
    do {
        let user = try await fetchUser(id: "123")
        self.user = user
    } catch {
        self.error = error
    }
}
```

**Actors (线程安全类):**
```swift
actor UserCache {
    private var cache: [String: User] = [:]

    func get(_ id: String) -> User? {
        cache[id]
    }

    func set(_ id: String, user: User) {
        cache[id] = user
    }
}
```

## SwiftUI vs UIKit

### 何时使用 SwiftUI
✅ 新项目 (iOS 13+)
✅ 首选声明式 UI
✅ 需要快速迭代
✅ 跨平台 (macOS, watchOS, tvOS)
✅ 代码量比 UIKit 少 40%

### 何时使用 UIKit
✅ 维护旧版应用
✅ 复杂的自定义
✅ 需要精细的控制
✅ 需要特定的 UIKit 功能
✅ 支持 iOS 13 之前的版本

### SwiftUI 基础

```swift
struct ContentView: View {
    @State private var count = 0

    var body: some View {
        VStack(spacing: 20) {
            Text("Count: \(count)")
                .font(.title)

            Button("Increment") {
                count += 1
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

**属性包装器 (Property Wrappers):**
- `@State`: 视图局部状态
- `@Binding`: 双向绑定
- `@StateObject`: 可观察对象所有者
- `@ObservedObject`: 可观察对象引用
- `@EnvironmentObject`: 依赖注入
- `@Published`: 可观察属性

## 架构模式

### MVVM (最流行)

```swift
// 模型 (Model)
struct User: Identifiable, Codable {
    let id: String
    let name: String
    let email: String
}

// 视图模型 (ViewModel)
@MainActor
class UserViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var error: Error?

    private let repository: UserRepository

    init(repository: UserRepository = UserRepository()) {
        self.repository = repository
    }

    func loadUsers() async {
        isLoading = true
        defer { isLoading = false }

        do {
            users = try await repository.fetchUsers()
        } catch {
            self.error = error
        }
    }
}

// 视图 (View)
struct UserListView: View {
    @StateObject private var viewModel = UserViewModel()

    var body: some View {
        List(viewModel.users) { user in
            Text(user.name)
        }
        .task {
            await viewModel.loadUsers()
        }
    }
}
```

### TCA (The Composable Architecture)

**何时使用:**
- 复杂的状态管理
- 可预测的状态更新
- 优秀的测试性
- 企业级应用

**权衡:**
- 学习曲线较陡
- 样板代码较多
- 非常适合大型团队

## 性能优化

### 编译器优化

**1. 使用 `final` 类:**
```swift
final class FastClass {
    // 编译器可以优化（无动态分发）
}
```

**2. 私有方法:**
```swift
private func optimize() {
    // 编译器可以内联
}
```

**3. 全模块优化:**
```bash
# Build Settings
SWIFT_WHOLE_MODULE_OPTIMIZATION = YES
```

### 内存管理

**ARC (自动引用计数):**
```swift
class Parent {
    var child: Child?
}

class Child {
    weak var parent: Parent?  // Weak 以避免保留环
}
```

**常见的保留环 (Retain Cycles):**
```swift
// ❌ 坏：保留环
class ViewController: UIViewController {
    var completion: (() -> Void)?

    func setup() {
        completion = {
            self.doSomething()  // 强捕获
        }
    }
}

// ✅ 好：Weak self
class ViewController: UIViewController {
    var completion: (() -> Void)?

    func setup() {
        completion = { [weak self] in
            self?.doSomething()
        }
    }
}
```

### SwiftUI 性能

**1. 使用 const 修饰符:**
```swift
Text("Hello")  // 每次渲染都会重新创建

vs

Text("Hello")
    .font(.title)  // 修饰符创建新视图

// 更好：提取静态视图
let titleText = Text("Hello").font(.title)
```

**2. 避免昂贵的计算:**
```swift
struct ExpensiveView: View {
    let data: [Item]

    // 每次渲染都会计算
    var sortedData: [Item] {
        data.sorted()  // ❌ 坏
    }

    // 更好：使用 @State 缓存或传递已排序数据
}
```

## 测试策略

### XCTest (单元测试)

```swift
import XCTest
@testable import MyApp

final class UserViewModelTests: XCTestCase {
    var viewModel: UserViewModel!
    var mockRepository: MockUserRepository!

    override func setUp() {
        super.setUp()
        mockRepository = MockUserRepository()
        viewModel = UserViewModel(repository: mockRepository)
    }

    func testLoadUsers() async throws {
        // 给定 (Given)
        let expectedUsers = [User(id: "1", name: "Test", email: "test@example.com")]
        mockRepository.usersToReturn = expectedUsers

        // 当 (When)
        await viewModel.loadUsers()

        // 那么 (Then)
        XCTAssertEqual(viewModel.users, expectedUsers)
        XCTAssertFalse(viewModel.isLoading)
        XCTAssertNil(viewModel.error)
    }
}
```

### XCUITest (UI 测试)

```swift
import XCTest

final class LoginUITests: XCTestCase {
    let app = XCUIApplication()

    override func setUp() {
        super.setUp()
        app.launch()
    }

    func testLoginFlow() {
        let emailField = app.textFields["emailField"]
        emailField.tap()
        emailField.typeText("test@example.com")

        let passwordField = app.secureTextFields["passwordField"]
        passwordField.tap()
        passwordField.typeText("password123")

        app.buttons["loginButton"].tap()

        XCTAssertTrue(app.staticTexts["Welcome"].waitForExistence(timeout: 5))
    }
}
```

**目标覆盖率:**
- 单元测试：70-80%+
- 关键路径：100%
- UI 测试：仅关键用户流程（较慢）

## iOS 特定功能

### WidgetKit (小组件)

```swift
import WidgetKit
import SwiftUI

struct SimpleWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: "SimpleWidget", provider: Provider()) { entry in
            SimpleWidgetView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .description("This is my widget")
        .supportedFamilies([.systemSmall, .systemMedium, .systemLarge])
    }
}
```

### 实时活动 (Live Activities) (iOS 16.1+)

```swift
import ActivityKit

struct OrderAttributes: ActivityAttributes {
    struct ContentState: Codable, Hashable {
        var status: String
        var estimatedTime: Date
    }

    var orderId: String
}

// 开始活动
let attributes = OrderAttributes(orderId: "123")
let initialState = OrderAttributes.ContentState(
    status: "Preparing",
    estimatedTime: Date().addingTimeInterval(1800)
)

let activity = try Activity.request(
    attributes: attributes,
    contentState: initialState
)
```

### App Clips (轻应用)

**特点:**
- 大小限制 <10MB
- 快速、轻量级体验
- 无需安装
- 通过 NFC, QR, Safari, Maps 唤起

## 人机界面指南 (HIG)

### 导航模式

**标签栏 (Tab Bar):**
- 2-5 个顶级部分
- 底部放置
- 始终可见
- 立即导航

**导航栏 (Navigation Bar):**
- 层级导航
- 自动返回按钮
- 标题和操作
- 大标题/内联标题模式

**模态演示 (Modal Presentation):**
- 中断任务
- 独立的流程
- 清晰的关闭操作
- 谨慎使用

### 设计原则

**清晰 (Clarity):**
- 清晰的文本（最小 11pt）
- 足够的对比度 (WCAG AA)
- 精确的图标

**顺从 (Deference):**
- 内容第一，UI 第二
- 半透明背景
- 极简的 UI 元素

**深度 (Depth):**
- 分层（表格，覆盖层）
- 视觉层级
- 动作提供含义

### 颜色

**系统颜色:**
```swift
Color.primary      // 自适应黑/白
Color.secondary    // 灰色
Color.accentColor  // 应用强调色
Color(uiColor: .systemBlue)
Color(uiColor: .label)
```

**深色模式:**
```swift
// 自动
Color.primary  // 适应亮/暗

// 自定义
Color("CustomColor")  // 在 Assets.xcassets 中定义
```

### SF Symbols (图标符号)

```swift
Image(systemName: "star.fill")
    .foregroundColor(.yellow)
    .font(.title)

// 渲染模式
Image(systemName: "heart.fill")
    .symbolRenderingMode(.multicolor)
```

## App Store 要求 (2024-2025)

### SDK 要求
- **当前**: Xcode 15+ 和 iOS 17 SDK（自 2024 年 4 月起必须）
- **即将到来**: Xcode 16+ 和 iOS 18 SDK（推荐用于 2025 年提交）

### 隐私
- **隐私清单 (Privacy manifest)**：第三方 SDK 必需
- **跟踪权限**: 用于广告的 ATT 框架
- **隐私营养标签**: 准确的数据收集信息
- **账户删除**: 要求应用内删除

### 功能 (Capabilities)
- **沙盒**: 所有应用均在沙盒中
- **Entitlements**: 仅请求所需的功能
- **后台模式**: 证明后台使用的合理性
- **HealthKit**: 隐私敏感，严格审查

### 提交清单
✅ 应用图标（所有所需尺寸）
✅ 截图（所有设备尺寸）
✅ 应用描述和关键词
✅ 隐私政策 URL
✅ 支持 URL
✅ 年龄分级问卷
✅ 出口合规性
✅ 在真实设备上测试
✅ 无崩溃或重大 Bug

## 常见陷阱

1.  **强引用循环**: 在闭包中使用 `[weak self]`
2.  **主线程阻塞**: 使用 async/await，避免同步操作
3.  **大图片**: 显示前调整大小
4.  **未处理的错误**: 始终处理 async throws
5.  **忽略安全区域**: 有意使用 `.ignoresSafeArea()`
6.  **未测试深色模式**: 为两种外观设计
7.  **硬编码字符串**: 从一开始就使用本地化
8.  **内存泄漏**: 定期使用 Instruments 分析

## 资源

**官方:**
- Swift 文档: https://swift.org/documentation/
- SwiftUI 教程: https://developer.apple.com/tutorials/swiftui
- HIG: https://developer.apple.com/design/human-interface-guidelines/
- WWDC 视频: https://developer.apple.com/videos/

**社区:**
- Hacking with Swift: https://www.hackingwithswift.com/
- Swift by Sundell: https://www.swiftbysundell.com/
- objc.io: https://www.objc.io/
- iOS Dev Weekly: https://iosdevweekly.com/
