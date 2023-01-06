---
title: C/C++图形库EasyX学习
date: 2022-07-13 19:56:40
categories:
- 编程
tags:
- C++
- EasyX
---

​	偶然在B站刷到了EasyX的教程，想了想我现在写C++确实全都是小黑框，于是快马加鞭的开始进行EasyX的学习，同时准备下一步学学QT（咕咕咕）

​	部分教程跟随B站@C语言Plus [视频链接](https://www.bilibili.com/video/BV11p4y1i74A)

​	部分引用EasyX官方说明文档 [在线文档链接](https://docs.easyx.cn/zh-cn/intro)

<!--more-->

同类笔记指路:

​	[CSDN@枳柯](https://blog.csdn.net/qq_60126616/article/details/120687524)

​	[CSDN@半生瓜のblog](https://blog.csdn.net/qq_51604330/article/details/116563569)

# EasyX是什么？

EasyX 是针对 C/C++ 的图形库，可以帮助使用C/C++语言的程序员快速上手图形和游戏编程。
比如，可以用 VC + EasyX 很快的用几何图形画一个房子，或者一辆移动的小车，可以编写俄罗斯方块、贪吃蛇、黑白棋等小游戏，可以练习图形学的各种算法，等等。

# 下载安装EasyX

准备一个任意vs系列的编译器(vs c++ 6.0, vs2011, vs2013, vs2017, vs2019等等)

（本博客使用VS2022作为使用实际参考）

官网下载EasyX并根据提示完成配置

```官网
https://easyx.cn/
```



# 源码中需要包含的头文件

```
#include <easyx.h>
//不含旧函数
#include <graphics.h>
//含有旧函数，兼容选项
```

# 常用函数

## 一、窗口函数

### 窗口的坐标位置描述

EasyX所创立窗口的原点位于显示窗口的左上角，以向右为x轴正向，向下为y轴正向，度量单位是像素点。



### 初始化绘图窗口函数

这个函数用于初始化绘图窗口。

```cpp
HWND initgraph(int width,int height,int flag = NULL);
```

#### 参数

width：绘图窗口的宽度。

height：绘图窗口的高度。

flag：绘图窗口的样式，默认为 NULL。可为以下值：

| 值             | 含义                           |
| -------------- | ------------------------------ |
| EW_DBLCLKS     | 在绘图窗口中支持鼠标双击事件。 |
| EW_NOCLOSE     | 禁用绘图窗口的关闭按钮。       |
| EW_NOMINIMIZE  | 禁用绘图窗口的最小化按钮。     |
| EW_SHOWCONSOLE | 显示控制台窗口。               |

#### 返回值

返回新建绘图窗口的句柄。

可以使用 | 使用多个flag参数

```
initgraph(640,480,NOCLOSE | EW_SHOWCONSOLE);
```

### 关闭绘图窗口

这个函数用于关闭绘图窗口。

①在程序结束前使用。

②在窗口需要关闭时使用。

```cpp
void closegraph();
```

### 清空绘图设备

这个函数使用当前背景色清空绘图设备。

在换背景色之后使用

```cpp
void cleardevice();
```

## 二、绘图函数

常用绘图（形状）函数有3x8种

### 填充样式

用画圆举例：

```
circle();//无填充

fillcircle();//有边框填充，前缀fill

solidcircle();//无边框填充，前缀solid
```

### 形状

类似以上的举例，常用以下八种图形

| 函数      | 图形             | 函数      | 图形             |
| --------- | ---------------- | --------- | ---------------- |
| circle    | 圆               | ellipse   | 椭圆             |
| pie       | 扇形             | polygon   | 多边形           |
| rectangle | 矩形             | roundrect | 圆角矩形         |
| line      | 线（只有第一种） | putpixel  | 点（只有第一种） |

### 设置颜色

#### 设置填充颜色

```cpp
void setfillcolor(COLORREF color);
```

#### 设置画线颜色

```cpp
void setlinecolor(COLORREF color);
```

#### 设置背景颜色

```cpp
void setbkcolor(COLORREF color);
```

使用示例

```cpp
// 设置背景色为蓝色
setbkcolor(BLUE);
// 用背景色清空屏幕
cleardevice();
```

#### 颜色表达

以下是几种设置画线颜色的方法：

```cpp
setlinecolor(0xff0000);
setlinecolor(BLUE);
setlinecolor(RGB(0, 0, 255));
setlinecolor(HSLtoRGB(240, 1, 0.5));
```

### 细节补充

#### 1.圆

这个函数用于画无填充的圆。

```cpp
void circle(int x,int y,int radius);
```

##### 参数

x：圆心 x 坐标。

y：圆心 y 坐标。

radius：圆的半径。

#### 2.椭圆

```cpp
void ellipse(int left,int top,int right,int bottom);
```

##### 参数

left：椭圆外切矩形的左上角 x 坐标。

top：椭圆外切矩形的左上角 y 坐标。

right：椭圆外切矩形的右下角 x 坐标。

bottom：椭圆外切矩形的右下角 y 坐标。

#### 3.矩形

这个函数用于画有边框的填充矩形。

```cpp
void fillrectangle(	int left,int top,int right,	int bottom);
```

##### 参数

left：矩形左部 x 坐标。

top：矩形顶部 y 坐标。

right：矩形右部 x 坐标。

bottom：矩形底部 y 坐标。

## 三、文字绘制函数

### 设置当前文字颜色

```cpp
void settextcolor(COLORREF color);
```

### 设置当前文字样式

```cpp
void settextstyle(	int nHeight,	int nWidth,	LPCTSTR lpszFace);
void settextstyle(const LOGFONT *font);
```

#### 参数

```
nHeight：指定高度（逻辑单位）。

nWidth：字符的平均宽度（逻辑单位）。如果为 0，则比例自适应。

lpszFace：字体名称。

nEscapement：字符串的书写角度，单位 0.1 度。

nOrientation：每个字符的书写角度，单位 0.1 度。

nWeight：字符的笔画粗细，范围 0~1000。0 表示默认粗细。详见 [LOGFONT]结构体。

bItalic：是否斜体。

bUnderline：是否有下划线。

bStrikeOut：是否有删除线。

fbCharSet：指定字符集。详见 [LOGFONT]结构体。

fbOutPrecision：指定文字的输出精度。详见 [LOGFONT]结构体。

fbClipPrecision：指定文字的剪辑精度。详见 [LOGFONT]结构体。

fbQuality：指定文字的输出质量。详见 [LOGFONT]结构体。

fbPitchAndFamily：指定以常规方式描述字体的字体系列。详见 [LOGFONT]结构体。

font：指向 [LOGFONT] 结构体的指针。
```

### 设置背景模式

这个函数用于设置当前设备图案填充和文字输出时的背景模式。

```cpp
void setbkmode(int mode);
```

#### 参数

mode：指定图案填充和文字输出时的背景模式，可以是以下值

| 值          | 描述                           |
| ----------- | ------------------------------ |
| OPAQUE      | 背景用当前背景色填充（默认）。 |
| TRANSPARENT | 背景是透明的。                 |

### 在指定位置输出字符串

```cpp
void outtextxy(	int x,	int y,	LPCTSTR str);
void outtextxy(	int x,	int y,	TCHAR c);
```

### 获取字符串实际占用大小

**可以用于文本的锤子和水平居中显示**

这个函数用于获取字符串实际占用的像素高度。

```cpp
int textheight(LPCTSTR str);
int textheight(TCHAR c);
```

这个函数用于获取字符串实际占用的像素宽度。

```cpp
int textwidth(LPCTSTR str);
int textwidth(TCHAR c);
```

#### 参数

str：指定的字符串指针。

c：指定的字符。

## 四、图像函数

### IMAGE

图像对象。

```cpp
class IMAGE(int width = 0, int height = 0);
```

#### 公有成员

int getwidth();

返回 IMAGE 对象的宽度，以像素为单位。

int getheight();

返回 IMAGE 对象的高度，以像素为单位。

operator =

实现 IMAGE 对象的直接赋值。该操作仅拷贝源图像的内容，不拷贝源图像的绘图窗口。

### 从文件中读取图像

这个函数用于从文件中读取图像。

```cpp
// 从图片文件获取图像(bmp/gif/jpg/png/tif/emf/wmf/ico)
void loadimage(
	IMAGE* pDstImg,			// 保存图像的 IMAGE 对象指针
	LPCTSTR pImgFile,		// 图片文件名
	int nWidth = 0,			// 图片的拉伸宽度
	int nHeight = 0,		// 图片的拉伸高度
	bool bResize = false	// 是否调整 IMAGE 的大小以适应图片
);
```

#### 参数

```
pDstImg：保存图像的 IMAGE 对象指针。如果为 NULL，表示图片将读取至绘图窗口。

pImgFile：图片文件名。支持 bmp / gif / jpg / png / tif / emf / wmf / ico 格式的图片。gif  格式的图片仅加载第一帧；gif 与 png 均不支持透明。

nWidth：图片的拉伸宽度。加载图片后，会拉伸至该宽度。如果为 0，表示使用原图的宽度。

nHeight：图片的拉伸高度。加载图片后，会拉伸至该高度。如果为 0，表示使用原图的高度。

bResize：是否调整 IMAGE 的大小以适应图片。

pResType：图片资源类型。

pResName：图片资源名称。
```

### 绘制指定图像

这个函数的几个重载用于在当前设备上绘制指定图像。

```cpp
// 绘制图像
void putimage(
	int dstX,				// 绘制位置的 x 坐标
	int dstY,				// 绘制位置的 y 坐标
	IMAGE *pSrcImg,			// 要绘制的 IMAGE 对象指针
	DWORD dwRop = SRCCOPY	// 三元光栅操作码
);
```

#### 备注

三元光栅操作码（即位操作模式），支持全部的 256 种三元光栅操作码，常用的几种如下：

| 值          | 含义                                                  |
| ----------- | ----------------------------------------------------- |
| DSTINVERT   | 目标图像 = NOT 目标图像                               |
| MERGECOPY   | 目标图像 = 源图像 AND 当前填充颜色                    |
| MERGEPAINT  | 目标图像 = 目标图像 OR (NOT 源图像)                   |
| NOTSRCCOPY  | 目标图像 = NOT 源图像                                 |
| NOTSRCERASE | 目标图像 = NOT (目标图像 OR 源图像)                   |
| PATCOPY     | 目标图像 = 当前填充颜色                               |
| PATINVERT   | 目标图像 = 目标图像 XOR 当前填充颜色                  |
| PATPAINT    | 目标图像 = 目标图像 OR ((NOT 源图像) OR 当前填充颜色) |
| SRCAND      | 目标图像 = 目标图像 AND 源图像                        |
| SRCCOPY     | 目标图像 = 源图像                                     |
| SRCERASE    | 目标图像 = (NOT 目标图像) AND 源图像                  |
| SRCINVERT   | 目标图像 = 目标图像 XOR 源图像                        |
| SRCPAINT    | 目标图像 = 目标图像 OR 源图像                         |

注：

1. AND / OR / NOT / XOR 为布尔运算。 
2. "当前填充颜色"是指通过 setfillcolor 设置的用于当前填充的颜色。 

## 五、消息处理函数（新版）

消息缓冲区可以缓冲 **63 个**未处理的消息。每次获取消息时，将从消息缓冲区取出一个最早发生的消息。消息缓冲区满了之后，不再接收任何消息。

相关函数如下：

| 函数或数据类型                   | 描述                                                 |
| -------------------------------- | ---------------------------------------------------- |
| [ExMessage](ExMessage.htm)       | 消息结构体。                                         |
| [flushmessage](flushmessage.htm) | 清空消息缓冲区。                                     |
| [getmessage](getmessage.htm)     | 获取一个消息。如果当前消息缓冲区中没有，就一直等待。 |
| [peekmessage](peekmessage.htm)   | 获取一个消息，并立即返回。                           |

### ExMessage

这个结构体用于保存鼠标消息，定义如下：

```cpp
struct ExMessage
{
	USHORT message;					// 消息标识
	union
	{
		// 鼠标消息的数据
		struct
		{
			bool ctrl		:1;		// Ctrl 键是否按下
			bool shift		:1;		// Shift 键是否按下
			bool lbutton	:1;		// 鼠标左键是否按下
			bool mbutton	:1;		// 鼠标中键是否按下
			bool rbutton	:1;		// 鼠标右键
			short x;				// 鼠标的 x 坐标
			short y;				// 鼠标的 y 坐标
			short wheel;			// 鼠标滚轮滚动值，为 120 的倍数
		};

		// 按键消息的数据
		struct
		{
			BYTE vkcode;			// 按键的虚拟键码
			BYTE scancode;			// 按键的扫描码（依赖于 OEM）
			bool extended	:1;		// 按键是否是扩展键
			bool prevdown	:1;		// 按键的前一个状态是否按下
		};

		// 字符消息的数据
		TCHAR ch;

		// 窗口消息的数据
		struct
		{
			WPARAM wParam;
			LPARAM lParam;
		};
	};
};
```

#### 成员

message

消息标识，可为以下值：

| 消息标识         | 消息类别           | 描述                   |
| ---------------- | ------------------ | ---------------------- |
| WM_MOUSEMOVE     | EM_MOUSE           | 鼠标移动消息。         |
| WM_MOUSEWHEEL    | 鼠标滚轮拨动消息。 |                        |
| WM_LBUTTONDOWN   | 左键按下消息。     |                        |
| WM_LBUTTONUP     | 左键弹起消息。     |                        |
| WM_LBUTTONDBLCLK | 左键双击消息。     |                        |
| WM_MBUTTONDOWN   | 中键按下消息。     |                        |
| WM_MBUTTONUP     | 中键弹起消息。     |                        |
| WM_MBUTTONDBLCLK | 中键双击消息。     |                        |
| WM_RBUTTONDOWN   | 右键按下消息。     |                        |
| WM_RBUTTONUP     | 右键弹起消息。     |                        |
| WM_RBUTTONDBLCLK | 右键双击消息。     |                        |
| WM_KEYDOWN       | EM_KEY             | 按键按下消息           |
| WM_KEYUP         | 按键弹起消息。     |                        |
| WM_CHAR          | EM_CHAR            | 字符消息。             |
| WM_ACTIVATE      | EM_WINDOW          | 窗口激活状态改变消息。 |
| WM_MOVE          | 窗口移动消息。     |                        |
| WM_SIZE          | 窗口大小改变消息。 |                        |

ctrl：Ctrl 键是否按下。仅当消息所属类别为 EM_MOUSE 时有效。

shift：Shift 键是否按下。仅当消息所属类别为 EM_MOUSE 时有效。

lbutton：鼠标左键是否按下。仅当消息所属类别为 EM_MOUSE 时有效。

mbutton：鼠标中键是否按下。仅当消息所属类别为 EM_MOUSE 时有效。

rbutton：鼠标右键是否按下。仅当消息所属类别为 EM_MOUSE 时有效。

x：当前鼠标 x 坐标（物理坐标）。仅当消息所属类别为 EM_MOUSE 时有效。

y：当前鼠标 y 坐标（物理坐标）。仅当消息所属类别为 EM_MOUSE 时有效。

wheel：鼠标滚轮滚动值，为 120 的倍数。仅当消息所属类别为 EM_MOUSE 时有效。

vkcode：按键的虚拟键码。仅当消息所属类别为 EM_KEY 时有效。

在微软网站上列出有所有的虚拟键码：https://docs.microsoft.com/windows/win32/inputdev/virtual-key-codes

scancode：按键的扫描码（依赖于 OEM）。仅当消息所属类别为 EM_KEY 时有效。

extended：按键是否为扩展按键，例如功能键和数字键盘。仅当消息所属类别为 EM_KEY 时有效。

prevdown：按键的前一个状态是否为按下。仅当消息所属类别为 EM_KEY 时有效。

ch：收到的字符。仅当消息所属类别为 EM_CHAR 时有效。

wParam：消息对应的 wParam 参数。仅当消息所属类别为 EM_WINDOW 时有效。

lParam：消息对应的 lParam 参数。仅当消息所属类别为 EM_WINDOW 时有效。



## 六、其他函数

# 常见问题

## 1.源文件问题

```
fatal error C1189: #error :  EasyX is only for C++
```

字面意思EasyX仅适用于C++，源文件应为.cpp而不是.c或者其他

## 2.字符集问题

```
错误	C2665	“outtextxy”: 2 个重载中没有一个可以转换所有参数类型	
error C2665: “outtextxy”: 2 个重载中没有一个可以转换所有参数类型
```

产生错误的函数定义

```
void outtextxy(	int x,	int y,	LPCTSTR str);
void outtextxy(	int x,	int y,	TCHAR c);
```

解决方法如下

①在字符串前面加上大写的L

```
    outtextxy(10,10, L"This is Lymone");
```

②在字符串前面加上大写的TEXT()或者用_T()或者……

```
    outtextxy(10,10, TEXT("This is Lymone"));
```

此方法在底层实现上与上一方法相同，如下：

```
#define TEXT(quote) __TEXT(quote) 
#define __TEXT(quote) L##quote 

#define _T(x)       __T(x)
#define _TEXT(x)    __T(x)
#define __T(x)      L ## x
```

③不需要添加任何代码，进项目->属性->配置属性->高级->字符集->改为多字节字符集，推荐使用（该路径为VS2022，其他版本请自己在相应位置寻找字符集设置）

