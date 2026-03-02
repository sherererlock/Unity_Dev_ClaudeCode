# 变更日志

Unity Editor Toolkit 的所有重要变更都将记录在此文件中。

格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)，
并且本项目遵守 [Semantic Versioning](https://semver.org/lang/zh-CN/)。

## [0.15.1] - 2025-12-02

### 新增
- **版本检查功能**：从 Unity 编辑器窗口检查更新
  - “检查更新”按钮，用于从 GitHub 获取最新版本
  - “打开 GitHub 发布页”按钮，用于快速访问发布页面
  - 本地版本与最新版本对比及更新状态显示
  - 通过异步 HTTP 请求 GitHub raw URL 进行实时版本检查

### 变更
- **文档**：更新了 Unity 包安装指南
  - 添加了依赖包安装步骤 (UniTask, unity-sqlite-net)
  - 添加了包更新说明
  - 移除了根目录下的 CHANGELOG 文件（发布说明现在在 GitHub Releases 中）

## [0.13.0] - 2025-12-02

### 新增
- **Shader 命令** (7 个新命令)
  - `shader list` - 列出项目中的所有 Shader
  - `shader find` - 按名称查找 Shader
  - `shader properties` - 获取 Shader 属性
  - `shader keywords` - 列出全局/Shader 关键字
  - `shader keyword-enable` - 启用全局 Shader 关键字
  - `shader keyword-disable` - 禁用全局 Shader 关键字
  - `shader keyword-status` - 检查关键字启用状态
- **文档**
  - `COMMANDS_MATERIAL.md` - 材质命令参考 (9 个命令)
  - `COMMANDS_SHADER.md` - Shader 命令参考 (7 个命令)

### 修复
- **控制台 Bug**：警告日志现在默认显示（`includeWarnings` 默认值更改为 `true`）
- **数据库历史记录删除**：清除时命令历史记录现在能从 SQLite 数据库中正确删除
- **项目数据库隔离**：每个 Unity 项目现在在 `Library/UnityEditorToolkit/` 文件夹中使用自己的数据库文件，而不是共享的持久化路径

### 变更
- 总命令数从 58 个增加到 86 个
- 更新文档以反映第二阶段状态

## [0.12.1] - 2025-12-01

### 修复
- Logger 类重命名为 ToolkitLogger 以避免编译冲突

## [0.12.0] - 2025-11-30

### 新增
- 材质命令 (9 个命令)
- 动画命令
- 编辑器命令扩展

## [0.11.0] - 2025-11-28

### 新增
- 预制件命令 (12 个命令)
- 资源管理命令 (9 个命令)

## [0.10.0] - 2025-11-25

### 新增
- 组件命令 (10 个命令)
- 菜单执行命令 (2 个命令)

## [0.9.0] - 2025-11-22

### 新增
- SQLite 数据库集成
- 基于 GUID 的 GameObject 持久化
- 多场景同步
- 带有撤销/重做的命令模式

## [0.8.0] - 2025-11-19

### 新增
- 初始实验性发布
- 核心命令：GameObject, Transform, Scene, Console, Wait, Chain
- 带有 JSON-RPC 2.0 的 WebSocket 服务器
- EditorPrefs 管理
