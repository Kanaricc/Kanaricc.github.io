# 一点点点分治

今天仍然是被吊起来打的一天。

点分治是一种用于解决树上路径问题的思路。它的关键就是选中一点，统计经过该点的答案后删除点，再递归到两个子树中进行同样的操作。

删除的点连带删除了一些已经计算过的路径。

当每次选择重心作为根时，能保证点分治的复杂度本身为$O(n \log n)$。当然你得保证$O(n)$做完你的事情，*同样清数组时也必须只清用到的*。

印象里之前有道树上DP的题，自己yy瞎糊的算法写的难看又难调，就有点这种感觉。不过那个是$O(n)$的，大概泛用上不如这个。

## 聪聪可可

聪聪和可可是兄弟俩，他们俩经常为了一些琐事打起来，例如家中只剩下最后一根冰棍而两人都想吃、两个人都想玩儿电脑（可是他们家只有一台电脑）……遇到这种问题，一般情况下石头剪刀布就好了，可是他们已经玩儿腻了这种低智商的游戏。

他们的爸爸快被他们的争吵烦死了，所以他发明了一个新游戏：由爸爸在纸上画 nn 个“点”，并用 n-1n−1 条“边”把这 nn 个“点”恰好连通（其实这就是一棵树）。并且每条“边”上都有一个数。接下来由聪聪和可可分别随即选一个点（当然他们选点时是看不到这棵树的），如果两个点之间所有边上数的和加起来恰好是 33 的倍数，则判聪聪赢，否则可可赢。

聪聪非常爱思考问题，在每次游戏后都会仔细研究这棵树，希望知道对于这张图自己的获胜概率是多少。现请你帮忙求出这个值以验证聪聪的答案是否正确。

### 分析

统计路径。但是这题dp的话能转移。

但是还是用点分治。

枚举一个点后，设$f(u,j)$表示到$u$点模3为$j$的路径数有多少。转移很明显，统计也很显然。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <queue>
#include <cstring>
using namespace std;
const int XV=20010;
using pii=pair<int,int>;
using ll=long long;
vector<pii> g[XV];


bool vis[XV];
int sz[XV];

int dfs1(int u,int fa){
    sz[u]=1;
    for(auto [v,w]:g[u]){
        if(vis[v] || v==fa)continue;
        sz[u]+=dfs1(v,u);
    }
    return sz[u];
}
int SZ;
int rt;
int maxsz[XV];

queue<int> ready_to_clear;
void getroot(int u,int fa){
    sz[u]=1;
    ready_to_clear.push(u);
    for(auto [v,w]:g[u]){
        if(vis[v] || v==fa)continue;
        getroot(v,u);
        sz[u]+=sz[v];
        maxsz[u]=max(maxsz[u],sz[v]);
    }
    maxsz[u]=max(maxsz[u],SZ-sz[u]);
    if(rt==0 || maxsz[u]<maxsz[rt])rt=u;
}

ll ans=0;
ll f[XV][3];
void cal(int u,int fa){

    for(auto [v,w]:g[u]){
        if(vis[v] || v==fa)continue;
        cal(v,u);

        int t[3];
        for(int i=0;i<3;i++){
            t[(i+w)%3]=f[v][i];
        }
        t[w%3]++;//由v直接出发

        if(u==rt){
            //从该子树到之前所有的子树
            ans+=t[0]*f[u][0];
            ans+=t[1]*f[u][2];
            ans+=t[2]*f[u][1];
        }
        for(int i=0;i<3;i++){
            f[u][i]+=t[i];
        }
    }

    if(u==rt)ans+=f[u][0];
}
void solve(int u){
    vis[u]=1;
    while(!ready_to_clear.empty()){
        int u=ready_to_clear.front();ready_to_clear.pop();
        f[u][0]=f[u][1]=f[u][2]=0;
        maxsz[u]=0;
    }
    cal(u,0);
    //cout<<"after"<<u<<",ans="<<ans<<endl;

    for(auto [v,w]:g[u]){
        if(vis[v])continue;
        SZ=sz[v];rt=0;
        getroot(v,0);
        solve(rt);
    }
}

ll gcd(ll a,ll b){
    return !b?a:gcd(b,a%b);
}

int main(){
    ios::sync_with_stdio(false);
    int nlen;cin>>nlen;
    for(int i=1;i<=nlen-1;i++){
        int u,v,w;cin>>u>>v>>w;
        g[u].emplace_back(v,w);
        g[v].emplace_back(u,w);
    }
    memset(vis,0,sizeof(vis));
    SZ=nlen;
    rt=0;getroot(1,0);
    solve(rt);


    ll win=ans*2+nlen;
    ll all=1LL*nlen*nlen;

    ll g=gcd(win,all);

    cout<<win/g<<"/"<<all/g<<endl;
}
```


## [HDU6643] yts1999 Doesn't Like This Problem

Mr. Bread has a tree T with n vertices, labeled by 1,2,…,n. Each vertex of the tree has a positive integer value wi.

The value of a non-empty tree T is equal to w1×w2×⋯×wn. A subtree of T is a connected subgraph of T that is also a tree.

Please write a program to calculate the number of non-empty subtrees of T whose values are not larger than a given number m.

### 分析

虽然我不知道为什么就显然点分治了，但是他们说是。要DP还是容易看出来的。

点分治能够解决路径问题，这种连通性问题也可以解决。也是枚举一个点后，统计包括这个点的连通块的答案。

选中该点后，进行DP。然后是他们说的非常套路的利用dfs序将树映射到序列上的做法。以$f(i,j)$表示在$i$点处值为$j$的方案数。

选中$i$：

<div>$$f(i,ja)+=f(i+1,a)$$</div>

不选$i$，那么同时要跳过i的整个子树：

<div>$$f(i,j)+=f(i+\text{sz}(i),j)$$</div>

另外是这道题的$j$范围太大。看到一个很骚的写法是存成$j$和$M/j$，然后每个都只有$\sqrt M$这么大…我寻思你再给我一遍这个题我也yy不出来这种存法。

总之有两个东西可以注意一下：

* 对树上涉及依赖关系的DP可以考虑使用dfs序映射。
* 我不知道这个处理j的办法有没有泛用性。

### 代码

> 代码中的重心、点分治的solve应该形成较为固定的写法。

[origin](https://www.cnblogs.com/hua-dong/p/11320013.html "origin")

```cpp
/// copied
#include <iostream>
#include <cstring>
#include <algorithm>
#include <vector>
#include <cmath>

using namespace std;
using ll=long long;
const int XN=2010;

vector<int> g[XN];
int dfn[XN],sz[XN],tick=0;
int re[XN];
int vis[XN];
int wage[XN];

int m;
int square;
int dfs(int u,int fa) {
    sz[u] = 1;
    dfn[u] = ++tick;
    re[tick] = u;
    for (auto v:g[u]) {
        if (vis[v] || v==fa)continue;
        sz[u] += dfs(v, u);
    }
    return sz[u];
}
int SZ;
int maxsz[XN],root=0;
void getroot(int u,int fa) {
    sz[u] = 1;
    maxsz[u] = 0;

    for (auto v:g[u]) {
        if (vis[v] || v == fa)continue;
        getroot(v, u);
        sz[u] += sz[v];
        maxsz[u] = max(maxsz[u], sz[v]);
    }
    maxsz[u] = max(maxsz[u], SZ - maxsz[u]);
    if (root == 0 || maxsz[u] < maxsz[root])root = u;
}

const int MOD=1000000007;
int M(int &x) {
    return x %= MOD;
}

int f[XN][XN],f2[XN][XN];
int cal() {
    //默认选中root
    int res = 0;
    for (int i = 0; i <= tick + 1; i++) {
        memset(f[i], 0, sizeof(f[i]));
        memset(f2[i], 0, sizeof(f2[i]));
    }
    f[tick + 1][1] = 1;
    for (int i = tick; i >= 1; i--) {
        int w = wage[re[i]];
        for (int j = 1; j <= min(m / w, square); j++) {
            int to = w * j;
            if (to <= square)M(f[i][to] += f[i + 1][j]);
            else M(f2[i][m / to] += f[i + 1][j]);
        }
        for (int j = w; j <= square; j++) {
            M(f2[i][j / w] += f2[i + 1][j]);
        }
        //不选j，那么要跳过整个子树
        for (int j = 1; j <= square; j++)M(f[i][j] += f[i + sz[re[i]]][j]);
        for (int j = 1; j <= square; j++)M(f2[i][j] += f2[i + sz[re[i]]][j]);
    }
    for (int i = 1; i <= square; i++)M(res += f[1][i]);
    for (int i = 1; i <= square; i++)M(res += f2[1][i]);
    //去掉不选root的情况
    res--;
    return M(res += MOD);
}
int ans=0;
void solve(int u) {
    vis[u] = 1;
    tick = 0;
    dfs(u, 0);
    M(ans += cal());
    for (auto v:g[u]) {
        if (vis[v])continue;
        SZ = sz[v];
        root = 0;
        getroot(v, 0);
        solve(root);
    }
}

int main() {
    int kase;
    cin >> kase;
    while (kase--) {
        int n;
        cin >> n >> m;
        square = sqrt(m);

        for (int i = 1; i <= n; i++)g[i].clear();
        memset(vis,0,sizeof(vis));
        tick = 0;
        ans = 0;

        for (int i = 1; i <= n; i++)cin >> wage[i];

        for (int i = 1; i <= n - 1; i++) {
            int u, v;
            cin >> u >> v;
            g[u].push_back(v);
            g[v].push_back(u);
        }
        root = 0;
        SZ = n;
        getroot(1, 0);
        solve(root);
        cout << ans << endl;
    }
}

```
