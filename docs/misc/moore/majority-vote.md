# 摩尔投票法

**by Acc_Robin**

!!! tip "tip"
    如果你觉得本文比较突兀，可以先看看原 OI-wiki 中的[主元素问题](https://oi-wiki.org/misc/main-element/)

## 简述

在 $O(1)$ 空间，$O(n)$ 时间求解序列主元素（主元素指出现次数超过 $\frac n2$ 的元素）。

## 详解

不断消去两个相邻的、不同的元素之后，剩下的一定是主元素。

### 需要支持的操作

1. 栈
2. 判断两元素是否相同。

### 可合并性

摩尔投票法支持合并：合并后的主元素要么不存在，要么一定是合并前两个集合主元素之一。

## 实现

```cpp
int moore(const vector<int>& vec) {
    int v = -1, c = 0;
    for (int x : vec) {
        if (x == v) ++c;
        else if (--c <= 0) v = x, c = 1;
    }
    return v;
}
```

## 应用

### 求解主元素

[P2397](http://luogu.com.cn/problem/P2397)：模板。

[[ARC070D] HonestOrUnkind](https://www.luogu.com.cn/problem/AT2348)：难点通常在发现要求的是主元素，毕竟这是一个较冷门的知识点。[题解](https://afoi-wiki.github.io/afoi-wiki/misc/moore/ARC070D/)

### 线段树维护摩尔投票

请注意区分主元素与众数的关系：主元素要求出现次数严格大于区间长度的一半（区间众数规约矩阵乘法）。

线段树每个节点存储一个对子 $(v,c)$，$v$ 表示**可能**是主元素的值，而 $c$ 就是摩尔投票相消去之后剩下的次数。

!!! note ""
    此处“可能”指的是：不是 $v$ 的值一定不是主元素，但 $v$ 不一定是主元素（有可能不存在主元素）

合并时如果两区间 $v$ 相同，则 $c$ 相加；否则大区间的 $v$ 为 $c$ 更大的那个区间的 $v$，$c$ 为两区间 $c$ 的差。

```cpp
struct T{
    int v, c;
};
auto mg=[](T a, T b)->T {
    if (a.v == b.v) return {a.v, a.c + b.c};
    return {a.c > b.c ? a.v : b.v, abs(a.c - b.c)};
};
```

这样维护只能保证：若区间存在主元素，则 $v$ 一定是主元素，但如果区间不存在主元素，$v$ 就不能保证了。

因此，应当在查询完区间 $v$ 之后，使用其它数据结构判断 $v$ 在区间出现次数是否严格大于区间长度的一半。通常可以对每个值开一颗平衡树，存储这个值出现的所有下标即可。

[P3765 总统选举](https://www.luogu.com.cn/problem/P3765)：线段树维护摩尔投票。
