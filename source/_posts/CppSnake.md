---
title: C++贪吃蛇项目
date: 2022-08-25 10:20:00
categories:
- 编程
tags: 
- C++
- EasyX
---

为了提高我的编程水平。在一众前辈的指引下，决定做个小项目，在完善项目的过程中逐渐提高自己的水准。

最终决定做个贪吃蛇小游戏吧，规则并不复杂，似乎和我的水平很搭。

本文提及的代码都可以在我(LymoneLM)的github仓库LymoneTest中找到。

<!--more-->

# 前期准备

首先敲定了使用的图形库为EasyX（当然主要也是因为目前EasyX也半生不熟）

## 浅浅的写写画画

（Hexo挂图有点麻烦啊）

![OneNote绘制的思维导图](https://s1.328888.xyz/2022/09/01/kmyTy.png)

打开OneNote，按照过去玩过的游戏，和对游戏工业的简单了解，初步的划分了编程的思路，大体的用已知的知识进行了构思。

## 技术验证

数据部分，地图方面准备使用一个字符型二维数组进行记录，初定大小为20*20。

蛇身则准备使用一个链队列进行存储，单进单出的蛇身节点和队列非常的相似。

### 链队列

我在数据结构方面的知识基本脱胎于程杰的《大话数据结构》，但是由于这本书主要是在高中的课余时间进行阅读的，所以并没有进行大量的实践，因此也出现了不小的岔子。

对于链队列，我最早的想法是使用struct结构体

```c++
struct sNode
{
	char x,y;
	sNode *next;
}
//以下为使用
sNode *p=new sNode;
```

然而这里出现了问题，如果是链队列，肯定需要循环申请内存，然而如果不为p指针释放内存无法再次申请。

于是我改用了类

关于链队列的部分可以参看  C++进阶学习 那篇博文的有关内容，这里仅放出最后的结果

```c++
class sNode
{
public:
	int x, y;
	sNode* snext;
};

class Linkqueue
{
	//链队列-逆
	//删尾加头 
public:
	void initQ();
	void enQ(sNode);
	sNode deQ();
	void destoryQ();
	void printQ();//devOnly 
	short countQ();
	bool pmapQ();
private:
	sNode* front;
	sNode* rear;
};
```

具体的实现参看SnakeData.cpp部分的文件

实际在实现的过程中参考了部分博文，但是原博文在实现程序的时候先创建了一个链队列类的对象作为链队列，并另外创建了一个对象，专门使用第二个对象的方法对链队列进行操作。

他的实现代码中所有的方法都含有一个引用传递的参数，传递一个该类的对象。我不知道这样做有什么好处，并且我在逻辑上也看不出这么做的优势，所以我并没有才用这个方法。

如果您了解我采用的方法的劣势或者明白他的这种实现的优势，恳请您能赐教。

### 绘制

绘制这部分总体来说问题并不大，参照帮助文档一步步的进行实现就好了。栅格绘制和简单图形绘制并没有任何问题，出现问题的是右边的文字绘制，在绘制过程中不断的发生报错，最后了解到这种绘制方式传入的字符串参数必须为TCHAR类型，终于通过改编例程中的代码实现了这个绘制。

```c++
void LSD::drawdeath()
{
    settextcolor(0x0000FF);
    settextstyle(40, 40, L"Minecraft");
    printf("WASTED\n");
    RECT r = { 0, 0, 699, 504 };
    drawtext(_T("WASTED"), &r, DT_CENTER | DT_VCENTER | DT_SINGLELINE);
    
    return;
}
void LSD::drawTitle()
{
    settextcolor(0xFFFFFF);
    settextstyle(14, 14, L"Minecraft");
    RECT r = { 500, 10, 699, 30 };
    drawtext(_T("LymoneSnake"), &r, DT_CENTER | DT_VCENTER | DT_SINGLELINE);
    r = { 500, 30, 699, 50 };
    drawtext(_T("按↑↓←→开始"), &r, DT_CENTER | DT_VCENTER | DT_SINGLELINE);
    settextstyle(20, 20, L"Minecraft");
    r = { 500, 50, 699, 90 };
    drawtext(_T("当前分数："), &r, DT_CENTER | DT_VCENTER | DT_SINGLELINE);
    return;
}
void LSD::drawScore(int score)
{
    TCHAR s[5];
    _stprintf_s(s, _T("%d"), score);

    settextcolor(0xFFFFFF);
    settextstyle(30, 30, L"Minecraft");
    RECT r = { 500, 90, 699, 140 };
    drawtext(s, &r, DT_CENTER | DT_VCENTER | DT_SINGLELINE);
    return;

}
```

Minecraft为使用的一个字体，理论上换用其他例如“黑体”是可以正常运行的。

# 一些迭代

## 画面绘制

最初的版本中，蛇每次移动后我都会重新绘制整个地图：

刷新map，绘制栅格，然后绘制蛇体

显而易见的，这会造成卡顿和频闪

在意识到这个问题之后我对链队列的实现进行了改进，在删除一个节点的同时返回这个节点的坐标，以此将这个坐标对应的位置画上一个黑色的图形，从而实现缩短蛇体。

这种方案显然更合适。

其实在采用这种绘制方式之后，map这个二维字符数组的作用就有点鸡肋了，但是随机苹果和蛇头的功能仍然依赖这个东西，后续我想添加的墙体功能也可能依赖于这个map，所以并没有对其进行彻底删除。

## 蛇头

作为判定的关键，蛇头的坐标显而易见的十分重要，在最初的版本中我使用

```C++
extern int hx, hy
```

来存储蛇头的坐标，显然这样做并不直观，所以随后我改用一个sNode对其进行存储。

## 退出机制

其实在任何时候按下ESC是可以退出的，这是在我不断调试的过程中加入的一个功能。完全没有在程序的界面中进行提示。

## 主循环中的延迟

这个是在调试的过程中更改最多的地方

起初我让检测输入的部分循环多次，后来发现因为消息函数的问题，其实这样做会花费大量的时间，导致等待时间非常的长，后来改为很少次数的输入判断，其实这样是够用的，因为键盘输入会保留在队列中等待处理。

早期测试中蛇非常难以操作，你必须进行预判才能正常游戏，显然这并不是正常的玩法。经过考量我才发现（真是太傻了），其实在每次蛇移动完之后再加一个延迟，给玩家一个观察的时间，这个问题就能很完美的解决了。

# 代码实现

我不打算提供编译好的程序了（毕竟程序本身和编译并不复杂）

您可以直接在我的仓库clone源代码或者直接复制下文进行编译即可

（甚至可以直接改编成单文件进行编译）

## LymoneSnake.cpp

主程序文件

```c++
#include <stdio.h>
#include <easyx.h>
#include "Snake.h"
#include <windows.h>

char map[20][20];
/*Wall-Head-Apple-Snake*/
ExMessage m;

char Nextstep;
/*Up-Down-Left-Right*/
int times = 4;
int delay = 100;

int snakelength=1;

void initMap()
{
    for (int xi = 0; xi < 20; xi++)
        for (int yi = 0; yi < 20; yi++)
        {
            map[xi][yi] = 0;
        }
    return;
}

//block 25*25
void testData();
void consolLoaded()
{
    #ifdef DEBUG
    printf("==ConsolLoaded==\n");
    #endif // DEBUG  
    return;
}

int main()
{
/*==初始化==*/
    #ifdef DEBUG
    initgraph(700, 505, EW_SHOWCONSOLE);
    consolLoaded();
    #else
    initgraph(700, 505);
    #endif // DEBUG

    LSD::drawNet();
    LSD::drawTitle();
    
    #ifdef DEBUG_TESTDATA
    testData();
    #endif // DEBUG_TESTDATA
    LSC::initrand();
    LSC::randhead();
    LSC::randapple();
    LSD::drawMap();

/*开始*/
    while (true)
    {
        //输入检定
        for (int i = 0; i < times; i++)
        {
            if (LSC::directionCheck())
            {
                return 0;
            }
            #ifdef DEBUG
            printf("%c", Nextstep);
            #endif // DEBUG
            Sleep(10);
        }
        #ifdef DEBUG
        printf("\n");
        #endif // DEBUG
        

        //死亡检定
        if (LSC::deathCheck())
        {
            LSD::drawdeath();
            m = getmessage();
            return 0;
        }
        else
        {
            //移动
            LSC::lengthCheck();
            LSD::drawScore(snakelength);
            //cleardevice();
            //LSD::drawNet();
            LSD::drawMap();
        }
        Sleep(delay*times);
    }//main while


    closegraph();//关闭绘图窗口
    return 0;

}

void testData()
{
    //testdata devonly
    printf("TestDataLoading\n");
    map[1][1] = 'W';
    map[2][2] = 'S';
    map[3][3] = 'W';
    map[4][4] = 'S';
    map[5][5] = 'W';
    map[6][6] = 'S';
    map[7][7] = 'A';
    map[8][8] = 'H';
    map[9][9] = 'A';
    map[10][10] = 'H';
}
```

## Snake.h

头文件，各种声明

```c++
//#define DEBUG
//#define DEBUG_TESTDATA

#ifndef SNAKEDRAW_H
#define SNAKEDRAW_H

namespace LSD
{
	/*棋盘绘制*/
	void drawNet();
	void drawMap();
	void cleanmap();
	/*文字绘制*/
	void drawTitle();
	void drawScore(int score);
	/*特殊画面*/
	void drawdeath();
}

#endif 


#ifndef SNAKECHECK_H
#define SNAKECHECK_H

namespace LSC
{
	/*Random*/
	void initrand(int);
	void initrand();
	void randapple();
	void randhead();
	/*Check*/
	void initwall();
	bool directionCheck();
	bool deathCheck();
	bool lengthCheck();

}

#endif


#ifndef SNAKEDATA_H
#define SNAKEDATA_H

class sNode
{
public:
	int x, y;
	sNode* snext;
};

class Linkqueue
{
	//链队列-逆
	//删尾加头 
public:
	void initQ();
	void enQ(sNode);
	sNode deQ();
	void destoryQ();
	void printQ();//devOnly 
	short countQ();
	bool pmapQ();
private:
	sNode* front;
	sNode* rear;
};

#endif
```



## SnakeCheck.cpp

各种判定的实现

```c++
//Check Frame
#include <cstdlib>
#include <stdio.h>
#include <EasyX.h>
#include <time.h>
#include "Snake.h"

extern char map[20][20];
extern ExMessage m;
extern char Nextstep;
sNode Head,Rear;
//extern int Head.x, Head.y;
extern int snakelength;
Linkqueue Q;

void LSC::initwall()
{
	//TODO: 墙体检入
}

void LSC::initrand(int seed)
{
	srand(seed);
	return;
}
void LSC::initrand()
{
	srand(time(0));
	return;
}

void LSC::randapple()
{
	short xi, yi;
	loop:
	xi = rand() % 20;
	yi = rand() % 20;
	if (map[xi][yi] == 0)
		map[xi][yi] = 'A';
	else
		goto loop;
	return;
}

void LSC::randhead()
{
	Q.initQ();
	short xi, yi;
loop:
	xi = rand() % 20;
	yi = rand() % 20;
	if (map[xi][yi] == 0)
	{
		map[xi][yi] = 'H';
		Head.x = xi;Head.y = yi;
		Q.enQ(Head);
	}
	else
		goto loop;
	return;
}

/*键盘输入*/
bool LSC::directionCheck()
{
	if (peekmessage(&m, EM_KEY))
	{
		switch (m.message)
		{
		case WM_KEYDOWN:
			if (m.vkcode == VK_UP)
			{
				Nextstep = 'U';
			}
			if (m.vkcode == VK_DOWN)
			{
				Nextstep = 'D';
			}
			if (m.vkcode == VK_LEFT)
			{
				Nextstep = 'L';
			}
			if (m.vkcode == VK_RIGHT)
			{
				Nextstep = 'R';
			}
			if (m.vkcode == VK_ESCAPE)
			{
				return true;
			}
			break;

		}
	}
	return false;

}

bool LSC::deathCheck()
{
	switch (Nextstep)
	{
	case 'U':
		if (Head.y - 1 < 0)
			return true;
		else if (map[Head.x][Head.y - 1] != 0&& map[Head.x][Head.y - 1] != 'A')
			return true;
		break;
	case 'D':
		if (Head.y + 1 >= 20)
			return true;
		else if (map[Head.x][Head.y + 1] != 0&& map[Head.x][Head.y + 1] != 'A')
			return true;
		break;
	case 'L':
		if (Head.x - 1 < 0)
			return true;
		else if (map[Head.x - 1][Head.y] != 0&& map[Head.x - 1][Head.y] != 'A')
			return true;
		break;
	case 'R':
		if (Head.x + 1 >= 20)
			return true;
		else if (map[Head.x + 1][Head.y] != 0&& map[Head.x + 1][Head.y] != 'A')
			return true;
		break;
	default:
		printf("Error:UndefinedDeathCheck\n");
	}
	return false;
}

bool LSC::lengthCheck()
{
	bool isA=false;
	//苹果检定并移动
	switch (Nextstep)
	{
	case 'U':
		if (map[Head.x][--Head.y] == 'A')
			isA = true;
		break;
	case 'D':
		if (map[Head.x][++Head.y] == 'A')
			isA = true;
		break;
	case 'L':
		if (map[--Head.x][Head.y] == 'A')
			isA = true;
		break;
	case 'R':
		if (map[++Head.x][Head.y] == 'A')
			isA = true;
		break;
	default:
		printf("Error:UndefinedLengthCheck\n");
	}
	if (isA)
	{
		snakelength++;
		LSC::randapple();
	}
	Q.enQ(Head);
		

	//长度检定
	int length = Q.countQ();
	if (length > snakelength)
	{
		Rear=Q.deQ();
		setfillcolor(0x000000);
		setlinecolor(0x000000);
		#ifdef DEBUG
		printf("%d,%d\n", Rear.x, Rear.y);
		#endif // DEBUG		
		fillcircle(25*Rear.x + 13, 25*Rear.y + 13, 11);
	}
	Q.pmapQ();
	return true;

}
```

## SnakeData.cpp

蛇体数据结构的实现

```c++
#include <stdio.h>
#include "Snake.h"

extern char map[20][20];

void Linkqueue::initQ()
{
	sNode* p = new sNode;
	front = p;
	rear = p;
	p->snext = NULL;
	return;
}
void Linkqueue::enQ(sNode node)
{
	sNode* p = new sNode;
	p->x = node.x;
	p->y = node.y;
	rear->snext = p;
	rear = p;
	p->x = node.x;
	p->y = node.y;
	return;
}
sNode Linkqueue::deQ()
{
	sNode rearNode;
	if (rear == front)
	{
		rearNode.x = -1;
		rearNode.y = -1;
	}
	sNode* p;
	p = front->snext;
	front->snext = p->snext;
	rearNode.x = p->x;
	rearNode.y = p->y;
	delete p;
	return rearNode;
}
void Linkqueue::destoryQ()
{
	while (rear != front)
		Linkqueue::deQ();
	delete front;
	return;
}
void Linkqueue::printQ()
{
	if (rear == front)
		return;
	sNode* p = front;
	do
	{
		p = p->snext;
		//std::cout << "X=" << p->x << "Y=" << p->y << std::endl;
	} while (p != rear);
	return;
}
short Linkqueue::countQ()
{
	if (rear == front)
		return 0;
	short num = 0;
	sNode* p = front;
	do
	{
		p = p->snext;
		++num;
	} while (p != rear);
	return num;
}
bool Linkqueue::pmapQ()
{
	if (rear == front)
		return false;
	LSD::cleanmap();
	sNode* p = front;
	do
	{
		p = p->snext;
		map[p->x][p->y] = 'S';
	} while (p != rear);
	map[p->x][p->y] = 'H';
	return true;
}
```

## SnakeDraw.cpp

绘制部分

```c++
//Draw Frame
#include <stdio.h>
#include <easyx.h>
#include "Snake.h"

extern char map[20][20];

void LSD::drawNet()
{
#ifdef DEBUG
    printf("PrintNet\n");
#endif // DEBUG


    setlinecolor(0xFFFFFF);
    for (int i = 0; i <= 20; i++)
        line(25 * i, 0, 25 * i, 500);
    for (int i = 0; i <= 20; i++)
        line(0, 25 * i, 500, 25 * i);
    return;

}

void LSD::drawMap()
{
#ifdef DEBUG
    printf("PrintMap\n");
#endif // DEBUG


    for (int xi = 0; xi < 20; xi++)
        for (int yi = 0; yi < 20; yi++)
        {
            short x = 25 * xi, y = 25 * yi;
            switch (map[xi][yi])
            {
            case 'w':
            case 'W':
                setfillcolor(0x2a2aa5);
                setlinecolor(0x2a2aa5);
                fillrectangle(x + 1, y + 1, x + 24, y + 24);
                break;
            case 'h':
            case 'H':
                setfillcolor(0x0000ff);
                setlinecolor(0x0000ff);
                fillcircle(x + 13, y + 13, 11);
                break;
            case 's':
            case 'S':
                setfillcolor(0xffffff);
                setlinecolor(0xffffff);
                fillcircle(x + 13, y + 13, 11);
                break;
            case 'a':
            case 'A':
                setfillcolor(0x6600aa);
                setlinecolor(0x00ff00);
                fillcircle(x + 13, y + 13, 11);
                break;
            }
        }
    return;

}
//cleardevice();//清空绘版
void LSD::cleanmap()
{
    for (int xi = 0; xi < 20; xi++)
        for (int yi = 0; yi < 20; yi++)
        {
            if (map[xi][yi] == 'S')
                map[xi][yi] = 0;
            if (map[xi][yi] == 'H')
                map[xi][yi] = 0;
        }
    return;
}

void LSD::drawdeath()
{
    settextcolor(0x0000FF);
    settextstyle(40, 40, L"Minecraft");
    printf("WASTED\n");
    RECT r = { 0, 0, 699, 504 };
    drawtext(_T("WASTED"), &r, DT_CENTER | DT_VCENTER | DT_SINGLELINE);
    

    return;

}
void LSD::drawTitle()
{
    settextcolor(0xFFFFFF);
    settextstyle(14, 14, L"Minecraft");
    RECT r = { 500, 10, 699, 30 };
    drawtext(_T("LymoneSnake"), &r, DT_CENTER | DT_VCENTER | DT_SINGLELINE);
    r = { 500, 30, 699, 50 };
    drawtext(_T("按↑↓←→开始"), &r, DT_CENTER | DT_VCENTER | DT_SINGLELINE);
    settextstyle(20, 20, L"Minecraft");
    r = { 500, 50, 699, 90 };
    drawtext(_T("当前分数："), &r, DT_CENTER | DT_VCENTER | DT_SINGLELINE);
    return;
}
void LSD::drawScore(int score)
{
    TCHAR s[5];
    _stprintf_s(s, _T("%d"), score);

    settextcolor(0xFFFFFF);
    settextstyle(30, 30, L"Minecraft");
    RECT r = { 500, 90, 699, 140 };
    drawtext(s, &r, DT_CENTER | DT_VCENTER | DT_SINGLELINE);
    return;

}
```

# 写在最后

其实这个东西在好久之前（大概半个月？）就已经实现了，在我的预想中，我想给它加入更好的UI，选关系统，内置地图，随机种子选择，Level选择，蛇的速度逐渐增加等等各种功能。

显而易见，到现在我并没有实现它。接下来可能会更忙，可能也没有时间去完成这件事了，所以说就先写完这篇搁置已久的博文吧。

如果有后续我可能会新开一篇博文，而不是对这篇博文进行重新编辑了。

不知道您是因为什么原因看到我这篇博文，虽然我确实写的很烂，但还是希望能对您产生一些帮助。
