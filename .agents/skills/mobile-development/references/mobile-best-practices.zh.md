# 移动开发最佳实践

现代移动开发的跨平台最佳实践 (2024-2025)。

## 移动优先设计原则

### 核心原则
1. **内容至上**：去除多余装饰，聚焦内容
2. **渐进式披露**：将复杂性隐藏在层级之后
3. **拇指友好**：主要操作触手可及
4. **性能预算**：启动 <2秒，屏幕加载 <1秒
5. **离线优先**：为不可靠的网络环境设计

### 触控目标
- **iOS**：最小 44x44px (HIG 指南)
- **Android**：最小 48x48px (Material Design)
- **最佳**：重要操作 44-57px
- **间距**：目标之间最小 8px

### 排版
- **iOS**：San Francisco (系统字体)
- **Android**：Roboto (Material)
- **最小**：正文 16px (无障碍)
- **行高**：1.5倍以提高可读性

## 性能优化

### 启动时间优化
**目标：**
- 冷启动：<2秒
- 温启动：<1秒
- 热启动：<0.5秒

**技术：**
- 推迟非关键初始化
- 懒加载依赖
- 仅预加载关键数据
- 数据就绪前显示 UI

### 内存管理
**目标：**
- 典型屏幕：<100MB
- 峰值使用：<200MB

**技术：**
- 图片分页/虚拟化
- 在后台释放资源
- 使用 Instruments/Profiler 进行分析
- 避免循环引用/内存泄漏

**React Native 示例：**
```javascript
// 长列表使用 FlatList 代替 ScrollView
<FlatList
  data={items}
  renderItem={({ item }) => <ItemCard item={item} />}
  keyExtractor={(item) => item.id}
  initialNumToRender={10}
  maxToRenderPerBatch={10}
  windowSize={5}
/>
```

### 网络优化
**技术：**
- 批量 API 请求
- 积极缓存
- 压缩图片 (WebP, AVIF)
- 静态资源使用 CDN
- 实现请求去重

**示例策略：**
```
用户打开屏幕
├─ 立即显示缓存数据 (stale-while-revalidate)
├─ 后台获取最新数据
└─ 最新数据到达时更新 UI
```

### 电池优化
**技术：**
- 批量网络请求
- 尽可能降低 GPS 精度
- 使用推送代替轮询
- 尊重 Doze 模式 (Android)
- 后台应用刷新 (iOS)

**目标：**
- 活跃使用：<5% 每小时
- 后台：<1% 每小时

## 离线优先架构

### 本地存储选项
**React Native:**
- AsyncStorage (小数据，<6MB)
- Realm (复杂对象，关系)
- SQLite (关系型数据)
- MMKV (最快键值对)

**Flutter:**
- SharedPreferences (小数据)
- Hive (NoSQL，快速)
- Drift (SQLite 封装)
- ObjectBox (对象数据库)

**iOS:**
- UserDefaults (小数据)
- Core Data (复杂对象)
- SwiftData (现代替代品)
- Realm

**Android:**
- SharedPreferences (小数据)
- Room (SQLite ORM)
- Realm
- DataStore (Preferences + Proto)

### 数据同步策略

**1. 透写缓存 (Write-Through Cache)**
```
用户进行更改
├─ 立即更新本地数据库
├─ 乐观更新 UI
├─ 队列同步操作
└─ 后台同步到服务器
```

**2. 混合同步 (推 + 拉)**
```
推送同步 (实时)
├─ 关键更新使用 WebSocket 连接
└─ 更改立即通知

拉取同步 (周期性)
├─ 非关键数据周期性轮询
├─ 应用前台时拉取
└─ 增量同步 (仅上次同步后的更改)
```

**3. 冲突解决**
- **最后写入胜出 (Last-write-wins)**：使用时间戳
- **操作转换 (Operational transformation)**：合并更改
- **CRDT**：无冲突复制数据类型
- **手动解决**：用户选择

### 示例：离线优先评论

```typescript
// React Native + TypeScript
class CommentService {
  async postComment(text: string, postId: string) {
    const tempId = generateTempId();
    const comment = {
      id: tempId,
      text,
      postId,
      synced: false,
      timestamp: Date.now()
    };

    // 1. 立即本地保存
    await db.comments.insert(comment);

    // 2. 更新 UI (乐观)
    eventBus.emit('comment:added', comment);

    // 3. 后台同步到服务器
    try {
      const serverComment = await api.postComment(text, postId);
      // 用服务器 ID 替换临时 ID
      await db.comments.update(tempId, {
        id: serverComment.id,
        synced: true
      });
    } catch (error) {
      // 标记为待同步，稍后重试
      await db.comments.update(tempId, {
        syncError: error.message
      });
      syncQueue.add({ type: 'comment', id: tempId });
    }
  }
}
```

## 移动分析与监控

### 分析平台 (2024-2025)

**Firebase Analytics (推荐)**
- 免费层级慷慨
- 移动端特定事件
- 集成 Crashlytics
- AI 驱动的洞察
- 支持所有平台

**Sentry**
- 错误追踪 + 性能
- 跨平台支持
- Source map 上传
- 发布追踪
- 自定义面包屑

**Amplitude**
- 产品分析
- 用户行为追踪
- 群组分析
- A/B 测试集成

### 需追踪的关键事件

**用户旅程：**
- 应用打开
- 屏幕浏览
- 功能使用
- 转化事件
- 用户留存

**性能：**
- 应用启动时间
- 屏幕加载时间
- API 延迟
- 无崩溃率
- ANR 率 (Android)

**业务：**
- 购买
- 订阅
- 广告展示
- 功能采用
- 推荐

### Crashlytics 集成

**React Native:**
```javascript
import crashlytics from '@react-native-firebase/crashlytics';

// 记录事件
crashlytics().log('User tapped purchase button');

// 设置用户属性
crashlytics().setUserId(user.id);

// 记录非致命错误
try {
  await riskyOperation();
} catch (error) {
  crashlytics().recordError(error);
}
```

**Flutter:**
```dart
import 'package:firebase_crashlytics/firebase_crashlytics.dart';

// 记录事件
FirebaseCrashlytics.instance.log('User tapped purchase');

// 设置用户 ID
FirebaseCrashlytics.instance.setUserIdentifier(userId);

// 记录错误
await FirebaseCrashlytics.instance.recordError(
  error,
  stackTrace,
  reason: 'API call failed',
);
```

## 推送通知最佳实践

### 平台
- **iOS**：APNs (苹果推送通知服务)
- **Android**：FCM (Firebase 云消息传递)
- **跨平台**：OneSignal, Firebase, AWS SNS

### 最佳实践

**1. 权限请求策略**
```
❌ 坏：应用启动时请求权限
✅ 好：用户看到价值后请求

流程：
1. 用户与功能交互
2. 显示自定义模态框解释好处
3. 请求系统权限
4. 优雅处理拒绝
```

**2. 个性化**
- 按行为细分用户
- 在最佳时间发送 (时区)
- 个性化内容
- A/B 测试消息

**3. 频率**
- 避免通知垃圾轰炸
- 尊重用户偏好
- 实现静音时段
- 分组相关通知

**4. 深度链接**
```javascript
// React Native
import messaging from '@react-native-firebase/messaging';

messaging().onNotificationOpenedApp(remoteMessage => {
  const { screen, params } = remoteMessage.data;
  navigation.navigate(screen, params);
});
```

**影响：**
- 适当个性化带来 25% 收入增长
- 使用预权限模态框的加入率为 88% (无则为 40%)

## 认证与授权

### 现代认证技术栈 (2024-2025)

**标准模式：**
```
OAuth 2.0 (授权)
├─ JWT (无状态认证令牌)
├─ 刷新令牌 (长期访问)
└─ 生物识别 (便捷重认证)
```

### 实现

**生物识别认证 (iOS)**
```swift
import LocalAuthentication

let context = LAContext()
var error: NSError?

if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
    context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics,
                          localizedReason: "Unlock your account") { success, error in
        if success {
            // 已认证
        }
    }
}
```

**生物识别认证 (Android)**
```kotlin
import androidx.biometric.BiometricPrompt

val promptInfo = BiometricPrompt.PromptInfo.Builder()
    .setTitle("Biometric login")
    .setSubtitle("Log in using your biometric credential")
    .setNegativeButtonText("Use account password")
    .build()

val biometricPrompt = BiometricPrompt(this, executor,
    object : BiometricPrompt.AuthenticationCallback() {
        override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
            // 已认证
        }
    })

biometricPrompt.authenticate(promptInfo)
```

### 安全令牌存储

**iOS: Keychain**
```swift
import Security

func saveToken(_ token: String, for key: String) {
    let data = token.data(using: .utf8)!
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecValueData as String: data,
        kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
    ]
    SecItemAdd(query as CFDictionary, nil)
}
```

**Android: EncryptedSharedPreferences**
```kotlin
import androidx.security.crypto.EncryptedSharedPreferences
import androidx.security.crypto.MasterKey

val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val sharedPreferences = EncryptedSharedPreferences.create(
    context,
    "secure_prefs",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)

sharedPreferences.edit().putString("auth_token", token).apply()
```

**React Native: react-native-keychain**
```javascript
import * as Keychain from 'react-native-keychain';

// 保存凭据
await Keychain.setGenericPassword('username', token, {
  accessControl: Keychain.ACCESS_CONTROL.BIOMETRY_CURRENT_SET,
  accessible: Keychain.ACCESSIBLE.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
});

// 检索凭据
const credentials = await Keychain.getGenericPassword();
const token = credentials.password;
```

## 应用商店部署

### App Store (iOS)

**要求 (2024-2025)：**
- Xcode 15+ 和 iOS 17 SDK (最低)
- Xcode 16+ 和 iOS 18 SDK (2025 推荐)
- 需要隐私清单
- 必须提供应用内账号删除

**发布流程：**
1. 在 Xcode 中归档
2. 上传至 App Store Connect
3. 提交审核
4. 分阶段发布 (7天推出)

**审核时间：**
- 平均：1-2 天
- 加急：1-2 小时 (仅限紧急情况)

**被拒原因：**
- 崩溃 (50%)
- 隐私违规 (25%)
- 信息不完整 (15%)
- 违反指南 (10%)

### Google Play (Android)

**要求 (2024-2025)：**
- 现在的目标版本 Android 14 (API 34)
- 2025年8月31日前目标版本 Android 15 (API 35)
- 需要隐私政策
- 需要数据安全表单

**发布流程：**
1. 构建签名 AAB (Android App Bundle)
2. 上传至 Play Console
3. 提交至生产轨道
4. 分阶段推出 (10% → 50% → 100%)

**审核时间：**
- 平均：1-3 天
- 更新：1-2 天

### 分阶段推出策略

**第 1 周：**
- 10% 用户
- 监控无崩溃率
- 观察关键 Bug

**第 2 周：**
- 50% 用户
- 验证性能指标
- 检查用户反馈

**第 3 周：**
- 100% 用户
- 如果指标健康则全面发布

**回滚触发条件：**
- 无崩溃率下降超过 5%
- 发现关键 Bug
- 重大用户投诉

## 跨平台比较

### Flutter vs React Native (2024-2025)

| 指标 | React Native | Flutter |
|--------|--------------|---------|
| **采用率** | 35% | 46% |
| **性能** | 80-90% | 85-95% |
| **应用体积** | 40-50MB | 15-20MB |
| **开发速度** | 快 | 非常快 |
| **商业化** | 12.57% | 5.24% |
| **开发者储备** | 20:1 比例 | 1 比例 |
| **适用场景** | JS 团队 | 性能优先 |

### 架构比较

**MVVM (小型应用)：**
```
View (视图)
 ↓
ViewModel (业务逻辑)
 ↓
Model (数据)
```

**整洁架构 (大型应用)：**
```
Presentation (表现层/UI)
 ↓
Domain (领域层/业务逻辑，用例)
 ↓
Data (数据层/仓库，API，数据库)
```

## 资源

**性能：**
- iOS: https://developer.apple.com/documentation/xcode/improving-your-app-s-performance
- Android: https://developer.android.com/topic/performance
- React Native: https://reactnative.dev/docs/performance

**分析：**
- Firebase: https://firebase.google.com/docs/analytics
- Sentry: https://docs.sentry.io/platforms/react-native/
- Amplitude: https://amplitude.com/docs

**安全：**
- OWASP Mobile: https://owasp.org/www-project-mobile-top-10/
- iOS Security: https://support.apple.com/guide/security/
- Android Security: https://source.android.com/docs/security

**测试：**
- Detox: https://wix.github.io/Detox/
- Appium: https://appium.io/docs/en/latest/
- XCTest: https://developer.apple.com/documentation/xctest
- Espresso: https://developer.android.com/training/testing/espresso
