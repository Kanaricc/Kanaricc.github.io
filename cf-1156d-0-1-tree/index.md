# [CF 1156D] 0-1 Tree

You are given a tree <!--more--> (an undirected connected acyclic graph) consisting of n vertices and n−1 edges. A number is written on each edge, each number is either 0 (let's call such edges 0-edges) or 1 (those are 1-edges).

Let's call an ordered pair of vertices (x,y) (x≠y) valid if, while traversing the simple path from x to y, we never go through a 0-edge after going through a 1-edge. Your task is to calculate the number of valid pairs in the tree.

## 分析

当经过一条`1`，就不能再经过`0`。所以路径只会有（称`1`为黑，`0`为白）

* 全`0`
* 全`1`
* 从某处划分后一半为`1`一半为`0`

维护从点`u`到其子树任意节点的白色路径，黑色路径总数。那么答案就可以统计了。

1. 从子树到自己的黑，白路径
2. 子树到子树的黑，白路径
3. 以`u`为分界点的黑白路径。包括子树到子树，子树到非子树。

子树到非子树的黑白节点已经包括在第1，2部分。

## 代码

```cpp
#include <algorithm>
#include <cstring>
#include <iostream>
#include <queue>
#include <vector>
#include <stack>
#include <map>
#include <set>
using namespace std;
using ll=long long;
using pii=pair<int,int>;
const int MAXV=200010,MAXE=400020;


struct Edge{
	int v,n,c;
}edges[MAXE];
int head[MAXV],idx=0;
void adde(int u,int v,int c){
	edges[++idx].v=v;
	edges[idx].n=head[u];
	edges[idx].c=c;
	head[u]=idx;
}
//从子树到本节点为白色路径或黑色路径的总数.
//不包括自己.
ll white[MAXV],black[MAXV];

void dfsCal(int u,int fa){
	for(int ei=head[u];ei;ei=edges[ei].n){
		Edge &e=edges[ei];
		if(e.v==fa)continue;
		dfsCal(e.v,u);

		white[u]+=e.c==0;
		black[u]+=e.c==1;
		if(e.c==0)white[u]+=white[e.v];
		if(e.c==1)black[u]+=black[e.v];
	}
}
ll ans=0;
void solve(int u,int fa,ll w,ll b){
	//子树到我
	ans+=black[u]*2;
	ans+=white[u]*2;
	//子树到子树
	for(int ei=head[u];ei;ei=edges[ei].n){
		Edge &e=edges[ei];
		if(e.v==fa)continue;

		if(e.c==1){
			ans+=(black[e.v]+1)*(black[u]-(black[e.v]+1));
		}
		if(e.c==0){
			ans+=(white[e.v]+1)*(white[u]-(white[e.v]+1));
		}

		if(e.c==0){
			ans+=(white[e.v]+1)*black[u];
		}

		
	}
	//跨过我
	ans+=b*white[u];
	ans+=w*black[u];

	//cout<<"arrive at "<<u<<" and collect "<<ans<<endl;

	for(int ei=head[u];ei;ei=edges[ei].n){
		Edge &e=edges[ei];
		if(e.v==fa)continue;

		if(e.c==1){
			solve(e.v,u,0,b+(black[u]-(black[e.v]+1)+1));
		}else{
			solve(e.v,u,w+(white[u]-(white[e.v]+1)+1),0);
		}
	}
}
int main(){
    ios::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);

	int nlen;cin>>nlen;
	for(int i=1;i<=nlen-1;i++){
		int u,v,c;
		cin>>u>>v>>c;
		adde(u,v,c);
		adde(v,u,c);
	}

	dfsCal(1,0);

	for(int i=1;i<=nlen;i++){
		//cout<<white[i]<<" "<<black[i]<<endl;
	}

	solve(1,0,0,0);
	cout<<ans<<endl;


	return 0;
}
```
