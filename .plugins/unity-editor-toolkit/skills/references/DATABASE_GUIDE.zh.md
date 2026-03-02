# Unity Editor Toolkit - 数据库指南

Unity Editor Toolkit 的 SQLite 数据库集成和实时游戏对象同步的完整指南。

## 概览

Unity Editor Toolkit 提供了完整的 SQLite 数据库集成，用于实现实时游戏对象同步、分析和版本控制功能。

### 主要特性

- **基于 GUID 的持久化**：通过在 Unity 会话间持久存在的 GUID 来标识游戏对象
- **实时同步**：每 1 秒自动同步所有已加载的场景
- **多场景支持**：同时同步所有已加载的场景
- **命令模式**：支持数据库操作的撤销/重做
- **自动迁移**：自动架构迁移系统
- **批量操作**：高效的批量操作（每批 500 个对象）
- **安全性**：通过参数化查询和事务安全防止 SQL 注入

## 数据库设置

### 1. Unity 编辑器设置

1. 打开 **Unity Editor Toolkit Server** 窗口
   ```
   Tools > Unity Editor Toolkit > Server Window
   ```

2. 切换到 **"Database"**（数据库）标签页

3. 点击 **"Connect"**（连接）以初始化 SQLite 数据库
   - 数据库文件：`{ProjectRoot}/.unity-websocket/unity-editor.db`
   - 自动创建带有迁移的架构

4. 启用实时同步（可选）
   - 点击 **"Start Sync"**（开始同步）以进行自动同步（1秒间隔）
   - 自动同步所有已加载的场景
   - 游戏对象会自动标记上 GUID 组件

### 2. 数据库文件位置

```
{ProjectRoot}/
└── .unity-websocket/
    ├── unity-editor.db          # SQLite 数据库
    ├── server-status.json       # WebSocket 状态
    └── uw.js                    # CLI 包装器
```

## CLI 命令

所有数据库命令均从 Unity 项目根目录运行：

```bash
cd <unity-project-root> && node .unity-websocket/uw <command> [options]
```

### 数据库管理 (`db`)

#### 连接到数据库

```bash
# 连接并初始化数据库
cd <unity-project-root> && node .unity-websocket/uw db connect

# 连接并禁用 WAL 模式（默认：启用）
cd <unity-project-root> && node .unity-websocket/uw db connect --no-wal
```

**WAL 模式** (Write-Ahead Logging / 预写式日志)：
- 默认：启用以获得更好的性能
- 多读者，单作者
- 使用 `--no-wal` 禁用以确保兼容性

#### 检查数据库状态

```bash
# 获取连接和健康状态
cd <unity-project-root> && node .unity-websocket/uw db status

# 以 JSON 格式输出
cd <unity-project-root> && node .unity-websocket/uw db status --json
```

输出包括：
- 连接状态
- 数据库文件路径和存在性
- 初始化状态
- 自动同步状态

#### 断开数据库连接

```bash
# 安全断开连接并清理
cd <unity-project-root> && node .unity-websocket/uw db disconnect
```

#### 重置数据库

```bash
# 删除并重新创建数据库及全新迁移
cd <unity-project-root> && node .unity-websocket/uw db reset

# 确认重置
cd <unity-project-root> && node .unity-websocket/uw db reset --yes
```

**警告**：会删除所有数据！

#### 运行迁移

```bash
# 应用挂起的迁移
cd <unity-project-root> && node .unity-websocket/uw db migrate

# 强制重新运行所有迁移
cd <unity-project-root> && node .unity-websocket/uw db clear-migrations
cd <unity-project-root> && node .unity-websocket/uw db migrate
```

### 场景同步 (`sync`)

#### 同步整个场景

```bash
# 同步当前活动场景到数据库
cd <unity-project-root> && node .unity-websocket/uw sync scene

# 保留现有数据（不清除）
cd <unity-project-root> && node .unity-websocket/uw sync scene --no-clear

# 跳过组件（仅同步游戏对象）
cd <unity-project-root> && node .unity-websocket/uw sync scene --no-components

# 跳过层级闭包表
cd <unity-project-root> && node .unity-websocket/uw sync scene --no-closure

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw sync scene --json
```

**闭包表 (Closure Table)**：存储父子关系以进行高效的层级查询。

#### 同步特定游戏对象

```bash
# 同步单个游戏对象
cd <unity-project-root> && node .unity-websocket/uw sync object "Player"

# 使用完整层级路径同步
cd <unity-project-root> && node .unity-websocket/uw sync object "Environment/Trees/Oak"

# 包含子对象
cd <unity-project-root> && node .unity-websocket/uw sync object "Player" --children

# 跳过组件
cd <unity-project-root> && node .unity-websocket/uw sync object "Player" --no-components
```

#### 检查同步状态

```bash
# 获取当前场景的同步状态
cd <unity-project-root> && node .unity-websocket/uw sync status

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw sync status --json
```

输出包括：
- Unity 对象计数
- 数据库对象计数
- 组件计数
- 闭包记录计数
- 同步状态

#### 清除同步数据

```bash
# 清除当前场景的所有同步数据
cd <unity-project-root> && node .unity-websocket/uw sync clear

# 确认清除
cd <unity-project-root> && node .unity-websocket/uw sync clear --yes
```

#### 自动同步控制

```bash
# 开始自动同步（1秒间隔）
cd <unity-project-root> && node .unity-websocket/uw sync start

# 停止自动同步
cd <unity-project-root> && node .unity-websocket/uw sync stop

# 获取自动同步状态
cd <unity-project-root> && node .unity-websocket/uw sync auto-status
```

自动同步状态包括：
- 运行状态
- 上次同步时间
- 成功/失败计数
- 同步间隔（毫秒）
- 批处理大小

### 分析 (`analytics`)

#### 场景分析

```bash
# 获取场景统计信息
cd <unity-project-root> && node .unity-websocket/uw analytics scene

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw analytics scene --json
```

输出包括：
- 游戏对象总数
- 组件总数
- 活动与非活动对象
- 对象层级深度
- 组件类型分布

#### 游戏对象分析

```bash
# 获取游戏对象统计信息
cd <unity-project-root> && node .unity-websocket/uw analytics objects

# 按活动状态过滤
cd <unity-project-root> && node .unity-websocket/uw analytics objects --active-only
```

#### 组件分析

```bash
# 获取组件类型分布
cd <unity-project-root> && node .unity-websocket/uw analytics components

# 前 N 个最常用的组件
cd <unity-project-root> && node .unity-websocket/uw analytics components --top 10
```

#### 标签分析

```bash
# 获取标签使用统计
cd <unity-project-root> && node .unity-websocket/uw analytics tags
```

#### 图层分析

```bash
# 获取图层使用统计
cd <unity-project-root> && node .unity-websocket/uw analytics layers
```

### 快照 (`snapshot`)

快照捕获特定时间点的游戏对象和组件的完整状态。

#### 创建快照

```bash
# 创建当前场景的快照
cd <unity-project-root> && node .unity-websocket/uw snapshot create "Before Refactor"

# 创建带有描述的快照
cd <unity-project-root> && node .unity-websocket/uw snapshot create "v1.0 Release" \
  --description "Scene state before major refactor"
```

#### 列出快照

```bash
# 列出所有快照
cd <unity-project-root> && node .unity-websocket/uw snapshot list

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw snapshot list --json
```

#### 获取快照详情

```bash
# 按 ID 获取快照
cd <unity-project-root> && node .unity-websocket/uw snapshot get 1

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw snapshot get 1 --json
```

#### 比较快照

```bash
# 比较两个快照
cd <unity-project-root> && node .unity-websocket/uw snapshot compare 1 2

# 仅显示差异
cd <unity-project-root> && node .unity-websocket/uw snapshot compare 1 2 --diff-only
```

输出包括：
- 添加的对象
- 移除的对象
- 修改的对象
- 组件变更

#### 恢复快照

```bash
# 从快照恢复场景
cd <unity-project-root> && node .unity-websocket/uw snapshot restore 1

# 确认恢复
cd <unity-project-root> && node .unity-websocket/uw snapshot restore 1 --yes
```

**警告**：会覆盖当前场景状态！

#### 删除快照

```bash
# 按 ID 删除快照
cd <unity-project-root> && node .unity-websocket/uw snapshot delete 1

# 确认删除
cd <unity-project-root> && node .unity-websocket/uw snapshot delete 1 --yes
```

### 变换历史 (`transform-history`)

追踪位置、旋转和缩放随时间的变化，支持撤销/重做。

#### 开始记录

```bash
# 开始记录变换更改
cd <unity-project-root> && node .unity-websocket/uw transform-history start

# 记录特定游戏对象
cd <unity-project-root> && node .unity-websocket/uw transform-history start "Player"
```

#### 停止记录

```bash
# 停止记录
cd <unity-project-root> && node .unity-websocket/uw transform-history stop
```

#### 获取历史

```bash
# 获取游戏对象的变换历史
cd <unity-project-root> && node .unity-websocket/uw transform-history get "Player"

# 限制结果数量
cd <unity-project-root> && node .unity-websocket/uw transform-history get "Player" --limit 10

# JSON 输出
cd <unity-project-root> && node .unity-websocket/uw transform-history get "Player" --json
```

#### 比较变换

```bash
# 比较两个时间戳之间的变换
cd <unity-project-root> && node .unity-websocket/uw transform-history compare "Player" \
  --from "2025-11-19T10:00:00" \
  --to "2025-11-19T11:00:00"
```

#### 撤销/重做

```bash
# 撤销上一次变换更改
cd <unity-project-root> && node .unity-websocket/uw transform-history undo "Player"

# 重做上一次撤销的更改
cd <unity-project-root> && node .unity-websocket/uw transform-history redo "Player"

# 撤销 N 步
cd <unity-project-root> && node .unity-websocket/uw transform-history undo "Player" --steps 3
```

#### 清除历史

```bash
# 清除特定游戏对象的历史
cd <unity-project-root> && node .unity-websocket/uw transform-history clear "Player"

# 清除所有历史
cd <unity-project-root> && node .unity-websocket/uw transform-history clear-all --yes
```

## 数据库架构

### 核心表

#### `scenes`
存储场景信息。
```sql
CREATE TABLE scenes (
    scene_id INTEGER PRIMARY KEY AUTOINCREMENT,
    scene_path TEXT NOT NULL UNIQUE,
    scene_name TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

#### `gameobjects`
存储带有基于 GUID 标识的游戏对象数据。
```sql
CREATE TABLE gameobjects (
    object_id INTEGER PRIMARY KEY AUTOINCREMENT,
    guid TEXT UNIQUE,                        -- 持久化 GUID
    instance_id INTEGER NOT NULL,
    scene_id INTEGER NOT NULL,
    object_name TEXT NOT NULL,
    parent_id INTEGER,
    tag TEXT,
    layer INTEGER,
    is_active BOOLEAN DEFAULT 1,
    is_static BOOLEAN DEFAULT 0,
    is_deleted BOOLEAN DEFAULT 0,            -- 软删除
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (scene_id) REFERENCES scenes(scene_id)
);
```

#### `gameobject_closure`
存储层级关系以进行高效查询。
```sql
CREATE TABLE gameobject_closure (
    ancestor_id INTEGER NOT NULL,
    descendant_id INTEGER NOT NULL,
    depth INTEGER NOT NULL,
    PRIMARY KEY (ancestor_id, descendant_id),
    FOREIGN KEY (ancestor_id) REFERENCES gameobjects(object_id),
    FOREIGN KEY (descendant_id) REFERENCES gameobjects(object_id)
);
```

#### `migrations`
追踪已应用的架构迁移。
```sql
CREATE TABLE migrations (
    migration_id INTEGER PRIMARY KEY AUTOINCREMENT,
    migration_name TEXT NOT NULL UNIQUE,
    applied_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 迁移历史

- **Migration_001**：初始架构（scenes, gameobjects, migrations）
- **Migration_002**：向 gameobjects 表添加 GUID 列

## 安全性

### SQL 注入预防

所有数据库操作都使用 **参数化查询**：

```csharp
// ✓ 安全 (参数化)
connection.Execute("DELETE FROM gameobjects WHERE guid = ?", guid);

// ✗ 不安全 (字符串拼接)
connection.Execute($"DELETE FROM gameobjects WHERE guid = '{guid}'");
```

**批量操作**：
- ≤100 项：单独的参数化 DELETE 语句
- >100 项：带有事务的临时表模式

### 事务安全

**防止嵌套事务**：
```csharp
ExecuteInTransaction(connection, () => {
    // 安全的事务执行
    // 检测并防止嵌套事务
    // 如果事务已启动，则优雅回退
});
```

### 内存安全

**域重载 (Domain Reload) 安全**：
```csharp
if (!isDisposed)
{
    await UniTask.SwitchToMainThread();
}
```

防止 Unity 域重载期间崩溃。

## 最佳实践

### 1. 连接管理

```bash
# 操作前始终检查状态
cd <unity-project-root> && node .unity-websocket/uw db status

# 会话开始时连接一次
cd <unity-project-root> && node .unity-websocket/uw db connect

# 会话结束时断开连接
cd <unity-project-root> && node .unity-websocket/uw db disconnect
```

### 2. 自动同步使用

**何时使用**：
- 频繁更改游戏对象的积极开发阶段
- 实时分析和监控
- 多场景编辑

**何时禁用**：
- 性能关键的操作
- 大型场景修改（使用手动同步）
- 预制件编辑

### 3. 快照工作流

```bash
# 重大更改前
cd <unity-project-root> && node .unity-websocket/uw snapshot create "Before Refactor"

# 进行更改...

# 如果出现问题
cd <unity-project-root> && node .unity-websocket/uw snapshot restore 1 --yes

# 更改成功后
cd <unity-project-root> && node .unity-websocket/uw snapshot create "After Refactor"
```

### 4. 性能优化

**批量操作**：
- 使用 `sync scene` 进行全场景同步（比单个对象更高效）
- 如果不需要，使用 `--no-components` 禁用组件
- 对于扁平层级，使用 `--no-closure` 跳过闭包表

**自动同步间隔**：
- 默认：1000ms（1秒）
- 如果需要，在 Unity 编辑器窗口中调整
- 在密集操作期间禁用

### 5. 分析工作流

```bash
# 获取场景概览
cd <unity-project-root> && node .unity-websocket/uw analytics scene

# 识别性能问题
cd <unity-project-root> && node .unity-websocket/uw analytics components --top 10

# 检查标签/图层使用情况
cd <unity-project-root> && node .unity-websocket/uw analytics tags
cd <unity-project-root> && node .unity-websocket/uw analytics layers
```

## 故障排除

### 数据库连接失败

```bash
# 检查 Unity 编辑器是否正在运行
cd <unity-project-root> && node .unity-websocket/uw status

# 检查数据库状态
cd <unity-project-root> && node .unity-websocket/uw db status

# 重新连接
cd <unity-project-root> && node .unity-websocket/uw db connect
```

### 同步问题

```bash
# 检查同步状态
cd <unity-project-root> && node .unity-websocket/uw sync status

# 清除并重新同步
cd <unity-project-root> && node .unity-websocket/uw sync clear --yes
cd <unity-project-root> && node .unity-websocket/uw sync scene
```

### 迁移错误

```bash
# 重置并重新应用迁移
cd <unity-project-root> && node .unity-websocket/uw db reset --yes
cd <unity-project-root> && node .unity-websocket/uw db migrate
```

### 性能问题

```bash
# 停止自动同步
cd <unity-project-root> && node .unity-websocket/uw sync stop

# 使用带有优化的手动同步
cd <unity-project-root> && node .unity-websocket/uw sync scene --no-closure
```

## 示例

### 完整工作流示例

```bash
# 1. 初始化数据库
cd /path/to/unity/project
node .unity-websocket/uw db connect

# 2. 同步当前场景
node .unity-websocket/uw sync scene

# 3. 创建快照
node .unity-websocket/uw snapshot create "Initial State"

# 4. 获取分析
node .unity-websocket/uw analytics scene --json

# 5. 开始自动同步
node .unity-websocket/uw sync start

# 6. 在 Unity 编辑器中进行更改...

# 7. 检查同步状态
node .unity-websocket/uw sync status

# 8. 创建另一个快照
node .unity-websocket/uw snapshot create "After Changes"

# 9. 比较快照
node .unity-websocket/uw snapshot compare 1 2

# 10. 清理
node .unity-websocket/uw sync stop
node .unity-websocket/uw db disconnect
```

### CI/CD 集成

```bash
#!/bin/bash
# 构建验证脚本

cd /path/to/unity/project

# 初始化
node .unity-websocket/uw db connect

# 同步并分析
node .unity-websocket/uw sync scene
node .unity-websocket/uw analytics scene --json > scene-stats.json

# 验证对象数量
OBJECT_COUNT=$(cat scene-stats.json | jq '.totalObjects')
if [ "$OBJECT_COUNT" -gt 10000 ]; then
    echo "❌ Scene too complex: $OBJECT_COUNT objects"
    exit 1
fi

# 创建用于回滚的快照
node .unity-websocket/uw snapshot create "CI Build $BUILD_NUMBER"

# 清理
node .unity-websocket/uw db disconnect
```

## 相关文档

- [README.md](../../README.md) - 主要插件文档
- [COMMANDS.md](./COMMANDS.md) - 完整 CLI 命令参考
- [SKILL.md](../SKILL.md) - 技能文档
- [Unity C# Package](../assets/unity-package/Editor/Database/) - 数据库实现

---

**版本**: 0.7.0
**最后更新**: 2025-11-19
**数据库版本**: Migration_002 (GUID 支持)
