---
title: SCNUCS-N 2020 Network
description: 
published: true
date: 2020-11-24T11:40:58.763Z
tags: 
editor: markdown
---

# A. Sum

题目要求从 $n$ 个数位中选取 $k$ 个数位并置为 $1$，显然一共有 $C_{n}^{k}$ 个满足条件的数。

然而直接求出所有满足条件的数过于麻烦。我们注意到对于任意一个数位，被置为 $1$ 的概率是相同的。具体而言就是各个数位都恰有 $C_{n-1}^{k-1}$ 种情况会被置为 $1$。那么我们考虑对这  $C_{n}^{k}$ 个数的各个数位分别进行求和后再相加。举个例子，对于样例 $1$：
$$
0011 \\
0101 \\
0110 \\
1001 \\
1010 \\
1100 \\
$$
每个数位都恰好有 $C_{4-1}^{2-1}=C_{3}^{1}=3$ 种情况被置为 $1$ 。我们对每一个数位上的数进行相加：

* 从右数起第 $1$ 位，$3 \times 2^0=3$；
* 从右数起第 $2$ 位，$3 \times 2^1=6$；
* 从右数起第 $3$ 位，$3 \times 2^2=12$；
* 从右数起第 $4$ 位，$3 \times 2^3=24$；
* **以此类推，从右数起第 $i$ 位，$C_{n-1}^{k-1} \times 2^{i-1}$**。

接下来，我们把各个数位求出来的结果相加，$3+6+12+24=45$，也就是样例的答案。

那么怎么用代码实现呢？最难实现的估计就是组合数了。聪明的同学已经开始开始使用百度搜索代码了，如果你顺利地找到了，直接套上就行了。如果你会**乘法逆元**的话当然是最好的，这里提供一种不需要乘法逆元的方法，但是**快速幂**（就是用很快的速度求出 $a^b$ 的值）和**取余运算的性质**是逃不掉要学的。

**在继续往下读之前，尝试借助网上资源完成 [洛谷 P1226](https://www.luogu.com.cn/problem/P1226)**，否则读起来就是一头雾水哦！

我们知道：
$$
C_{n}^{m}=\frac{n!}{m! \cdot (n-m)!}=\frac{n \times (n-1) \times (n-2) \times \cdots \times (n-m+1)}{m \times (m-1) \times \cdots \times 2 \times 1}
$$
然而在给定的数据范围下，未经约分的分子分母的值无法直接使用 long long 存储，而且非常显然地 $\frac{a}{b} \bmod p$ 并不总是等于 $\frac{a \bmod p}{b \bmod p}$。
> 举例：$a=21,b=7,p=5$。

但是我们知道组合数算出来最终的值一定是一个整数，所以我们可以先给分子分母进行约分。

> 可以尝试使用数学归纳法证明组合数一定是整数，思路是 $C_{n+1}^{k}=C_{n}^{k-1}+C_{n}^{k}$。

考虑对分子和分母进行质因数分解，也就是将分子分母都转换为下面的形式，其中 $p_i$ 是质数（有兴趣深入了解的同学可以百度搜索**算术基本定理**）。
$$
(p_1)^{cnt_1} \times (p_2)^{cnt_2} \times \cdots  \times(p_n)^{cnt_n}
$$
怎么分解质因数呢？事实上和如何判断一个数是不是质数是差不多的，使用试除法就可以了。

```cpp
int fac[60];

for (int i = 2; i * i <= x; i++)
{
    while (x % i == 0)
    {
        fac[i]++;
        x /= i;
    }
}
if (x != 1)
{
    fac[x]++;
}
```

质因数分解后，约分的实现就很简单了：在对分子做质因数分解就增加 $fac[i]$ 的值，在对分母做质因数分解就相应地减少 $fac[i]$ 的值。由于组合数算出来最终的值一定是一个整数，我们可以肯定数组 $fac$ 的每一项在最后都大于等于 $0$。最后计算组合数的值时，我们只需要遍历这个数组，计算出 $\prod i^{fac[i]}$ 的值就行了。举个例子：
$$
C_{6}^{3}=\frac{6 \times 5 \times 4}{3 \times 2 \times 1}=\frac{2^3\times 3^1 \times 5^1}{2^1 \times 3^1}=2^{3-1}\times 3^{1-1} \times 5^1=20
$$

* 在对分子做质因数分解之后，$fac[2]=3,fac[3]=1,fac[5]=1$；

* 在对分母做质因数分解之后，$fac[2]=3-1=2,fac[3]=1-1=0,fac[5]=1$。

计算 $\prod i^{fac[i]}$ 时候，我们就可以放心地一边做乘法一边进行取模了。下面的 $qpow()$ 就是自己实现的快速幂函数啦！


```cpp
long long c = 1;
for (int i = 2; i < 50; i++)
{
    c = c * qpow(i, fac[i]) % MOD;
}
```

得出了组合数的值之后，我们将各个数位求出来的结果相加，得到的就是答案啦！

```cpp
long long ans = 0;
for (int i = 0; i < n; i++)
{
    ans = (ans + qpow(2, i) * c % MOD) % MOD;
}
```

要注意的是，由于是多组测试数据，处理完每组数据后别忘了给 $fac$ 数组置零。



# B. 初中生的难题

给定两个数 $a,b$，怎么计算这两个数的异或值呢？

```cpp
int ans = a ^ b;
```

然而如果直接 $O(nm)$ 暴力计算的话，计算量过大是会超时的。

那么如何减少计算量呢？通常就是各种预处理。我们还是从样例入手：

我们计算了 $(1 \oplus 4) +(2 \oplus 4) + (3 \oplus 4) + \cdots$，我们在二进制角度下看看这些计算。
$$
\begin {array}{ccc}
&00001\\
\oplus&00100\\
\hline
=&00101
\end {array}
$$

$$
\begin {array}{ccc}
&00010\\
\oplus&00100\\
\hline
=&00110
\end {array}
$$

$$
\begin {array}{ccc}
&00011\\
\oplus&00100\\
\hline
=&00111
\end {array}
$$

$$
\begin {array}{ccc}
&00101\\
+&00110\\
+&00111\\
\hline
=&10010
\end {array}
$$

前 $3$ 个竖式分别在计算 $(1 \oplus 4), (2 \oplus 4)$ 和 $(3 \oplus 4)$ 的值。第 $4$ 个竖式在把前 $3$ 个竖式计算得到的值相加。我们观察一下第 $4$ 个竖式的三个被加数。
$$
00101 \\
00110 \\
00111
$$

* 从右数起第 $1$ 位，一共有 $2$ 个 $1$；
* 从右数起第 $2$ 位，一共有 $2$ 个 $1$；
* 从右数起第 $3$ 位，一共有 $3$ 个 $1$；

不难想到这些 $1$ 都是前 $3$ 个竖式中来的。

* 对于右数起第 $1$ 位，竖式 $1$ 贡献了一个 $1$，竖式 $2$ 贡献了一个 $1$。
* 对于右数起第 $2$ 位，竖式 $2$ 贡献了一个 $1$，竖式 $3$ 贡献了一个 $1$。
* 对于右数起第 $3$ 位，竖式 $1,2,3$ 各献了一个 $1$。

为什么这些竖式会得以贡献 $1$ 呢？这就得回到异或的计算法则了。

异或是一个位运算，每个数位的计算相互独立，不会有什么进位和退位。对两个数 $a,b$ 作异或运算，那么对于任意一个数位，$0  \oplus 0 = 0, 1  \oplus 0 = 1, 0  \oplus 1 = 1,1  \oplus 1 = 0$。正如异或的名字所述，对于某一个数位，计算结果为 $1$ 当且仅当 $a$ 的这个数位和 $b$ 的这个数位恰好只有一个是 $1$。

回到题目，我们要计算 $A[i]\oplus B[j]$，对于任意一个 $B[j]$，不妨对它的每一个数位作以下的分类讨论：

* 假设这个数位是 $1$，要想这个数位为答案贡献 $1$，$A[i]$ 的这个数位必须是 $0$。如果对于数组 $A$ 的 $n$ 个数，如果有 $x$ 个数的这个数位为 $0$，那么数组 $A$ 所有数和 $B[j]$ 异或时这个数位就会贡献 $x$ 个 $1$；

* 假设这个数位是 $0$，同理，如果对于数组 $A$ 的 $n$ 个数，如果有 $y$ 个数的这个数位为 $1$，那么数组 $A$ 所有数和 $B[j]$ 异或时这个数位就会贡献 $y$ 个 $1$。

那么接下来的任务就很简单了，我们对数组 $A$ 进行预处理，即对于每个数位，统计一共有多少个数这个数位是 $1$，统计一共有多少个数这个数位是 $0$。接下来我们开始遍历数组 $B$，对于数组 $B$ 的每一项 $B[j]$，我们作上面的分类讨论，然后我们就像 $A$ 题一样，把各个数位求出来的结果相加，得到的就是 $\sum_{i=0}^{n}(A[i] \oplus B[j])$ 的值，把对于每个 $B[j]$ 算出来的值再给加起来就是答案啦！在求和的时候别忘了一边求和一边对 $10^9+7$ 取模哦！