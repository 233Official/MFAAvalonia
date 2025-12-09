# 现有功能解析：截图功能

本文档通过分析「截图功能」的完整实现，帮助开发者理解 MFAAvalonia 的功能开发流程。

## 功能概述

截图功能允许用户：

1. 连接设备后获取屏幕截图
2. 在界面中预览截图
3. 保存截图到本地文件

## 文件结构

```text
MFAAvalonia/
├── ViewModels/Pages/
│   └── ScreenshotViewModel.cs    # 业务逻辑
└── Views/Pages/
    ├── ScreenshotView.axaml      # 界面定义
    └── ScreenshotView.axaml.cs   # 代码后置
```

---

## 第一步：创建 ViewModel

### 完整代码分析

```csharp
// ViewModels/Pages/ScreenshotViewModel.cs
using Avalonia.Media;
using Avalonia.Platform.Storage;
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;
// ... 其他引用

namespace MFAAvalonia.ViewModels.Pages;

public partial class ScreenshotViewModel : ViewModelBase
{
    // ① 可观察属性 - 使用源生成器自动生成 PropertyChanged
    [ObservableProperty] private Bitmap? _screenshotImage;
    [ObservableProperty] private string _taskName = string.Empty;

    // ② 命令 - 使用 [RelayCommand] 自动生成 ICommand
    [RelayCommand]
    private void Screenshot()
    {
        // 截图逻辑
    }

    [RelayCommand]
    private void SaveScreenshot()
    {
        // 保存逻辑
    }
}
```

### 关键点解析

#### ① 可观察属性

```csharp
[ObservableProperty] private Bitmap? _screenshotImage;
```

- 使用 `[ObservableProperty]` 特性，源生成器自动生成：
  - 公共属性 `ScreenshotImage`
  - `PropertyChanged` 事件通知
- **命名规则**: 私有字段 `_screenshotImage` → 公共属性 `ScreenshotImage`

#### ② RelayCommand

```csharp
[RelayCommand]
private void Screenshot() { }
```

- 自动生成 `ScreenshotCommand` 属性（类型为 `IRelayCommand`）
- 可直接在 XAML 中绑定

#### ③ 与 MaaProcessor 交互

```csharp
// 检查连接状态
if (MaaProcessor.Instance.MaaTasker is not { IsInitialized: true })
{
    // 未连接时，先连接再截图
    MaaProcessor.Instance.TaskQueue.Enqueue(new MFATask { ... });
    MaaProcessor.Instance.Start(true, checkUpdate: false);
}
else
{
    // 已连接，直接截图
    var bitmap = MaaProcessor.Instance.GetBitmapImage();
}
```

#### ④ 线程安全的 UI 更新

```csharp
DispatcherHelper.PostOnMainThread(() =>
{
    var oldImage = ScreenshotImage;
    ScreenshotImage = bitmap;  // 必须在 UI 线程更新
    oldImage?.Dispose();       // 释放旧资源
});
```

---

## 第二步：创建 View

### AXAML 界面定义

```xml
<!-- Views/Pages/ScreenshotView.axaml -->
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:pages="clr-namespace:MFAAvalonia.ViewModels.Pages"
             x:DataType="pages:ScreenshotViewModel"
             x:Class="MFAAvalonia.Views.Pages.ScreenshotView">
    <Grid Margin="15">
        <Grid.RowDefinitions>
            <RowDefinition Height="*" />      <!-- 图片区域 -->
            <RowDefinition Height="Auto" />   <!-- 按钮区域 -->
        </Grid.RowDefinitions>
        
        <!-- 截图预览 -->
        <Image Grid.Row="0" 
               Source="{Binding ScreenshotImage}"
               Stretch="Uniform" />
        
        <!-- 操作按钮 -->
        <StackPanel Grid.Row="1" Orientation="Horizontal">
            <Button Command="{Binding ScreenshotCommand}"
                    Content="截图测试" />
            <Button Command="{Binding SaveScreenshotCommand}"
                    Content="保存截图" />
        </StackPanel>
    </Grid>
</UserControl>
```

### AXAML 关键点解析

#### ① DataType 编译绑定

```xml
x:DataType="pages:ScreenshotViewModel"
```

- 启用编译时绑定检查
- 提供 IntelliSense 支持
- 性能更优

#### ② 属性绑定

```xml
Source="{Binding ScreenshotImage}"
```

- 绑定到 ViewModel 的 `ScreenshotImage` 属性
- 属性变化时自动更新 UI

#### ③ 命令绑定

```xml
Command="{Binding ScreenshotCommand}"
```

- 绑定到自动生成的命令
- `[RelayCommand]` 方法名 `Screenshot` → 命令名 `ScreenshotCommand`

### 代码后置

```csharp
// Views/Pages/ScreenshotView.axaml.cs
public partial class ScreenshotView : UserControl
{
    public ScreenshotView()
    {
        // 设置 DataContext
        DataContext = Instances.ScreenshotViewModel;
        InitializeComponent();
        
        // 可选的额外配置
        RenderOptions.SetBitmapInterpolationMode(image, BitmapInterpolationMode.HighQuality);
    }
}
```

---

## 第三步：注册到 Instances

在 `Helper/Instances.cs` 中，ViewModel 通常通过懒加载方式注册：

```csharp
private static ScreenshotViewModel? _screenshotViewModel;
public static ScreenshotViewModel ScreenshotViewModel => 
    _screenshotViewModel ??= new ScreenshotViewModel();
```

或使用 `[LazyStatic]` 特性（需要 LazyStaticGenerator）。

---

## 数据流总结

```text
用户点击"截图"按钮
       ↓
ScreenshotCommand 执行
       ↓
Screenshot() 方法调用
       ↓
MaaProcessor.GetBitmapImage()
       ↓
DispatcherHelper.PostOnMainThread()
       ↓
ScreenshotImage = bitmap
       ↓
PropertyChanged 事件触发
       ↓
UI 自动更新显示图片
```

---

## 学到的模式

| 模式 | 用法 |
|------|------|
| `[ObservableProperty]` | 自动生成可观察属性 |
| `[RelayCommand]` | 自动生成命令 |
| `DispatcherHelper` | 线程安全的 UI 更新 |
| `MaaProcessor.Instance` | 访问 MAA 核心功能 |
| `ToastHelper` | 显示消息通知 |
| `Instances.Xxx` | 获取全局服务实例 |
