# [HDU6638] Snowy Smile

There are n pirate chests buried in Byteland, labeled by 1,2,…,n. The i-th chest's location is (xi,yi), and its value is wi, wi can be negative since the pirate can add some poisonous gases into the chest. When you open the i-th pirate chest, you will get wi value.

You want to make money from these pirate chests. You can select a rectangle, the sides of which are all paralleled to the axes, and then all the chests inside it or on its border will be opened. Note that you must open all the chests within that range regardless of their values are positive or negative. But you can choose a rectangle with nothing in it to get a zero sum.

Please write a program to find the best rectangle with maximum total value.

The first line of the input contains an integer T(1≤T≤100), denoting the number of test cases.

In each test case, there is one integer n(1≤n≤2000) in the first line, denoting the number of pirate chests.

For the next n lines, each line contains three integers xi,yi,wi(−109≤xi,yi,wi≤109), denoting each pirate chest.

It is guaranteed that ∑n≤10000.


## 分析
首先，我没做出来。

<!--more-->

这道题实际上就是在要求你用小于$O(N^3)$的复杂度求出最大和子矩阵。注意到该题的点**稀疏**，所以以点为考虑对象。

三方的做法，枚举矩阵的上边界和下边界，维护纵向上的和，求最大字段和。当以点考虑时，上下边界就可以直接由**排序后**的点决定。之后，使用线段树维护最大子段和。

## 代码
```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <cstring>
using namespace std;
using ll=long long;
constexpr int MAXN=400010;

ll d_sub[MAXN],d_pre[MAXN],d_suf[MAXN];
ll d_sum[MAXN];
int lc[MAXN],rc[MAXN];
int idx=0;
void build(int &n,int l,int r){
    if(!n)n=++idx;
    d_sum[n]=d_sub[n]=d_pre[n]=d_suf[n]=0;
    if(l==r){
        return;
    }
    int mid=(l+r)/2;
    build(lc[n],l,mid);
    build(rc[n],mid+1,r);
    //combine data
}
void collect(int node){
    d_sum[node]=d_sum[lc[node]]+d_sum[rc[node]];

    d_pre[node]=max(d_pre[lc[node]],d_sum[lc[node]]+d_pre[rc[node]]);
    d_suf[node]=max(d_suf[rc[node]],d_sum[rc[node]]+d_suf[lc[node]]);

    d_sub[node]=max(max(d_sub[lc[node]],d_sub[rc[node]]),d_suf[lc[node]]+d_pre[rc[node]]);
}

void modify(int x,int l,int r,int L,int R,int node){
    if(l<=L && R<=r){
        //only single point to modify
        d_sub[node]=d_sub[node]+x;
        d_pre[node]=d_pre[node]+x;
        d_suf[node]=d_suf[node]+x;
        d_sum[node]+=x;
        return;
    }
    int mid=(L+R)/2;
    if(l<=mid)modify(x,l,r,L,mid,lc[node]);
    if(mid<r)modify(x,l,r,mid+1,R,rc[node]);

    collect(node);
}
int root;
ll query_all(){
    return d_sub[root];
}

struct Chest{
    int x,y,v;
    bool operator<(const Chest &other)const{
        if(x==other.x)return y<other.y;
        return x<other.x;
    }
} chests[MAXN];

vector<int> refy;
int main(){
    ios::sync_with_stdio(false);
    int kase;cin>>kase;
    while(kase--){
        int nlen;cin>>nlen;
        refy.clear();
        for(int i=0;i<nlen;i++){
            Chest &chest=chests[i];
            cin>>chest.x>>chest.y>>chest.v;
            refy.push_back(chest.y);
        }
        sort(chests,chests+nlen);

        sort(refy.begin(),refy.end());
        auto refyend=unique(refy.begin(),refy.end());
        int maxy=0;
        for(int i=0;i<nlen;i++){
            chests[i].y=lower_bound(refy.begin(),refyend,chests[i].y)-refy.begin()+1;
            maxy=max(maxy,chests[i].y);
        }

        int lastx=0;
        ll ans=0;

        for(int i=0;i<nlen;i++){
            if(lastx==chests[i].x)continue;
            lastx=chests[i].x;
            //cout<<"start from "<<lastx<<endl;

            build(root,1,maxy);
            int nextx=chests[i].x;
            for(int j=i;j<nlen;j++){
                if(chests[j].x!=nextx){
                    nextx=chests[j].x;
                    ans=max(ans,query_all());
                }
                modify(chests[j].v,chests[j].y,chests[j].y,1,maxy,root);
            }
            ans=max(ans,query_all());
        }
        cout<<max(0ll,ans)<<endl;
    }

    return 0;
}
```
