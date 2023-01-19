---
title: '7-10 排座位（25分）[并查集算法]'
categories:
  - 算法
tags:
  - C++
  - PTA
  - 数据结构
abbrlink: b67676c7
date: 2022-10-03 10:00:25
---

置宴席最微妙的事情，就是给前来参宴的各位宾客安排座位。无论如何，总不能把两个死对头排到同一张宴会桌旁！这个艰巨任务现在就交给你，对任何一对客人，请编写程序告诉主人他们是否能被安排同席。

<!--more-->

### 输入格式：

输入第一行给出3个正整数：`N`（≤100），即前来参宴的宾客总人数，则这些人从1到`N`编号；`M`为已知两两宾客之间的关系数；`K`为查询的条数。随后`M`行，每行给出一对宾客之间的关系，格式为：`宾客1 宾客2 关系`，其中`关系`为1表示是朋友，-1表示是死对头。注意两个人不可能既是朋友又是敌人。最后`K`行，每行给出一对需要查询的宾客编号。

这里假设朋友的朋友也是朋友。但敌人的敌人并不一定就是朋友，朋友的敌人也不一定是敌人。只有单纯直接的敌对关系才是绝对不能同席的。

### 输出格式：

对每个查询输出一行结果：如果两位宾客之间是朋友，且没有敌对关系，则输出`No problem`；如果他们之间并不是朋友，但也不敌对，则输出`OK`；如果他们之间有敌对，然而也有共同的朋友，则输出`OK but...`；如果他们之间只有敌对关系，则输出`No way`。

### 输入样例：

```in
7 8 4
5 6 1
2 7 -1
1 3 1
3 4 1
6 7 -1
1 2 1
1 4 1
2 3 -1
3 4
5 7
2 3
7 2
```

### 输出样例：

```out
No problem
OK
OK but...
No way
```

代码长度限制

16 KB

时间限制

200 ms

内存限制

64 MB

## 解题

### 错误示范

22/25分代码如下

思路为暴力搜索

```C++
#include <bits/stdc++.h>
using namespace std;
int main()
{
	int _friend[101][101];
	int _energy[101][101];
	for (int i = 1; i <= 100; i++)
	{
		_friend[i][0] = 0;
		_energy[i][0] = 0;
	}
	int N, M, K;
	cin >> N >> M >> K;
	int p1, p2, rope;
	for (int i = 0; i < M; i++)
	{
		cin >> p1 >> p2 >> rope;
		if (rope == 1)
		{
			_friend[p1][0]++;
			_friend[p1][_friend[p1][0]] = p2;
			_friend[p2][0]++;
			_friend[p2][_friend[p2][0]] = p1;
		}
		if (rope == -1)
		{
			_energy[p1][0]++;
			_energy[p1][_energy[p1][0]] = p2;
			_energy[p2][0]++;
			_energy[p2][_energy[p2][0]] = p1;
		}
	}
	for (int i = 0; i < K; i++)
	{
		bool is_friend = false, is_energy = false;
		cin >> p1 >> p2;
		for (int xi = 1; xi <= _friend[p1][0]; xi++)
		{
			if (_friend[p1][xi] == p2 || is_friend)
			{
				is_friend = true;
				break;
			}
			for (int yi = 1; yi <= _friend[p2][0]; yi++)
			{
				if (_friend[p1][xi] == _friend[p2][yi])
				{
					is_friend = true;
					break;
				}
			}
		}
		for (int xi = 1; xi <= _energy[p1][0]; xi++)
		{
			if (_energy[p1][xi] == p2)
			{
				is_energy = true;
				break;
			}
		}
		for (int xi = 1; xi <= _energy[p2][0]; xi++)
		{
			if (_energy[p2][xi] == p1)
			{
				is_energy = true;
				break;
			}
		}
		// 10 00 11 01
		if (is_friend)
		{
			if (is_energy)
				cout << "OK but..." << endl;
			else
				cout << "No problem" << endl;
		}
		else
		{
			if (is_energy)
				cout << "No way" << endl;
			else
				cout << "OK" << endl;
		}
	}

	return 0;
}

```

这个代码始终有一个数据过不去，原因不明。

### 正确解题

这个题目的解答要用到并查集算法

代码如下，后文有并查集补充

```c++
#include <bits/stdc++.h>
using namespace std;
class DisjointSet
{
    public:
    int father[101];
    //int rank[101];
    DisjointSet(int N)
    {
        for(int i=1;i<=N;i++)
        {
            father[i]=i;
            //rank[i]=1;
        }
    }
    int find(int x)
    {
        return father[x]==x ? x : (father[x]=find(father[x]));
    }
    void merge(int x,int y)
    {
        father[y]=find(x);
    }

};
int main()
{
    int N;cin>>N;
    DisjointSet T(N);
    bool energy[101][101]={{}};

    int M;cin>>M;
    int K;cin>>K;
    int temp1,temp2,rope;
    while(M--)
    {
        cin>>temp1>>temp2>>rope;
        if(rope==1)
            T.merge(temp1,temp2);
        else if(rope==-1)
        {
            energy[temp1][temp2]=true;
            energy[temp2][temp1]=true;
        }
    }
    while(K--)
    {
        bool is_friend=false,is_energy=false;
        cin>>temp1>>temp2;
        if(T.find(temp1)==T.find(temp2))
            is_friend=true;
        is_energy=energy[temp1][temp2];
        if(is_friend)
		{
			if(is_energy)
				cout<<"OK but..."<<endl;
			else
				cout<<"No problem"<<endl;
		}
		else
		{
			if(is_energy)
				cout<<"No way"<<endl;
			else
				cout<<"OK"<<endl;
		}
    }
    return 0;
}
```

本代码**并未**用到**按秩排序**，数据量小为原因之一，之二则是因为本人认为按秩排序并不符合逻辑，在代码的运行过程中由于**路径压缩**的存在，**秩**会变得非常混乱，完全没必要使用了，可能在足够大的数据量中才有一定的用武之地。

## 并查集（Disjoint Set）

并查集被很多OIer认为是最简洁而优雅的数据结构之一，主要用于解决一些**元素分组**的问题。它管理一系列**不相交的集合**，并支持两种操作：

- **合并**（Union）：把两个不相交的集合合并为一个集合。
- **查询**（Find）：查询两个元素是否在同一个集合中。

##### 一、概念及其介绍

并查集是一种树型的数据结构，用于处理一些不相交集合的合并及查询问题。

并查集的思想是用一个数组表示了整片森林（parent），树的根节点唯一标识了一个集合，我们只要找到了某个元素的的树根，就能确定它在哪个集合里。

##### 二、适用说明

并查集用在一些有 **N** 个元素的集合应用问题中，我们通常是在开始时让每个元素构成一个单元素的集合，然后按一定顺序将属于同一组的元素所在的集合合并，其间要反复查找一个元素在哪个集合中。这个过程看似并不复杂，但数据量极大，若用其他的数据结构来描述的话，往往在空间上过大，计算机无法承受，也无法在短时间内计算出结果，所以只能用并查集来处理。

##### 简单实现

###### 初始化

```c
int fa[MAXN];
inline void init(int n)
{
    for (int i = 1; i <= n; ++i)
        fa[i] = i;
}
```

假如有编号为1, 2, 3, ..., n的n个元素，我们用一个数组fa[]来存储每个元素的父节点（因为每个元素有且只有一个父节点，所以这是可行的）。一开始，我们先将它们的父节点设为自己。

###### 查询

```c
int find(int x)
{
    if(fa[x] == x)
        return x;
    else
        return find(fa[x]);
}
```

我们用递归的写法实现对代表元素的查询：一层一层访问父节点，直至根节点（根节点的标志就是父节点是本身）。要判断两个元素是否属于同一个集合，只需要看它们的根节点是否相同即可。

###### 合并

```c
inline void merge(int i, int j)
{
    fa[find(i)] = find(j);
}
```

合并操作也是很简单的，先找到两个集合的代表元素，然后将前者的父节点设为后者即可。当然也可以将后者的父节点设为前者，这里暂时不重要。本文末尾会给出一个更合理的比较方法。

##### 优化实现

显而易见，这种简单的方法几乎不对树进行维护，极有可能造成树变成链，复杂度直线上升的惨剧。那么如何进行维护呢？既然我们在进行查询的时候会进行递归，不如在这个递归的过程中直接把沿途节点的父节点都设置为根节点。

###### 合并（路径压缩）

```c
int find(int x)
{
    if(x == fa[x])
        return x;
    else{
        fa[x] = find(fa[x]);  //父节点设为根节点
        return fa[x];         //返回父节点
    }
}
```

以上代码常常简写为一行：

```c
int find(int x)
{
    return x == fa[x] ? x : (fa[x] = find(fa[x]));
}
```

注意赋值运算符=的优先级没有三元运算符?:高，这里要加括号。

路径压缩优化后，并查集的时间复杂度已经比较低了，绝大多数不相交集合的合并查询问题都能够解决。然而，对于某些时间卡得很紧的题目，我们还可以进一步优化。

###### 按秩合并

有些人可能有一个误解，以为路径压缩优化后，并查集始终都是一个**菊花图**（只有两层的树的俗称）。但其实，由于路径压缩只在查询时进行，也只压缩一条路径，所以并查集最终的结构仍然可能是比较复杂的。例如，现在我们有一棵较复杂的树需要与一个单元素的集合合并：

我们应该把简单的树往复杂的树上合并，而不是相反。因为这样合并后，到根节点距离变长的节点个数比较少。

我们用一个数组rank[]记录每个根节点对应的树的深度（如果不是根节点，其rank相当于以它作为根节点的**子树**的深度）。一开始，把所有元素的rank（**秩**）设为1。合并时比较两个根节点，把rank较小者往较大者上合并。

路径压缩和按秩合并如果一起使用，时间复杂度接近 O(n) ，但是很可能会破坏rank的准确性。

**初始化（按秩合并）**

```c
inline void init(int n)
{
    for (int i = 1; i <= n; ++i)
    {
        fa[i] = i;
        rank[i] = 1;
    }
}
```

**合并（按秩合并）**

```c
inline void merge(int i, int j)
{
    int x = find(i), y = find(j);    //先找到两个根节点
    if (rank[x] <= rank[y])
        fa[x] = y;
    else
        fa[y] = x;
    if (rank[x] == rank[y] && x != y)
        rank[y]++;                   //如果深度相同且根节点不同，则新的根节点的深度+1
}
```
