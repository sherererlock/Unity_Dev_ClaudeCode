# 移动端调试策略

2024-2025 年移动开发的综合调试技术、工具和最佳实践。

## 移动调试思维

### 独特的移动挑战

1.  **设备多样性** - 数千种设备/操作系统组合
2.  **资源限制** - 有限的 CPU、内存、电池
3.  **网络多变性** - 从 WiFi 到 2G，以及离线场景
4.  **平台差异** - iOS 与 Android 的行为差异
5.  **真机测试** - 模拟器无法反映真实性能
6.  **有限的调试访问** - 无法 SSH 进入生产设备

### 调试哲学

**黄金法则：**
1.  **在真机上测试** - 模拟器在性能上会撒谎
2.  **持续复现** - 间歇性 Bug 需要可复现的步骤
3.  **先检查显而易见之处** - 网络、权限、资源
4.  **隔离平台** - 是 iOS 特有、Android 特有，还是两者皆有？
5.  **监控资源** - CPU、内存、电池、网络
6.  **阅读日志** - 设备日志包含关键线索

## 平台特定调试工具

### iOS 调试

**1. Xcode 调试器**

```swift
// 断点调试
func fetchUserData(userId: String) {
    // 在此处设置断点
    let url = URL(string: "https://api.example.com/users/\(userId)")!

    // LLDB 命令：
    // po userId - 打印对象
    // p url - 打印变量
    // bt - 回溯堆栈 (backtrace)
    // c - 继续 (continue)
    // step - 单步进入 (step into)
    // next - 单步跳过 (step over)
}
```

**LLDB 高级命令：**
```bash
# 条件断点
breakpoint set --name fetchUserData --condition userId == "123"

# 观察点 (值改变时中断)
watchpoint set variable self.counter

# 打印视图层级
po UIApplication.shared.keyWindow?.value(forKey: "recursiveDescription")

# 打印所有属性
po self.value(forKey: "description")
```

**2. Instruments (性能分析)**

**Time Profiler** - CPU 使用率
```
1. Xcode → Product → Profile
2. 选择 "Time Profiler"
3. 在使用 App 时录制
4. 识别热点方法 (高自身时间/self time)
```

**Allocations** - 内存使用
```
1. 选择 "Allocations" 工具
2. 寻找内存增长
3. 按对象类型过滤
4. 查找分配堆栈跟踪
```

**Leaks** - 内存泄漏
```
1. 选择 "Leaks" 工具
2. 泄漏显示为红色
3. 点击泄漏查看堆栈跟踪
4. 修复循环引用
```

**Network** - API 调试
```
1. 选择 "Network" 工具
2. 查看所有 HTTP 请求
3. 响应时间、大小
4. 失败的请求高亮显示
```

**3. 视图调试**

```swift
// Xcode 中的视图层级
// Debug → View Debugging → Capture View Hierarchy

// 运行时检查
#if DEBUG
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            Text("Hello")
        }
        .onAppear {
            // 打印视图树以供调试
            print(Mirror(reflecting: self.body))
        }
    }
}
#endif
```

**4. Console.app (系统日志)**

```bash
# 按进程过滤日志
log stream --predicate 'processImagePath contains "YourApp"' --level debug

# 按子系统过滤
log stream --predicate 'subsystem == "com.yourcompany.yourapp"'

# 仅显示错误
log stream --predicate 'processImagePath contains "YourApp"' --level error
```

**5. Network Link Conditioner**

```
设置 → 开发者 → Network Link Conditioner

模拟：
- 3G, LTE, WiFi
- 高延迟
- 丢包
- 带宽限制
```

### Android 调试

**1. Android Studio 调试器**

```kotlin
// 断点调试
fun fetchUserData(userId: String) {
    // 在此处设置断点
    val url = "https://api.example.com/users/$userId"

    // 调试器命令：
    // 计算表达式：Alt+F8 (Windows) / Cmd+F8 (Mac)
    // 单步跳过 (Step over)：F8
    // 单步进入 (Step into)：F7
    // 恢复 (Resume)：F9
}
```

**高级调试器功能：**
```kotlin
// 条件断点
// 右键点击断点 → Condition: userId == "123"

// 日志点 (记录日志而不暂停)
// 右键点击断点 → More → 勾选 "Evaluate and log"

// 异常断点
// Run → View Breakpoints → + → Java Exception Breakpoints
```

**2. Android Profiler**

**CPU Profiler:**
```
View → Tool Windows → Profiler → CPU
- 记录跟踪 (Record trace)
- 识别慢速方法
- 火焰图显示调用层级
```

**Memory Profiler:**
```
View → Tool Windows → Profiler → Memory
- 跟踪分配
- 堆转储 (Heap dump) 分析
- 查找内存泄漏
```

**Network Profiler:**
```
View → Tool Windows → Profiler → Network
- 所有 HTTP 请求
- 请求/响应详情
- 时间轴视图
```

**3. 布局检查器 (Layout Inspector)**

```
Tools → Layout Inspector

功能：
- 3D 视图层级
- 实时布局更新
- 视图属性
- 约束可视化
```

**4. ADB (Android 调试桥)**

```bash
# 查看设备日志
adb logcat

# 按应用过滤
adb logcat | grep com.yourcompany.yourapp

# 按标签过滤
adb logcat MyTag:D *:S

# 清除日志
adb logcat -c

# 安装 APK
adb install app-debug.apk

# 卸载应用
adb uninstall com.yourcompany.yourapp

# 截屏
adb shell screencap -p /sdcard/screenshot.png
adb pull /sdcard/screenshot.png

# 录屏
adb shell screenrecord /sdcard/demo.mp4
adb pull /sdcard/demo.mp4
```

**5. 网络模拟**

```bash
# 模拟器网络节流
# Settings → Network → Network Profile

# 或通过 ADB
adb shell setprop net.dns1 8.8.8.8
```

### React Native 调试

**1. React DevTools**

```bash
# 安装
npm install -g react-devtools

# 启动
react-devtools

# 在 App 中：摇晃设备 → "Debug with React DevTools"
```

**2. Flipper (推荐)**

```bash
# 安装
npm install -g flipper

# 在 App 中配置
# 添加 flipper 包到你的应用
npm install --save-dev react-native-flipper

# 功能：
# - 布局检查器
# - 网络检查器
# - Redux DevTools
# - 数据库查看器
# - Shared Preferences 查看器
```

**3. Chrome DevTools**

```javascript
// 在 App 中：摇晃设备 → "Debug"
// 打开 Chrome DevTools

// Console.log 出现在 Chrome 中
console.log('User data:', userData);

// 在源代码中设置断点
debugger; // 暂停执行

// Network 标签页显示 API 调用
fetch('https://api.example.com/users')
  .then(res => res.json())
  .then(data => console.log(data));
```

**4. React Native Debugger (独立应用)**

```bash
# 安装
brew install --cask react-native-debugger

# 启动
open "rndebugger://set-debugger-loc?host=localhost&port=8081"

# 功能：
# - Redux DevTools
# - React DevTools
# - 网络检查器
# - 控制台 (Console)
```

**5. 性能监视器**

```javascript
// 显示应用内性能覆盖层
// 摇晃设备 → "Show Perf Monitor"

// 显示：
// - RAM 使用量
// - JS 帧率
// - UI 帧率
// - 视图数量
```

**6. LogBox**

```javascript
// 忽略特定警告
import { LogBox } from 'react-native';

LogBox.ignoreLogs([
  'Warning: componentWillReceiveProps',
]);

// 忽略所有日志 (不推荐)
LogBox.ignoreAllLogs();
```

### Flutter 调试

**1. DevTools**

```bash
# 从 VS Code 启动
# Debug → Open DevTools

# 或从命令行启动
flutter pub global activate devtools
flutter pub global run devtools

# 功能：
# - Widget 检查器
# - 时间轴视图
# - 内存分析器
# - 网络分析器
# - 日志视图
```

**2. Widget 检查器**

```dart
// 在 DevTools 中：Inspector 标签页

// 调试绘制 (显示布局边框)
// Ctrl+Shift+P → "Toggle Debug Painting"

// 打印 Widget 树
debugDumpApp();

// 打印渲染树
debugDumpRenderTree();

// 打印图层树
debugDumpLayerTree();
```

**3. 性能覆盖层 (Performance Overlay)**

```dart
void main() {
  runApp(
    MaterialApp(
      showPerformanceOverlay: true, // FPS 计数器
      debugShowCheckedModeBanner: false,
      home: MyApp(),
    ),
  );
}
```

**4. 日志记录**

```dart
import 'dart:developer' as developer;

// 简单打印
print('User ID: $userId');

// 结构化日志
developer.log(
  'User logged in',
  name: 'app.auth',
  error: error,
  stackTrace: stackTrace,
);

// 时间轴事件
developer.Timeline.startSync('fetchUsers');
await fetchUsers();
developer.Timeline.finishSync();
```

**5. 断点调试**

```dart
// 在 VS Code 或 Android Studio 中设置断点
Future<User> fetchUser(String id) async {
  // 在此处断点
  final response = await http.get(Uri.parse('https://api.example.com/users/$id'));

  // 调试控制台命令：
  // p variable - 打印变量
  // 单步跳过 (Step over)：F10
  // 单步进入 (Step into)：F11
  // 继续 (Continue)：F5
  return User.fromJson(jsonDecode(response.body));
}
```

## UI 调试

### 布局问题

**iOS (SwiftUI):**
```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Text("Hello")
        }
        .border(Color.red) // 调试边框
        .background(Color.yellow.opacity(0.3)) // 调试背景
    }
}

// 打印布局信息
Text("Hello")
    .onAppear {
        print("Frame: \(UIScreen.main.bounds)")
    }
```

**Android (Jetpack Compose):**
```kotlin
@Composable
fun DebugLayout() {
    Column(
        modifier = Modifier
            .border(2.dp, Color.Red) // 调试边框
            .background(Color.Yellow.copy(alpha = 0.3f)) // 调试背景
    ) {
        Text("Hello")
    }
}

// 在开发者选项中显示布局边界
// Settings → Developer Options → Show layout bounds
```

**React Native:**
```javascript
// 调试边框
<View style={{ borderWidth: 1, borderColor: 'red' }}>
  <Text>Hello</Text>
</View>

// 布局动画调试
import { LayoutAnimation, UIManager } from 'react-native';

UIManager.setLayoutAnimationEnabledExperimental &&
  UIManager.setLayoutAnimationEnabledExperimental(true);

// 检查器 (Inspector)
// 摇晃设备 → "Toggle Inspector"
// 显示元素层级和样式
```

**Flutter:**
```dart
// 调试绘制
void main() {
  debugPaintSizeEnabled = true; // 显示布局辅助线
  debugPaintBaselinesEnabled = true; // 显示文本基线
  debugPaintLayerBordersEnabled = true; // 显示图层边框
  runApp(MyApp());
}

// Widget 边界
Container(
  decoration: BoxDecoration(
    border: Border.all(color: Colors.red, width: 2),
  ),
  child: Text('Hello'),
)
```

### 动画调试

**缓慢动画：**
```dart
// Flutter: 减慢动画
timeDilation = 5.0; // 慢 5 倍

// React Native: 慢动画
import { Animated } from 'react-native';
Animated.timing(value, {
  toValue: 1,
  duration: 3000, // 增加持续时间
});
```

**动画性能：**
```swift
// iOS: Core Animation Instrument
// Instruments → Core Animation
// 检查：
// - 丢帧 (Dropped frames)
// - 离屏渲染 (Off-screen rendering)
// - 图层混合 (Blending layers)
```

## 性能调试

### 帧率问题 (< 60 FPS)

**诊断：**

**React Native:**
```javascript
// 启用性能监视器
// 显示 JS 和 UI 线程 FPS

// 常见问题：
// 1. 渲染中进行繁重计算
// 2. 长列表未使用虚拟化
// 3. 不必要的重新渲染
```

**解决方案：**
```javascript
// ❌ 坏：渲染中进行繁重计算
function UserList({ users }) {
  const sortedUsers = users.sort((a, b) => a.name.localeCompare(b.name));
  return <FlatList data={sortedUsers} />;
}

// ✅ 好：记忆 (Memoize) 昂贵的操作
function UserList({ users }) {
  const sortedUsers = useMemo(
    () => users.sort((a, b) => a.name.localeCompare(b.name)),
    [users]
  );
  return <FlatList data={sortedUsers} />;
}

// ❌ 坏：ScrollView 处理大数据
<ScrollView>
  {users.map(user => <UserCard key={user.id} user={user} />)}
</ScrollView>

// ✅ 好：使用带虚拟化的 FlatList
<FlatList
  data={users}
  renderItem={({ item }) => <UserCard user={item} />}
  keyExtractor={item => item.id}
  windowSize={5}
  initialNumToRender={10}
/>
```

**Flutter:**
```dart
// 检查：
// - 构建阶段过长
// - 布局阶段过长
// - 绘制阶段过长

// 使用 const 构造函数
// ❌ 坏
Widget build(BuildContext context) {
  return Container(child: Text('Hello'));
}

// ✅ 好
Widget build(BuildContext context) {
  return const Text('Hello');
}

// 避免昂贵的构建
// 对有状态 Widget 使用 Key
ListView.builder(
  itemBuilder: (context, index) {
    return UserCard(
      key: ValueKey(users[index].id), // 保留状态
      user: users[index],
    );
  },
)
```

### 内存问题

**检测：**

**iOS:**
```
Xcode → Debug Navigator → Memory
- 观察内存图
- 寻找持续增长
```

**Android:**
```
Android Studio → Profiler → Memory
- 获取堆转储 (Take heap dump)
- 分析保留对象 (Retained objects)
```

**常见原因：**

```javascript
// React Native: 内存泄漏

// ❌ 坏：未移除事件监听器
useEffect(() => {
  EventEmitter.on('data', handleData);
  // 缺少清理
}, []);

// ✅ 好：清理
useEffect(() => {
  EventEmitter.on('data', handleData);
  return () => {
    EventEmitter.off('data', handleData);
  };
}, []);

// ❌ 坏：未清除定时器
useEffect(() => {
  setInterval(() => {
    console.log('tick');
  }, 1000);
}, []);

// ✅ 好：清除定时器
useEffect(() => {
  const timer = setInterval(() => {
    console.log('tick');
  }, 1000);
  return () => clearInterval(timer);
}, []);
```

```dart
// Flutter: 销毁控制器
class MyWidget extends StatefulWidget {
  @override
  _MyWidgetState createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  late TextEditingController _controller;

  @override
  void initState() {
    super.initState();
    _controller = TextEditingController();
  }

  @override
  void dispose() {
    _controller.dispose(); // 必须销毁
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return TextField(controller: _controller);
  }
}
```

## 网络调试

### HTTP 调试

**iOS (Proxyman / Charles)**
```
1. 安装 Proxyman (免费) 或 Charles
2. 配置设备代理
3. 安装 SSL 证书
4. 查看所有 HTTP 流量
```

**Android (Charles / Flipper)**
```
1. 安装 Charles Proxy
2. 配置设备代理：Settings → WiFi → Modify → Proxy
3. 安装 Charles 证书
4. 查看所有 HTTP 请求/响应
```

**React Native (Flipper Network Plugin)**
```javascript
// 自动捕获所有 fetch/axios 请求
fetch('https://api.example.com/users')
  .then(res => res.json())
  .then(data => console.log(data));

// 在 Flipper 中查看：
// - 请求/响应头
// - 请求/响应体
// - 计时信息
```

**Flutter (DevTools Network Tab)**
```dart
// 自动捕获 HTTP 请求
final response = await http.get(
  Uri.parse('https://api.example.com/users')
);

// 在 DevTools Network 标签页中查看：
// - 所有 HTTP 请求
// - 标头和正文
// - 响应时间
```

### 网络模拟

**测试场景：**
- 慢速网络 (3G, 2G)
- 高延迟 (500ms+)
- 丢包 (10%)
- 离线模式

**iOS:**
```
Settings → Developer → Network Link Conditioner
```

**Android:**
```
模拟器：Settings → Network → Network Profile
```

## 崩溃调试

### 崩溃报告服务

**Firebase Crashlytics (推荐)**

**React Native:**
```javascript
import crashlytics from '@react-native-firebase/crashlytics';

// 记录自定义事件
crashlytics().log('User pressed purchase button');

// 设置用户标识符
crashlytics().setUserId(userId);

// 记录非致命错误
try {
  await fetchData();
} catch (error) {
  crashlytics().recordError(error);
}

// 强制崩溃以进行测试
crashlytics().crash();
```

**Flutter:**
```dart
import 'package:firebase_crashlytics/firebase_crashlytics.dart';

// 捕获错误
FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterError;

// 捕获异步错误
runZonedGuarded(() {
  runApp(MyApp());
}, (error, stackTrace) {
  FirebaseCrashlytics.instance.recordError(error, stackTrace);
});

// 记录自定义事件
FirebaseCrashlytics.instance.log('User pressed purchase');

// 设置用户 ID
FirebaseCrashlytics.instance.setUserIdentifier(userId);
```

**iOS Native:**
```swift
import FirebaseCrashlytics

// 记录事件
Crashlytics.crashlytics().log("User tapped button")

// 设置用户 ID
Crashlytics.crashlytics().setUserID(userId)

// 记录错误
Crashlytics.crashlytics().record(error: error)
```

**Android Native:**
```kotlin
import com.google.firebase.crashlytics.FirebaseCrashlytics

// 记录事件
FirebaseCrashlytics.getInstance().log("User tapped button")

// 设置用户 ID
FirebaseCrashlytics.getInstance().setUserId(userId)

// 记录异常
FirebaseCrashlytics.getInstance().recordException(exception)
```

### 分析崩溃报告

**iOS (Xcode Organizer):**
```
Window → Organizer → Crashes
- 符号化的崩溃日志
- 堆栈跟踪
- 崩溃计数
```

**Android (Play Console):**
```
Play Console → Quality → Crashes & ANRs
- 崩溃堆栈跟踪
- 受影响设备
- 操作系统版本
```

**阅读堆栈跟踪：**
```
Fatal Exception: java.lang.NullPointerException
Attempt to invoke virtual method 'java.lang.String User.getName()' on a null object reference
    at com.example.app.UserService.displayUser(UserService.kt:42)
    at com.example.app.MainActivity.onCreate(MainActivity.kt:23)

修复：
1. 检查 UserService.kt 第 42 行
2. User 对象为空
3. 在访问 getName() 之前添加空值检查
```

## 常见调试场景

### 1. 应用启动时崩溃

**步骤：**
1. 检查崩溃日志
2. 寻找初始化错误
3. 验证依赖项是否加载
4. 检查权限

**示例：**
```javascript
// React Native: 缺少原生依赖
// Error: Invariant Violation: Native module cannot be null

// 修复：链接原生模块
npx react-native link <module-name>
# 或
cd ios && pod install
```

### 2. UI 不更新

**React Native:**
```javascript
// ❌ 坏：直接修改状态
this.state.users.push(newUser); // 不会触发重新渲染

// ✅ 好：创建新状态
this.setState({ users: [...this.state.users, newUser] });
```

**Flutter:**
```dart
// ❌ 坏：未调用 setState
void addUser(User user) {
  users.add(user); // 不会重建
}

// ✅ 好：调用 setState
void addUser(User user) {
  setState(() {
    users.add(user);
  });
}
```

### 3. 图片不加载

**常见原因：**
1. 错误的 URL
2. CORS 问题
3. SSL 证书问题
4. 网络超时

**调试：**
```javascript
// React Native
<Image
  source={{ uri: imageUrl }}
  onError={(error) => console.log('Image error:', error)}
  onLoad={() => console.log('Image loaded')}
/>

// 检查 Network 标签页是否有 404, 403 等
```

### 4. 键盘遮挡输入框

**React Native:**
```javascript
import { KeyboardAvoidingView } from 'react-native';

<KeyboardAvoidingView
  behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
  style={{ flex: 1 }}
>
  <TextInput placeholder="Email" />
</KeyboardAvoidingView>
```

**Flutter:**
```dart
// Scaffold 自动处理
Scaffold(
  resizeToAvoidBottomInset: true, // 默认值
  body: TextField(),
)
```

### 5. 导航不工作

**React Navigation:**
```javascript
// ❌ 坏：Navigation 属性不可用
function MyComponent() {
  navigation.navigate('Home'); // 错误
}

// ✅ 好：使用 hook 或属性
function MyComponent({ navigation }) {
  // 或
  // const navigation = useNavigation();

  navigation.navigate('Home');
}
```

## 生产环境调试

### 远程日志

**LogRocket (会话回放)**
```javascript
import LogRocket from '@logrocket/react-native';

LogRocket.init('your-app-id');

// 识别用户
LogRocket.identify(userId, {
  name: user.name,
  email: user.email,
});

// 回放用户会话，包含：
// - 控制台日志
// - 网络请求
// - UI 交互
// - Redux 动作
```

### 用于调试的功能开关 (Feature Flags)

```javascript
import { useFlags } from 'launchdarkly-react-native-client-sdk';

function MyComponent() {
  const { debugMode } = useFlags();

  if (debugMode) {
    console.log('Debug info:', userData);
  }

  return <View>...</View>;
}

// 为特定用户远程启用调试模式
```

### 用于 Bug 调查的 A/B 测试

```javascript
// 逐步推出修复
if (abTest.variant === 'fixed') {
  return <FixedComponent />;
} else {
  return <OriginalComponent />;
}

// 监控每个变体的崩溃率
```

## 调试清单

**提交 Bug 之前：**
- [ ] 在真机上复现
- [ ] 检查 iOS 和 Android 两端
- [ ] 在多个 OS 版本上测试
- [ ] 验证网络连接
- [ ] 检查应用权限
- [ ] 审查最近的代码更改
- [ ] 检查崩溃日志

**调查：**
- [ ] 启用调试日志
- [ ] 使用平台调试器
- [ ] 如果慢，进行性能分析
- [ ] 监控内存使用
- [ ] 检查网络请求
- [ ] 检查 UI 层级

**生产环境问题：**
- [ ] 检查崩溃报告仪表板
- [ ] 审查用户报告的问题
- [ ] 分析受影响的 OS 版本
- [ ] 检查受影响的设备
- [ ] 审查最近的应用发布
- [ ] 比较无崩溃率

**修复之后：**
- [ ] 在真机上测试
- [ ] 在受影响的 OS 版本上验证
- [ ] 添加回归测试
- [ ] 分阶段发布 (10% → 100%)
- [ ] 监控崩溃率

## 资源

**通用：**
- React Native 调试: https://reactnative.dev/docs/debugging
- Flutter DevTools: https://docs.flutter.dev/tools/devtools
- iOS 调试: https://developer.apple.com/documentation/xcode/debugging
- Android 调试: https://developer.android.com/studio/debug

**崩溃报告：**
- Firebase Crashlytics: https://firebase.google.com/docs/crashlytics
- Sentry: https://docs.sentry.io/platforms/react-native/
- Bugsnag: https://docs.bugsnag.com/

**性能：**
- iOS Instruments: https://developer.apple.com/instruments/
- Android Profiler: https://developer.android.com/studio/profile
- Flipper: https://fbflipper.com/

**网络：**
- Proxyman: https://proxyman.io/
- Charles Proxy: https://www.charlesproxy.com/
- Flipper Network Plugin: https://fbflipper.com/docs/features/network-plugin/
