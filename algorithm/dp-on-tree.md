---
title: Dynamic Programming on Tree Structures (Luogu P2014)
description: 
published: true
date: 2020-08-01T02:44:23.057Z
tags: algorithm, luogu, dp, solution, tree
editor: markdown
---

## 题目
https://www.luogu.com.cn/problem/P2014

## 思路

> 以下内容**仅为个人理解，可能存在错误**。

### 树的孩子兄弟表示法
![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/p2014-2.gif)

想要使用孩子兄弟表示法储存上面这样的一棵树，我们就要从根节点开始，依次用链表存储各个节点的孩子节点和兄弟节点。因此，该链表中的节点应包含以下三部分内容：

* 节点的值。
* 指向孩子节点的指针。
* 指向兄弟节点的指针。

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/p2014-1.gif)

在这里，每个节点是一门课程，孩子节点其实就是后续课程，兄弟节点其实就是前导课程相同的一门课程。不妨将第 i 门课程编号为 i，那么每一门课都会有一个独一无二的课程编号。这个课程编号本应放在数据域，但是考虑到它的独一无二性和连续性，也方便我们查找某一课程的「后续课程」和「前导课程相同的一门课程」的课程编号，以获取它的学分，我们不妨直接用数组下标来储存这个，就不开数据域了。

### 树形动归

> **TODO**：此部分自认为没有理解透彻，下面给出的思路也不够自然。

考虑之前发过的几道题，LXY 点菜，我们逐道菜过目；上山采药，我们逐件药品过目。这里我们照葫芦画瓢，就逐门课过目。树形动归和线性动归又有些啥不同呢？首先要遵循从最简单的子问题开始，逐步扩大子问题规模的原则，从子树上动归，最后进行合并。如何体现合并这一操作呢？就是在过目某一节点（课程）的时候同时考虑这个节点（课程）的后续课程。

不妨就将根节点（假设其编号为 now）的所有孩子过目完，假设现在过目了 a 门课程，选择了 b 门课程。对于这个第 a 门课程依然是两个选择，**选和不选**。不选的话和我过目 a - 1 门课程，选择了 b 门课程效果是一样的，而第 a 门课程我不选自然就拿不了学分，其后续课程我也选不了就无需考虑了。但如果我选择呢，我不但选了第 a 门课程，而且还要考虑它的后续课程。假设已知我过目完 a - 1 门课程时，选择了 b - c 门课程。过目第 a 门课程显然我就考虑了第 a 门课的后续课程并选择了 c 门课程（包含 a）。此时为了求出过目了 a 门课程，选择了 b 门课程的最大学分，我就要求出过目 a - 1 门课程，选择了 b - c 门课程的最大学分和考虑了第 a 门课的**所有**后续课程并选择了 c 门课程（包含 a）的最大学分。而由于我们从子树开始动归，所以后者我们是已经求出过的。考虑完选和不选两种情况后，取最大值即可。

假设 `son` 是 `now` 的一个子节点，有：

```cpp
f[now][i][j] = max(f[now][i - 1][j], f[son][所有节点数][k] + f[now][i - 1][j - k]);
```

和线性动态规划一样，由于过目了 i 门课程的情况永远由过目 i - 1 门课程的情况推出来，于是这一维可以被去掉。

```cpp
f[now][j] = max(f[now][j], f[son][k] + f[now][j - k]);
```

## AC 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
//n、m 如题所述，f[i][j] 表示 i 为子树的根节点，在子树中选 j 门课的最大学分
int n, m, f[1000][1000], fa;
struct node //用结构体储存节点信息
{
    int pre, to; //pre 为兄弟节点在数组 e 中的下标，to 为孩子节点在数组 e 中的下标
} e[1000];       //考虑到课程编号的唯一性，这题 e 数组下标可用来表示课程编号，我们也不用开数据域了

void dp(int now) //在「根节点编号为 now 的树」中动归
{
    //遍历 now 的所有孩子节点，也就是先访问 now 的「第一个」儿子节点，然后再访问这个儿子节点的兄弟节点
    for (int i = e[now].to; i != 0; i = e[i].pre)
    {
        dp(i);                           //在「根节点编号为 i 的树」中动归
        for (int j = m + 1; j >= 1; j--) //关于为什么要反向遍历，前面的文章有提及
        {
            for (int k = 0; k < j; k++)
                f[now][j] = max(f[now][j], f[i][k] + f[now][j - k]); //状态转移方程
        }
    }
}
int main()
{
    ios::sync_with_stdio(false); //是输入输出挂
    cin.tie(0);                  //还是输入输出挂
    cout.tie(0);                 //依然是输入输出挂
    cin >> n >> m;
    //我们令 i 为节点（课程）的编号
    //不妨假定存在一门编号为 0 的课，是所有课程的先修课程且学分为 0
    //这样子我们就可以将所有的课程标在一棵树上
    for (int i = 1; i <= n; i++) //接下来我们从 i = 1 开始读入 n 个节点（课程）
    {
        cin >> fa;           //fa 是节点 i 的父节点（先修课程）的编号
        cin >> f[i][1];      //显然 j == 1 时最大学分就是编号为 i 的这门课本身的学分
        e[i].pre = e[fa].to; //指明节点 i 的兄弟节点是节点 fa 的「第一个」儿子节点
        e[fa].to = i;        //更新节点 fa 的「第一个」儿子节点为节点 i
    }
    dp(0); //从编号为 0 的根节点开始动归
    //输出 0 为根节点，选 m + 1 门课的最大学分
    //m + 1 门课是因为我们无中生有了一门「所有课程的先修课」，也就是编号为 0 的这门课
    cout << f[0][m + 1] << endl;
    return 0;
}

```