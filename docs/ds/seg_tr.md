**by 木xx木大**

退役一年后再学线段树，修正了之前博客中的锅，补充了一些例题，并增加了少量之前没来得及写的内容。我也不知道为啥写着写着就字数翻倍破 1w 了（捂脸）。

内容大致按照我认为的难度排序。

感谢 a___ 和 zyt1253679098 对我的帮助和指导！

## 一、线段树维护矩阵与动态 DP

线段树可以维护**和、积、最值**等满足**结合律**的区间运算。

对于一些只有单点修改且维护的信息具有递推性的题目，由于运算具有结合律，可以将两区间合并写成矩阵乘法的形式，省去一些麻烦的讨论。

在序列上时，可以直接将 dp 转移写成矩阵乘法的形式，方便维护。

在树上时，可以把树剖了，分别维护重链的信息 $f$ 和轻链的信息 $g$。修改时，一条条链依次向上跳，每条链的信息用线段树维护，链首的父亲的 $g$ 会被更新。当然也可以用全局平衡二叉树维护。

但写成矩乘的形式可能会带来巨大的常数，所以更好的做法是只维护矩阵中有用的值。

>例题1：[CF575A Fibonotci](https://www.luogu.com.cn/problem/CF575A) 
>
>$\{s_n\}^\infty_{n=0}$是一个正整数序列。
>
>给定 $s_0,s_1, \dots ,s_{n-1}$，对于 $i \ge n$，有 $m$ 个 $i$ 给定 $s_i$，剩下的满足 $s_i = s_{i \bmod n}$。
>	
>$\{ f_n \}_{n=0}^\infty$同样是一个正整数序列。
>	
>$f_0 = 0,f_1 = 1$，对于 $i \ge 2$，$f_i = s_{i-1}f_{i-1} + s_{i-2}f_{i-2}$，求 $f_k \bmod p$
>	
>$n,m \le 5 \times 10^4$，$k \le 10^{18}$，$s_i,p \le 10^9$。

看到近似Fibonacci的递推，我们想到矩阵乘法，朴素递推式为
$$
	\left[ \begin{matrix} f_i,f_{i+1}\end{matrix} \right] \times\left[ \begin{matrix}0 , s_i \\   1,s_{i+1} \end{matrix} \right]=\left[ \begin{matrix} f_{i+1},f_{i+2}\end{matrix} \right]
$$
$s_n$ 有循环节，而 $m$ 个特殊位置可以看作计算时对循环节进行 $m$ 次修改。如果没有修改，我们可以暴力算出每个循环节内的答案，用矩阵快速幂求总答案。而修改最多只会涉及 $m$ 个循环节，复杂度 $O(nm)$。

考虑优化：在原矩阵乘积的基础上进行修改。注意到一个 $s_i$ 的改变只会影响两个矩阵，那么我们需要单点修改，全局查询，用线段树维护即可。对于一个有特殊点的周期，我们先修改，算完这个循环节后再改回去即可。

>例题2：[P3781 [SDOI2017]切树游戏](https://www.luogu.com.cn/problem/P3781) 
>
>给定一棵有点权的树，权值上限为 $m$。要求支持两种操作：
>
>* 将编号为 $x$的结点的权值修改为 $y$
>* 询问有多少个非空连通子图，满足其所有点的权值的异或和恰好为 $k$。
>
>$n,q \le 3 \times 10^4$，$m \le 128$。

首先考虑不带修时的暴力 dp：设 $f_{u,i}$ 表示以 $u$ 为根、异或和为 $i$ 的联通子树的个数，$h_{u,i}$ 表示 $u$ 子树内 $f_{v,i}$ 的和，转移为 
$$
	f_{u,k}+=\sum_{i \oplus j=k}f_{u,i}\times f_{v,j},h_{u,i}=f_{u,i}+\sum h_{v,i}
$$

复杂度为 $O(nm^2)$ 。发现转移式是一个异或卷积的形式，于是可以先 FWT 一次，转移时直接用点值相乘，复杂度为 $O(nm)$

带修怎么办？动态 dp！	
设 $lf_{u,i}=\prod_{v\in lightson_{u}}(1+f_{v,i})$,$lh_{u,i}=\sum_{v\in lightson_{u}}h_{v,i}$ 。
设 $F,H,LF,LH$ 分别表示 $f,h,lf,lh$ 的生成函数，设 $v$ 为 $u$ 的重儿子，则转移为	
$$
\begin{aligned}
		&f_{u,i}=(1+f_{v,i})\times lf_{u,i}\\
		&h_{u,i}=h_{v,i}+lh_{u,i}+(1+f_{v,i})\times lf_{u,i}\\
		&\begin{pmatrix} F_v,H_v,1\\\end{pmatrix}\times\begin{pmatrix}\ &LF_u,&LF_u,&0\\&0,&1,&0\\&LF_u,&LH_u+LF_u,&1\end{pmatrix}=\begin{pmatrix} F_u,H_u,1\\\end{pmatrix}
	\end{aligned}
$$
优化常数：如下式，其实我们只需要维护 $a,b,c,d$ 处的四个值即可。
$$
		\begin{aligned}
			\begin{pmatrix}&a_1,&b_1,&0\\&0,&1,&0\\&c_1,&d_1,&1\end{pmatrix}\times\begin{pmatrix}\ &a_2,&b_2&0\\&0,&1,&0\\&c_2,&d_2,&1\end{pmatrix}=\begin{pmatrix}\ &a_1a_2,&a_1b_2+b_1,&0\\&0,&1,&0\\&a_2c_1+c_2,&c_1b_2+d_1+d_2,&1\end{pmatrix}
		\end{aligned}
$$
还有一个细节：跳重链修改父亲 $lf$ 的值的时候可能会出现除以 0 的情况，所以必须再开一个变量记 $lf$ 中 0 的个数，除以 0 就直接减一，通过这个变量推出 $lf$ 的真实值。

用树剖写的话，复杂度 $O(qm\log^2n)$，loj 能过。洛谷只有 80 pts，需要写全局平衡二叉树。

##### 其他题目

* [CF573D Bear and Cavalry](https://www.luogu.com.cn/problem/CF573D)
* [CF1286D LCC](https://www.luogu.com.cn/problem/CF1286D)
* [P4719 【模板】"动态 DP"&动态树分治](https://www.luogu.com.cn/problem/P4719)
* [P6021 洪水](https://www.luogu.com.cn/problem/P6021)
* [P5281 [ZJOI2019]Minimax搜索](https://www.luogu.com.cn/problem/P5281)（鸽了）

## 二、线段树维护树上信息

### 1.线段树维护直径

* 方法一：基于贪心的思想和直径的性质，当合并两个点集时，新直径的端点一定是**被合并的两个点集直径四个端点中的某两个**。

* 方法二：把点拍扁到欧拉序上，两点间的距离即 $dep_u+dep_v-2\min\limits_{i=u}^vdep_{i}$。维护最大、最小深度 $mx,mi$，以及 $lm=\max\{dep_u-2\min\limits_{i=l}^udep_i\},rm=\max\{dep_u-2\min\limits_{i=u}^rdep_i\}$ 和直径 $s$ 。合并两个点集时，新点集的直径的两个端点要么全在左边（即 $s_{lson}$），要么全在右边（即 $s_{rson}$），要么跨越中点。

  ```cpp
  void pushup(int ro)
  {
  	t[ro].mx=max(t[ro<<1].mx,t[ro<<1|1].mx),t[ro].mi=min(t[ro<<1].mi,t[ro<<1|1].mi);
  	t[ro].lm=max(max(t[ro<<1].lm,t[ro<<1|1].lm),t[ro<<1|1].mx-2*t[ro<<1].mi);
  	t[ro].rm=max(max(t[ro<<1].rm,t[ro<<1|1].rm),t[ro<<1].mx-2*t[ro<<1|1].mi);
  	t[ro].s=max(max(t[ro<<1].s,t[ro<<1|1].s),max(t[ro<<1].mx+t[ro<<1|1].lm,t[ro<<1|1].mx+t[ro<<1].rm));
  }
  ```

这两个方法不适用存在负边权的情况。相较于更普适的点分树和 DDP，这两种方法的优点在于代码短，常数小！

##### 题目

* [题解 P2056 [ZJOI2007]捉迷藏](https://www.luogu.com.cn/blog/flyingfan/p2056-zjoi2007-zhuo-mi-zang-xian-duan-shu-wei-hu-zhi-jing-post)

* [P6845 [CEOI2019] Dynamic Diameter](https://www.luogu.com.cn/problem/P6845)

* [CF1149C Tree Generator™](https://www.luogu.com.cn/problem/CF1149C)（括号序列）

### 2.线段树维护极小连通子树

这种题目通常会标记一些点为关键点，每次操作可能会改变关键点，你需要求出所有关键点形成的极小连通子树的边权和。

DFS 序求出后，假设关键点按照 DFS 序排序后是 $a_1,a_2,…,a_k$，那么所有关键点形成的极小连通子树的边权和的两倍等于 $dist(a_1,a_2)+dist(a_2,a_3)+\dots +dist(a_{k−1},a_k)+dist(a_k,a_1)$。这个信息便于单点修改，区间合并，可以用线段树/ set 维护。

用线段树维护:上式即边权和为 $\sum_{i=1}^k dep_{a_i}-\sum_{i=1}^{k-1}dep_{lca(a_i,a_{i+1})}-dep_{lca(a_1,a_k)}$。以 DFS 序为下标，叶子节点存点的深度;合并两个区间时，总答案为两区间答案之和减去两区间相邻点 LCA 的深度。

用 set 维护：假设插入 $x$，它DFS序左右两边分别是 $y$ 和 $z$。那么答案加上 $dist(x,y)+dist(x,z)−dist(y,z)$ 即可。（特判边界情况，即 DFS 序是最小的或者是最大的），删除同理。

##### 题目

* [P5327 [ZJOI2019]语言](https://www.luogu.com.cn/problem/P5327)
* [P3320 [SDOI2015]寻宝游戏](https://www.luogu.com.cn/problem/P3320)

## 三、线段树上二分

对于一些需要用二分套线段树上查询的问题，因为线段树本身就是一个分治的结构，所以可以直接在线段树的结构上进行二分。这样做可以去掉一个 log。

>例题3：[P5537 【XR-3】系统设计](https://www.luogu.com.cn/problem/P5537)
>
>现有一棵 $n$ 个点的有根树和一个长度为 $m$ 的序列 $a$，接下来需要实现 $q$ 个操作。
>操作分两种：
>
>* 1 x l r 表示设定起点为有根树的节点 $x$，接下来依次遍历 $l \sim r$。当遍历到 $i$ 时，从当前节点走向它的编号第 $a_i$小的儿子。如果某一时刻当前节点的儿子个数小于 $a_i$，或者已经遍历完 $l \sim r$，则在这个点停住，并输出这个点的编号，同时停止遍历。
>* 2 t k 表示将序列中第 $t$ 个数 $a_t$修改为 $k$。
>
>$n,m,q \le 5 \times 10^5,1\le a_i\le n$。

性质：任意一对祖先 - 后代之间的路径都可以按题目所述的规则表示成一个**固定**整数序列，并且这个序列是可以**合并**，即如果 $u$ 是 $v$ 的祖先，$v$ 是 $w$ 的祖先，则 $(u,v)$ 的序列与 $(v,w)$ 的序列拼接起来就是 $(u,w)$ 的序列。 

我们可以用一个序列（对应着从根到这个结点的走法）来表示一个结点。 在 dfs 过程中按编号给子节点排序，预处理出从根到所有节点路径的数字序列， 把序列 $\to$ 结点的映射关系用哈希表存下来，并用线段树维护给定序列的哈希值，就可以快速判断从一个结点按照一个序列走若干步能否走到一个结点，以及查询如果能走到，走到的是哪个结点。 在线段树上二分答案即可。

因为我们要在线段树上的给定区间 $[L,R]$ 上二分，而非全局二分，所以查询时可能不能直接调用已经算出来的值，在二分时必须现算区间 $[L,mid]$ 的哈希值。这样做是两只 log。

怎么优化？

万能的 zyt1253679098 学长告诉我们：在一次询问中，查询的区间左端点相同；且触发查询的条件为查询区间的 $L>t[ro].l$。因此只有本次往左儿子走之后才可能查询，查询的区间也一定是左儿子的子区间。换句话说，之后可能查的区间在当前的查询中都会经过！那么记忆化一下就只有一只 log 了

>例题4：[P4364 [九省联考2018]IIIDX](https://www.luogu.com.cn/problem/P4364)
>给定序列 $d$，要求你将 $d$ 重新排列，使重排后的 $d$ 满足 $d_i \geq d_{\left\lfloor \frac ik \right\rfloor}$ 且 $d$ 的字典序最大。
>
>$n\le 5\times 10^5,k,d\le 10^9$，$k$ 为浮点数。

容易发现，$i\to \lfloor\frac i k\rfloor$ 构成了一个树形结构。对于 $d_i$ 互不相同的情况，把 $d_i$ 排序后按子树大小贪心分配后缀即可。对于点 $u$ 的子树，选择当前最大的 $siz_u$个权值，然后递归求解。	

但当 $d_i$ 相同时，可能会存在一种情况：可以将 $u$ 子树内的一个较大值与 $u$ 的兄弟交换使得答案更优。如 $n=4;k=3;a=\{1,1,2,2\}$，正解为 $1,2,2,1$，而直接贪心答案为 $1,1,2,2$。

改变贪心策略。设 $cnt_i$ 表示 $\ge i$ 的数的个数，$cnt$ 是单调不降的。当我们选了一个数 $x$ 放在当前子树 $u$ 的根上，则 $cnt[1,x]$ 全部要减小 $siz_u$（给子树预留位置）。那么我们要找的实际上是 $cnt_i\ge siz_u$ 的最大的 $i$ 。区间减可以用线段树维护，查询直接在线段树上二分即可。

实现时要注意：如果一个点有父亲，那么查这个点的答案之前要把它父亲为子树预留的大小删去，且多个点父亲相同时只需要删一次。

## 四、线段树合并

merge(x,y)的操作：

* $x=0$，返回 $y$

* $y=0$，返回 $x$
* 两个点都是叶子，直接合并信息；
* 否则，分别递归合并左、右儿子：merge($ls_x,ls_y$)，merge($rs_x,rs_y$) ；再将 $x,y$ 两个结点上的信息合并，返回合并后的结点。

复杂度分析：每次递归会将 $x,y$ 中的一个点删除，这件事发生的总次数不超过总点数 $O(n\log n)$，总复杂度 $O(n\log n)$

当需要**按对应位置合并一些数组**，但这些数组并不满，且合并操作较为简单时,可以采用线段树合并。解决类似的树上统计问题时也可以用 dsu on tree、长链剖分、数组启发式合并等。这类树上统计问题也可以推广到后缀树上。

相比其他方法，线段树合并的优点是支持线段树所支持的区间修改、查询等操作，可以维护的信息更复杂。

一个巧妙的应用是线段树合并优化 DP（整体 DP）

### 1.树上统计问题

>例题5：[P5327 [ZJOI2019]语言](https://www.luogu.com.cn/problem/P5327)
>
>给定一棵 $n$ 个点的树和 $m$ 条树上路径，问有多少有序点对 $(i,j)$ 满足这两点至少被同一条路径覆盖。
>
>$1\leq n,m\leq 10^5$

首先有序点对不好求，先算无序点对个数，最后答案除以 2 即可。问题转化为对每个点求它能通过给定路径到达的点的个数。

每个点能到达的点集为所有经过它的路径的并集，这个并集为一个连通块，即**包含所有经过它的路径的端点的极小连通子树**。极小连通子树的维护方法讲过了。于是我们有了一个暴力：对每个点开一棵线段树，将经过其路径的端点插入，维护极小连通子树，复杂度 $O(mn\log n)$。

另一个暴力：对于每个点，将经过它的路径上的点权赋为 1 ，用**差分**实现，全局求和，复杂度 $O(mn\log n)$。

把两个思路合起来，将路径差分后插入线段树，用线段树合并向上传递信息，复杂度 $O((n+m)\log^2 n)$ 或 $O((n+m)\log n)$（欧拉序求 LCA）。

##### 其他题目

* [P4556雨天的尾巴](https://www.luogu.com.cn/problem/P4556)
* [P3899 [湖南集训]更为厉害](https://www.luogu.com.cn/problem/P3899)

### 2.整体dp

>例题6：[P6773 [NOI2020] 命运](https://www.luogu.com.cn/problem/P6773) 
>
>给定一棵 $n$ 个点的树和 $m$ 条限制，你可以给树上的每一条边赋一个 0 或 1 的权值。对于所有限制 $(u,v)$（保证 $u$ 为 $v$ 的祖先） 你需要保证 $u$ 到 $v$ 上至少有一条边的权值为 1，求赋值方案数。
>
>$1\leq n,m\leq 5\times 10^5$

性质：下端点相同的限制只有上端点深度最大的有用。	

设 $dp_{u,i}$ 表示下端点在 $u$ 的子树内且未被满足的限制的上端点深度最大的为 $i$ 时，$u$ 子树内赋值的方案数。特别地，$dp_{u,0}$ 为所有限制都满足的方案数。答案即为 $dp_{1,0}$
$$
dp_{u,i}'=dp_{u,i}\sum_{j=0}^{dep_u}dp_{v,j}+dp_{u,i}\sum_{j=0}^idp_{v,j}+dp_{v,i}\sum_{j=0}^{i-1}dp_{u,j}
$$

第一个 $\sum$ 是 $(u,v)$ 为 1 的情况，后两个是为 0 的情况。容易发现 dp 式子可以前缀和优化。

用线段树合并优化这个 dp 式子。先查询 $\sum_{j=0}^{dep_u}dp_{v,j}$，merge 的时候先处理左区间，处理的时候顺便求出前缀和，处理右区间的时候就可以直接调用了。

>例题7：[P5298 [PKUWC2018]Minimax](https://www.luogu.com.cn/problem/P6773)
>
>有一棵树，给出每个叶节点的点权（互不相同），每个节点 $x$ 至多有两个子节点，且其点权有 $p_x$（$p_x$ 不为0且不为1）的概率是子节点点权较大值，有 $1-p_x$ 的概率是子节点点权较小值。假设根节点点权有 $m$ 种可能性，其中权值第 $i$ 小的可能点权是 $V_i$，可能性为 $D_i$，求$\sum_{i=1}^mi\cdot V_i\cdot D_i^2$。
>
>$1\leq n\leq 3\times 10^5,0<p\times 10000<10000$

由于概率不为 0 且不为1，所以所有点权都可能取到。$V_i$ 排个序即可得到，我们只需要求 $D_i$。设 $f_{i,j}$ 为 $i$ 节点出现 $j$ 的概率。	
$$
f_{i,j} = f_{l,j} * (p_i \sum_{k=1}^{j-1}f_{r,k} + (1-p_i)\sum_{k=j+1}^{m}f_{r,k}) + f_{r,j} * (p_i \sum_{k=1}^{j-1}f_{l,k} + (1-p_i)\sum_{k=j+1}^{m}f_{l,k})
$$

现在我们需要一个数据结构，能够维护数组之间的转移，并能维护前缀和与后缀和，想到线段树合并。

考虑线段树合并时的行为：

* 两个合并点都为空，dp 值一定为 0 

* 两个合并点都不为空，直接递归

* 有一个为空时，以 $r$ 为空为例，转移式为
  $$
  f_{i,j} = f_{l,j} * (p_i \sum_{k=1}^{j-1}f_{r,k} + (1-p_i)\sum_{k=j+1}^{m}f_{r,k}) 
  $$
  即给整个区间打上 $(p_i \sum_{k=1}^{j-1}f_{r,k} + (1-p_i)\sum_{k=j+1}^{m}f_{r,k})$ 的乘法标记。前缀/后缀和在 merge 时顺便计算一下即可。

##### 其他题目

* [题解 P4365 [九省联考2018]秘密袭击coat](https://www.luogu.com.cn/blog/flyingfan/p4365-jiu-xing-lian-kao-2018-mi-mi-xi-ji-coat-cha-zhi-xian-duan-shu-post)

## 五、线段树维护线段/直线

即李超线段树。详见[李超树学习笔记](https://www.luogu.com.cn/blog/flyingfan/li-chao-shu)

## 六、线段树优化建图

常见操作有三种：

* 从 $x$ 向区间 $[l,r]$ 的点连边

  建一棵“正线段树“，其中父亲向儿子连边。找到 $[l,r]$ 在“正线段树”上对应的节点，让 $x$ 向这些点连边。

* 从区间 $[l,r]$ 的点向 $x$ 连边

  建一棵“反线段树“，其中儿子向父亲连边。找到 $[l,r]$ 在“正线段树”上对应的节点，让这些点向 $x$ 连边。

* 从区间 $[x,y]$ 的点向区间 $[l,r]$ 的点连边

  建一个虚点，让 $[x,y]$ 在“反线段树”上对应的节点向虚点连边，虚点向“正线段树”上 $[l,r]$ 对应的节点连边

对于有 $n$ 个点，$m$ 个连边操作的图，最终连出的总边数为 $O(m\log n)$ 级别。

##### 题目

* [CF786B Legacy](https://www.luogu.com.cn/problem/CF786B)
* [P5025 [SNOI2017]炸弹](https://www.luogu.com.cn/problem/P5025)
* [CF786E ALT](https://www.luogu.com.cn/problem/CF786E)

## 七、线段树结构分析

经常有这样一类问题：模拟线段树上的若干操作，然后查询相关的计数信息。以下结论对广义线段树（mid 为 $[l,r)$ 中的任意整数）均适用

**1.区间 $[l,r]$ 可以表示为线段树上若干个区间的不交并，称这些区间构成 $[l,r]$ 的区间拆分。**

**2.任何一个包含于 $[1,n]$ 的区间的区间拆分存在，且大小最小的区间拆分唯一。最小区间拆分的大小不超过 $2\log n+c$，$c$ 为常数**

**3.$[1,n]$ 的广义线段树内 $[l,r]$ 的最小区间拆分大小为 $2\times (r-l+1)-|S|$，其中 $|S|$ 表示线段树上完全包含于 $[l,r]$ 的区间数量。**

证明：提取出所有包含于 $[l,r]$ 区间的线段树上的区间，它们构成了一个完满二叉树森林。这个森林的叶子节点个数为 $r-l+1$，则森林的总点数为 $2(r-l+1)-rt$，其中 $rt$ 表示根节点个数，也等价于区间定位数，移项得 $rt=2(r-l+1)-|S|$

##### 题目

* [P7143 [THUPC2021 初赛] 线段树](https://www.luogu.com.cn/problem/P7143) （也可以不用上面的结论，直接分治成两个子问题然后记搜）

**4.广义线段树上 $[l,r]$ 的最小区间拆分可以这样表示：
记 $L,R$ 分别表示 $[l-1,l-1],[r+1,r+1]$ 对应的节点，记 $U=lca(L,R)$，则定位出来的区间为  $L$ 到 $U$ 的左儿子 的链上（下文称左链）所有节点的 不在链上的右儿子 和所有  $R$ 到 $U$ 的右儿子 的链上（下文称右链）所有节点的 不在链上的左儿子**

例如，在 $[1,10]$ 的标准线段树上，$[3,8]$ 的最小区间拆分如下，黄色表示最小区间拆分，绿色部分为左、右链。（图来自 ix35 博客，侵删歉）

![img](https://cdn.luogu.com.cn/upload/image_hosting/drwqvpt2.png)

此外，如果 $l=1$ 或 $r=n$，我们可以在广义线段树上额外建出 0 和 $n+1$ 这两个位置。

>例题8：[P5210 [ZJOI2017]线段树](https://www.luogu.com.cn/problem/P5210)
>
>给定 $[1,n]$ 的广义线段树，有 $m$ 组询问，每组询问给定区间 $[l,r]$ 和线段树上结点 $p$，求 $p$ 到所有 $[l,r]$ 的最小区间拆分对应结点的距离和。
>
>$1\leq n,m\leq 2\times 10^5$

设 $[l,r]$ 的最小区间拆分为 $S$ 
$$
\sum_{v\in S}dis(u,v)=|S|dep_u+\sum_{v\in S}dep_v-2\sum_{v\in S}dep_{lca(u,v)}
$$

分别维护 $sizl_u,sizr_u,dl_u,dr_u$ 表示 $u$ 的所有祖先的左/右儿子的兄弟的个数/深度和，$|S|$ 和 $\sum_{v\in S}dep_v$ 差分一下就可以得到。后半部分需要精细的讨论，以左链为例，记 $u$ 和 $L$ 的 $lca$ 为 $V$

* $V$ 为 $U$ 或 $U$ 的祖先时，左链上所有的点和 $u$ 的 $lca$ 均 $V$
* 否则，$V$ 以上的点与 $u$ 的 $lca$ 为它的父亲，$V$ 以下的点与 $u$ 的 $lca$ 为 $V$ ；但当 $u$ 恰好在为 $V$ 的右儿子的子树里，$u$ 和 $V$ 的右儿子的 $lca$ 就为 $V$ 的右儿子

**5.modify 操作对 `lazytag` 的影响**

设操作区间为 $[ql,qr]$（图来自小粉兔的题解，侵删歉）
  ![img](https://cdn.luogu.com.cn/upload/pic/55713.png)

   1. 若 $[l,r]\cap [ql,qr]=[l,r]$ 且它的父亲不满足上述性质 （即下图中的浅蓝色部分），那么这个区间在这轮操作中一定会被覆盖到，**即操作后它们必然有懒标记** 。
   2. 若 $[l,r]\cap [ql,qr]\neq \emptyset$ 且不满足性质①（即下图中的紫色部分），那么这个区间的 lazytag 一定会被下传， **即操作后它们必然无懒标记**。
   3. 若 $[l,r]\cap [ql,qr]=\emptyset$ 但它的父亲区间与 $[ql,qr]$ 有交集（即下图中的黄色部分），那么这个区间在这轮操作中只能接受祖先的 lazytag， **操作后它们有无懒标记取决于操作前这个节点到根的链上有无懒标记**。 
   4. 若 $[l,r]\cap [ql,qr]=\emptyset$ 但它的父亲区间与 $[ql,qr]$ 交集为空（即下图中的橙色部分），那么这轮操作与这个区间无关
   5. 若 $[l,r]\cap [ql,qr]=[l,r]$ 且它的父亲满足上述性质 （即下图中的深蓝色部分），那么这个区间在这轮操作中一定不会被覆盖到。

   总结一下，设 $f_i$ 表示 $i$ 是否有标记，$g_i$ 表示 $i$ 到根的路径上（包括 $i$）是否存在一个有标记的点

   1. 浅蓝色部分：$f_i=g_i=1$
   2. 紫色部分：$f_i=g_i=0$
   3. 黄色部分：$f_i=g_i=g_i'$
   4. 橙色部分：$f_i=f_i',g_i=g_i'$
   5. 深蓝色部分：$f_i=f_i',g_i=1$

>例题9：[P5280 [ZJOI2019]线段树](https://www.luogu.com.cn/problem/P5280)
>
>初始时有一棵什么标记都没有的 $[1,n]$ 上的线段树。操作有两种：
>
>修改：把当前所有的线段树复制一份，然后对新增线段树实行一次区间修改操作。
>
>查询：求所有线段树中有懒标记的节点总数。 
>
>$1\le n,q\le 10^5$

由于概率具有**可乘性**，我们将问题转化为求概率，最后再$\times 2^{now}$ 即可，$now$ 为当前修改操作的次数。我们设 $f_u$ 表示结点 $u$ 有懒标记的概率。由于第3类节点是否有 tag 与其祖先有关，我们再设 $g_u$ 表示每个节点 $u$ 到根的路径上有懒标记的概率。 最终答案即为$\sum f_u\times 2^{now}$。

对于第1类节点，一半保持原样，一半有标记，那么 $f_u=\frac {f_u}{2}+\frac {1}{2}$，$g_u=\frac {g_u}{2}+\frac {1}{2}$

对于第2类节点，一半保持原样，一半无标记，那么 $f_u=\frac {f_u}{2}$，$g_u=\frac {g_u}{2}$

对于第3类节点，一半保持原样，另一半的标记取决于 $u$ 到根的路径 $f_u=\frac{f_u+g_u}{2}$，$g_u$ 不变。

对于第4类节点，标记不受影响，到根路径上的标记也不受影响。我们不用管。

对于第5类节点，标记不受影响，操作后新一半到根路径上一定有标记。$f_u$ 不变，$g_u=\frac {g_u}{2}+\frac {1}{2}$。

1,2,3类节点都只有 $O(\log n )$ 个，可以直接修改。

第5类节点个数过多，只能用打懒标记的方法维护。若当前节点到根的路径上有 $x$ 个标记，那么只有 $x$ 次操作**都不执行**才可能使当前区间 lazytag 为 0，$g_u=1-\frac{1-g_u}{2^x}$

>例题10：[P6630 [ZJOI2020] 传统艺能](https://www.luogu.com.cn/problem/P6630)
>
>给定一个 $[1,n]$ 的广义线段树，每次操作等概率选取一个区间 $[l,r]$，将其最小区间拆分集合中的区间打上懒标记，同时下传沿途标记，求 $k$ 次操作后期望有多少个点被打上标记。 
>
>$1\le n\le 2\times 10^5,1\le k \le 10^9$

对每个点算出其最终有 tag 的概率，相加即为答案。

分别计算出一次操作后每个节点作为以上 5 类点的概率，将操作对 $f,g$ 的影响用矩阵表示出来，矩阵快速幂即可。 

## 八、线段树维护单调栈（前缀最值相关）

>例题11：[P4198 楼房重建](https://www.luogu.com.cn/problem/P4198)
>
>维护一个序列，支持单点修改，查询前缀严格最大值的个数，即满足 $\displaystyle \max_{j = 1}^{i - 1} \{ a_j \} < a_i$ 的 $i$ 的个数。

设 $s$ 表示仅考虑此区间时区间严格最大值的个数，$mx$ 表示区间最大值，设 $solve(ro,x)$ 表示 $ro$ 所覆盖的区间中考虑前缀最大值 $x$ 后的区间严格最大值个数

* 如果 $x< mx_{ls}$ ，处于右区间的前缀最大值必定大于 $mx_{ls}$，所以也$>x$，则 $x$ 不会对右区间造成影响。答案为 $solve(ls,x)+s_{ro}-s_{ls}$（注意，这里不是 $s_{rs}$）
* 如果 $x\ge mx_{ls}$，左区间所有的数都不可能成为前缀最大值，答案为 $solve(rs,x)$

由于每次只往一边递归，所以这个 $solve$ 函数的复杂度是 $O(\log n)$ 的。合并两个区间时，$s_{ro}=s_{ls}+solve(rs,mx_{ls})$，修改时会调用 $O(\log n)$ 次 $solve$ 函数，单次修改的复杂度为 $O(\log^2n)$。

但有时我们维护的信息不具有可减性（如取 $\max,\min$ 等），为了让以上做法适应更一般的情况，我们更改 $s$  的定义为：只考虑 $[l,r]$ 区间的影响时，$[mid+1,r]$ 中的答案。那么 $solve$ 函数中的第一种情况就可以改为 $solve(ls,x)+s_{ro}$，更新答案的操作改为 $s_{ro}=solve(rs,mx_{ls})$，同时查询操作要改为 $solve(rt,0)$

>例题12：[题解 CF671E Organizing a Race](https://www.luogu.com.cn/blog/flyingfan/cf671e-organizing-a-race-xian-duan-shu-wei-hu-qian-zhui-zui-zhi-tan-x) （前面的推导过于复杂，这里仅给出转化后的题意）
>
>给出 $a_i,b_i$，设 $c_i=a_i-\min\limits_{j=1}^{i} b_j$，支持 $b$ 的区间加，查询 $c_i\le k$ 的 $i$ 的最大值。

线段树上每个节点维护 $\min \{a_i\},\min \{b_i\}$ 和仅考虑这个区间时右子树的答案 $ans=\min \{c_i\}$。对于修改操作，直接给 $b$ 加，给 $ans$ 减并打 tag 。对于 pushup 操作，用一个类似楼房重建的每次向一边递归的函数维护当前节点信息，代码长这样

  ```cpp
  ll pushup(int ro,int l,int r,ll p)
  {
  	if(l==r)return t[ro].ami-p;
  	pushdown(ro);
  	return t[ro<<1].bmi<p?min(pushup(ro<<1,l,mid,p),t[ro].ans):min(t[ro<<1].ami-p,pushup(ro<<1|1,mid+1,r,p));
  }
  ```

  对于查询操作，一般情况下可以直接线段树上二分，但这里略有不同。考虑一个 $query(ro,x)$ 操作，表示前缀最小值为 $x$ 时 $c_i\le k$ 的 $i$ 的最大值

  * $bmi_{ls}<x$ 时，$x$ 不影响右区间，那么 $ans_{ro}\le k$ 时向右子树递归，否则向左子树递归，时间复杂度为 $O(\log n)$
  * $bmi_{ls}\ge x$ 时，左子树完全被 $x$ 控制了。先向右子树递归，如果有合法的直接 return；否则在左子树中查$a_i-x\le k$ 的最大的 $i$ ，移项得 $a_i\le k+x$，这就是一个经典的线段树上二分了。因为最多进行 $O(\log n)$ 次线段树上二分，总复杂度为$O(\log^2n)$

  代码长这样

  ```cpp
  int query(int ro,int l,int r,ll &x)
      {
          if(l==r)
          {
              int tmp=t[ro].ami-x<=m?l:0;
              x=min(x,t[ro].bmi);
              return tmp;
          }
          pushdown(ro);
          if(x>t[ro<<1].bmi)
          {
              if(t[ro].ans<=m)return query(ro<<1|1,mid+1,r,x=t[ro<<1].bmi);
              int tmp=query(ro<<1,l,mid,x);
              x=min(x,t[ro].bmi);
              return tmp;
          }
          else 
          {
              int tmp=(t[ro<<1].ami<=m+x?query2(ro<<1,l,mid,m+x):0);
              return max(tmp,query(ro<<1|1,mid+1,r,x));
          }
      }
  ```

##### 其他题目

（感觉这个 trick 外面可以套很多东西，全是大思维题）

* [P4425 [HNOI/AHOI2018]转盘](https://www.luogu.com.cn/problem/P4425)
* PKUSC2021 D2T2

## 九、线段树维护历史值

* #### 历史最大值

  >例题13：[P4314 CPU监控](https://www.luogu.com.cn/problem/P4314)
  >
  >维护一个序列 $a$，支持区间加，区间赋值，查询区间最大值，查询区间历史最大值。
  >
  >$1\le n,m\le 10^5$

  先考虑如果只有加操作怎么做。假设每个点开了一个队列，存这个点被打过的所有标记，那么 `pushdown` 操作即为将父亲节点的队列中的元素全部放进儿子节点的队列，每放入一个值，则更新 $x\leftarrow x'+tag,mx\leftarrow\max(mx,x)$。但我们不可能真的存一个队列。设队列中加法标记的前缀和为 $s[1\dots k]$，则所有更新进行完后应有 $x\leftarrow x'+s_k,mx\leftarrow\max\{x'+s_i\}=x'+\max\{s_i\}$。那么我们只需要维护历史加的最大值即可。这个东西怎么维护呢？考虑合并两个队列（的前缀和）$s_{son}[1\dots k_1],s_{fa}[1\dots k_2]$ 的过程

  $$
  s_{son}[i]=\begin{cases}s_{son}'[i]\quad(i\le k_1)\\s'_{son}[k_1]+s_{fa}[i-k_1]\quad(k_1<i\le k_1+k_2)\end{cases}
  $$

  则 $\max\{s_{son}[i]\}=\max(\max \{s'_{son}[i]\},s_{son}'[k_1]+\max\{s_{fa}[i]\})$ ，则我们需要维护 $s_{son}[k_1]$ ，即目前的加法标记即可。总结一下，代码如下

  ```cpp
  void getsum(int ro,int sum,int hsum)//hsum:父节点上一次pushdown后的历史加法标记最大值
  {
   t[ro].hsum=max(t[ro].hsum,t[ro].sum+hsum);//历史加法标记最大值
   t[ro].ans[1]=max(t[ro].ans[1],t[ro].ans[0]+hsum);//历史最大值
   t[ro].ans[0]+=sum;//当前最大值
   t[ro].sum+=sum; //当前加法标记
  }
  ```

  再加上赋值操作后，如果队列中有两种标记，不便于处理。可以发现，若存在一个赋值标记，则这个区间中的所有数会变成一样的，那么之后的加法标记都可以看成赋值标记。因此，此时的队列可以表示为一个加法标记队列紧跟着一个赋值标记队列。加法标记按上面说的处理。对于赋值标记 $a[1\dots k]$，最终产生的贡献即为 $\max\{a_i\}$，再维护一个历史最大赋值标记即可。

  >例题14：[P6109 [Ynoi2009] rprmq](https://www.luogu.com.cn/problem/P6109) 
  >
  >有一个 $n \times n$ 的矩阵，初始全是 0，有 $m$ 次矩形加操作和 $q$ 次矩形查询最大值操作。先进行所有修改操作，然后进行所有查询操作。 
  >
  >$n,m\le 5\times 10^4,q\le 5\times 10^5$

  首先要注意到，这题是所有修改进行完后再查询！

  对于修改操作，可以看作在 $l_1$ 时刻对 $[l_2,r_2]$ 区间加 $x$ ，在 $r_1+1$ 时刻对 $[l_2,r_2]$ 区间减 $x$，这样就把矩形的一维转变为了时间。询问变成了区间历史最大值。

  如何查询规定时间区间的历史最大值呢？考虑对时间进行猫树分治，分别处理左、右区间内部的询问和跨越中点的询问。处理一个区间前保证 $[1,l)$ 的修改已经进行，然后进行 $[l,mid]$ 的修改。对于右区间，先打一个 tag 把历史最大值重置为当前最大值，边修改边处理询问，这样查询到的历史最大值就是 $[mid,r]$ 时间的历史最大值了。然后撤回右区间的修改并再次重置历史最大值，在 $[l,mid]$ 倒着撤回修改并查询历史最值。

  实现时要注意的细节：如果有多个修改操作在同一时刻，必须按加的值从小到大处理，因为如果同时 $+x,-y$ ，可能的历史最值应为 $x-y$ 而非 $x$ ；打重置标记前必须先下放标记。

  ##### 其他题目

  * [P6349 [PA2011]Kangaroos](https://www.luogu.com.cn/problem/P6349) （这个其实是 KDT，但维护 tag 的方法是相同的）

    

* #### 历史版本和

  问题：维护一个数列 $A$，要求支持区间加，区间查历史版本和。

  记历史版本和序列为 $B$，并将“更新 $B$ 序列”也看作一个操作，每次修改完都进行一次这个操作，进行这个操作的方法是给全局打上一个 `upd` 标记。记 $sum$ 表示区间和，$hsum$ 表示历史版本和，$s[i]$ 表示前 $i$ 个标记中加法操作的前缀和。仍然假设每个点开了一个队列，存这个点被打过的所有标记。队列中的元素对$hsum$ 的贡献为
  $$
  \sum[q[i]=upd](sum+s[i]\times len)=sum\sum[q[i]=upd]+len\sum[q[i]=upd]s[i]
  $$
  则我们需要维护 $\sum[q[i]=upd]$ 和 $\sum[q[i]=upd]s[i]$，即更新标记的个数和加法标记的历史版本和。合并两个队列时，更新标记的个数直接把两部分加起来即可
  $$
  \begin{aligned}
  &\sum[q_3[i]=upd]s_3[i]\\
  =&\sum_{i=1}^{k_1}[q_1[i]=upd]s_1[i]+\sum_{i=1}^{k_2}[q_2[i]=upd](s_2[i]+s_1[k_1])\\
  =&s_1[k_1]\sum[q_2[i]=upd]+\sum_{i=1}^{k_1}[q_1[i]=upd]s_1[i]+\sum_{i=1}^{k_2}[q_2[i]=upd]s_2[i]\\
  \end{aligned}
  $$
  再维护 $s[k]$ ，即当前的加法标记即可。

  >例题15：[P3246 [HNOI2016]序列](https://www.luogu.com.cn/problem/P3246)（未实现的口胡做法）
  >
  >给出一个序列，每次询问一个区间的所有子区间最小值之和。
  >
  >$1\le n,q\le 10^5$

  设 $f[l][r]$ 表示 $[l,r]$ 中的最小值。每次询问的答案是 $f$ 的一个矩阵的和。

  按照 $r$ 从小到大的顺序维护，将不同的 $r$ 看作不同历史版本，**将矩阵和转化为历史版本和**。$r$ 每右移一次，会使一个区间的 $l$ 对应的 $f[r]$ 发生改变，可用单调栈维护区间端点，用线段树维护 $f[r]$ 发生的改变。

  **总结：二维问题可将一维看作时间轴，转化为历史版本问题。**

  ##### 其他题目：

  * [CF997E Good Subsegments](https://www.luogu.com.cn/problem/CF997E)


## 十、势能分析与区间最值操作

线段树能够通过打懒标记实现区间修改的条件有两个：

* 能够快速处理懒标记对区间询问结果的影响

* 能够快速实现懒标记的合并

有这样一类线段树题目：

* 不能通过懒标记实现区间修改，即满足 $[l,r]\subset [ql,qr]$ 也有可能继续递归，直到区间 $[l,r]$ 满足某些条件。
* 题目中的操作会使信息量趋向于减小（如开根、整除、按位与、取模等），使区间出现某种特征，如递减、趋同等。

此时应尽可能地进行剪枝，得到的做法经势能分析均摊复杂度可能是对的。

>例题16：[Loj6029. 「雅礼集训 2017 Day1」市场](https://loj.ac/p/6029)
>
>给出一段长度为 $n$ 的序列，要求支持 $m$ 次操作，包括：
>
>- 1 l r c，对于 $i\in[l,r]$，$a_i\leftarrow a_i+c$
>- 2 l r d，对于 $i\in[l,r]$，$a_i\leftarrow \lfloor\frac{a_i}{d}\rfloor$
>- 3 l r，询问 $\min\limits_{i=l}^{r} a_i$
>- 4 l r，询问 $\sum\limits_{i=l}^r a_i$
>
>$1\le n,m\le 10^5,c\in[-10^4,10^4],d\in[2,10^9]$

“ 容均摊”法： 找出一种能概括信息复杂度的“特征值”，证明其消长和时间消耗有关，最终通过求和得到复杂度。 

本题中线段树上一个节点的**容**为该区间数字的极差，即 $\max\limits_{i=l}^{r} a_i-\min\limits_{i=l}^{r} a_i$。记值域大小为 $S$

如果极差为 0 ，则所有数据均相同，之后的修改打标记即可。

如果极差不为 0 ，暴力递归下去，每次除法操作可以让极差除以 $d$，除 $O(\log S)$ 次极差就会变为 0。

整区间的操作后容不降，非整区间的操作可能使容增大，每次修改最多涉及到 $O(\log n)$ 个非整区间，最多使容增加 $O(\log n\log S)$。总复杂度 $O((n+m\log n)\log S)$

然而下取整可能带来 1 的偏差，因此极差为 1 的时候除完可能极差不变。例如 $5,6, 5, 6$ 除以 $3$ 后变为 $1,2,1,2$。这种情况需要特殊处理，这里给出一个网上通用的方法：观察发现此时两种数除前后的差相同（$5-1=6-2=4$），则将除法变为减法。极差为 0 时的除法也可以转化为减法，则只需要维护加法标记即可。

>例题16：[CF765F Souvenirs](https://www.luogu.com.cn/problem/CF765F)（未自己实现，做法来自 CF 官方题解）
>
>给出一个长为 $n$ 的序列 $a$，有 $m$ 组询问。
>
>每组询问给出 $l,r$，求 $\min\limits_{i,j\in[l,r],i\neq j}|a_i-a_j|$
>
>$1 \leq n \leq 10^5,1 \leq m \leq 3\times 10^5,0 \leq a_i \leq 10^9$

将询问离线，每次移动右端点，维护此时每个左端点对应的答案，查询后缀 $\min$

去掉绝对值：只考虑 $j<i$ 且 $a_j\ge a_i$ 的情况，将 $a$ 值翻转再做一次即可。

找到 $i$ 左边所有可能产生答案的左端点。首先找到 $j<i$ 且 $a_j\ge a_i$ 的最大 $j$，这可以用线段树上二分实现。若还存在一个可能产生答案的 $j'$，则 $a_j>a_{j'}>a_i$ 且 $a_{j'}-a_i<a_j-a_{j'}$，移项得 $a_{j'}-a_i<\frac{1}{2}(a_j-a_i)$ ，差值每次减半，则最多有 $O(\log a)$ 个可能产生答案的左端点。用树状数组维护每个左端点对应的答案。

总复杂度 $O(n\log n\log a+m\log n)$

### Segment tree Beats

* 问题 1（HDU5306 Gorgeous Sequence）：

给定一个数列 $a$，有 $m$ 次操作，操作有两种：区间 $[l,r]$ 中的所有数变成 $\min(a_i,x)$；查询区间和。 

大致思路：

* 维护最大值 $mx$，最大值的个数 $cnt$，严格次大值 $mx2$

* $mx\le x$，不会修改当前区间，直接返回

* $mx2 \le x<mx$ ，本次操作只改变最大值，$mx\leftarrow x$，用 $mx2,cnt$ 算贡献

* $x<mx2$ ，继续递归

复杂度分析：

每个节点的容为该区间的数字种类数，所有点容之和为 $O(n\log n)$。若一个点递归时被经过，其容至少减少 1。总复杂度 $O((n+m)\log n)$

* 问题 2：同时有取 $\min,\max$ 操作

$\min,\max$ 操作都会让容变小，上述复杂度分析仍成立。再维护最小值和次小值即可。

* 问题 3：加入区间加操作

维护两套标记，一套负责最大值，另一套负责非最大值。

操作可看作两类：对区间最大值加，对区间所有数加

标记下传时，最大值标记只下放给最大值较大的儿子（相同则都下传）。

吉老师论文给出的复杂度：$O(m\log^2  n)$，而实际运行中更接近 $O(m\log n)$。事实上，只要在给定操作集合下能够维护区间最大值、最大值个数、次大值，这一复杂度均成立。吉司机线段树同时也支持查询历史最值。

>例题 17：[CF1290E Cartesian Tree](https://www.luogu.com.cn/problem/CF1290E)
>
>给定序列 $a$。初始为空。 第 $i$ 次操作会将 $i$ 插入到 $a$ 的某一个位置中。每次插入后询问如果按照大根方式建出笛卡尔树，所有子树大小之和。 
>
>$n \le 150000$  

考虑笛卡尔树的结构，记 $i$ 左/右边第一个比 $i$ 大的为 $l_i/r_i$。则 $(l_i,r_i)$ 中的数均在笛卡尔树上 $i$ 的子树内，子树大小为 $r_i-l_i-1$。$r_i$ 与 $l_i$ 无关，可以分别求。以 $r$ 为例，插入当前最大值 $x$ 到位置 $p$ 后

* $i<p$，$r_i\leftarrow \min(r_i,p)$。即区间取 $\min $
* $r_p=x+1$。即单点修改
* $i>p$，$r_i\leftarrow r_i+1$。即区间加

直接套用 beats 即可。

### 参考资料

* [从《楼房重建》出发浅谈一类使用线段树维护前缀最大值的算法](https://www.luogu.com.cn/blog/PinkRabbit/Segment-Tree-and-Prefix-Maximums) （by 小粉兔）
* [关于线段树上的一些进阶操作](https://www.luogu.com.cn/blog/command-block/guan-yu-xian-duan-shu-shang-di-yi-suo-jin-jie-cao-zuo)（by command_block）
* [NOI 一轮复习 III：数据结构](https://www.luogu.com.cn/blog/ix-35/noi-yi-lun-fu-xi-iii-shuo-ju-jie-gou)（by ix35）
* 《区间最值操作与历史最值问题》（by 吉如一）

### 附赠题单

[线段树进阶](https://www.luogu.com.cn/training/185693)

