# 实战教程：开发设备状态卡片（完整指南）

本教程将带你**从零开始**完成一个功能的完整开发流程，包括环境准备、编码、调试、测试和验证。

## 教程目标

创建一个「设备状态卡片」组件，显示：

- 设备连接状态（已连接/未连接）
- 设备类型（ADB/Win32）
- 设备名称
- 刷新按钮

---

# 第一阶段：开发环境准备

## 1.1 确保开发环境就绪

打开终端，检查 .NET SDK：

```bash
dotnet --version
# 预期输出: 10.0.xxx 或更高版本
```

## 1.2 克隆并打开项目

```bash
# 如果尚未克隆
git clone https://github.com/SweetSmellFox/MFAAvalonia.git
cd MFAAvalonia

# 还原依赖
dotnet restore
```

## 1.3 使用 IDE 打开

推荐使用以下 IDE：

- **JetBrains Rider**（推荐，对 Avalonia 支持最好）
- **Visual Studio 2022**
- **VS Code** + C# 扩展

打开 `MFAAvalonia.sln` 文件。

## 1.4 首次构建验证

```bash
dotnet build
```

**预期结果**：构建成功，无错误。

> **⚠️ 如果构建失败**：检查 .NET SDK 版本，确保已安装 .NET 10.0。

---

# 第二阶段：创建功能代码

## 2.1 创建 ViewModel 文件

**操作步骤**：

1. 在 IDE 中，导航到 `MFAAvalonia/ViewModels/UsersControls/`
2. 右键 → 新建文件 → `DeviceStatusCardViewModel.cs`
3. 复制以下完整代码：

```csharp
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;
using MFAAvalonia.Extensions.MaaFW;
using MFAAvalonia.Helper;
using System;

namespace MFAAvalonia.ViewModels.UsersControls;

public partial class DeviceStatusCardViewModel : ViewModelBase
{
    // ===== 可观察属性 =====
    
    [ObservableProperty]
    private bool _isConnected;

    [ObservableProperty]
    private string _deviceType = "未知";

    [ObservableProperty]
    private string _deviceName = "无设备";

    [ObservableProperty]
    private string _statusText = "未连接";

    [ObservableProperty]
    private DateTime _lastUpdateTime = DateTime.Now;

    // ===== 构造函数 =====
    
    public DeviceStatusCardViewModel()
    {
        // 初始化时刷新状态
        RefreshStatus();
    }

    // ===== 命令 =====
    
    [RelayCommand]
    private void RefreshStatus()
    {
        try
        {
            // 检查 MaaTasker 是否已初始化
            var tasker = MaaProcessor.Instance.MaaTasker;
            IsConnected = tasker is { IsInitialized: true };

            if (IsConnected)
            {
                // 获取设备类型
                var controllerType = Instances.TaskQueueViewModel.CurrentController;
                DeviceType = controllerType == MaaControllerTypes.Adb ? "ADB 设备" : "Win32 窗口";

                // 获取设备名称
                DeviceName = Instances.TaskQueueViewModel.CurrentDevice?.Name ?? "未知设备";
                StatusText = "已连接";
            }
            else
            {
                DeviceType = "未知";
                DeviceName = "无设备";
                StatusText = "未连接";
            }

            LastUpdateTime = DateTime.Now;
            
            // 调试日志
            LoggerHelper.Info($"[DeviceStatusCard] 刷新状态: Connected={IsConnected}, Type={DeviceType}");
        }
        catch (Exception e)
        {
            LoggerHelper.Error("刷新设备状态失败", e);
            StatusText = "获取失败";
        }
    }

    // ===== 计算属性 =====
    
    public string FormattedUpdateTime => LastUpdateTime.ToString("HH:mm:ss");
}
```

## 2.2 创建 View 文件

**操作步骤**：

1. 导航到 `MFAAvalonia/Views/UserControls/`
2. 创建文件 `DeviceStatusCardView.axaml`：

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
             xmlns:suki="clr-namespace:SukiUI.Controls;assembly=SukiUI"
             xmlns:vm="clr-namespace:MFAAvalonia.ViewModels.UsersControls"
             mc:Ignorable="d" d:DesignWidth="300" d:DesignHeight="200"
             x:DataType="vm:DeviceStatusCardViewModel"
             x:Class="MFAAvalonia.Views.UserControls.DeviceStatusCardView">

    <suki:GlassCard Margin="10" Padding="15">
        <Grid RowDefinitions="Auto,Auto,Auto,Auto,Auto">
            
            <!-- 标题行 -->
            <StackPanel Grid.Row="0" Orientation="Horizontal" Margin="0,0,0,10">
                <TextBlock Text="设备状态" 
                           FontSize="16" 
                           FontWeight="Bold"/>
                
                <!-- 状态指示点 - 简化版，直接用绿/灰色 -->
                <Ellipse Width="10" Height="10" 
                         Margin="10,0,0,0"
                         VerticalAlignment="Center"
                         Fill="{Binding IsConnected, 
                                Converter={StaticResource BooleanToConnectedColorConverter}}"/>
            </StackPanel>
            
            <!-- 状态文本 -->
            <TextBlock Grid.Row="1" 
                       Text="{Binding StatusText}"
                       Opacity="0.7"
                       Margin="0,0,0,5"/>
            
            <!-- 设备类型 -->
            <StackPanel Grid.Row="2" Orientation="Horizontal" Margin="0,5">
                <TextBlock Text="类型：" Opacity="0.7"/>
                <TextBlock Text="{Binding DeviceType}"/>
            </StackPanel>
            
            <!-- 设备名称 -->
            <StackPanel Grid.Row="3" Orientation="Horizontal" Margin="0,5">
                <TextBlock Text="设备：" Opacity="0.7"/>
                <TextBlock Text="{Binding DeviceName}" 
                           TextTrimming="CharacterEllipsis"
                           MaxWidth="200"/>
            </StackPanel>
            
            <!-- 刷新按钮和时间 -->
            <Grid Grid.Row="4" Margin="0,10,0,0" ColumnDefinitions="*,Auto">
                <TextBlock Grid.Column="0" 
                           Opacity="0.5"
                           FontSize="11"
                           VerticalAlignment="Center">
                    <Run Text="更新于 "/>
                    <Run Text="{Binding FormattedUpdateTime}"/>
                </TextBlock>
                
                <Button Grid.Column="1"
                        Command="{Binding RefreshStatusCommand}"
                        Content="刷新"
                        Classes="Flat"/>
            </Grid>
        </Grid>
    </suki:GlassCard>
</UserControl>
```

3. 创建代码后置文件 `DeviceStatusCardView.axaml.cs`：

```csharp
using Avalonia.Controls;

namespace MFAAvalonia.Views.UserControls;

public partial class DeviceStatusCardView : UserControl
{
    public DeviceStatusCardView()
    {
        InitializeComponent();
    }
}
```

## 2.3 注册 ViewModel 到 Instances

**操作步骤**：

1. 打开 `MFAAvalonia/Helper/Instances.cs`
2. 找到其他 ViewModel 的定义位置（搜索 `ViewModel`）
3. 添加以下代码：

```csharp
// 在文件顶部添加 using（如果没有）
using MFAAvalonia.ViewModels.UsersControls;

// 在类中添加（找一个合适的位置，比如其他 ViewModel 附近）
private static DeviceStatusCardViewModel? _deviceStatusCardViewModel;
public static DeviceStatusCardViewModel DeviceStatusCardViewModel => 
    _deviceStatusCardViewModel ??= new DeviceStatusCardViewModel();
```

## 2.4 中间构建验证

**关键步骤**：在继续之前，先验证代码能否编译！

```bash
dotnet build
```

**预期结果**：构建成功。

**常见错误及解决**：

| 错误信息 | 原因 | 解决方法 |
|----------|------|----------|
| `CS0246: 找不到类型名"DeviceStatusCardViewModel"` | 命名空间错误 | 检查 `using` 语句 |
| `CS1061: 不包含"XX"的定义` | 属性名拼写错误 | 检查属性名大小写 |
| `AXAML: 找不到类型` | AXAML 命名空间错误 | 检查 `xmlns:vm` 定义 |

---

# 第三阶段：调试运行

## 3.1 启动调试模式

**方式一：命令行**

```bash
cd MFAAvalonia
dotnet run
```

**方式二：IDE**

- **Rider**: 点击右上角绿色播放按钮，或按 `Shift+F10`
- **VS**: 按 `F5`
- **VS Code**: 使用 `dotnet run` 或配置 launch.json

## 3.2 使用 Avalonia 开发者工具

应用启动后，按 **F12** 打开 Avalonia 开发者工具（仅 Debug 模式）。

**开发者工具功能**：

- **Visual Tree**: 查看控件层级
- **Properties**: 查看控件属性
- **Events**: 监控事件
- **Console**: 查看绑定错误

## 3.3 添加组件到界面进行测试

为了测试我们的组件，需要将它添加到一个现有页面。

**临时测试方法** - 修改 `Views/Pages/SettingsView.axaml`：

1. 在文件顶部添加命名空间：

```xml
xmlns:uc="clr-namespace:MFAAvalonia.Views.UserControls"
xmlns:helper="clr-namespace:MFAAvalonia.Helper"
```

2. 在页面合适位置（比如最顶部）添加组件：

```xml
<uc:DeviceStatusCardView 
    DataContext="{x:Static helper:Instances.DeviceStatusCardViewModel}"
    Margin="10"/>
```

## 3.4 运行并验证

1. 重新运行应用：`dotnet run`
2. 导航到设置页面
3. 检查设备状态卡片是否显示
4. 点击「刷新」按钮，观察状态变化

**验证清单**：

- [ ] 卡片正确显示
- [ ] 状态文本显示「未连接」或「已连接」
- [ ] 刷新按钮可点击
- [ ] 点击刷新后时间更新

---

# 第四阶段：调试技巧

## 4.1 添加调试日志

在 ViewModel 的关键位置添加日志：

```csharp
[RelayCommand]
private void RefreshStatus()
{
    LoggerHelper.Info("[DeviceStatusCard] 开始刷新状态...");
    
    try
    {
        // ... 业务逻辑
        LoggerHelper.Info($"[DeviceStatusCard] 刷新完成: {StatusText}");
    }
    catch (Exception e)
    {
        LoggerHelper.Error("[DeviceStatusCard] 刷新失败", e);
    }
}
```

## 4.2 查看日志

日志文件位置：`logs/mfa-[日期].log`

```bash
# macOS/Linux
tail -f logs/mfa-*.log | grep DeviceStatusCard

# Windows PowerShell
Get-Content logs/mfa-*.log -Wait | Select-String DeviceStatusCard
```

## 4.3 设置断点调试

**Rider/VS**：

1. 在代码行号左侧点击，添加红色断点
2. 按 F5 以调试模式启动
3. 触发断点后查看变量值

**推荐断点位置**：

- `RefreshStatus()` 方法开头
- `IsConnected = ...` 赋值处
- `catch` 块内

## 4.4 使用开发者工具检查绑定

1. 按 F12 打开开发者工具
2. 切换到 Console 标签
3. 查看是否有绑定错误（红色提示）

**常见绑定错误**：

```text
Binding error: Property 'DeviceType' not found on 'null'
```

**解决**：检查 DataContext 是否正确设置。

---

# 第五阶段：测试验证

## 5.1 功能测试清单

按以下顺序测试：

### 测试 1：初始状态

| 步骤 | 操作 | 预期结果 |
|------|------|----------|
| 1 | 启动应用 | 卡片显示「未连接」 |
| 2 | 查看设备类型 | 显示「未知」 |
| 3 | 查看设备名称 | 显示「无设备」 |

### 测试 2：刷新功能

| 步骤 | 操作 | 预期结果 |
|------|------|----------|
| 1 | 点击「刷新」按钮 | 时间更新为当前时间 |
| 2 | 观察日志 | 显示刷新成功信息 |

### 测试 3：连接设备后

| 步骤 | 操作 | 预期结果 |
|------|------|----------|
| 1 | 在主界面连接模拟器 | 连接成功 |
| 2 | 回到设置页 | - |
| 3 | 点击「刷新」 | 状态变为「已连接」 |
| 4 | 查看设备类型 | 显示「ADB 设备」 |
| 5 | 查看设备名称 | 显示实际设备名 |

## 5.2 边界情况测试

| 场景 | 操作 | 预期结果 |
|------|------|----------|
| 设备断开 | 断开模拟器后刷新 | 状态恢复「未连接」 |
| 快速连点 | 连续点击刷新 | 不崩溃，正常响应 |
| 长设备名 | 连接名称很长的设备 | 文本截断显示 |

## 5.3 UI 验证

- [ ] 卡片边框渲染正常
- [ ] 文字清晰可读
- [ ] 按钮 hover 效果正常
- [ ] 暗色/亮色主题下显示正常

---

# 第六阶段：完善与收尾

## 6.1 清理调试代码

如果添加了临时测试代码（如在 SettingsView 中添加的组件），决定是否保留。

## 6.2 代码审查清单

- [ ] 无未使用的 `using` 语句
- [ ] 无硬编码字符串（应使用 LangKeys）
- [ ] 异常都有被 try-catch 捕获
- [ ] 日志级别使用正确（Info/Warning/Error）

## 6.3 提交代码

```bash
# 查看改动
git status
git diff

# 添加文件
git add MFAAvalonia/ViewModels/UsersControls/DeviceStatusCardViewModel.cs
git add MFAAvalonia/Views/UserControls/DeviceStatusCardView.axaml
git add MFAAvalonia/Views/UserControls/DeviceStatusCardView.axaml.cs
git add MFAAvalonia/Helper/Instances.cs

# 提交
git commit -m "feat: 添加设备状态卡片组件"
```

---

# 常见问题排查

## Q1: 组件不显示

**检查清单**：

1. DataContext 是否正确设置？
2. 命名空间是否正确引入？
3. 控件是否在可见区域内？

**调试方法**：

```xml
<!-- 临时添加背景色，确认控件位置 -->
<uc:DeviceStatusCardView Background="Red" ... />
```

## Q2: 绑定不工作

**检查清单**：

1. 属性名是否拼写正确？（区分大小写）
2. 是否使用了 `[ObservableProperty]`？
3. DataContext 链是否完整？

**调试方法**：

```xml
<!-- 显示 DataContext 类型 -->
<TextBlock Text="{Binding}" />
```

## Q3: 命令不触发

**检查清单**：

1. 方法是否有 `[RelayCommand]` 特性？
2. 命令名是否正确？（方法名 + Command）
3. 按钮是否被禁用？

## Q4: 编译错误

```bash
# 清理并重新构建
dotnet clean
dotnet build
```

---

# 完整文件清单

## 新增文件

| 文件路径 | 说明 |
|----------|------|
| `ViewModels/UsersControls/DeviceStatusCardViewModel.cs` | ViewModel |
| `Views/UserControls/DeviceStatusCardView.axaml` | AXAML 视图 |
| `Views/UserControls/DeviceStatusCardView.axaml.cs` | 代码后置 |

## 修改文件

| 文件路径 | 修改内容 |
|----------|----------|
| `Helper/Instances.cs` | 添加 ViewModel 注册 |
| 使用组件的页面 | 添加组件引用 |

---

# 总结

通过本教程，你完成了：

- ✅ **环境准备**：验证开发环境，首次构建
- ✅ **编码实现**：创建 ViewModel、View、注册服务
- ✅ **中间验证**：每步完成后进行构建检查
- ✅ **调试运行**：使用开发者工具和日志调试
- ✅ **功能测试**：系统性验证各种场景
- ✅ **代码收尾**：清理、审查、提交

这就是 MFAAvalonia 中一个功能从零到完成的完整流程！
