---
title: 面向Unity的C#学习记录
categories:
  - 编程
tags:
  - C#
  - Unity
abbrlink: ddf9c66f
date: 2023-01-07 19:25:37
---

​	最近又把戴森球计划(Dyson Sphere Program)下回来了，并且发现有个叫创世之书模组挺有意思的。了解到现在模组作者都苦于游戏的数字ID，决定尝试一下能不能解决这个问题。解决问题第一步，简单学一下C#。

<!--more-->

# 一、C#基础

## 1.我遇到的一些没见过东西

### 特性（Attribute）

**特性（Attribute）**是用于在运行时传递程序中各种元素（比如类、方法、结构、枚举、组件等）的行为信息的声明性标签。您可以通过使用特性向程序添加声明性信息。一个声明性标签是通过放置在它所应用的元素前面的方括号（[ ]）来描述的。

特性（Attribute）用于添加元数据，如编译器指令和注释、描述、方法、类等其他信息。.Net 框架提供了两种类型的特性：*预定义*特性和*自定义*特性。
