# RSA 算法

[TOC]

## 1 算法描述

### 1.1 密钥的产生

> （1）选两个保密的大素数 $p,q$ ；
>
> （2）计算 $n=p\times q,\varphi(n)=(p-1)(q-1)$ ，其中 $\varphi(n)$ 是 $n$ 的欧拉函数值；
>
> （3）选一整数 $e$ ，满足 $1<e<\varphi(n)$ ，且 $gcd(\varphi(n),e)=1$ ；
>
> （4）计算 $d$ 满足
>
> &emsp;&emsp;&emsp;&emsp;$d\:\cdot\:e\equiv 1\:mod\:\varphi(n)$
>
> 即 $d$ 是 $e$ 在模 $\varphi(n)$ 下的乘法逆元，因 $e$ 与 $\varphi(n)$ 互素，故他们的乘法逆元一定存在；
>
> （5）以 $\{e,n\}$ 为公钥， $\{d,n\}$ 为私钥。

### 1.2 加密

加密时首先将铭文比特串分组，使得每个分组对应的十进制数小于 $n$ ，即分组长度小于 ${log}_2n$ 。然后每个明文分组 $m$ ，作为加密运算：

>&emsp;&emsp;&emsp;$c\equiv m^e\:mod\:n$ 

### 1.3 解密

对密文分组的解密运算为：

> &emsp;&emsp;&emsp;$m\equiv c^d\:mod\:n$

