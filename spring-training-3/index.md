# Spring Training 3

## HDU6319 Ascending Rating

Before the start of contest, there are n ICPC contestants waiting in a long queue. They are labeled by 1 to n from left to right. It can be easily found that the i-th contestant's QodeForces rating is ai.
Little Q, the coach of Quailty Normal University, is bored to just watch them waiting in the queue. He starts to compare the rating of the contestants. He will pick a continous interval with length m, say [l,l+m−1], and then inspect each contestant from left to right. Initially, he will write down two numbers maxrating=−1 and count=0. Everytime he meets a contestant k with strictly higher rating than maxrating, he will change maxrating to ak and count to count+1.
Little T is also a coach waiting for the contest. He knows Little Q is not good at counting, so he is wondering what are the correct final value of maxrating and count. Please write a program to figure out the answer.

### 分析

一开始想着从左到右滑动窗口维护一个单调队列，结果发现没法统计count。

反向滑动就好了，我还没太反应过来为啥。唯一捉住的一点想法是“数据无用”的时刻。在正向维护时，后想加入但是未被加入的元素可能因为前者pop掉而漏掉，但是反向维护时只会弹出已经使用的无用元素，任何时候新元素都会被加入。

## HDU6331 Walking Plan

There are n intersections in Bytetown, connected with m one way streets. Little Q likes sport walking very much, he plans to walk for q days. On the i-th day, Little Q plans to start walking at the si-th intersection, walk through at least ki streets and finally return to the ti-th intersection.
Little Q's smart phone will record his walking route. Compared to stay healthy, Little Q cares the statistics more. So he wants to minimize the total walking length of each day. Please write a program to help him find the best route.

### 分析

……果然还是菜，别人过得像飞一样。先是读错了题，把ki streets理解成了路径权的限制，就翻了车。后发现是k条之后，猜了发是不是倍增，又争执了很久到底最短路能不能蹦。最后结论是大概可以蹦。（既然过了那就也是了）

结果发现倍增的话答案没法统计。大哥们说分块，才终于看到了一点希望。wa了数发才过。

## HDU6325 Interstellar Travel

After trying hard for many years, Little Q has finally received an astronaut license. To celebrate the fact, he intends to buy himself a spaceship and make an interstellar travel.
Little Q knows the position of n planets in space, labeled by 1 to n. To his surprise, these planets are all coplanar. So to simplify, Little Q put these n planets on a plane coordinate system, and calculated the coordinate of each planet (xi,yi).
Little Q plans to start his journey at the 1-th planet, and end at the n-th planet. When he is at the i-th planet, he can next fly to the j-th planet only if xi<xj, which will cost his spaceship xi×yj−xj×yi units of energy. Note that this cost can be negative, it means the flight will supply his spaceship.
Please write a program to help Little Q find the best route with minimum total cost.

### 分析

这题如果你在a和b两点之间找第三点，然后计算代价差，就会发现最终的代价里出现了a到b的直线方程。代价变成了该直线上第三点的值和真实值的差， ~~可是见鬼了这是怎么发现的。~~

结论是一个点在直线之上，就要进行中转。所以最终答案大方向是求一个凸包。

## HDU6321 Dynamic Graph Matching
In the mathematical discipline of graph theory, a matching in a graph is a set of edges without common vertices.
You are given an undirected graph with n vertices, labeled by 1,2,...,n. Initially the graph has no edges.
There are 2 kinds of operations :
+ u v, add an edge (u,v) into the graph, multiple edges between same pair of vertices are allowed.
- u v, remove an edge (u,v), it is guaranteed that there are at least one such edge in the graph.
Your task is to compute the number of matchings with exactly k edges after each operation for k=1,2,3,...,n2. Note that multiple edges between same pair of vertices are considered different.

### 分析

数据范围能够状压。

用$f(i,state)$表示此次操作完后，已经选择的点有state所能构成的方案数。对于加边uv，就有如下转移。表示在未选择uv的已匹配状态里再选上uv构成新state。

<div>$$f(i,s)=\sum f(i-1,s-\{u,v\})$$</div>

表示在删除uv时，从方案中再次扣去这些状态。

<div>$$f(i,s)-=f(i,s-\{u,v\})$$</div>

把第一维滚动掉，这两个公式对转移顺序没有要求，每次更新都不会更新被用来转移的状态，瞎掰写。
