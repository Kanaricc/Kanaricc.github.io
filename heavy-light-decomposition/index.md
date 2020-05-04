# 树链剖分

树链剖分可以用来维护树上路径的信息。把树上的节点拆成不超过$O(\log n)$段连续的路径（链），以映射到线段树或者什么的构来维护数据。

依照子树的大小，将最大子树作为重边，其他子树作为轻边，从而在树上拆分出多条链。为同一条链上的节点连续编号，完成剖分。至于剩下的，依照编号一条链可以连续的映射到数据结构的某个取间内，用类似于倍增LCA的方法在树上得到每一个询问所覆盖的链的起始与结尾然后继续其他操作就可以了。

## 通常策略

* 对于子树的大小统计，dfs即可。
* 之后节点编号使用先序的dfs序，并且**优先进入重边**，以保证链上节点的编号连续。
* 为了在链上快速跳转，还需要记录每个节点所在链的头。

具体的代码可以看下面两个题目。~~其实我也是从别处抄了板子~~

## HDU 3966 Aragorn's Story

题目要求区间修改某条路径上所有节点的权值，查询某个点的权值。

就剖，剖完线段树还是树状数组区间修改单点查询。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <cmath>
#include <map>
#include <queue>
#include <cstring>
#include <cstdio>
using namespace std;
using ll=long long;
using pii=pair<int,int>;
const int MAXN=50010;

vector<int> g[MAXN];
int vlen,elen,qlen;

int tick=0;
int fa[MAXN],sz[MAXN],dep[MAXN];
int son[MAXN],top[MAXN];
int dfn[MAXN],rnk[MAXN];

void dfs1(int u,int f){
	fa[u]=f;
	dep[u]=dep[f]+1;
	sz[u]=1;
	son[u]=0;
	for(auto v:g[u]){
		if(v==f)continue;
		dfs1(v,u);
		sz[u]+=sz[v];
		if(sz[v]>sz[son[u]]){
			son[u]=v;
		}
	}
}

void dfs2(int u,int f){
	top[u]=f;
	dfn[u]=++tick;
	rnk[tick]=u;
	if(son[u]==0)return;
	dfs2(son[u],f);
	for(auto v:g[u]){
		if(v!=son[u] && v!=fa[u])dfs2(v,v);
	}
}
const int FTN=4*(MAXN+10);
int ft[FTN];
int lowbit(int x){
	return x&-x;
}
void ftadd(int pos,int x){
	if(pos<=0)return;
	for(;pos<FTN;pos+=lowbit(pos))ft[pos]+=x;
}
int ftget(int pos){
	if(pos<=0)return 0;
	int res=0;
	for(;pos;pos-=lowbit(pos))res+=ft[pos];
	return res;
}

int query(int pos){
	return ftget(dfn[pos]);
}

void modify(int l,int r,int x){
	ftadd(l,x);
	ftadd(r+1,-x);
}

void solve(int x,int y,int delta){
	int fx=top[x],fy=top[y];
	
	while(fx!=fy){
		if(dep[fx]>dep[fy])modify(dfn[fx],dfn[x],delta),x=fa[fx];
		else modify(dfn[fy],dfn[y],delta),y=fa[fy];

		fx=top[x];
		fy=top[y];
	}
	if(x!=y){
		if(dfn[x]<dfn[y])modify(dfn[x],dfn[y],delta);
		else modify(dfn[y],dfn[x],delta);
	}else modify(dfn[x],dfn[y],delta);
}

int num[MAXN];

int main(){

	while(~scanf("%d%d%d",&vlen,&elen,&qlen)){
		for(int i=0;i<=vlen;i++)g[i].clear();
		memset(ft,0,sizeof(ft));
		memset(fa,0,sizeof(fa));
		memset(sz,0,sizeof(sz));
		memset(dep,0,sizeof(dep));
		memset(son,0,sizeof(son));
		memset(top,0,sizeof(top));
		memset(dfn,0,sizeof(dfn));
		memset(rnk,0,sizeof(rnk));
		tick=0;

		for(int i=1;i<=vlen;i++){
			scanf("%d",num+i);
		}
		for(int i=0;i<elen;i++){
			int u,v;
			scanf("%d%d",&u,&v);
			g[u].push_back(v);
			g[v].push_back(u);
		}
		dfs1(1,0);
		dfs2(1,1);
		/*
		for(int i=1;i<=vlen;i++)
			cout<<dfn[i]<<" ";
		cout<<endl;
		*/
		for(int i=1;i<=vlen;i++){
			modify(dfn[i],dfn[i],num[i]);
		}
		/*
		for(int i=1;i<=vlen;i++){
			cout<<query(i)<<" ";
		}
		cout<<endl;
		*/

		while(qlen--){
			char opt[10];
			scanf("%s",opt);
			if(opt[0]=='I' || opt[0]=='D'){
				int l,r,x;
				scanf("%d%d%d",&l,&r,&x);
				if(opt[0]=='D')x=-x;
				solve(l,r,x);
			}else{
				int u;
				scanf("%d",&u);
				printf("%d\n",query(u));
			}
		}
	}	


    return 0;
}

```

## CF343D Water Tree

稍微有点不一样。修改1要求为点的整个子树打标记，修改2要求取消节点到根节点上的标记。

按照dfs序的编号方式，为每一个节点记录进入时间和离开时间，就能得到该节点子树映射后的连续区间。那么浇水就可以套一个线段树直接改了。清空水往根节点跳然后修改一路上经过的链。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <cmath>
#include <map>
#include <queue>
#include <cstring>
#include <cstdio>
using namespace std;
using ll=long long;
using pii=pair<int,int>;
const int MAXN=500010;

vector<int> g[MAXN];
int vlen,elen,qlen;

int tick=0;
int fa[MAXN],sz[MAXN],dep[MAXN];
int son[MAXN],top[MAXN];
int dfn[MAXN],outdfn[MAXN];

void dfs1(int u,int f){
	fa[u]=f;
	dep[u]=dep[f]+1;
	sz[u]=1;
	son[u]=0;
	for(auto v:g[u]){
		if(v==f)continue;
		dfs1(v,u);
		sz[u]+=sz[v];
		if(sz[v]>sz[son[u]]){
			son[u]=v;
		}
	}
}

void dfs2(int u,int f){
	top[u]=f;
	dfn[u]=++tick;
	if(son[u]==0){
		outdfn[u]=++tick;
		return;
	}
	dfs2(son[u],f);
	for(auto v:g[u]){
		if(v!=son[u] && v!=fa[u])dfs2(v,v);
	}
	outdfn[u]=++tick;
}
const int FTN=5*(MAXN*2+10);

int dat[FTN];
int lc[FTN],rc[FTN];
int idx=0;
int root=0;

void build(int &n,int l,int r){
	if(!n)n=++idx;
	if(l>=r){
		dat[n]=2;
		return;
	}
	int mid=(l+r)/2;
	build(lc[n],l,mid);
	build(rc[n],mid+1,r);
}
void pushdown(int n){
	if(dat[n]==0)return;
	dat[lc[n]]=dat[n];
	dat[rc[n]]=dat[n];
	dat[n]=0;
}
void fill_water(int l,int r,int L,int R,int n){
	if(l<=L && R<=r){
		dat[n]=1;
		return;
	}
	pushdown(n);

	int mid=(L+R)/2;
	if(l<=mid)fill_water(l,r,L,mid,lc[n]);
	if(mid<r)fill_water(l,r,mid+1,R,rc[n]);
}
void f**k_water(int l,int r,int L,int R,int n){
	if(l<=L && R<=r){
		dat[n]=2;
		return;
	}
	pushdown(n);

	int mid=(L+R)/2;
	if(l<=mid)f**k_water(l,r,L,mid,lc[n]);
	if(mid<r)f**k_water(l,r,mid+1,R,rc[n]);
}

void F**K_WATER(int u){
	while(u!=0){
		//cout<<"clear "<<dfn[u]<<endl;
		f**k_water(dfn[top[u]],dfn[u],1,tick,root);
		u=fa[top[u]];
	}
}

int query(int pos,int L,int R,int n){
	//1有水
	if(L==pos && R==pos)return dat[n]==1;
	pushdown(n);

	int mid=(L+R)/2;
	if(pos<=mid)return query(pos,L,mid,lc[n]);
	else return query(pos,mid+1,R,rc[n]);
}



int main(){
	ios::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);

	cin>>vlen;
	elen=vlen-1;
	for(int i=1;i<=elen;i++){
		int u,v;cin>>u>>v;
		g[u].push_back(v);
		g[v].push_back(u);
	}

	dfs1(1,0);
	dfs2(1,1);

	build(root,1,tick);
	/*
	for(int i=1;i<=vlen;i++){
		cout<<dfn[i]<<" ";
	}
	cout<<endl;
	for(int i=1;i<=vlen;i++){
		cout<<outdfn[i]<<" ";
	}
	cout<<endl;
	*/

	int qlen;cin>>qlen;
	while(qlen--){
		int opt,u;cin>>opt>>u;
		if(opt==1){
			fill_water(dfn[u],outdfn[u]-1,1,tick,root);
			//cout<<"add "<<dfn[u]<<"-"<<outdfn[u]<<endl;
		}else if(opt==2){
			F**K_WATER(u);
		}else{
			cout<<query(dfn[u],1,tick,root)<<"\n";
		}
	}

    return 0;
}
```
