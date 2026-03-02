**Language**: Simplified Chinese | [English](./TEST_GUIDE.md) | [한국어](./TEST_GUIDE.ko.md)

---

# Unity Editor Toolkit - 测试指南

运行 Unity Test Framework 测试的完整指南。

## 目录

1. [先决条件](#先决条件)
2. [测试设置](#测试设置)
3. [在 Unity 编辑器中运行测试](#在-unity-编辑器中运行测试)
4. [通过命令行运行测试](#通过命令行运行测试)
5. [理解测试结果](#理解测试结果)
6. [测试覆盖率](#测试覆盖率)
7. [故障排除](#故障排除)
8. [CI/CD 集成](#cicd-集成)
9. [最佳实践](#最佳实践)
10. [附加资源](#附加资源)
11. [快速命令参考](#快速命令参考)

---

## 先决条件

### 所需 Unity 版本

- **Unity 2020.3 或更高版本**

### Unity Test Framework (测试框架)

| Unity 版本 | 测试框架状态 |
|------------|--------------|
| **Unity 2019.2+** | ✅ **自动包含** 在所有项目中 |
| **Unity 6+ (2023.x+)** | ✅ **核心包 (Core Package)** (版本固定于编辑器) + 新功能 |

> **重要提示**：自 Unity 2019.2 起，Test Framework 已自动包含。在 Unity 6+ 中，它被归类为“核心包”，版本锁定为编辑器版本。

**Unity 6+ 新功能** (自 2023.2 起):
- ✅ 测试重试 (Retry) 和重复 (Repeat) 功能
- ✅ 随机测试执行顺序 (`-randomOrderSeed`)
- ✅ 自动生成 `TestFileReferences.json`
- ✅ SRP 测试自动更新

### 包安装

Unity Editor Toolkit 包包含以下测试程序集：

```
unity-package/
├── Tests/
│   ├── Editor/
│   │   ├── UnityEditorToolkit.Editor.Tests.asmdef
│   │   ├── UnityMainThreadDispatcherTests.cs (10 个测试)
│   │   ├── GameObjectCachingTests.cs (13 个测试)
│   │   ├── Vector3ValidationTests.cs (20 个测试)
│   │   └── JsonRpcProtocolTests.cs (23 个测试)
│   └── Runtime/
│       └── UnityEditorToolkit.Tests.asmdef
```

**总计：66 个自动化测试**

---

## 测试设置

### 1. 安装包

如果尚未安装，请通过 UPM 添加包：

```
Window → Package Manager → + → Add package from git URL
```

输入：
```
https://github.com/Dev-GOM/claude-code-marketplace.git?path=/plugins/unity-editor-toolkit/skills/assets/unity-package
```

### 2. 验证测试框架

**所有 Unity 版本 (2019.2+)**:
- 测试框架 **自动包含**
- 无需安装
- Test Runner 可立即使用

**如果缺少 Test Runner 窗口** (罕见情况):
1. `Window → Package Manager`
2. 验证 "Test Framework" 是否列在 "Built-in packages" (内置包) 中
3. 如果不可见，重启 Unity 编辑器

**Unity 6+ 特有**:
- 测试框架是 **核心包** (无法更改版本)
- 版本锁定为您的 Unity 编辑器版本
- 包含新功能：重试、重复、随机顺序

### 3. 打开 Test Runner

打开 Test Runner 窗口：

```
Window → General → Test Runner
```

或使用键盘快捷键：`Ctrl + Alt + T` (Windows/Linux) 或 `Cmd + Alt + T` (macOS)

---

## 在 Unity 编辑器中运行测试

### 方法 1: Test Runner 窗口 (推荐)

#### 第 1 步: 打开 Test Runner

`Window → General → Test Runner`

#### 第 2 步: 选择测试模式

点击 **"EditMode"** 选项卡 (当前所有测试均为 EditMode)

#### 第 3 步: 查看测试层级

您应该看到：

```
▼ UnityEditorToolkit.Editor.Tests
  ▼ UnityMainThreadDispatcherTests (10 tests)
  ▼ GameObjectCachingTests (13 tests)
  ▼ Vector3ValidationTests (20 tests)
  ▼ JsonRpcProtocolTests (23 tests)
```

#### 第 4 步: 运行测试

**运行所有测试：**
- 点击顶部的 `Run All` 按钮

**运行特定测试套件：**
- 右键点击测试类 (例如 "UnityMainThreadDispatcherTests")
- 选择 `Run`

**运行单个测试：**
- 右键点击单个测试 (例如 "Instance_Should_CreateSingleton")
- 选择 `Run`

#### 第 5 步: 查看结果

测试结果显示在 Test Runner 窗口中：

- ✅ **绿色对勾**: 测试通过
- ❌ **红色 X**: 测试失败
- ⏭️ **灰色横线**: 测试跳过

**示例输出:**
```
✓ UnityMainThreadDispatcherTests.Instance_Should_CreateSingleton (0.003s)
✓ UnityMainThreadDispatcherTests.Enqueue_Should_ExecuteAction_OnMainThread (0.012s)
✓ GameObjectCachingTests.FindGameObject_Should_FindExistingObject (0.008s)
✓ Vector3ValidationTests.ToVector3_Should_Throw_On_NaN_X (0.002s)

Total: 66 tests
Passed: 66
Failed: 0
Inconclusive: 0
Skipped: 0
Time: 2.345s
```

---

### 方法 2: 运行选定的测试

1. 在 Test Runner 中选择一个或多个测试
2. 点击 `Run Selected` 按钮

### 方法 3: 重新运行失败的测试

在一次有失败的测试运行之后：

1. 点击 `Rerun Failed` 按钮
2. 只有失败的测试会再次运行

---

## 通过命令行运行测试

### Windows

```batch
"C:\Program Files\Unity\Hub\Editor\2022.3.0f1\Editor\Unity.exe" ^
  -runTests ^
  -batchmode ^
  -projectPath "C:\Path\To\Your\UnityProject" ^
  -testResults "C:\Path\To\Results\test-results.xml" ^
  -testPlatform EditMode ^
  -logFile "C:\Path\To\Logs\unity-test.log"
```

### macOS / Linux

```bash
/Applications/Unity/Hub/Editor/2022.3.0f1/Unity.app/Contents/MacOS/Unity \
  -runTests \
  -batchmode \
  -projectPath "/path/to/your/UnityProject" \
  -testResults "/path/to/results/test-results.xml" \
  -testPlatform EditMode \
  -logFile "/path/to/logs/unity-test.log"
```

### 参数解释

- `-runTests`: 运行测试并退出
- `-batchmode`: 无 UI 运行 (headless 模式)
- `-projectPath`: Unity 项目路径
- `-testResults`: 输出 XML 文件路径 (NUnit 格式)
- `-testPlatform EditMode`: 运行 EditMode 测试
- `-logFile`: Unity 日志输出文件

### 退出代码

- `0`: 所有测试通过
- `2`: 部分测试失败
- `3`: 运行错误 (例如，编译错误)

---

## 理解测试结果

### Test Runner 结果

每个测试显示：

1. **状态图标**
   - ✅ 绿色: 通过
   - ❌ 红色: 失败
   - ⚠️ 黄色: 不确定 (Inconclusive)
   - ⏭️ 灰色: 跳过

2. **测试名称**
   - 格式: `ClassName.TestMethodName`
   - 示例: `UnityMainThreadDispatcherTests.Instance_Should_CreateSingleton`

3. **执行时间**
   - 以秒显示 (例如 `0.012s`)

4. **失败详情** (如果失败)
   - 堆栈跟踪 (Stack trace)
   - 预期值 vs. 实际值
   - 断言消息 (Assertion message)

### XML 测试结果

通过命令行运行时，结果保存为 NUnit XML 格式：

```xml
<?xml version="1.0" encoding="utf-8"?>
<test-results>
  <test-suite name="UnityEditorToolkit.Editor.Tests">
    <results>
      <test-case name="UnityMainThreadDispatcherTests.Instance_Should_CreateSingleton"
                 executed="True"
                 success="True"
                 time="0.003"/>
      <!-- 更多测试用例... -->
    </results>
  </test-suite>
</test-results>
```

---

## 测试覆盖率

### 当前测试覆盖率

| 组件 | 测试数量 | 覆盖范围 |
|-----------|-------|----------|
| **UnityMainThreadDispatcher** | 10 个测试 | 关键线程安全 |
| **GameObject Caching** | 13 个测试 | 性能优化 |
| **Vector3 Validation** | 20 个测试 | 安全性 (NaN/Infinity) |
| **JSON-RPC Protocol** | 23 个测试 | 协议合规性 |
| **总计** | **66 个测试** | **核心功能** |

### 已测试内容

#### 1. 线程安全 (关键)
- ✅ 单例实例创建
- ✅ 主线程执行验证
- ✅ 按顺序执行多个操作
- ✅ 异常处理
- ✅ 并发访问 (5 个线程)
- ✅ 从后台线程调用 Unity API

#### 2. 性能 (缓存)
- ✅ 带缓存的 GameObject 搜索
- ✅ 缓存命中性能 (快 10x-100x)
- ✅ 销毁时的缓存失效
- ✅ 非活动 GameObject 处理
- ✅ 嵌套 GameObject 支持
- ✅ 大规模缓存 (100+ 对象)

#### 3. 安全性 (验证)
- ✅ NaN 检测 (x, y, z)
- ✅ 正无穷大 (PositiveInfinity) 检测
- ✅ 负无穷大 (NegativeInfinity) 检测
- ✅ 有效值接受 (零, 正数, 负数)
- ✅ 浮点精度保留
- ✅ 边缘情况 (MaxValue, MinValue)

#### 4. 协议合规性 (JSON-RPC 2.0)
- ✅ 请求序列化/反序列化
- ✅ 响应序列化
- ✅ 带有请求 ID 保留的错误响应
- ✅ 错误代码 (-32700, -32600, -32601, -32602, -32603)
- ✅ 参数反序列化
- ✅ 复杂参数处理
- ✅ JSON-RPC 2.0 规范合规性

### 未测试内容 (未来计划)

- ⏳ WebSocket 通信 (集成测试)
- ⏳ 处理器实现 (GameObject, Transform 等)
- ⏳ 服务器生命周期 (启动, 停止, 重连)
- ⏳ 控制台日志收集
- ⏳ 场景加载
- ⏳ 撤销系统集成

---

## 故障排除

### 问题: "Test Framework package not found" (找不到测试框架包)

**Unity 6+ (2023.x+)**:
- 此问题不应发生 (Test Framework 是内置的)
- 如果缺少 Test Runner 窗口，请验证 Unity 安装

**Unity 2020.3 - 2022.x**:
1. 打开 Package Manager (包管理器)
2. 搜索 "Test Framework"
3. 安装 "Test Framework" 包
4. 重启 Unity 编辑器

### 问题: "Assembly reference errors" (程序集引用错误)

**解决方案:**
1. 检查 `Runtime/` 中是否存在 `UnityEditorToolkit.asmdef`
2. 验证测试程序集定义引用了主程序集：
   ```json
   {
     "references": ["UnityEditorToolkit"]
   }
   ```
3. 重新导入包：右键点击包 → Reimport (重新导入)

### 问题: Tests not appearing in Test Runner (测试未显示在 Test Runner 中)

**解决方案:**
1. 确保已设置 `UNITY_INCLUDE_TESTS` 定义 (对于 testables 自动设置)
2. 检查程序集定义是否包含 `"autoReferenced": false`
3. 重启 Unity 编辑器
4. 点击 Test Runner 窗口中的 "Refresh" (刷新)

### 问题: "DllNotFoundException: websocket-sharp"

**解决方案:**
测试不需要 websocket-sharp (没有实际的服务器连接)，但如果发生错误：

1. 检查 `ThirdParty/websocket-sharp/websocket-sharp.dll` 是否存在
2. 运行安装脚本 (参见 QUICKSTART.md)
3. 重启 Unity 编辑器

### 问题: Tests fail with "NullReferenceException" (测试失败并显示空引用异常)

**解决方案:**
1. 检查测试是否需要 GameObject 设置
2. 验证 `[SetUp]` 方法是否在每个测试前运行
3. 检查 `[TearDown]` 是否正确清理
4. 单独运行测试以隔离问题

### 问题: "UnityMainThreadDispatcher tests fail" (UnityMainThreadDispatcher 测试失败)

**解决方案:**
1. 确保测试使用 `[UnityTest]` 属性 (而不是 `[Test]`) 以支持协程
2. 添加 `yield return null;` 以等待 Update() 执行
3. 检查是否有多个调度器实例冲突 (在 TearDown 中清理)

### 问题: Slow test execution (测试执行缓慢)

**预期时间:**
- 所有测试: ~2-5 秒
- UnityMainThreadDispatcher: ~0.5s (包含帧等待)
- GameObjectCaching: ~0.3s
- Vector3Validation: ~0.1s (纯逻辑)
- JsonRpcProtocol: ~0.2s (序列化)

**如果较慢:**
1. 检查 Unity 是否在测试期间重新编译
2. 关闭不必要的编辑器窗口
3. 禁用自动刷新: `Edit → Preferences → Asset Pipeline → Auto Refresh (off)`

### 问题: "Could not load assembly" (无法加载程序集)

**解决方案:**
1. 删除 `Library/` 文件夹
2. 重启 Unity (强制重新编译)
3. 等待编译完成
4. 重新打开 Test Runner

---

## CI/CD 集成

### GitHub Actions 示例

```yaml
name: Unity Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - uses: game-ci/unity-test-runner@v2
        with:
          projectPath: ./
          testMode: EditMode
          artifactsPath: test-results

      - uses: actions/upload-artifact@v3
        if: always()
        with:
          name: test-results
          path: test-results
```

### Jenkins 示例

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh '''
                    /Applications/Unity/Hub/Editor/2022.3.0f1/Unity.app/Contents/MacOS/Unity \
                      -runTests \
                      -batchmode \
                      -projectPath $WORKSPACE \
                      -testResults test-results.xml \
                      -testPlatform EditMode
                '''
            }
        }
    }
    post {
        always {
            junit 'test-results.xml'
        }
    }
}
```

---

## 最佳实践

### 何时运行测试

1. **提交代码前**: 运行所有测试
2. **修改核心代码后**: 运行受影响的测试
3. **发布前**: 运行完整测试套件
4. **在 CI/CD 中**: 每次推送/PR

### 编写新测试

添加新功能时，请遵循现有模式：

```csharp
[Test]
public void FeatureName_Should_DoExpectedBehavior()
{
    // Arrange: Set up test conditions (设置测试条件)
    var testObject = new GameObject("Test");

    // Act: Execute the code being tested (执行被测代码)
    var result = handler.DoSomething(testObject);

    // Assert: Verify expected outcome (验证预期结果)
    Assert.IsNotNull(result);
    Assert.AreEqual(expectedValue, result);

    // Cleanup: Destroy objects (清理：销毁对象)
    Object.DestroyImmediate(testObject);
}
```

### 测试命名约定

格式: `MethodName_Should_ExpectedBehavior_WhenCondition` (方法名_应该_预期行为_当条件满足时)

示例:
- ✅ `FindGameObject_Should_ReturnNull_WhenNotFound`
- ✅ `ToVector3_Should_Throw_On_NaN_X`
- ✅ `Enqueue_Should_ExecuteAction_OnMainThread`

---

## 附加资源

### Unity 文档

- [Unity Test Framework 手册](https://docs.unity3d.com/Packages/com.unity.test-framework@latest)
- [编写测试](https://docs.unity3d.com/Packages/com.unity.test-framework@latest/index.html?subfolder=/manual/workflow-create-test.html)
- [测试属性](https://docs.unity3d.com/Packages/com.unity.test-framework@latest/index.html?subfolder=/manual/reference-attribute.html)

### NUnit 文档

- [NUnit 断言](https://docs.nunit.org/articles/nunit/writing-tests/assertions/assertion-models/constraint.html)
- [测试属性](https://docs.nunit.org/articles/nunit/writing-tests/attributes.html)

### Unity Editor Toolkit

- [主 README](../README.md)
- [快速入门指南](./QUICKSTART.md)
- [命令路线图](./COMMANDS.md)
- **已实现的命令类别:**
  - [连接与状态](./COMMANDS_CONNECTION_STATUS.md)
  - [GameObject 与层级](./COMMANDS_GAMEOBJECT_HIERARCHY.md)
  - [Transform (变换)](./COMMANDS_TRANSFORM.md)
  - [场景管理](./COMMANDS_SCENE.md)
  - [Asset 数据库与编辑器](./COMMANDS_EDITOR.md)
  - [控制台与日志](./COMMANDS_CONSOLE.md)

---

## 快速命令参考

```bash
# 打开 Test Runner
Window → General → Test Runner
快捷键: Ctrl+Alt+T (Win/Linux) 或 Cmd+Alt+T (Mac)

# 运行所有测试
点击 Test Runner 中的 "Run All" 按钮

# 通过命令行运行 (Windows)
Unity.exe -runTests -batchmode -projectPath "path" -testResults "results.xml" -testPlatform EditMode

# 查看测试结果
在任何 XML 查看器或 CI 工具中打开 test-results.xml
```

---

**测试总数**: 66
