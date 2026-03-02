# websocket-sharp DLL 安装指南

Unity Editor Toolkit 需要 websocket-sharp 来进行 WebSocket 通信。

## 安装步骤

### 第 1 步：下载 websocket-sharp

**选项 A：下载预编译 DLL（推荐）**

1. 访问：https://github.com/sta/websocket-sharp/releases
2. 下载最新版本（例如 `websocket-sharp.zip` 或 `websocket-sharp.dll`）
3. 如果需要，请解压缩

**直接下载链接：**
- 最新稳定版：https://github.com/sta/websocket-sharp/releases/download/1.0.3-rc11/websocket-sharp.dll

**选项 B：从源码构建**

```bash
git clone https://github.com/sta/websocket-sharp.git
cd websocket-sharp/websocket-sharp
# 使用您的 C# 编译器或 Visual Studio 进行构建
```

### 第 2 步：添加 DLL 到 Unity 项目

**将 DLL 复制到此确切位置：**

```
Packages/com.devgom.unity-editor-toolkit/ThirdParty/websocket-sharp/websocket-sharp.dll
```

**完整路径示例：**

**Windows:**
```
D:\YourUnityProject\Packages\com.devgom.unity-editor-toolkit\ThirdParty\websocket-sharp\websocket-sharp.dll
```

**macOS/Linux:**
```
/Users/YourName/UnityProjects/YourProject/Packages/com.devgom.unity-editor-toolkit/ThirdParty/websocket-sharp/websocket-sharp.dll
```

### 第 3 步：在 Unity 中配置导入设置

1. Unity 会自动检测到该 DLL
2. 在 Project（项目）窗口中选中 `websocket-sharp.dll`
3. 在 Inspector（检视）窗口中验证设置：
   - **Any Platform（任意平台）**: ✓ 勾选
   - **Editor（编辑器）**: ✓ 包含
   - **Standalone（独立程序）**: ✗ 排除（仅限编辑器使用）

4. 点击 **Apply（应用）**

### 第 4 步：验证安装

**方法 1：检查控制台**

添加 DLL 后，Unity 不应显示任何导入错误。检查 Console（控制台，快捷键 Ctrl+Shift+C）确认：
- ✓ 没有 "missing assembly"（缺少程序集）警告
- ✓ 没有 websocket-sharp 相关错误

**方法 2：测试服务器**

1. 创建 GameObject → 添加 `UnityEditorServer` 组件
2. 进入 Play Mode（播放模式）
3. 控制台应显示：`✓ Unity Editor Server started on ws://127.0.0.1:9500`

## 故障排除

### 错误："Assembly 'websocket-sharp' not found"

**解决方案：**
- 验证 DLL 是否在正确的位置：`ThirdParty/websocket-sharp/websocket-sharp.dll`
- 重启 Unity 编辑器
- 重新导入包：右键点击包 → Reimport（重新导入）

### 错误："Could not load file or assembly"

**解决方案：**
- 检查 DLL 平台设置（应勾选 Any Platform）
- 尝试不同的 websocket-sharp 版本
- 确保您使用的是 .NET Framework 4.x（而不是 .NET Standard）

### DLL 未出现在 Project 窗口中

**解决方案：**
1. 关闭 Unity
2. 删除项目中的 `Library/` 文件夹
3. 重新打开 Unity（这将重新导入所有资产）

### Unity 2020.3+ 兼容性

websocket-sharp 1.0.3-rc11 兼容：
- Unity 2020.3 LTS
- Unity 2021.3 LTS
- Unity 2022.3 LTS
- Unity 6 (2023.2+)

## 替代方案：NuGet for Unity

如果您已安装 NuGet for Unity：

1. 安装 NuGet for Unity：https://github.com/GlitchEnzo/NuGetForUnity
2. 打开 NuGet 窗口：`NuGet → Manage NuGet Packages`
3. 搜索："websocket-sharp"
4. 点击 **Install（安装）**

**优点：**
- 自动依赖管理
- 易于更新
- 无需手动复制 DLL

**缺点：**
- 需要额外的包（NuGet for Unity）
- 项目体积略微增加

## 安装后的文件结构

```
Packages/com.devgom.unity-editor-toolkit/
├── ThirdParty/
│   └── websocket-sharp/
│       ├── websocket-sharp.dll          ← 您添加的文件
│       ├── websocket-sharp.dll.meta     ← Unity 创建的文件
│       └── README.md                    ← 本文件
├── Runtime/
├── Editor/
└── ...
```

## 验证清单

- [ ] 已从官方来源下载 DLL
- [ ] DLL 已放置在 `ThirdParty/websocket-sharp/` 文件夹中
- [ ] Unity 已检测并导入 DLL（控制台无错误）
- [ ] 导入设置已配置（任意平台，仅限编辑器）
- [ ] 测试服务器在 Play Mode 下成功启动
- [ ] 控制台中无程序集解析错误

## 许可说明

websocket-sharp 采用 MIT 许可证授权。

**websocket-sharp 许可证：**
```
MIT License

Copyright (c) 2010-2021 sta.blockhead

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
```

## 支持

如遇问题：
- **websocket-sharp**: https://github.com/sta/websocket-sharp/issues
- **Unity Editor Toolkit**: https://github.com/Dev-GOM/claude-code-marketplace/issues
