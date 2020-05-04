# Distribution of Books

zz6d likes reading very much, so he bought a lot of books. One day, zz6d brought n books to a classroom in school. The books of zz6d is so popular that K students in the classroom want to borrow his books to read. Every book of zz6d has a number i (1<=i<=n). Every student in the classroom wants to get a continuous number books. Every book has a pleasure value, which can be 0 or even negative (causing discomfort). Now zz6d needs to distribute these books to K students. The pleasure value of each student is defined as the sum of the pleasure values of all the books he obtains.Zz6d didn't want his classmates to be too happy, so he wanted to minimize the maximum pleasure of the K classmates. zz6d can hide some last numbered books and not distribute them,which means he can just split the first x books into k parts and ignore the rest books, every part is consecutive and no two parts intersect with each other.However,every classmate must get at least one book.Now he wonders how small can the maximum pleasure of the K classmates be.

1<=T<=10

1<=n<=2*105 

1<=k<=n 

-109<=ai<=109

# 分析
最大值最小，考虑二分答案。思考题目是否具有单调性：当最大值极大时，书可以随便分，当最大值极小时，可能会出现无法凑齐的状况，目测满足。

题目要求分书时必须连续分，可以使用动态规划来做。假设二分的答案为lim

<div>$$f(i)=\max \{ f(j) | \sum_{k=j+1}^i a_k \leq lim  \}+1$$</div>

将求和改为前缀和，$pre(i)$。

<div>$$f(i)=\max \{ f(j) | pre(i)-pre(j-1) \leq lim  \}+1$$</div>

动态规划的复杂度为$O(n^2)$，太慢，考虑优化。

每次转移都从先前已经出现的满足要求的f中转移。限制条件转一下，就是

<div>$$pre(i)-lim \leq pre(?)$$</div>

* 当有2个f对应的前缀和相同，我们选择更大的那个

所以可以直接维护已经出现的每种前缀和所对应的最大f。可以离散化后使用权值线段树。复杂度变为$O(n\lg n)$

总复杂度为$O(n\lg n \lg n)$。

# 代码
```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;
using ll=long long;
const int MAXN=900010;

ll dat[MAXN];
int lc[MAXN],rc[MAXN];
int idx=0;
int build(int &n,int l,int r){
    if(!n)n=++idx;
    dat[n]=-0x3f3f3f3f;
    if(l>=r)return dat[n];

    int mid=(l+r)/2;
    dat[n]=max(build(lc[n],l,mid),build(rc[n],mid+1,r));
    return dat[n];
}
void collectchild(int node){
    dat[node]=max(dat[lc[node]],dat[rc[node]]);
}

int query_n(int l,int r,int L,int R,int node){
    if(l<=L && R<=r)return dat[node];
    int mid=(L+R)/2;
    int res=-0x3f3f3f3f;
    if(l<=mid)res=max(res,query_n(l,r,L,mid,lc[node]));
    if(mid<r)res=max(res,query_n(l,r,mid+1,R,rc[node]));
    return res;
}

void modify(int l,int r,ll x,int L,int R,int node){
    if(L>=R){
        dat[node]=max(dat[node],x);
        return;
    }
    int mid=(L+R)/2;
    if(l<=mid)modify(l,r,x,L,mid,lc[node]);
    if(mid<r)modify(l,r,x,mid+1,R,rc[node]);
    collectchild(node);
}
ll prefix[MAXN];
ll bprefix[MAXN];
ll num[MAXN];

int nlen,sel;
int rlen;
int root;
bool check(ll x){
    build(root,0,rlen-1);

    int zero=lower_bound(bprefix,bprefix+rlen,0)-bprefix;
    modify(zero,zero,0,0,rlen-1,root);

    for(int i=1;i<=nlen;i++){
        int start=lower_bound(bprefix,bprefix+rlen,bprefix[prefix[i]]-x)-bprefix;
        int dp=query_n(start,rlen-1,0,rlen-1,root)+1;
        if(dp>=sel)return true;
        modify(prefix[i],prefix[i],dp,0,rlen-1,root);
    }
    return false;
}


int main(){
    //debug
    /*
    int opt;
    build(root,0,10-1);
    while(cin>>opt){
        if(opt==1){
            int pos,x;cin>>pos>>x;
            modify(pos,pos,x,0,10-1,root);
        }else{
            int l,r;cin>>l>>r;
            cout<<query_n(l,r,0,10-1,root)<<endl;
        }
    }
    */

    int kase;cin>>kase;
    while(kase--){
        cin>>nlen>>sel;
        for(int i=1;i<=nlen;i++){
            cin>>num[i];
        }
        bprefix[0]=prefix[0]=0;
        for(int i=1;i<=nlen;i++)bprefix[i]=prefix[i]=prefix[i-1]+num[i];
        //discrete
        sort(bprefix,bprefix+nlen+1);
        rlen=unique(bprefix,bprefix+nlen+1)-bprefix;
        for(int i=0;i<=nlen;i++)prefix[i]=lower_bound(bprefix,bprefix+rlen,prefix[i])-bprefix;

        


        //binary
        ll l=-1e15,r=1e15;
        while(l+1<r){
            ll mid=(l+r)/2;
            if(check(mid)){
                r=mid;
            }else l=mid;
        }
        for(ll i=l;i<=r;i++){
            if(check(i)){
                cout<<i<<endl;
                break;
            }
        }
    }
    return 0;
}
```

多组询问没有清理干净数组，WA了好几发。
