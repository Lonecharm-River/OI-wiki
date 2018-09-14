学习状压 dp 之前，请确认你已经完成了[动态规划初步](/dp/)部分内容的学习

(建议学习[位运算](/math/bit/)部分的内容)

### 状压 dp 简介

状压 dp 是动态规划的一种，借由将状态压缩（通常压缩为某整形）以达到节约空间和时间的目的

#### 常用格式

```cpp
int maxn=1<<n; //规定状态的上界
for (int i=0;i<maxn;i++){
	if (i&(i<<1)) continue;//如果i情况不成立就忽略
	Type[++top]=i;//记录情况i到Type数组中
}
for (int i=1;i<=top;i++){
	if (fit(situation[1],Type[i]))
    	dp[1][Type[i]]=1;//初始化第一层
}
for (int i=2;i<=层数(dp上界);i++){
	for (int l=1;l<=top;l++)//穷举本层情况
    	for (int j=1;j<=top;j++)//穷举上一层情况(上一层对本层有影响时)
        	if (situation[i],Type[l]和Type[j]符合题意)
            	dp[i][l]=dp[i][l]+dp[i-1][j];//改变当前层(i)的状态(l)的方案种数
}
for (int i=1;i<=top;i++) ans+=dp[上界][Type[i]];
```

#### 典型例题

[旅行商问题/售货员的难题](https://www.luogu.org/problemnew/show/P1171)

其递推式为：

$$f[V][0]=0$$
$$f[S][v]=min\{ f[S\cup \{ u\}][u]+d(v,u)|u\notin S, v\in S\}$$

其中$f[S][v]$表示的是从$v$出发访问所有不属于集合$S$的顶点，最终回到$0$的路径的权重总和的最小值。而集合$V$表示为所有城市的集合，不难理解从$0$到$0$就是路程为$0$，并且再将其它所有情况均赋值为无穷大（找一个达不到的值即可），以防计算不存在的情况。大家也可以举几个例子帮助自己理解。

```cpp
#include <cstdio>
#include <algorithm>
#define INF 20000
using namespace std;
int f[(1<<20)][20], d[20][20];
int main()
{
    int i, j, k, n, smax;
    scanf("%d", &n);
    smax = (1 << n) - 1;
    for(i = 0; i < n; i += 1)
        for(j = 0; j < n; j += 1)
            scanf("%d", &d[i][j]);
    for(i = 0; i <= smax; i += 1)
        for(j = 0; j < n; j += 1)
            f[i][j] = INF;
    f[smax][0] = 0;
    for(i = smax-1; i; i -= 1)
        for(j = 0; j < n; j += 1)
            if(i >> j & 1)
                for(k = 0; k < n; k += 1)
                    if(!(i >> k & 1))
                        f[i][j] = cmin(f[i][j], f[1<<k|i][k] + d[j][k]);
    printf("%d", f[0][0]);
    return 0;
}
```

但是面对如此卡常的题目，不开O2优化怎么拿满分呢（不能用状压DP以外的算法）？我们回过头来再看递推式。

最开始$S$二进制位（位数小于$N$）全部是$1$的时候，**仅$0$号点为$0$，其它都是$INF$**。而有些$INF$是可以推到底层的。那什么样的情况满足其$f$恒为$INF$呢？首先注意到$f[V][x],x\neq 0$的情况恒为$INF$，那这个状态又会推到哪里呢？根据递推式：

$$f[S][v]=min\{ f[S\cup \{ u\}][u]+d(v,u)|u\notin S, v\in S\}$$

我们有：

$$f[S][v]=min\{ f[V][u]+d(v,u)|u\neq \{ 0\} ,u\notin S, v\in S\}$$

容易得到$f[S][v]$必然为$INF$（因为$u$只有一种可能）。

同理，对于递推式：

$$f[S][v]=min\{ f[S\cup \{ u\}][u]+d(v,u)|\{ 0\} \in S ,u\notin S, v\in S\}$$

$f[S][v]$必然由一个比当前集合$S$（包含元素$0$）多一个元素的集合$S'$得来，而$S'$又以类似方式得来，最终它们共同的来源均为：

$$f[V][u]+d(v,u)|u\neq \{ 0\} ,u\notin S, v\in S$$

故对于任何满足$\{ 0\} \in S$的$f[S][v]$，它们的值恒为$INF$。用二进制表示法来说，**只要$S\And 1$为真，那么就无需考虑**。

那么程序就可以加速了，从而拿到$100$分！

```cpp
#include <cstdio>
#define INF 20000
int f[(1<<20)][20], d[20][20];
inline int cmin(int a, int b)
{
    return a < b ? a : b;
}
int main()
{
    int i, j, k, n, smax;
    scanf("%d", &n);
    smax = (1 << n) - 1;
    for(i = 0; i < n; i += 1)
        for(j = 0; j < n; j += 1)
            scanf("%d", &d[i][j]);
    for(i = 0; i <= smax; i += 1)
        for(j = 0; j < n; j += 1)
            f[i][j] = INF;
    f[smax][0] = 0;
    for(i = smax-1; i; i -= 2)/*此处优化*/
        for(j = 0; j < n; j += 1)
            if(i >> j & 1)
                for(k = 0; k < n; k += 1)
                    if(!(i >> k & 1))
                        f[i][j] = cmin(f[i][j], f[1<<k|i][k] + d[j][k]);
    int ans = INF;
    for(i = 1; i < n; i += 1)
        ans = cmin(ans, f[1 << i][i] + d[0][i]);
    printf("%d", ans);
    return 0;
}
```

[\[USACO06NOV\] 玉米田 Corn Fields](https://www.luogu.org/problemnew/show/P1879)

显然，这是一道典型的动态规划题目，但由于方案数过多，应使用状压 dp 避免超时

本题所 "压缩" 的是 "每行可行的状态" 和 "每行土地的状态", 而储存答案的 dp 数组就应同时体现这两个特点 (所以本题 dp 数组为二维)

具体实现方法同上方伪代码

[例题代码](https://www.luogu.org/paste/kto3ua68)
