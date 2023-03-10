---
title: c++进阶学习
categories:
  - 编程
tags:
  - C++
abbrlink: 2b8bac97
date: 2022-07-16 10:13:57
---

之前的C++学习都是浅尝辄止，这一次直接购入《C++ Primer Plus》系统深入的学习一些C++的知识，尤其C++擅长的面向对象部分。

引用表在文章末尾

<!--more-->

# union

union即为联合，它是一种特殊的类。通过关键字union进行定义，一个union可以有多个数据成员。
在任意时刻，联合中只能有一个数据成员可以有值。当给联合中某个成员赋值之后，该联合中的其它成员就变成未定义状态了。
在C/C++程序的编写中，当多个基本数据类型或复合[数据结构](https://so.csdn.net/so/search?q=数据结构&spm=1001.2101.3001.7020)要占用同一片内存时，我们要使用联合体；当多种类型，多个对象，多个事物只取其一时（我们姑且通俗地称其为“n 选1”），我们也可以使用联合体来发挥其长处。
union主要是共享[内存](https://so.csdn.net/so/search?q=内存&spm=1001.2101.3001.7020)，分配内存以其最大的结构或对象为大小，即sizeof最大的。

```cpp
union myun 
{
struct { int x; int y; int z; }u; 
int k; 
}a; 
int main() 
{ 
a.u.x =4;
a.u.y =5; 
a.u.z =6; 
a.k = 0; 
printf("%d %d %d\n",a.u.x,a.u.y,a.u.z);
return 0;
}
1234567891011121314
```

由于union类型是共享内存，以size最大的结构作为自己的大小，这样的话，myun这个结构就包含u这个结构体，而大小也等于u这个结构体 的大小，在内存中的排列为声明的顺序x,y,z从低到高，然后赋值的时候，在内存中，就是x的位置放置4，y的位置放置5，z的位置放置6，现在对k赋 值，对k的赋值因为是union，要共享内存，所以从union的首地址开始放置，首地址开始的位置其实是x的位置，这样原来内存中x的位置就被k所赋的值代替了，就变为0了，这个时候要进行打印，就直接看内存里就行了，x的位置也就是k的位置是0，而y，z的位置的值没有改变，所以应该是0,5,6。

EasyX的ExMessage的定义就涉及到了union的使用，有兴趣可以看看我写的EasyX的博客。

# 引用及致谢

union部分：[CSDN@W Y](https://blog.csdn.net/qq_45611002/article/details/119675452)