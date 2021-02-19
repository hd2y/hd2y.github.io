---
title: "WPF 系列教程：控件之布局"
date: "2020/04/24 16:02:31"
updated: "2020/04/24 16:10:49"
permalink: "wpf-controls-and-layout"
categories:
 - [开发, C#, WPF]
---

布局（Layout）控件用于管理子元素的大小、尺寸、位置和排列方式。

## 边框（Border）

`Border` 可以在另一个元素周围绘制边框与背景。

需要注意的是，`Border` 内最多只能有一个子级。

若要显示多个子元素，需要在父 `Border` 中放置其他 [Panel](https://docs.microsoft.com/en-us/dotnet/api/system.windows.controls.panel?view=netframework-4.8) 元素，可以将子元素放在该 `Panel` 元素中。

```xml
<Border Background="LightBlue" BorderBrush="Black" BorderThickness="2" CornerRadius="45" Padding="25" />
```

## 子弹装饰器（BulletDecorator）

`BulletDecorator` 是一个布局控件，该控件可以将项目符号与另一个可视对象对齐。

```xml
<StackPanel>
    <BulletDecorator Margin="5" VerticalAlignment="Center" Background="Yellow">
        <BulletDecorator.Bullet>
            <Image Width="50" Source="pack://application:,,,/WpfExample;component/images/bullet.png"/>
        </BulletDecorator.Bullet>
        <TextBlock TextWrapping="Wrap" VerticalAlignment="Center" HorizontalAlignment="Left" Foreground ="Purple">A Simple BulletDecorator</TextBlock>
    </BulletDecorator>
    <BulletDecorator Margin="5" VerticalAlignment="Center" Background="Yellow">
        <BulletDecorator.Bullet>
            <Image Width="20" Source="pack://application:,,,/WpfExample;component/images/apple.jpg"/>
        </BulletDecorator.Bullet>
        <TextBlock TextWrapping="Wrap" VerticalAlignment="Center" HorizontalAlignment="Left" Foreground ="Purple">A Simple BulletDecorator</TextBlock>
    </BulletDecorator>
    <BulletDecorator Margin="5" VerticalAlignment="Center" Background="Yellow">
        <BulletDecorator.Bullet>
            <Ellipse Margin="5,0,0,0" Height="10" Width="10" Fill="Purple" HorizontalAlignment="Left" ></Ellipse>
        </BulletDecorator.Bullet>
        <TextBlock Margin="10,0,0,0" TextWrapping="Wrap" VerticalAlignment="Center" HorizontalAlignment="Left" Foreground ="Purple">A Simple BulletDecorator</TextBlock>
    </BulletDecorator>
    <BulletDecorator Margin="5" VerticalAlignment="Center" Background="Yellow">
        <BulletDecorator.Bullet>
            <Ellipse Margin="5,0,0,0" Height="10" Width="10" Fill="Purple" HorizontalAlignment="Left" ></Ellipse>
        </BulletDecorator.Bullet>
        <TextBlock Margin="10,0,0,0" TextWrapping="Wrap" VerticalAlignment="Center" HorizontalAlignment="Left" Foreground ="Purple">A Simple BulletDecorator</TextBlock>
    </BulletDecorator>
    <BulletDecorator Margin="5" VerticalAlignment="Center" Background="Yellow">
        <BulletDecorator.Bullet>
            <CheckBox />
        </BulletDecorator.Bullet>
        <TextBlock Margin="10,0,0,0" TextWrapping="Wrap" Width="200" VerticalAlignment="Center" HorizontalAlignment="Left" Foreground ="Purple">A very very veryvery very very very long text BulletDecorator</TextBlock>
    </BulletDecorator>
</StackPanel>
```

如上代码所展示的是一个使用 `BulletDecorator` 控件实现一个无序列表。可以任意定义 `Bullet` 以及 `Child` 内的内容。

以上例子分别使用了图形与圆形放置在 `Bullet` 中作为列表的符号，`Child` 内容固定为一段文本内容，当文本换行时，文本的第一行将与列表符号对齐。

该控件除可以作为列表提供展示外，还可以在 `Bullet` 中放置单选框、多选框、输入框等。

## Canvas（画布）

定义一个区域，可在其中使用相对于 `Canvas` 区域的坐标以显式方式来定位子元素。

```xml
<Canvas Height="400" Width="400">
    <Canvas Height="100" Width="100" Top="0" Left="0" Background="Red"/>
    <Canvas Height="100" Width="100" Top="100" Left="100" Background="Green"/>
    <Canvas Height="100" Width="100" Top="50" Left="50" Background="Blue"/>
</Canvas>
```

需要注意的是 `Canvas` 作为一个存储控件的容器，其不会自动调整内部元素的大小与排列。

`Canvas` 默认也不会裁剪超出自身范围的内容，溢出的内容将会显示在 `Canvas` 外，可以通过通过调整 `ClipToBounds` 属性，来控制是否裁剪多出的内容。

## 停靠面板（DockPanel）

定义一个区域，从中可以按相对位置水平或垂直排列各个子元素。

```xml
<DockPanel>
    <Button DockPanel.Dock="Top" Background="Aqua">1(Top)</Button>
    <Button DockPanel.Dock="Left" Background="Green">2(Left)</Button>
    <Button DockPanel.Dock="Right" Background="Yellow">3(Right)</Button>
    <Button DockPanel.Dock="Bottom" Background="Blue">4(Bottom)</Button>
    <DockPanel Background="Orange" LastChildFill="False">
        <Button Height="20">Button 1</Button>
        <Button Height="20">Button 2</Button>
        <Button DockPanel.Dock="Right" Height="20">Button 3</Button>
    </DockPanel>
</DockPanel>
```

`SetDock` 方法更改某个元素相对于同一容器中的其他元素的位置。对齐属性（如 `HorizontalAlignment`）更改元素相对于其父元素的位置。

默认情况下 `DockPanel` 中的最后一个元素将填充剩余空间，而不考虑最后一个元素设置的任何其他 `Dock` 值，可以通过指定 `LastChildFill` 属性并设置停靠方向来调整。

默认情况下，`Panel` 元素不接收焦点。 若要强制 `Panel` 元素接收焦点，请将 `Focusable` 属性设置为 `true`。

## 扩展器（Expander）

`Expander` 是一个可以展开和折叠的控件，它包含标头 `Header` 和内容 `Content` 两个部分。

```xml
<Expander Background="Tan" HorizontalAlignment="Left" ExpandDirection="Down" IsExpanded="True" Width="100">
    <Expander.Header>
        <TextBlock>hd2y</TextBlock>
    </Expander.Header>
    <StackPanel HorizontalAlignment="Center">
        <TextBlock><Hyperlink>用户中心</Hyperlink></TextBlock>
        <TextBlock><Hyperlink>博客</Hyperlink></TextBlock>
        <TextBlock><Hyperlink>注销</Hyperlink></TextBlock>
    </StackPanel>
</Expander>
```

`Content` 和 `Header` 属性可以是任何类型（如字符串、图像或面板）。

如果展开窗口的内容对窗口而言太大，则可以将 `Expander` 的内容包装在 `ScrollViewer` 控件中以提供可滚动的内容。 `Expander` 控件不自动提供滚动功能。

## 网格（Grid）

`Grid` 用来定义由列和行组成的灵活的网格区域。

```xml
<Grid VerticalAlignment="Top" HorizontalAlignment="Left" ShowGridLines="True" Width="250" Height="100">
    <Grid.ColumnDefinitions>
        <ColumnDefinition />
        <ColumnDefinition />
        <ColumnDefinition />
    </Grid.ColumnDefinitions>
    <Grid.RowDefinitions>
        <RowDefinition />
        <RowDefinition />
        <RowDefinition />
        <RowDefinition />
    </Grid.RowDefinitions>

    <TextBlock FontSize="20" FontWeight="Bold" Grid.ColumnSpan="3" Grid.Row="0">2005 Products Shipped</TextBlock>
    <TextBlock FontSize="12" FontWeight="Bold" Grid.Row="1" Grid.Column="0">Quarter 1</TextBlock>
    <TextBlock FontSize="12" FontWeight="Bold" Grid.Row="1" Grid.Column="1">Quarter 2</TextBlock>
    <TextBlock FontSize="12" FontWeight="Bold" Grid.Row="1" Grid.Column="2">Quarter 3</TextBlock>
    <TextBlock Grid.Row="2" Grid.Column="0">50000</TextBlock>
    <TextBlock Grid.Row="2" Grid.Column="1">100000</TextBlock>
    <TextBlock Grid.Row="2" Grid.Column="2">150000</TextBlock>
    <TextBlock FontSize="16" FontWeight="Bold" Grid.ColumnSpan="3" Grid.Row="3">Total Units: 300000</TextBlock>
</Grid>
```

默认情况下列宽与行高是等比例分配，可以设置 `Width` 与 `Height` 属性为 `Auto` 让宽高自适应。

宽高除可以设置为 `Auto` 外，其默认值为 `1*`，这里 `*` 会使用设定了固定值外的剩余空间，按前面设置的数值进行等比例分配。

固定值我们可以设置 `100`、`100px`、`100pt`、`100cm`、`100in`，默认单位为 `px`，所以 `100` 与 `100px` 是同样的。

在子元素中，我们可以通过设置 `ColumnSpan` 与 `RowSpan`，让这个元素跨列或跨行显示。

## 网格拆分器（GridSplitter）

利用 `Grid` 我们可以很方便的划分单元格，并通过设定宽高来控制每行每列所占用的空间。

但是如果想要让用户来改变每行每列的宽高，就需要使用 `GridSplitter` 来实现这样的效果。

例如以下代码，定义了一个 `GridSplitter`，通过覆盖列的边缘来调整 Grid 中列的大小：

```xml
<Grid>
  <Grid.ColumnDefinitions>
    <ColumnDefinition/>
    <ColumnDefinition/>
  </Grid.ColumnDefinitions>
  <GridSplitter Grid.Column="0" Background="Blue" Width="5" HorizontalAlignment="Right" VerticalAlignment="Stretch"/>
</Grid>
```

而以下代码中，`GridSplitter` 将在 `Grid` 中占用一列：

```xml
<Grid.ColumnDefinitions>
    <ColumnDefinition Width="2*"/>
    <ColumnDefinition Width="Auto"/>
    <ColumnDefinition/>
</Grid.ColumnDefinitions>
<GridSplitter Grid.Column="1" HorizontalAlignment="Center" VerticalAlignment="Stretch" Background="Black" ShowsPreview="True" Width="5"/>
```

## 组框（GroupBox）

`GroupBox` 控件可以在用户界面的内容外创建一个带有边框与标题的容器。

```xml
<GroupBox>
    <GroupBox.Header>
        <Label>health information</Label>
    </GroupBox.Header>
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="30"/>
            <RowDefinition Height="30"/>
            <RowDefinition/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="100"/>
            <ColumnDefinition/>
        </Grid.ColumnDefinitions>
        <Label VerticalAlignment="Center" HorizontalAlignment="Right">height:</Label>
        <Label Grid.Row="1" VerticalAlignment="Center" HorizontalAlignment="Right">weight:</Label>
        <TextBox Grid.Row="0" Grid.Column="1" Margin="3"></TextBox>
        <TextBox Grid.Row="1" Grid.Column="1" Margin="3"></TextBox>
    </Grid>
</GroupBox>
```

`GroupBox` 的效果类似 `html` 中的 `fieldset` 标签。

另外需要注意的是，`GroupBox` 是 `HeaderedContentControl`，这意味着其 `Content` 和 `Header` 属性可以是任何类型（如字符串、图像或面板）。

## 分离器（Separator）

`Separator` 控件是用来分隔控件中的多个项。

```xml
<ToolBarTray Background="White">
    <ToolBar Band="1" BandIndex="1">
        <Button>New</Button>
        <Button>Open</Button>
        <Button>Save</Button>
        <Separator/>
        <Button>Cut</Button>
        <Button>Copy</Button>
        <Button>Paste</Button>
        <Separator/>
        <Button>Print</Button>
        <Button>Preview</Button>
    </ToolBar>
</ToolBarTray>
```

## 滚动条（ScrollBar）

`ScrollBar` 提供一个滚动条的控件，其位置对应于值。

```xml
<ScrollBar Orientation="Horizontal" Width ="4in" Value="50" Minimum="1" Maximum="100" />
```

## 滚动查看器（ScrollViewer）

`ScrollViewer` 定义可滚动的区域，可以包含其他可见元素。

```xml
<ScrollViewer HorizontalScrollBarVisibility="Auto">
    <StackPanel VerticalAlignment="Top" HorizontalAlignment="Left">
        <TextBlock TextWrapping="Wrap" Margin="0,0,0,20">Scrolling is enabled when it is necessary. Resize the window, making it larger and smaller.</TextBlock>
        <Rectangle Fill="Red" Width="500" Height="500"></Rectangle>
    </StackPanel>
</ScrollViewer>
```

## 堆栈面板（StackPanel）

`StackPanel` 可以将子元素排列成水平或垂直的一行。

```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition/>
        <RowDefinition/>
    </Grid.RowDefinitions>
    <StackPanel>
        <Button>Test1</Button>
        <Button>Test2</Button>
        <Button>Test3</Button>
    </StackPanel>
    <StackPanel Grid.Row="1" Orientation="Horizontal">
        <Button>Test4</Button>
        <Button>Test5</Button>
        <Button>Test6</Button>
    </StackPanel>
</Grid>
```

默认子元素会垂直排列，可以修改 `Orientation` 属性为 `Horizontal`，子元素将水平排列。

## 查看框（ViewBox）

`ViewBox` 组件的作用是拉伸或延展位于其中的组件，使之有更好的布局及视觉效果。

```xml
<Viewbox Stretch="Fill" StretchDirection="Both" Width="400" Height="60">
    <Button>测试</Button>
</Viewbox>
```

`ViewBox` 内只能有一个子元素，另外可以通过设置 `Stretch` 与 `StretchDirection` 属性，来设置内容的缩放效果。

`Stretch` 可选项有：
+ `None`：原始大小。
+ `Fill`：填充，子元素占满整个 `Viewbox`，不保证宽高比例。
+ `Uniform`：等比例缩放。在 `Viewbox` 能完整显示子元素的最大尺寸；
+ `UniformToFill`：等比例缩放填充，子元素在保证缩放的情况下填充 `Viewbox`，多余的部分被裁剪。

`StretchDirection` 可选项有：
+ `Both`：内容根据 `Stretch` 模式进行缩放以适合父项的大小；
+ `DownOnly`：内容仅在大于父级时缩放。如果内容较小，则不会执行任何缩放。
+ `UpOnly`：内容仅在小于父级时缩放。如果内容较大，则不执行缩放。

## 窗口（Window）

`Window` 提供了创建、配置、显示、管理窗口与对话框的生命周期的能力。

```xml
<Window xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" Title="Main Window in Markup Only" Height="300" Width="300" />
```

用户与独立应用程序之间的交互点是一个窗口。`WPF` 窗口包含两个不同的区域：
+ 承载 `Windows` 修饰的非工作区，包括图标、标题、系统菜单、最小化按钮、最大化按钮、还原按钮、关闭按钮和边框。
+ 承载特定于应用程序的内容的工作区。

`Window` 封装了创建、配置、显示和管理窗口和对话框的声明周期的功能，并提供了以下关键服务：
+ 生命周期管理：`Activate`、`Activated`、`Close`、`Closed`、`Closing`、`Deactivated`、`Hide`、`IsActive`、`Show`、`SourceInitialized`。
+ 窗口管理：`GetWindow`、`OwnedWindowsOwner`。
+ 外观和行为：`AllowsTransparency`、`ContentRendered`、`DragMove`、`Icon`、`Left`、`LocationChanged`、`ResizeMode`、`RestoreBounds`、`ShowActivated`、`ShowInTaskbar`、`SizeToContent`、`StateChanged`、`Title`、`Top`、`Topmost`、`WindowStartupLocation`、`WindowState`、`WindowStyle`。
+ 对话框：`DialogResult`，`ShowDialog`。

此外，`Application` 公开了对管理应用程序中的所有窗口的特殊支持：应用程序维护当前在应用程序中实例化的所有窗口的列表。 此列表由 `Windows` 属性公开。

默认情况下，将使用对应用程序中实例化的第一个 `Window` 的引用自动设置 `MainWindow`。 从而使窗口成为主应用程序窗口。

`Window` 主要用于显示独立应用程序的窗口和对话框。但是，对于需要在窗口级别导航的应用程序（如向导），可以改用 `NavigationWindow`。`NavigationWindow` 派生自 `Window` 并通过浏览器样式的导航支持对其进行扩展。

> **注：** 本文的目的是为了熟悉常见的布局控件，所以存在大段的复制粘贴。
> 示例代码来自 `MSDN`，内容已经上传至 `github`：[hd2y/WpfExample](https://github.com/hd2y/WpfExample)

> 参考：
> + MSDN：[Controls by Category](https://docs.microsoft.com/en-us/dotnet/framework/wpf/controls/controls-by-category)
> + CSDN：[继续聊 WPF —— Thumb 控件](https://blog.csdn.net/tcjiaan/article/details/6961335)
