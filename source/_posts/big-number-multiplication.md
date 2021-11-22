---
title: 大数模乘
catalog: true
date: 2021-06-13 14:26:50
subtitle: Big Number Modular Multiplication
header-img: "/img/article_header/crypto-bg.png"
tags:
- cryptography
- big number
toc_nav_num: true
---

# 简介

大数模乘是密码学中最基础几种操作之一。考虑$\mathbb{Z}_p$（$p$为256位）上两个元素$a$和$b$的模$p$乘法，最简单的实现思路为：

1. 计算`a*b`，结果长度为512位
2. 执行一个模运算（512位模256位，“除-乘-减”循环）

这样做的效率自然是非常差的，所以实际实现上通常会采用Montgomery来优化**连续**乘法的性能（单次乘法就不需要了）。连续乘法在椭圆曲线运算和模$p$求幂上的使用是非常广泛的，而Montgomery模乘给这些操作带来的性能提升也是显著的。而且Montgomery模乘本身是保持线性的，即数据只要不进行求逆运算，就可以一直以Montgomery模乘所需要的形式进行运算，直到需要计算原值的时候再进行转换。

Montgomery模乘用到以下事实：**对于某些模数而言，求模运算是不需要循环的。**这些模数包括2的幂和（广义）梅森素数（形式为$p = \sum_{i=1}^n a_i\times 2^{b_i}(a_i \in \{1,-1\}, b \in [0, 255])$）。密码规范在选择素数参数的时候一般会选择广义梅森素数来降低运算复杂度，但在椭圆曲线领域，数域参数$p$和椭圆曲线群的阶$n$恰好都是广义梅森素数，而且项数不多的概率就不很高了。所以NIST给出的椭圆曲线和国密SM2使用的椭圆曲线中，都指定$p$为某个梅森素数，但$n$就只是一个一般的大素数了。所以在计算$\mathbb{Z}_n$上的模乘时，就会使用到Montgomery模乘将乘法转换到模$m$的域上。

# 定义及性质

令$m'=m^{-1}\mod n, n'=(mm'-1)/n$，定义$f:\mathbb{Z}_n\rightarrow \mathbb{Z}_n$为$f(a)=a\cdot m \mod n$，定义逆映射$f':\mathbb{Z}_n \rightarrow \mathbb{Z}_n$为$f'(a') = a' \cdot m' \mod n$。 定义Montgomery模乘运算$*$:

$$
a' * b' \mod n = a' \cdot b' \cdot m' \mod n
$$

这里看起来有一个模$n$的操作，但其实是不需要进行模$n$运算的。因为注意到
$$
\begin{aligned}
a'\cdot b' \cdot m' \mod n& = \frac{a'b'm'm}{m} \mod n\\
	& = \frac{a'b'(nn'+1)}{m} \mod n \\
	& = \frac{a'b'(nn'+1) \mod mn}{m} \mod n\\
	& = \frac{a'b'\mod mn + a'b'nn' \mod mn}{m} \mod n \\
	& = \frac{a'b' + (a'b'n'\mod m)\cdot n}{m} \mod n
\end{aligned}
$$
看一下最后结果模$n$之前的范围。$a'b'\in [0,(n-1)^2], (a'b'n' \mod m)\cdot n \in [0,(m-1)n]$，令模$n$之前的结果为$R$，则$R \geq 0$，且$R \leq \left((n-1)^2+(m-1)n\right)/m < (2mn)/m = 2n$。所以最终结果最多会比实际所需的结果大一个$n$，因而最终的乘法定义为
$$
a'*b' = \left\{\begin{array}{ll}
\frac{a'b'+(a'b'n'\mod m)\cdot n}{m}&, \text{does not exceeds $n$}\\
\frac{a'b'+(a'b'n'\mod m)\cdot n}{m}-n&, \text{otherwise}
\end{array}\right.
$$

- 乘法同态：$f(a)*f(b)=f(a\cdot b \mod n)$
- 加法同态：$f(a)+f(b)\mod n= f(a+b \mod n)$ 

考虑一组数$a_1, a_2, \cdots, a_k \in \mathbb{Z}_n$，要计算

$$r=F(a_1, a_2, \cdots, a_k)$$

$F$只涉及模$n$的加和乘运算，可以先将这组数映射到$a_1' = f(a_1), a_2' = f(a_2), \cdots, a_k' = f(a_k) \in \mathbb{Z}_p$上，然后计算

$$r' = F'(a_1', a_2', \cdots, a_k')$$

$F'$与$F$对应，只是乘法使用$*$进行替换，最终计算$r = f'(r')$。

# 应用场景

可以看出两个映射都要执行一个模$n$运算，所以单次模乘使用Montgomery模乘是会代入额外负载的，但是连续模乘使用Montgomery就会大幅提升效率。

# 参考文献
[1] Montgomery P L. Modular multiplication without trial division[J]. Mathematics of computation, 1985, 44(170): 519-521.