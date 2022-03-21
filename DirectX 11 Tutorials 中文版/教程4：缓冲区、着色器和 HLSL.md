# 教程4：缓冲区、着色器和 HLSL

[原文链接](http://www.rastertek.com/dx11s2tut04.html)

本教程将介绍如何在 DirectX 11 中编写顶点和像素着色器，还将介绍如何在 DirectX 11 中使用顶点和索引缓冲区，这些是渲染 3D 图形时需要理解和利用的最基本概念。

## 顶点缓冲区

首先要理解的概念是顶点缓冲区，为了说明这个概念，让我们以球体的 3D 模型为例：

![3D球体模型图1](img/tutorial_4_1.gif)

3D 球体模型实际上由成百上千个三角形组成：

![3D球体模型图2](img/tutorial_4_2.gif)

球体模型中的每个三角形都有三个点，我们称每个点为顶点，因此，为了渲染球体模型，我们需要将构成球体的所有顶点放入一个特殊的数据数组中，我们称之为顶点缓冲区；一旦球体模型的所有点都被放置到了顶点缓冲区中，我们就可以将顶点缓冲区发送到 GPU，来将模型渲染出来。

## 索引缓冲区

索引缓冲区与顶点缓冲区相关，它们的目的是记录顶点缓冲区中每个顶点的位置；然后，GPU 使用索引缓冲区快速查找顶点缓冲区中的特定顶点。

索引缓冲区的概念类似于在书中使用目录的概念，它有助于以更高的速度找到您要查找的主题。

DirectX 文档表示，使用索引缓冲区还可以增加将顶点数据缓存到显存中更快位置的可能性。

因此，出于性能方面的考虑，强烈建议使用索引缓冲区。

## 顶点着色器

顶点着色器是一种小程序，主要用于将顶点从顶点缓冲区转换到三维空间，它还可以进行其他计算，例如计算每个顶点的法线。

GPU 将为需要处理的每个顶点调用顶点着色器程序，例如，一个 5000 个多边形组成的模型每帧将运行 15000 次你定义的顶点着色器程序，来绘制这个模型；因此，如果你将图形程序锁定为每秒 60 帧，它将每秒调用 90 万次顶点着色器来绘制 5000 个三角形。

显而易见，编写高效的顶点着色器非常重要。

## 像素着色器

像素着色器是为我们绘制的多边形着色而编写的小程序，它们由 GPU 为屏幕上的每个可见像素运行。

颜色、纹理、光照以及你计划对多边形面的大多数效果设置都由像素着色器程序处理。

由于 GPU 将调用像素着色器的次数，因此必须高效地编写像素着色器。

## HLSL

HLSL 是我们在 DirectX 11 中用来编写这些顶点和像素着色器程序的语言，它的语法与C语言基本相同，只是有一些预定义的类型。

HLSL 程序文件由 全局变量、类型定义、顶点着色器、像素着色器和几何体着色器 组成。

由于这是第一个 HLSL 教程，我们将使用 DirectX 10 开始一个非常简单的 HLSL 程序。

## 更新后的框架

![框架示意图](img/tutorial_4_3.gif)

本教程的框架已经更新，在 `GraphicsClass` 下，我们添加了三个新类，分别称为 `CameraClass`、`ModelClass` 和 `ColorShaderClass`。

`CameraClass` 将处理我们之前讨论过的视图矩阵，它将处理相机在世界坐标系下的位置，并在着色器需要绘制并确定我们从何处观看场景时将其传递给着色器。

`ModelClass` 将处理 3D 模型的几何图形，为了简单起见，在本教程中 3D 模型将只是一个三角形。

最后，`ColorShaderClass` 将负责调用 HLSL 着色器将模型渲染到屏幕上。

在本篇教程代码开始之前，让我们先看一下 HLSL 着色器程序。

## Color.vs

这将是我们的第一个着色器程序。

着色器是对模型进行实际渲染的小程序，这些着色器用 HLSL 编写，并存储在名为 color.vs 和 color.ps 的源文件中。另外，我把这些文件和 .cpp 与 .h 文件一起放到了引擎中，请注意，你需要在 VisualStudio 中为它们创建一个新的 过滤器/文件夹。确保右键单击着色器文件并选择“属性”，在弹出窗口中，内容部分中应该没有任何内容，项目类型部分中应该显示“不参与构建”，否则你将收到关于程序主入口点的编译错误。

现在，这个着色器的目的只是绘制彩色三角形，因为我在第一个 HLSL 教程中尽可能地保持简单。以下是顶点着色器的代码：

```hlsl
////////////////////////////////////////////////////////////////////////////////
// Filename: color.vs
////////////////////////////////////////////////////////////////////////////////
```

在着色器程序中，从全局变量开始：这些全局变量可以从外部的 C++ 代码进行修改，你可以使用多种类型的变量，例如 int 或 float，然后在外部设置它们以供着色器程序使用。

通常，你会将大多数全局变量放在名为 `cbuffer` 的缓冲区对象类型中，即使它只是一个全局变量。逻辑地组织这些缓冲区对于着色器的有效执行以及显卡存储缓冲区的方式非常重要。

在本例中，我将三个矩阵放在同一个缓冲区中，因为我将同时更新它们的每一帧。

```hlsl
/////////////
// GLOBALS //
/////////////
cbuffer MatrixBuffer
{
    matrix worldMatrix;
    matrix viewMatrix;
    matrix projectionMatrix;
};
```

与 C 类似，我们可以创建自己的类型定义：我们将使用 HLSL 可用的不同类型，例如 `float4`，这使对着色器的编程更容易和可读。

在本例中，我们将创建具有 `x、y、z、w` 位置向量和 红、绿、蓝、alpha 颜色的类型。`POSITION`、`COLOR` 和 `SV_POSITION` 是向 GPU 传递变量使用的语义。

我必须在这里创建两个不同的结构，因为顶点着色器和像素着色器的语义不同，尽管在其他方面结构相同：`POSITION` 适用于顶点着色器，`SV_POSITION` 适用于像素着色器，而 `COLOR` 适用于两者；如果需要多个相同类型，则必须在末尾添加一个数字，如`COLOR0、COLOR1`等。

```hlsl
//////////////
// TYPEDEFS //
//////////////
struct VertexInputType
{
    float4 position : POSITION;
    float4 color : COLOR;
};

struct PixelInputType
{
    float4 position : SV_POSITION;
    float4 color : COLOR;
};
```

当 GPU 处理来自已发送给它的顶点缓冲区的数据时，会调用顶点着色器,这个我命名为 `ColorVertexShader` 的顶点着色器将为顶点缓冲区中的每个顶点调用。顶点着色器的输入必须与顶点缓冲区中的数据格式以及着色器源文件（在本例中为 `VertexInputType`）中的类型定义相匹配。顶点着色器的输出将被发送到像素着色器。在这种情况下，输出类型被称为 `PixelInputType`，这也是上面定义的。

考虑这一点，你可以看到顶点着色器创建了一个 `PixelInputType` 类型的输出变量。然后获取输入顶点的位置，并将其乘以 世界、视图和投影矩阵，这将根据我们的视角将顶点放置在正确的位置，以便在 3D 空间中进行渲染，然后放置到 2D 屏幕上。

之后，输出变量获取输入颜色的副本，然后返回的输出将用作像素着色器输入。还要注意，我刻意在输入时将 `W` 值设置为 1.0，否则它是未定义的，因为我们只读取位置的 `XYZ` 向量。

```hlsl
////////////////////////////////////////////////////////////////////////////////
// Vertex Shader
////////////////////////////////////////////////////////////////////////////////
PixelInputType ColorVertexShader(VertexInputType input)
{
    PixelInputType output;
    

    // Change the position vector to be 4 units for proper matrix calculations.
    input.position.w = 1.0f;

    // Calculate the position of the vertex against the world, view, and projection matrices.
    output.position = mul(input.position, worldMatrix);
    output.position = mul(output.position, viewMatrix);
    output.position = mul(output.position, projectionMatrix);
    
    // Store the input color for the pixel shader to use.
    output.color = input.color;
    
    return output;
}
```

## Color.ps

像素着色器为渲染到屏幕的多边形上绘制每个像素，在这个像素着色器中，它使用 `PixelInputType` 作为输入，并返回一个 `float4` 作为输出，它代表最终的像素颜色。

这个像素着色器程序非常简单，因为我们只是告诉它将像素的颜色设置为与颜色的输入值相同的颜色，请注意，像素着色器从顶点着色器输出获取其输入。

```hlsl
////////////////////////////////////////////////////////////////////////////////
// Filename: color.ps
////////////////////////////////////////////////////////////////////////////////


//////////////
// TYPEDEFS //
//////////////
struct PixelInputType
{
    float4 position : SV_POSITION;
    float4 color : COLOR;
};


////////////////////////////////////////////////////////////////////////////////
// Pixel Shader
////////////////////////////////////////////////////////////////////////////////
float4 ColorPixelShader(PixelInputType input) : SV_TARGET
{
    return input.color;
}
```

## Colorshaderclass.h

我们将使用 `ColorShaderClass` 调用 HLSL 着色器来绘制 GPU 上的 3D 模型。

```cpp
////////////////////////////////////////////////////////////////////////////////
// Filename: colorshaderclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _COLORSHADERCLASS_H_
#define _COLORSHADERCLASS_H_


//////////////
// INCLUDES //
//////////////
#include <d3d11.h>
#include <d3dcompiler.h>
#include <directxmath.h>
#include <fstream>
using namespace DirectX;
using namespace std;


////////////////////////////////////////////////////////////////////////////////
// Class name: ColorShaderClass
////////////////////////////////////////////////////////////////////////////////
class ColorShaderClass
{
private:
```

以下是将与顶点着色器一起使用的 `cBuffer` 类型的定义。

此 `typedef` 必须与顶点着色器中的 `typedef` 完全相同，因为模型数据需要与着色器中的 `typedef` 匹配，才能进行正确渲染。

```cpp
	struct MatrixBufferType
	{
		XMMATRIX world;
		XMMATRIX view;
		XMMATRIX projection;
	};

public:
	ColorShaderClass();
	ColorShaderClass(const ColorShaderClass&);
	~ColorShaderClass();
```