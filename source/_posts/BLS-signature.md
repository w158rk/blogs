---
title: BLS签名
catalog: true
header-img: "/img/article_header/crypto-bg.png"
tags:
- cryptography
- signature
subtitle: BLS-signature
toc_nav_num: true
date: 2021-06-29 19:55:01
---

BLS签名是Dan Boneh, Ben Lynn, Hovav Shacham在2001年提出的基于双线性对的签名方案。其中Boneh和Shacham相对比较著名。Boneh和Franklin首次实现了标识加密，开创了后来IBE、ABE等一系列技术的先河。Shacham第一次提出了ROP攻击，大致可以算是现在广泛被研究的各类恶意代码攻击和相关防御技术的开端。

这个方案是非常简单的，但是基于双线性对的签名方案通常都不是非常好证明。

## 方案

- 系统参数：双线性对：$e: \mathbb{G}\times\mathbb{G} \rightarrow \mathbb{G}_T$.群是$p$阶的，生成元$g$。
- 密钥：私钥$x \in \mathbb{G}_p$, 公钥$v = g^x$。
- 签名：$h = H(M), \sigma=h^x$
- 验证：验证$e(v, h) = e(\sigma, g)$

## 方案证明

方案看起来很简单，直觉上也是正确的，但是并不很容易证明。



## 引用

Boneh D., Lynn B., Shacham H. (2001) Short Signatures from the Weil Pairing. In: Boyd C. (eds) Advances in Cryptology — ASIACRYPT 2001. ASIACRYPT 2001. Lecture Notes in Computer Science, vol 2248. Springer, Berlin, Heidelberg. https://doi.org/10.1007/3-540-45682-1_30