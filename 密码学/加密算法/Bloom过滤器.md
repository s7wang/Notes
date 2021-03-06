# Bloom 过滤器

[TOC]

## 1 提出一个问题

在我们细述Bloom过滤器之前，我们先抛出一个问题：给你一个巨大的数据集（百万级、亿级…），怎么判断一个元素是否在此数据集中？或者怎么判断一个元素不在此数据集中？

思考这个问题的时候，最先想到的可能是哈希表，在数据集规模较小的时候，这个方法是可行的，当然，数据集巨大的时候也可以采用分布式哈希表的方式。当数据集规模较大时，尤其是应用中只需要判断一个元素不在此数据集中的情况时，我们可以借鉴哈希表的思路，使用Bloom过滤器解决这个问题。既然我们只关心元素在不在，不关心元素值是什么，只要把元素映射为一个布尔值表示在不在就足够了。下面细述Bloom过滤器数据结构的设计。

# 2 Bloom 过滤器

### 2.1 原理

>1.首先需要k个hash函数，每个函数可以把key散列成为1个整数；
>
>2.初始化时，需要一个长度为 m 比特的数组，每个比特位初始化为0；
>
>3.某个key加入集合时，用k个hash函数计算出k个散列值，并把数组中对应的比特位置为1；
>
>4.判断某个key是否在集合时，用k个hash函数计算出k个散列值，并查询数组中对应的比特位，如果所有的比特位都是1，认为在集合中。

### 2.2 特点

> 优点：
>
> 1.相比于其它的数据结构，布隆过滤器在空间和时间方面都有巨大的优势。布隆过滤器存储空间和插入/查询时间都是常数。另外, Hash 函数相互之间没有关系，方便由硬件并行实现。布隆过滤器不需要存储元素本身，在某些对保密要求非常严格的场合有优势；
>
> 2.布隆过滤器可以表示全集，其它任何数据结构都不能；
>
> 3.k 和 m 相同，使用同一组 Hash 函数的两个布隆过滤器的交并差运算可以使用位操作进行。
>
> 
>
> 缺点：
>
> 1.但是布隆过滤器的缺点和优点一样明显。误算率（False Positive）是其中之一。随着存入的元素数量增加，误算率随之增加。但是如果元素数量太少，则使用散列表足矣；
>
> 2.一般情况下不能从布隆过滤器中删除元素. 我们很容易想到把位列阵变成整数数组，每插入一个元素相应的计数器加1, 这样删除元素时将计数器减掉就可以了。然而要保证安全的删除元素并非如此简单。首先我们必须保证删除的元素的确在布隆过滤器里面. 这一点单凭这个过滤器是无法保证的。另外计数器回绕也会造成问题。



## 2 案例

### 2.1 集合表示和元素查询

下面我们具体来看Bloom Filter是如何用位数组表示集合的。初始状态时，Bloom Filter是一个包含m位的位数组，每一位都置为0。

![Bloom_001](assets/Bloom_001.jpg)

为了表达 $S=\{x_1,x_2,\ldots,x_n\}$ 这样一个n个元素的集合，Bloom Filter使用k个相互独立的哈希函数（Hash Function），它们分别将集合中的每个元素映射到 $\{1,\ldots,m\}$ 的范围中。对任意一个元素x，第 $i$ 个哈希函数映射的位置 $hi(x)$ 就会被置为 $1\quad (1\leq i\leq k)$ 。注意，如果一个位置多次被置为1，那么只有第一次会起作用，后面几次将没有任何效果。在下图中，$k=3$ ，且有两个哈希函数选中同一个位置（从左边数第五位）。  

![Bloom_002](assets/Bloom_002.jpg)



在判断 $y$ 是否属于这个集合时，我们对 $y$ 应用 $k$ 次哈希函数，如果所有 $hi(y)$ 的位置都是 $1\quad (1\leq i\leq k)$ ，那么我们就认为y是集合中的元素，否则就认为 $y$ 不是集合中的元素。下图中 $y_1$ 就不是集合中的元素。$y_2$ 或者属于这个集合，或者刚好是一个false positive。

![Bloom_003](assets/Bloom_003.jpg)

### 2.2 错误率估计



