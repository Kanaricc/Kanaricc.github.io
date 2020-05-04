# Colorful Tree

There is a tree having n nodes, the i-th node of which has a type of color, denoted by an integer $c_i$

The path between every two nodes is unique, of which we define the value is the number of distinct types of colors appearing on it.

Calculate the sum of values of all possible paths, $\frac{n(n-1)}{2}$ in total, between two different nodes on the tree.

## 输入范围
多组数据,约50;节点数$2 \times 10^5$.

# 分析
没想出来该怎么做.统计路径颜色的答案没有什么合并的好方法,同时也不能按照每种颜色单独考虑.

后来经过dalao点拨,该题中的"统计一条路径上颜色种类"的要求可以转化为求其**反面**"没有某种颜色的路径有多少种".

如此,对于每一种颜色就可以使用$n(n-1)/2-size$来求其对答案的贡献了.So,来解决这个问题.

原图是一棵树,如果某点$N$为颜色$c$,那么经过$N$的子树任意点跨越$N$的路径都有该颜色.所有不包括该颜色$c$的路径只**能**出现在以$N$为切点的其他2部分.考虑其中一个部分,任意选中其中2个节点即可构建一条路径,但还是必须满足刚刚的条件(不能跨越颜色$c$的节点).以dfs递归进去即可.

考虑能否递归合并已有答案.能够得到的数据有*子节点中2部分的节点数目*,显然能够合并出以当前节点为界划分的2部分中的一部分,自然可以在递归返回后推出另一部分.

# 代码
```cpp
#include <iostream>
#include <set>
#include <algorithm>
#include <cstdlib>
#include <cstring>
#include <cmath>
#include <vector>
using namespace std;
typedef long long ll;

const int MAXV=200010;
vector<int> g[MAXV];
inline void adde(int u,int v){
    g[u].push_back(v);
}
int color[MAXV];
ll sz[MAXV];
ll gsz;
ll res=0;
void dfs(int u,int fa){
    gsz++;
    for(auto v:g[u]){
        if(v==fa)continue;
        ll b_gsz=gsz;
        ll b_sz=sz[color[u]];
        dfs(v,u);

        ll d=(gsz-b_gsz)-(sz[color[u]]-b_sz);
        res+=d*(d-1)/2;
        sz[color[u]]+=d;
    }
    sz[color[u]]++;
}
int flag[MAXV];
int main(){
    ios::sync_with_stdio(false);
    int n;
    int kase=0;
    while(cin>>n){
        for(int i=1;i<=n;i++)g[i].clear();
        memset(sz,0,sizeof(sz));
        gsz=0;
        res=0;

        vector<int> discol;
        for(int i=1;i<=n;i++){
            cin>>color[i];
            flag[color[i]]=1;
        }
        for(int i=1;i<=n;i++){
            if(flag[i])discol.push_back(i);
        }
        for(int i=0;i<n-1;i++){
            int u,v;cin>>u>>v;
            adde(u,v);
            adde(v,u);
        }

        int disnum=discol.size();
        dfs(1,0);
        ll ans=(ll)disnum*n*(n-1)/2;
        for(auto i:discol){
            ll t=n-sz[i];
            ans-=t*(t-1)/2;
        }
        cout<<"Case #"<<++kase<<": "<<ans-res<<endl;
    }

    return 0;
}
```
