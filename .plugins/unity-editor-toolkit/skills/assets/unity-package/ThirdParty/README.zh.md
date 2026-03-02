# Third-Party Dependencies

本指南介绍了 Unity Editor Toolkit 中使用的外部库列表及其安装方法。

## 📦 当前包含的库

### 1. websocket-sharp
- **用途**：Unity Editor ↔ CLI WebSocket 通信
- **版本**：已包含
- **位置**：`ThirdParty/websocket-sharp/`

## 🆕 为了 PostgreSQL DB 功能额外需要的库

### 2. Npgsql (PostgreSQL .NET Driver)
- **用途**：在 Unity 中连接 PostgreSQL 数据库
- **版本**：6.x (推荐：6.0.11)
- **下载**：[NuGet Gallery - Npgsql](https://www.nuget.org/packages/Npgsql/)
- **安装位置**：`ThirdParty/Npgsql/`

#### Npgsql 安装步骤：

**选项 1：使用 NuGet Package Explorer（推荐）**

1. 下载 NuGet Package Explorer：https://github.com/NuGetPackageExplorer/NuGetPackageExplorer/releases
2. 运行 NuGet Package Explorer
3. 选择 `File > Open Package from Online Feed...`
4. 搜索 "Npgsql"，选择版本 6.0.11
5. 在右侧面板确认 `lib/netstandard2.1/` 文件夹
6. 将以下 DLL 文件复制到 `ThirdParty/Npgsql/` 文件夹：
   - `Npgsql.dll`
   - `System.Buffers.dll`
   - `System.Memory.dll`
   - `System.Runtime.CompilerServices.Unsafe.dll`
   - `System.Threading.Tasks.Extensions.dll`

**选项 2：使用 NuGet CLI**

```bash
# 安装 NuGet
dotnet tool install --global NuGet.CommandLine

# 下载 Npgsql
cd ThirdParty
mkdir Npgsql
cd Npgsql
nuget install Npgsql -Version 6.0.11 -Framework netstandard2.1

# 复制 DLL 文件
cp Npgsql.6.0.11/lib/netstandard2.1/*.dll ./
cp System.Buffers.*/lib/netstandard2.1/*.dll ./
cp System.Memory.*/lib/netstandard2.1/*.dll ./
cp System.Runtime.CompilerServices.Unsafe.*/lib/netstandard2.1/*.dll ./
cp System.Threading.Tasks.Extensions.*/lib/netstandard2.1/*.dll ./

# 删除临时文件夹
rm -rf Npgsql.6.0.11/ System.Buffers.*/ System.Memory.*/ System.Runtime.CompilerServices.Unsafe.*/ System.Threading.Tasks.Extensions.*/
```

**选项 3：手动下载**

1. 访问 https://www.nuget.org/packages/Npgsql/6.0.11
2. 点击右侧的 "Download package"
3. 将 `.nupkg` 文件重命名为 `.zip`
4. 解压缩
5. 将 `lib/netstandard2.1/` 文件夹中的 DLL 文件复制到 `ThirdParty/Npgsql/`

#### 最终文件夹结构：

```
ThirdParty/
├── Npgsql/
│   ├── Npgsql.dll
│   ├── Npgsql.dll.meta (Unity 自动生成)
│   ├── System.Buffers.dll
│   ├── System.Buffers.dll.meta
│   ├── System.Memory.dll
│   ├── System.Memory.dll.meta
│   ├── System.Runtime.CompilerServices.Unsafe.dll
│   ├── System.Runtime.CompilerServices.Unsafe.dll.meta
│   ├── System.Threading.Tasks.Extensions.dll
│   └── System.Threading.Tasks.Extensions.dll.meta
└── websocket-sharp/
    └── (现有文件)
```

---

### 3. UniTask (Unity 异步编程)
- **用途**：数据库异步操作（防止 DB 查询时阻塞 Unity 主线程）
- **版本**：2.x (推荐：2.5.4)
- **下载**：[GitHub - UniTask](https://github.com/Cysharp/UniTask/releases)
- **安装位置**：`ThirdParty/UniTask/`

#### UniTask 安装步骤：

**选项 1：Unity Package Manager (Git URL) - 推荐**

1. Unity Editor 菜单：`Window > Package Manager`
2. 点击左上角 `+` 按钮
3. 选择 `Add package from git URL...`
4. 输入：`https://github.com/Cysharp/UniTask.git?path=src/UniTask/Assets/Plugins/UniTask`
5. 点击 `Add`

**选项 2：下载 .unitypackage**

1. 访问 https://github.com/Cysharp/UniTask/releases
2. 下载最新发布的 `UniTask.{version}.unitypackage`
3. 在 Unity Editor 中选择 `Assets > Import Package > Custom Package...`
4. 选择下载的 `.unitypackage`
5. 导入 (Import)

**选项 3：手动添加 DLL (高级)**

1. 访问 https://github.com/Cysharp/UniTask/releases
2. 下载源代码 (zip)
3. 从 `src/UniTask/Assets/Plugins/UniTask/Runtime/` 文件夹复制 `UniTask.dll`
4. 粘贴到 `ThirdParty/UniTask/` 文件夹

#### 最终文件夹结构 (使用选项 3 时)：

```
ThirdParty/
├── UniTask/
│   ├── UniTask.dll
│   └── UniTask.dll.meta
├── Npgsql/
│   └── (上述 DLL 文件)
└── websocket-sharp/
    └── (现有文件)
```

---

## ✅ 安装确认

1. 重启 Unity Editor
2. 在 `Console` 标签页检查是否有 DLL 加载错误
3. 菜单：`Tools > Unity Editor Toolkit > Server Window`
4. 检查 Database 标签页（Phase 1 完成后显示）

---

## 🔧 故障排除

### DLL 冲突错误

```
Assembly 'Npgsql' has already been loaded from a different location.
```

**解决方案：**
1. 检查 Unity 项目的 `Packages/` 文件夹，移除重复的 Npgsql
2. 删除 `Library/ScriptAssemblies/`
3. 重启 Unity Editor

### .NET Standard 2.1 兼容性错误

```
The type or namespace name 'System.Buffers' could not be found
```

**解决方案：**
1. 确认使用 Unity 2020.3 或更高版本
2. `Edit > Project Settings > Player > Other Settings`
3. `Api Compatibility Level`：选择 `.NET Standard 2.1`
4. 重启 Unity Editor

### UniTask 重复错误

```
Multiple precompiled assemblies with the same name UniTask.dll
```

**解决方案：**
1. 如果使用了选项 1 (UPM)，请移除选项 3 (手动 DLL)
2. 或者反之

---

## 📚 参考文档

- [Npgsql 文档](https://www.npgsql.org/doc/index.html)
- [UniTask 文档](https://github.com/Cysharp/UniTask)
- [Unity .NET Profile 支持](https://docs.unity3d.com/Manual/dotnetProfileSupport.html)

---

**最后更新**：2025-11-14
**阶段 (Phase)**：1 (基础设施建设)
