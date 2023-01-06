---
title: C++泛型编程学习
date: 2022-09-30 11:33:17
categories:
- 编程
tags: 
- C++
- 泛型编程
- STL
---

看学长用STL，馋疯了。

22.12.1 我又来继续学了

<!--more-->

# 预备知识：模板

为了提高代码的重用性，C++中引入了泛型编程的概念，泛型编程的基础就是模板。

## 模板函数

### 基本语法

```c++
template<typename T>
void func( T a)
{
    return;
}
```

其中 T可任意更换，typename 可换为 class

在调用的时候，必须给模板函数确定的数据类型

分为自动推断和明确给出两种

```C++
//自动推断
int a;
func(a);
//给出
func<int>(a);
```

如此便实现了将数据类型作为函数的参数

### 例程

```C++
#include <bits/stdc++.h>
using namespace std;

template<typename T>
void mysort(T arry[],int len)
{
	for(int xi=0; xi<len; xi++)
	{
		for(int yi=0; yi<len; yi++)
		{
			if(arry[xi]>arry[yi])
			{
				T c;
				c=arry[xi],arry[xi]=arry[yi],arry[yi]=c;
			}
		}
	}
	for(int xi=0; xi<len; xi++)
		cout<<arry[xi]<<" ";
	return;
}

void test()
{
	int arryint[]= {5,3,7,8,2,9,0};
	char arrychar[]="ILOVEYOU";
	int len1=sizeof(arryint)/sizeof(int)
	         ,len2=sizeof(arrychar)/sizeof(char);
	mysort(arryint,len1);
	mysort(arrychar,len2);
	return;
}

int main()
{
	test();
	return 0;
}
```



### 注意事项

#### 关于调用函数时参数发生的类型转换

- 调用函数模板，使用自动推导时，不会发生隐式类型转换

- 调用函数模板，显式指定类型，可以发生隐式类型转换

#### 关于函数重载的调用规则

1. 如果函数模板和普通函数都可以实现，优先调用普通函数

2. 可以通过空模板参数列表来强制调用函数模板

   ```C++
   func<>(a);
   ```

3. 函数模板可以发生重载

4. 如果函数模板可以产生更好的匹配，优先调用函数模板



# 容器

## 一、vector







# 参考链接

[C++ STL总结 | 行码棋 (wyqz.top)](https://wyqz.top/p/870124582.html)

