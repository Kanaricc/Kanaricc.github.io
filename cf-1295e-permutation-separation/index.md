# [CF 1295E] Permutation Separation

You are given a permutation 𝑝1,𝑝2,…,𝑝𝑛 (an array where each integer from 1 to 𝑛 appears exactly once). The weight of the 𝑖-th element of this permutation is 𝑎𝑖.

At first, you separate your permutation into two non-empty sets — prefix and suffix. More formally, the first set contains elements 𝑝1,𝑝2,…,𝑝𝑘, the second — 𝑝𝑘+1,𝑝𝑘+2,…,𝑝𝑛, where 1≤𝑘<𝑛.

After that, you may move elements between sets. The operation you are allowed to do is to choose some element of the first set and move it to the second set, or vice versa (move from the second set to the first). You have to pay 𝑎𝑖 dollars to move the element 𝑝𝑖.

Your goal is to make it so that each element of the first set is less than each element of the second set. Note that if one of the sets is empty, this condition is met.

For example, if 𝑝=[3,1,2] and 𝑎=[7,1,4], then the optimal strategy is: separate 𝑝 into two parts [3,1] and [2] and then move the 2-element into first set (it costs 4). And if 𝑝=[3,5,1,6,2,4], 𝑎=[9,1,9,9,1,9], then the optimal strategy is: separate 𝑝 into two parts [3,5,1] and [6,2,4], and then move the 2-element into first set (it costs 1), and 5-element into second set (it also costs 1).

Calculate the minimum number of dollars you have to spend.

## 分析

首先要解决的问题是如何断开。枚举排列的某个位置断开，似乎没有什么好办法转移，枚举数值上断开的位置显得更可靠一些。不过断开数值后，仍然没有什么好办法去直接求得从排列的何处断开。

考虑再次枚举排列的断开位置。当确定排列的一个断点后，由于已经知道两个子集所包含的数，可以$O(N)$算出到达目标状态需要的代价，在此以左侧小右侧大为准。当向右移动断点后，注意到左侧原有的需要移动数字**仍然需要移动**。于是，针对一个子集的分割，能够用通过线段树$O(n\lg n)$来求出在排列任何一个位置断开的代价。

接下来考虑数值上的断点移动后是否能够维护代价变化。当一个新的数字$a$被从大集合划归到小集合后，在排列中，以该数字$a$位置，左侧所有的断点都需要付出额外的代价将该数挪到左侧，而右侧所有的断点都不再需要支付代价来将该数挪到右侧。于是，这个代价也是可以维护的。

## 代码

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
#include <cmath>
#include <limits>
using namespace std;
using ll=long long;
const int MAXN=200010;
 
int lc[MAXN*4],rc[MAXN*4];
ll flag[MAXN*4],dmin[MAXN*4];
int idx=0;
void build(int &n,int l,int r){
    if(!n)n=++idx;
    if(l==r){
        flag[n]=dmin[n]=0;
        return;
    }
    int mid=(l+r)/2;
    build(lc[n],l,mid);
    build(rc[n],mid+1,r);
}
 
void pushdown(int n){
    if(!flag[n])return;
    flag[lc[n]]+=flag[n];
    flag[rc[n]]+=flag[n];
    dmin[lc[n]]+=flag[n];
    dmin[rc[n]]+=flag[n];
    flag[n]=0;
}
 
void collect(int n){
    dmin[n]=min(dmin[lc[n]],dmin[rc[n]]);
}
 
void modify(ll x,int l,int r,int L,int R,int n){
    if(l<=L && R<=r){
        flag[n]+=x;
        dmin[n]+=x;
        return;
    }
    pushdown(n);
    int mid=(L+R)/2;
    if(l<=mid)modify(x,l,r,L,mid,lc[n]);
    if(mid<r)modify(x,l,r,mid+1,R,rc[n]);
    collect(n);
}
 
ll query(int l,int r,int L,int R,int n){
    if(l<=L && R<=r){
        return dmin[n];
    }
    pushdown(n);
    int mid=(L+R)/2;
    ll res=numeric_limits<ll>::max();
    if(l<=mid)res=min(res,query(l,r,L,mid,lc[n]));
    if(mid<r)res=min(res,query(l,r,mid+1,R,rc[n]));
    return res;
}
int root=0;
int permu[MAXN];
ll wage[MAXN];
int pos[MAXN];
int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
 
 
    int nlen;cin>>nlen;
    for(int i=1;i<=nlen;i++)cin>>permu[i];
    for(int i=1;i<=nlen;i++)pos[permu[i]]=i;
    for(int i=1;i<=nlen;i++)cin>>wage[i];
    int ptr=0;
 
    if(nlen==1){
        cout<<0<<endl;
        return 0;
    }
    //init
    //以第i位切割，前i个为一个集合
    build(root,1,nlen);
    for(int i=1;i<=nlen-1;i++){
        modify(wage[i],i,nlen-1,1,nlen,root);
    }
 
    ll ans=numeric_limits<ll>::max();
    while(ptr<=nlen){
        ans=min(ans,query(1,nlen-1,1,nlen,root));
        ptr++;
        if(pos[ptr]-1>=1)
            modify(wage[pos[ptr]],1,pos[ptr]-1,1,nlen,root);
        if(pos[ptr]<=nlen-1)
            modify(-wage[pos[ptr]],pos[ptr],nlen-1,1,nlen,root);
    }
 
    cout<<ans<<endl;
 
 
    return 0;
}
```
