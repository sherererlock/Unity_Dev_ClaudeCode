---
name: unity-test-runner
description: 从命令行执行并分析 Unity 测试框架的测试。此技能通过检测 Unity 编辑器、配置测试参数（EditMode/PlayMode）、通过 CLI 运行测试、解析 XML 结果以及生成详细的失败报告，来实现 Unity 项目测试执行的自动化。当需要运行 Unity 测试、验证游戏逻辑或调试测试失败时，请使用此技能。
---

# Unity 测试运行器 (Unity Test Runner)

## 概述

此技能支持直接从命令行自动执行和分析 Unity 测试框架（Unity Test Framework）的测试。它处理完整的测试工作流：在不同平台（Windows/macOS/Linux）上检测 Unity 编辑器安装、配置测试参数、在 EditMode（编辑模式）或 PlayMode（播放模式）下执行测试、解析 NUnit XML 结果，并生成带有可操作见解的详细失败报告。

## 何时使用此技能

在以下情况下使用此技能：
- 从命令行执行 Unity 测试框架的测试
- 运行 PlayMode 或 EditMode 测试以验证游戏逻辑
- 分析测试失败并生成失败报告
- 将 Unity 测试集成到 CI/CD 管道中
- 通过详细的堆栈跟踪和文件位置调试测试失败
- 在提交前验证 Unity 项目更改

**示例用户请求：**
- "运行我项目中的所有 Unity 测试"
- "执行 PlayMode 测试并向我展示结果"
- "运行 Combat（战斗）类别的测试"
- "检查我的 Unity 测试是否通过"
- "仅运行 EditMode 测试"

## 工作流

当调用此技能时，请遵循以下工作流：

### 1. 检测 Unity 编辑器安装

使用 `find-unity-editor.js` 脚本自动定位 Unity 编辑器：

```bash
node scripts/find-unity-editor.js --json
```

**脚本行为：**
- 扫描特定平台的默认安装路径
- 检测所有已安装的 Unity 版本
- 默认返回最新版本
- 可以使用 `--version <version>` 标志指定特定版本

**输出：**
```json
{
  "found": true,
  "editorPath": "C:\\Program Files\\Unity\\Hub\\Editor\\2021.3.15f1\\Editor\\Unity.exe",
  "version": "2021.3.15f1",
  "platform": "win32",
  "allVersions": ["2021.3.15f1", "2020.3.30f1"]
}
```

**如果找到多个版本：**
1. 向用户展示所有可用版本
2. 询问用户确认使用哪个版本
3. 或者默认使用最新版本

**如果未找到 Unity 编辑器：**
- 报告错误及已搜索的路径
- 要求用户手动提供 Unity 编辑器路径
- 存储该路径以供将来使用

### 2. 验证 Unity 项目路径

使用跨平台检查确认当前目录包含有效的 Unity 项目：

```typescript
// Use Read tool to check for Unity project indicators
Read({ file_path: "ProjectSettings/ProjectVersion.txt" })

// Use Glob to verify Assets directory exists
Glob({ pattern: "Assets/*", path: "." })
```

**验证步骤：**
1. 验证 `Assets/` 目录是否存在
2. 验证 `ProjectSettings/ProjectVersion.txt` 是否存在
3. 读取 `ProjectVersion.txt` 以获取 Unity 版本
4. 如果编辑器版本与项目版本不匹配，发出警告

**ProjectVersion.txt 示例：**
```
m_EditorVersion: 2021.3.15f1
m_EditorVersionWithRevision: 2021.3.15f1 (e8e88743f9e5)
```

### 3. 配置测试设置

确定测试执行参数。如果未指定参数，请使用 `AskUserQuestion` 工具：

**必需设置：**
- **测试模式 (Test Mode)**：EditMode、PlayMode 或 Both（两者）
- **测试平台 (Test Platform)**：EditMode 测试使用 "EditMode"，PlayMode 可以指定平台（例如 "StandaloneWindows64", "Android", "iOS"）

**可选设置：**
- **测试类别 (Test Categories)**：分号分隔的列表（例如 "Combat;AI;Physics"）
- **测试过滤器 (Test Filter)**：正则表达式模式或分号分隔的测试名称
- **结果输出路径 (Results Output Path)**：默认为项目根目录下的 `TestResults.xml`

**配置示例：**
```typescript
AskUserQuestion({
  questions: [{
    question: "Which test mode should be executed?",
    header: "Test Mode",
    multiSelect: false,
    options: [
      { label: "EditMode Only", description: "Fast unit tests without Play Mode" },
      { label: "PlayMode Only", description: "Full Unity engine tests" },
      { label: "Both Modes", description: "Run all tests (slower)" }
    ]
  }]
})
```

### 4. 通过命令行执行测试

构建并执行 Unity 命令行测试命令：

**命令结构：**
```bash
<UnityEditorPath> -runTests -batchmode -projectPath <ProjectPath> \
  -testPlatform <EditMode|PlayMode> \
  -testResults <OutputPath> \
  [-testCategory <Categories>] \
  [-testFilter <Filter>] \
  -logFile -
```

**示例命令：**

**EditMode 测试：**
```bash
"C:\Program Files\Unity\Hub\Editor\2021.3.15f1\Editor\Unity.exe" \
  -runTests -batchmode \
  -projectPath "D:\Projects\MyGame" \
  -testPlatform EditMode \
  -testResults "TestResults-EditMode.xml" \
  -logFile -
```

**带类别过滤器的 PlayMode 测试：**
```bash
"C:\Program Files\Unity\Hub\Editor\2021.3.15f1\Editor\Unity.exe" \
  -runTests -batchmode \
  -projectPath "D:\Projects\MyGame" \
  -testPlatform PlayMode \
  -testResults "TestResults-PlayMode.xml" \
  -testCategory "Combat;AI" \
  -logFile -
```

**执行注意事项：**
- 对于长时间运行的测试，使用带有 `run_in_background: true` 的 `Bash` 工具
- 适当设置超时（默认：5-10 分钟，根据测试数量调整）
- 监控输出以获取进度指示
- 捕获 stdout 和 stderr

**执行示例：**
```typescript
Bash({
  command: `"${unityPath}" -runTests -batchmode -projectPath "${projectPath}" -testPlatform EditMode -testResults "TestResults.xml" -logFile -`,
  description: "Execute Unity EditMode tests",
  timeout: 300000, // 5 minutes
  run_in_background: true
})
```

### 5. 解析测试结果

测试完成后，使用 `parse-test-results.js` 解析 NUnit XML 结果：

```bash
node scripts/parse-test-results.js TestResults.xml --json
```

**脚本输出：**
```json
{
  "summary": {
    "total": 10,
    "passed": 7,
    "failed": 2,
    "skipped": 1,
    "duration": 12.345
  },
  "failures": [
    {
      "name": "TestPlayerTakeDamage",
      "fullName": "Tests.Combat.PlayerTests.TestPlayerTakeDamage",
      "message": "Expected: 90\n  But was: 100",
      "stackTrace": "at Tests.Combat.PlayerTests.TestPlayerTakeDamage () [0x00001] in Assets/Tests/Combat/PlayerTests.cs:42",
      "file": "Assets/Tests/Combat/PlayerTests.cs",
      "line": 42
    }
  ],
  "allTests": [...]
}
```

**结果分析：**
1. 提取测试摘要统计信息
2. 识别所有失败的测试
3. 从堆栈跟踪中提取文件路径和行号
4. 按类型（断言、异常、超时）对失败进行分类

### 6. 分析测试失败

对于每个失败的测试，使用 `references/test-patterns.json` 分析失败原因：

**分析步骤：**

1. **加载测试模式数据库：**
```typescript
Read({ file_path: "references/test-patterns.json" })
```

2. **将失败消息与模式匹配：**
   - 断言失败：`Expected: <X> But was: <Y>`
   - 空引用失败：`Expected: not null But was: <null>`
   - 超时失败：`TimeoutException|Test exceeded time limit`
   - 线程错误：`Can't be called from.*main thread`
   - 对象生命周期问题：`has been destroyed|MissingReferenceException`

3. **确定失败类别：**
   - ValueMismatch（值不匹配）：断言值不正确
   - NullValue（空值）：意外的空引用
   - Performance（性能）：超时或执行缓慢
   - TestSetup（测试设置）：Setup/TearDown 失败
   - ObjectLifetime（对象生命周期）：访问已销毁的对象
   - Threading（线程）：错误的线程执行

4. **生成修复建议：**
   - 从 test-patterns.json 加载通用解决方案
   - 将解决方案与失败模式匹配
   - 提供具体的代码示例

**失败分析示例：**

```markdown
**Test**: Tests.Combat.PlayerTests.TestPlayerTakeDamage
**Location**: Assets/Tests/Combat/PlayerTests.cs:42
**Result**: FAILED

**Failure Message**:
Expected: 90
  But was: 100

**Analysis**:
- Category: ValueMismatch (Assertion Failure)
- Pattern: Expected/actual value mismatch
- Root Cause: Player health not decreasing after TakeDamage() call

**Possible Causes**:
1. TakeDamage() method not implemented correctly
2. Player health not initialized properly
3. Damage value passed incorrectly

**Suggested Solutions**:
1. Verify TakeDamage() implementation:
   ```csharp
   public void TakeDamage(int damage) {
       health -= damage; // Ensure this line exists
   }
   ```

2. Check test setup:
   ```csharp
   [SetUp]
   public void SetUp() {
       player = new Player();
       player.Health = 100; // Ensure proper initialization
   }
   ```

3. Verify test assertion:
   ```csharp
   player.TakeDamage(10);
   Assert.AreEqual(90, player.Health); // Expected: 90
   ```
```

### 7. 生成测试报告

为用户创建全面的测试报告：

**报告结构：**

```markdown
# Unity Test Results

## Summary
- **Total Tests**: 10
- **✓ Passed**: 7 (70%)
- **✗ Failed**: 2 (20%)
- **⊘ Skipped**: 1 (10%)
- **Duration**: 12.35s

## Test Breakdown
- **EditMode Tests**: 5 passed, 1 failed
- **PlayMode Tests**: 2 passed, 1 failed

## Failed Tests

### 1. Tests.Combat.PlayerTests.TestPlayerTakeDamage
**Location**: Assets/Tests/Combat/PlayerTests.cs:42

**Failure**: Expected: 90, But was: 100

**Analysis**: Player health not decreasing after TakeDamage() call.

**Suggested Fix**: Verify TakeDamage() implementation decreases health correctly.

---

### 2. Tests.AI.EnemyTests.TestEnemyChasePlayer
**Location**: Assets/Tests/AI/EnemyTests.cs:67

**Failure**: TimeoutException - Test exceeded time limit (5s)

**Analysis**: Infinite loop or missing yield in coroutine test.

**Suggested Fix**: Add `[UnityTest]` attribute and use `yield return null` in test loop.

---

## Next Steps
1. Review failed test locations and fix implementation
2. Re-run tests after fixes by re-invoking the skill
3. Consider adding more assertions for edge cases
```

**报告交付：**
- 以格式化的 Markdown 呈现报告
- 突出显示关键故障
- 提供文件:行号引用以便快速导航
- 如果用户请求，提供修复特定故障的帮助

## 最佳实践

使用此技能时：

1. **首先运行 EditMode 测试** - 它们速度更快，能捕捉基本的逻辑错误
   - 将 PlayMode 测试保留用于 Unity 特定功能
   - 对纯 C# 逻辑和数据结构使用 EditMode

2. **使用测试类别** - 过滤测试以加快迭代
   - `-testCategory "Combat"` 仅运行 Combat（战斗）测试
   - 在特定功能的积极开发期间很有帮助

3. **监控测试时长** - 设置适当的超时
   - EditMode：通常 1-3 分钟
   - PlayMode：通常 5-15 分钟
   - 根据测试数量调整超时

4. **检查 Unity 版本兼容性** - 确保编辑器与项目版本匹配
   - 版本不匹配可能导致测试失败
   - 测试结果在不同版本之间可能不一致

5. **立即解析结果** - 不要等待人工审查
   - 自动解析能更快发现问题
   - 提供可操作的文件:行号信息

6. **分析失败模式** - 寻找常见原因
   - 类似的失败通常表明存在系统性问题
   - 修复根本原因，而不是个别症状

7. **保留测试结果** - 保留 XML 文件以供调试
   - 结果包含完整的堆栈跟踪
   - 有助于比较测试运行

8. **处理长时间运行的测试** - 使用后台执行
   - 使用 `BashOutput` 工具监控进度
   - 向用户提供状态更新

## 资源

### scripts/find-unity-editor.js

跨平台 Unity 编辑器路径检测脚本。自动扫描 Windows、macOS 和 Linux 的默认安装目录，检测所有已安装的 Unity 版本，并返回最新版本或特定请求的版本。

**用法：**
```bash
# Find latest Unity version
node scripts/find-unity-editor.js --json

# Find specific version
node scripts/find-unity-editor.js --version 2021.3.15f1 --json
```

**输出**：包含 Unity 编辑器路径、版本、平台和所有可用版本的 JSON。

### scripts/parse-test-results.js

Unity 测试框架输出的 NUnit XML 结果解析器。从 XML 结果中提取测试统计信息、失败详情、堆栈跟踪和文件位置。

**用法：**
```bash
# Parse test results with JSON output
node scripts/parse-test-results.js TestResults.xml --json

# Parse with formatted console output
node scripts/parse-test-results.js TestResults.xml
```

**输出**：包含测试摘要、包括文件路径和行号的失败详情以及完整测试列表的 JSON。

### references/test-patterns.json

Unity 测试模式、NUnit 断言、常见失败模式和最佳实践的综合数据库。包括：
- NUnit 断言参考（相等性、集合、异常、Unity 特定）
- 带有正则表达式匹配的常见失败模式
- 失败类别和根本原因分析
- 带有代码示例的解决方案模板
- EditMode 与 PlayMode 指南
- Unity 特定测试模式（协程、场景、预制件、物理）
- 测试最佳实践

**用法**：分析测试失败时加载此文件，以将失败消息与模式匹配并生成修复建议。
