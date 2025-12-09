# 开发者指南

本文档为 MFAAvalonia 项目的开发者提供开发环境搭建、构建运行、编码规范等指导。

## 环境要求

| 组件 | 版本要求 |
|------|----------|
| .NET SDK | 10.0 或更高 |
| IDE | Visual Studio 2022 / Rider / VS Code |
| 操作系统 | Windows 10+, Linux (X11/Wayland), macOS 10.15+ |

## 快速开始

### 1. 克隆仓库

```bash
git clone https://github.com/SweetSmellFox/MFAAvalonia.git
cd MFAAvalonia
```

### 2. 还原依赖

```bash
dotnet restore
```

### 3. 构建项目

```bash
# Debug 构建
dotnet build

# Release 构建
dotnet build -c Release
```

### 4. 运行应用

```bash
cd MFAAvalonia
dotnet run
```

---

## 发布构建

### 单平台发布

```bash
# Windows x64
dotnet publish -c Release -r win-x64

# Linux x64
dotnet publish -c Release -r linux-x64

# macOS x64
dotnet publish -c Release -r osx-x64

# macOS ARM64 (Apple Silicon)
dotnet publish -c Release -r osx-arm64
```

发布输出位于：`bin/x64/Release/{rid}/publish/`

### 发布说明

- 发布为**单文件**可执行程序（`PublishSingleFile=true`）
- **不**包含运行时（`SelfContained=false`），需用户安装 .NET Runtime
- 库文件自动移动到 `libs/` 子目录

---

## 项目结构

详见 [项目结构文档](./project-structure.md)。

关键开发目录：

```
MFAAvalonia/
├── ViewModels/      # 添加新 ViewModel
├── Views/           # 添加新视图
├── Extensions/MaaFW/# 扩展 MaaFramework 功能
├── Helper/          # 添加辅助工具类
└── Configuration/   # 配置相关代码
```

---

## 开发任务

### 添加新页面

1. **创建 ViewModel**

```csharp
// ViewModels/Pages/MyPageViewModel.cs
public partial class MyPageViewModel : ViewModelBase
{
    [ObservableProperty]
    private string _title = "My Page";
}
```

2. **创建 View**

```xml
<!-- Views/Pages/MyPageView.axaml -->
<UserControl xmlns="https://github.com/avaloniaui"
             x:Class="MFAAvalonia.Views.Pages.MyPageView"
             x:DataType="vm:MyPageViewModel">
    <TextBlock Text="{Binding Title}" />
</UserControl>
```

3. **注册页面**（在 `App.axaml.cs` 的 `ConfigureViews` 方法中）

### 添加配置项

1. 在 `ConfigurationKeys.cs` 添加键名：

```csharp
public const string MyNewSetting = "MyNewSetting";
```

2. 在代码中使用：

```csharp
var value = ConfigurationManager.Current.GetValue(
    ConfigurationKeys.MyNewSetting, 
    defaultValue
);
```

### 添加多语言文本

1. 编辑 `Assets/Localization/Strings.resx` 添加键值
2. 使用 T4 模板重新生成 `LangKeys.cs`
3. 在代码中使用：

```csharp
var text = LanguageHelper.GetLocalizedString(LangKeys.MyKey);
```

---

## 编码规范

### 命名约定

| 类型 | 规范 | 示例 |
|------|------|------|
| 类 | PascalCase | `TaskQueueViewModel` |
| 方法 | PascalCase | `StartTask()` |
| 属性 | PascalCase | `IsRunning` |
| 私有字段 | _camelCase | `_isRunning` |
| 常量 | PascalCase | `MaxRetryCount` |
| 局部变量 | camelCase | `taskList` |

### MVVM 规范

- **View 不应包含业务逻辑**
- 使用 `[ObservableProperty]` 生成属性
- 使用 `[RelayCommand]` 生成命令
- 避免在 ViewModel 中直接引用 View

### 异步编程

```csharp
// ✅ 推荐
public async Task DoWorkAsync()
{
    await Task.Run(() => { /* CPU 密集型 */ });
}

// ❌ 避免
public void DoWork()
{
    Task.Run(() => { /* ... */ }).Wait(); // 阻塞线程
}
```

---

## 调试技巧

### 启用诊断工具

Debug 模式下自动包含 `Avalonia.Diagnostics`：

- 按 `F12` 打开开发者工具
- 查看可视化树和属性

### 日志查看

日志文件位于：`logs/mfa-[date].log`

```csharp
// 记录日志
LoggerHelper.Info("信息");
LoggerHelper.Warning("警告");
LoggerHelper.Error("错误", exception);
```

### 常见问题

**Q: 界面不更新？**  
A: 确保属性使用 `[ObservableProperty]` 或继承 `ObservableObject`

**Q: 绑定错误？**  
A: 检查 `x:DataType` 是否正确设置

**Q: 原生库加载失败？**  
A: 检查 `runtimes/` 目录结构和 `PrivatePathHelper.cs`

---

## 贡献流程

1. **Fork 仓库**
2. **创建功能分支**: `git checkout -b feature/my-feature`
3. **提交更改**: `git commit -m 'Add some feature'`
4. **推送分支**: `git push origin feature/my-feature`
5. **创建 Pull Request**

### 提交规范

```
<type>: <subject>

<body>
```

类型：

- `feat`: 新功能
- `fix`: 修复
- `docs`: 文档
- `style`: 格式
- `refactor`: 重构
- `test`: 测试
- `chore`: 构建/工具

---

## 相关资源

- [Avalonia 文档](https://docs.avaloniaui.net/)
- [CommunityToolkit.Mvvm](https://learn.microsoft.com/zh-cn/dotnet/communitytoolkit/mvvm/)
- [MaaFramework 文档](https://github.com/MaaXYZ/MaaFramework/tree/main/docs)
- [SukiUI 示例](https://github.com/kikipoulet/SukiUI)
