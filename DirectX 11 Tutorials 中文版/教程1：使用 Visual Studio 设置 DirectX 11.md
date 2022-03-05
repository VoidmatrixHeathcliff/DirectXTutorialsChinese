# 教程1：使用 Visual Studio 设置 DirectX 11

[原文链接](http://www.rastertek.com/dx11s2tut01.html)

在编写任何图形代码之前，我们都需要有相关的工具，这些工具中的最关键的是编译器，并且它最好是内置在一个好的 IDE 中。

我使用 Visual Studio 2015 来编写项目。还有其他几种，有些是免费的，想用哪个由你自己来决定。我个人推荐 Visual Studio 2015 社区版，因为它是一个优秀的免费 IDE 。你可以从 Visual Studio 官网下载 Visual Studio 2015 社区版。在安装 VisualStudio 2015 时，请确保选择“自定义”并选择“安装全部”，以便所有 Visual C++ 组件都安装好，否则它将主要设置为 C# 语言开发。

你需要的第二个工具是 Windows 10 SDK 。

Windows 10 SDK包含编写 DirectX 11 应用程序所需的所有 DirectX 11 头文件、静态链接库和动态链接库等。如果您安装了Visual Studio 2015，则 SDK 已随它一起安装。否则，您可以从 Windows 开发中心（msdn）网站下载适用于 Windows 10 的 Window s独立 SDK。下载并安装 SDK 后，您将拥有编译 DirectX 11 程序所需的文件。Windows 10 SDK 的文档也都在Windows 开发中心的网站上。您可以在那里找到 Direct3D 11 编程指南，其中包含所有 DirectX 11 文档以及一些示例代码。

一旦安装了 IDE 和 SDK，现在就可以将 IDE 设置为与 Windows 10 SDK 一起使用，以便编写 DirectX 11 应用程序。

请注意，对于某些 IDE 而言，你需要先安装它们再考虑安装 Windows 10 SDK。


## 配置 Visual Studio 2015

在 Visual Studio 2015 环境下，我进行了以下步骤的配置：

首先你需要创建一个空的 Win32 项目，所以选择 `File -> New -> Project`。在新项目菜单中选择 `使用Win32模板 -> Visual C++ -> Win32`。然后从选项中选择 `Win32 Project`。给这个项目起一个名字（我的叫 “mine Engine”），选择工程位置，然后点击 “Ok”。

点击 “下一步” 后，你将跳转到另一个菜单：在 “其他选项” 下，在 “空项目” 框中打勾，然后取消选中安全开发生命周期（SDL）检查。点击 “完成”，到此为止，你现在应该已经有了一个基本设置完成的 Win32 空项目。

然后，在顶部栏上，您将看到解决方案平台下拉列表中的值 “x86”，单机此项并重新选择为 “x64”。这会将项目设置为64位，而不是默认的32位。


## 总结

我们的 DirectX 11 开发环境现在应该已经设置好，可以开始编写 DirectX 11 应用程序了。


## 练习

1. 快速浏览 Windows 开发中心（msdn）网站上的 DirectX 11 编程指南。