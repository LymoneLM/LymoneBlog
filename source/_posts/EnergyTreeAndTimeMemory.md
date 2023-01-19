---
title: Tokitsukaze and Energy Tree谈变量大小与时间复杂度
categories:
  - 算法
tags:
  - C++
  - ACM
mathjax: true
abbrlink: bf5a535
date: 2023-01-18 20:10:25
---

链接：https://ac.nowcoder.com/acm/contest/46810/D
来源：牛客网

Tokitsukaze 有 nnn 个节点的有根能量树，根为 111。最开始，树上每个节点的能量都是 000。

<!--more-->

# 正文

时间限制：C/C++ 1秒，其他语言2秒
空间限制：C/C++ 262144K，其他语言524288K
64bit IO Format: %lld
        

## 题目描述                    

Tokitsukaze 有 $n$ 个节点的有根能量树，根为 $1$。最开始，树上每个节点的能量都是 $0$。 

 现在 Tokitsukaze 有 $n$ 个能量球，第 $i$ 个能量球拥有 $v_i$ 能量。她想把这 $n$ 个能量球分别放置在能量树的每个节点上，使能量树的每个节点都恰好有一个能量球。

 Tokitsukaze 每次只能放置一个能量球，所以她将进行 $n$ 次操作。每一次操作，她会选择一个能量球，再选择一个没有能量球的能量树节点 $x$，把刚刚选择的能量球放置在节点 $x$ 上。在这之后，Tokitsukaze 能获得以 $x$ 为根的子树中的所有能量球的能量 (包括节点 $x$ 的能量球能量)。

 在放置完所有能量球后，Tokitsukaze 可能获得的总能量最多是多少？

## 输入描述:


第一行包含一个整数 $n (1≤n≤2⋅10^5)$。

第二行包含 $n-1$ 个整数 $f_2,f_3,…,f_n (1≤f_i≤i−1)$，表示节点 $i$ 的父亲是 $f_i$。

第三行包含 $n$ 个整数 $v_1,v_2,…,v_n (1≤v_i≤10^5)$，分别表示能量球的能量。


## 输出描述:


输出一个整数，表示 Tokitsukaze 可能获得的最多总能量。              

## 输入

>5
>1 1 3 3
>1 1 2 2 3


## 输出

>22

# 代码

```cpp
#include <bits/stdc++.h>

using namespace std;
using ll = long long;
int dt[200010], mul[200010];

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, temp;
    ll all = 0;

    cin >> n;
    mul[1] = 1;
    for (int i = 2; i <= n; i++) {
        cin >> temp;
        mul[i] = mul[temp] + 1;
    }
    for (int i = 1; i <= n; i++) {
        cin >> dt[i];
    }
    sort(dt + 1, dt + n + 1, greater<ll>());
    sort(mul + 1, mul + n + 1, greater<ll>());
    for (int i = 1; i <= n; i++) {
        all += ll(mul[i]) * dt[i]; // 2e5*1e5>2e10
    }
    cout << all << endl;

    return 0;
}
```

# Others

## 故事从26行开始

​	代码26行`all += ll(mul[i]) * dt[i];`相当奇怪，这是我wa了三次的原因，mul[i]最大取值应该是$2\times10^5$,dt[i]的最大取值应该是$10^5$，理论上最大取值应该是$2\times10^{10}$,刚好不超过int的范围，但是这题大概是出题人搞事情的吧，总之这里如果不使得计算结果为`long long`会导致**WA**。

​	联想到最近一次又一次的**TLE**,我感觉是时候正视题目上方的时间空间限制了。

### 变量范围

​	ok，第一步，先搞懂基本的变量范围。

![变量范围](https://s1.ax1x.com/2023/01/18/pS3vrkD.png)

#### 能开多少

268,435,456B=262144KB=256MB

B就是Byte，也就是所谓的字节

1Byte为8bit，也可以通过这个简单的估算整型的大小

可以简单估算得到下表

| 类型      | 大约能开多少    | 最大值           |
| --------- | --------------- | ---------------- |
| (u)char/bool | $2.5\times10^9(实际10^8)$ | (255)127/1 |
| int       | $6.7\times10^8(实际10^7)$ | $2\times10^{10}$ |
| long long | $3.3\times10^8(实际10^7)$      | $9\times10^{19}$ |

**以上数据以256M为准**

### 时间限制

这张图选自`AOP`的小群，原出处不明

![常见时间限制对应时间复杂度呃算法](https://s1.ax1x.com/2023/01/18/pS3vste.jpg)

希望2023年我能看懂这张图
