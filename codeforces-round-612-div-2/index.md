# Codeforces Round #612 (Div. 2)

## C
无脑DP.设状态$f(i,j,0/1)$表示已经设置了$i$位,用了$j$个偶数,第$i$位是偶数/奇数.然后

<div>$$
\begin{aligned}
f(i,j,0)&=\min\{f(i-1,j-1,0),f(i-1,j-1,1)+1\} \\
f(i,j,1)&=\min\{f(i-1,j,0)+1,f(i-1,j,1)\}
\end{aligned}
$$</div>

以$i=1$作为开始,因为第1位不能算代价.

```cpp
// missing?
```

## D
一个事实是,两棵子🌳的间大小关系不会影响其内部节点的既定关系.

以此,直接dfs,从底层向上构造相对大小顺序.合并时子树间的相对大小没有影响,所以直接前后接在一起就好,再把当前节点插到合适的位置使其满足c的条件.
全部完成后再对节点统一标号.

最开始先标了号,然后越写越麻烦,生气.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;
const int MAXN=2010;
const int MAXV=2010,MAXE=4020;
int tick=0;
 
struct Edge{
    int v,n;
}edges[MAXE];
int head[MAXV];
int idx=0;
int sz[MAXV];
void adde(int u,int v){
    edges[++idx].v=v;
    edges[idx].n=head[u];
    head[u]=idx;
}
int name[MAXV];
int cur[MAXV][MAXV];
int dfs0(int u,int fa){
    sz[u]=1;
    for(int ei=head[u];ei;ei=edges[ei].n){
        Edge &e=edges[ei];
        if(e.v==fa)continue;
        sz[u]+=dfs0(e.v,u);
    }
    if(sz[u]==1){
        name[u]=tick;
        cur[u][tick]++;
        tick+=2000;
    }
    return sz[u];
}
int target[MAXV];
bool ok=1;
vector<int> dfs(int u,int fa){
    vector<int> res;
    for(int ei=head[u];ei;ei=edges[ei].n){
        Edge &e=edges[ei];
        if(e.v==fa)continue;
        vector<int> cur=dfs(e.v,u);
        for(auto it=cur.begin();it!=cur.end();it++){
            res.push_back(*it);
        }
    }
    if(target[u]>res.size())ok=0;
    else res.insert(res.begin()+target[u],u);
    return res;
}
int main(){
    int vlen;cin>>vlen;
    int root;
    for(int i=1;i<=vlen;i++){
        int fa;
        cin>>fa>>target[i];
        if(fa!=0){
            adde(fa,i);
        }else root=i;
    }
 
    vector<int> res=dfs(root,0);
    if(ok){
        cout<<"YES"<<endl;
        for(int i=0;i<res.size();i++){
            name[res[i]]=i;
        }
        for(int i=1;i<=vlen;i++){
            cout<<name[i]+1<<" ";
        }
        cout<<endl;
    }else{
        cout<<"NO"<<endl;
    }
 
    return 0;
}
```
