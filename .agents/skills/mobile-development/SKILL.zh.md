---
name: mobile-development
description: 使用 React Native, Flutter, Swift/SwiftUI 和 Kotlin/Jetpack Compose 构建现代移动应用。涵盖移动优先设计原则、性能优化（电池、内存、网络）、离线优先架构、平台特定指南（iOS HIG, Material Design）、测试策略、安全最佳实践、无障碍性、应用商店发布以及移动开发思维。适用于构建移动应用、实施移动 UX 模式、针对移动限制进行优化，或进行原生与跨平台决策时使用。
license: MIT
version: 1.0.0
---

# 移动开发技能

使用现代框架、最佳实践和移动优先思维模式进行生产级移动开发。

## 适用场景

- 构建移动应用程序（iOS、Android 或跨平台）
- 实施移动优先设计和 UX 模式
- 针对移动限制（电池、内存、网络、小屏幕）进行优化
- 制定原生与跨平台技术决策
- 实施离线优先架构和数据同步
- 遵循平台特定指南（iOS HIG, Material Design）
- 优化移动应用性能和用户体验
- 实施移动安全和身份验证
- 测试移动应用程序（单元测试、集成测试、端到端测试）
- 发布到 App Store 和 Google Play

## 技术选型指南

**跨平台框架：**
- **React Native**：利用 JavaScript 专长，Web 代码共享，成熟的生态系统（12.1万星，67% 熟悉度）
- **Flutter**：性能关键型应用，复杂的动画，增长最快（17万星，46% 采用率）

**原生开发：**
- **iOS (Swift/SwiftUI)**：极致 iOS 性能，最新特性，Apple 生态系统集成
- **Android (Kotlin/Jetpack Compose)**：极致 Android 性能，Material Design 3，平台优化

参见：`references/mobile-frameworks.md` 获取详细的框架对比

## 移动开发思维

**移动开发十诫：**

1.  **性能是基石，而非特性** - 加载时间超过 3 秒，70% 的用户会放弃应用
2.  **每字节、每毫秒都至关重要** - 移动设备的限制是真实存在的
3.  **默认离线优先** - 网络不可靠，设计时需考虑这一点
4.  **用户场景 > 开发者环境** - 思考真实的使用场景
5.  **通过平台意识避免平台锁定** - 尊重平台惯例
6.  **迭代，而非完美** - 发布、测量、改进的循环是生存之道
7.  **设计之初即考虑安全和无障碍性** - 绝非事后诸葛亮
8.  **在真机上测试** - 模拟器在性能表现上会撒谎
9.  **架构随复杂度扩展** - 简单的应用不要过度设计
10. **持续学习是生存之道** - 移动领域发展迅速

参见：`references/mobile-mindset.md` 获取思维模式和决策框架

## 参考导航

**核心技术：**
- `mobile-frameworks.md` - React Native, Flutter, Swift, Kotlin, 框架对比矩阵，各框架适用场景
- `mobile-ios.md` - Swift 6, SwiftUI, iOS 架构模式, HIG, App Store 要求, 平台能力
- `mobile-android.md` - Kotlin, Jetpack Compose, Material Design 3, Play Store, Android 特定功能

**最佳实践与开发思维：**
- `mobile-best-practices.md` - 移动优先设计, 性能优化, 离线优先架构, 安全, 测试, 无障碍性, 部署, 分析
- `mobile-debugging.md` - 调试工具, 性能分析, 崩溃分析, 网络调试, 平台特定调试
- `mobile-mindset.md` - 思维模式, 决策框架, 平台特定思维, 常见陷阱, 调试策略

## 核心最佳实践 (2024-2025)

**性能目标：**
- 应用启动：<2 秒（超过 3 秒 70% 用户放弃）
- 内存使用：典型屏幕 <100MB
- 网络请求：积极进行批处理和缓存
- 电池影响：尊重 Doze 模式和后台限制
- 动画：60 FPS（每帧 16.67ms）

**架构：**
- 中小型应用使用 MVVM（分离清晰，可测试）
- 大型企业级应用使用 MVVM + Clean Architecture
- 离线优先，采用混合同步（推送 + 拉取）
- 状态管理：Zustand (React Native), Riverpod 3 (Flutter), StateFlow (Android)

**安全 (OWASP Mobile Top 10)：**
- 身份验证使用 OAuth 2.0 + JWT + 生物识别
- 敏感数据使用 Keychain (iOS) / KeyStore (Android)
- 网络安全使用证书锁定 (Certificate pinning)
- 绝不硬编码凭据或 API 密钥
- 实施正确的会话管理

**测试策略：**
- 单元测试：业务逻辑覆盖率 70%+
- 集成测试：关键用户流程
- 端到端 (E2E) 测试：Detox (React Native), Appium (跨平台), XCUITest (iOS), Espresso (Android)
- 发布前强制进行真机测试

**部署：**
- 使用 Fastlane 实现跨平台自动化
- 分阶段发布：内部 → 封闭测试 → 开放测试 → 生产环境
- 强制要求：iOS 17 SDK (2024), Android 15 API 35 (2025年8月)
- CI/CD 节省 20% 开发时间

## 快速决策矩阵

| 需求 | 选择 |
|------|--------|
| JavaScript 团队，Web 代码共享 | React Native |
| 性能关键，复杂动画 | Flutter |
| 极致 iOS 性能，最新特性 | Swift/SwiftUI 原生 |
| 极致 Android 性能，Material 3 | Kotlin/Compose 原生 |
| 快速原型开发 | React Native + Expo |
| 桌面 + 移动端 | Flutter |
| 具备 JavaScript 技能的企业 | React Native |
| 资源有限的初创公司 | Flutter 或 React Native |
| 游戏或重度图形应用 | 原生 (Swift/Kotlin) 或 Unity |

## 框架快速对比 (2024-2025)

| 标准 | React Native | Flutter | Swift/SwiftUI | Kotlin/Compose |
|-----------|--------------|---------|---------------|----------------|
| **Star 数** | 12.1万 | 17万 | N/A | N/A |
| **采用率** | 35% | 46% | 仅限 iOS | 仅限 Android |
| **性能** | 80-90% 原生 | 85-95% 原生 | 100% 原生 | 100% 原生 |
| **开发速度** | 快 (热重载) | 非常快 (热重载) | 快 (Xcode 预览) | 快 (Live Edit) |
| **学习曲线** | 易 (JavaScript) | 中 (Dart) | 中 (Swift) | 中 (Kotlin) |
| **UI 范式** | 基于组件 | 基于 Widget | 声明式 | 声明式 |
| **社区** | 巨大 (npm) | 增长中 | Apple 生态系统 | Android 生态系统 |
| **最适合** | JS 团队，Web 共享 | 性能，动画 | 纯 iOS 应用 | 纯 Android 应用 |

## 实施清单

**项目设置：**
- 选择框架 → 初始化项目 → 配置开发环境 → 设置版本控制 → 配置 CI/CD → 团队规范

**架构：**
- 选择模式 (MVVM/Clean) → 设置文件夹结构 → 状态管理 → 导航 → API 层 → 错误处理 → 日志记录

**核心功能：**
- 身份验证 → 数据持久化 → API 集成 → 离线同步 → 推送通知 → 深度链接 → 分析

**UI/UX：**
- 设计系统 → 平台指南 → 无障碍性 → 响应式布局 → 深色模式 → 本地化 → 动画

**性能：**
- 图片优化 → 懒加载 → 内存分析 → 网络优化 → 电池测试 → 启动时间优化

**质量：**
- 单元测试 (70%+) → 集成测试 → E2E 测试 → 无障碍测试 → 性能测试 → 安全审计

**安全：**
- 安全存储 → 身份验证流程 → 网络安全 → 输入验证 → 会话管理 → 加密

**部署：**
- 应用图标/启动页 → 截图 → 商店详情 → 隐私政策 → TestFlight/内部测试 → 分阶段发布 → 监控

## 平台特定指南

**iOS (Human Interface Guidelines):**
- 原生导航模式（标签栏，导航栏）
- iOS 设计模式（下拉刷新，滑动操作）
- San Francisco 字体，iOS 颜色系统
- 触觉反馈，3D Touch/Haptic Touch
- 尊重安全区域和刘海屏

**Android (Material Design 3):**
- Material 导航（底部导航，导航抽屉）
- 悬浮操作按钮，Material 组件
- Roboto 字体，Material You 动态取色
- 触摸反馈（波纹效果）
- 尊重系统栏和手势

## 常见陷阱

1.  **仅在模拟器上测试** - 真机才能反映真实性能
2.  **忽视平台惯例** - 用户期望平台特定的交互模式
3.  **不处理离线情况** - 网络故障在所难免
4.  **糟糕的内存管理** - 导致崩溃和糟糕的用户体验
5.  **硬编码凭据** - 安全漏洞
6.  **无无障碍性** - 排除了 15%+ 的用户
7.  **过早优化** - 基于指标而非假设进行优化
8.  **过度设计** - 从简单开始，按需扩展
9.  **跳过真机测试** - 模拟器无法显示电池/网络问题
10. **不尊重电池** - 后台处理必须有正当理由

## 性能预算

**推荐目标：**
- **应用大小**：初始下载 <50MB，总计 <200MB
- **启动时间**：<2 秒可交互
- **屏幕加载**：缓存数据 <1 秒
- **网络请求**：API 调用 <3 秒
- **内存**：典型屏幕 <100MB，峰值 <200MB
- **电池**：每小时活跃使用消耗 <5%
- **帧率**：60 FPS（每帧 16.67ms）

## 资源

**官方文档：**
- React Native: https://reactnative.dev/
- Flutter: https://flutter.dev/
- iOS HIG: https://developer.apple.com/design/human-interface-guidelines/
- Material Design: https://m3.material.io/
- OWASP Mobile: https://owasp.org/www-project-mobile-top-10/

**工具与测试：**
- Detox E2E: https://wix.github.io/Detox/
- Appium: https://appium.io/
- Fastlane: https://fastlane.tools/
- Firebase: https://firebase.google.com/

**社区：**
- React Native Directory: https://reactnative.directory/
- Pub.dev (Flutter packages): https://pub.dev/
- Awesome React Native: https://github.com/jondot/awesome-react-native
- Awesome Flutter: https://github.com/Solido/awesome-flutter
