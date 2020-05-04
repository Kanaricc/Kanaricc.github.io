# 一点点决策单调性

艹了天天都被不知道的知识点教育，知道的又做不出来。 :lei:

在动态规划中，决策单调性指对于$j < j'$，$f(j')$的最佳转移$f(i')$位置$i'$一定不小于$f(j)$的$i$。

假设

<div>$$f(r)=\min{f(k-1)+w(k,r)}$$</div>

为了证明这一点，首先假设

<div>$$f(r)=f(k-1)+w(k,r)$$</div>

此时我们有$\forall i < k$，

<div>$$f(k-1)+w(k,r) < f(i-1)+w(i,r)$$</div>

此后，将r后挪1。我们必须要证明$\forall i < k$，

<div>$$f(k-1)+w(k,r+1) < f(i-1)+w(i,r+1)$$</div>

也就是证明对于两边的两个增量$\Delta$有

<div>$$w(k,r+1) - w(k,r) < w(i,r+1) - w(i,r)$$</div>

## 使用分治

一旦有决策单调性了，我们就可以通过优于$n^2$的总复杂度完成规划。

例如使用分治。当我们找到区间$[ l, r ]$的中点$m$的决策点$p$时，那么$[l, m-1]$区间只能从不晚于$p$转移，$[m+1, r]$只能从不早于$p$转移。

l和r是当前要决策的区间，L和R是转移位置。

```cpp
void solve(int l,int r,int L,int R) {
    if(l>r)return;
    int mid=(l+r)/2;
    int p=0;
    for(int i=L;i<=min(mid,R);i++){
        //这里的下标写法要和方程配套
        if(f[i-1]+w[i,mid]<f[mid]){
            f[mid]=f[i-1]+w[i,mid];
            p=i;
        }
    }
    solve(l,mid-1,L,p);
    solve(mid+1,r,p,R);
}
```

## 【CF868F】yts1999 Doing Minimization

You are given an array of n integers a1... an. The cost of a subsegment is the number of unordered pairs of distinct indices within the subsegment that contain equal elements. Split the given array into k non-intersecting non-empty subsegments so that the sum of their costs is minimum possible. Each element should be present in exactly one subsegment.

### 分析

设$f(i,j)$表示前j个分i段。

这个题目就满足决策单调。一个更长的区间+1之后只有可能比短的区间增加更多的代价。

接下来的问题是怎么算这道题里的$w(i,j)$。$n^2$显然不行。看到有人整了个类似莫队的东西，不过我还没太弄明白这个复杂度是怎么算的，看起来是和分治的内层循环一致所以就对复杂度没有更多的贡献了。

一共进行k轮分治，$O(kn\log n)$

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;
using ll=long long;
const int XN=100010;

ll f[30][XN];
ll wage;
int cl,cr;
int num[XN];
ll cnt[XN];
void jump(int l,int r){
    while(cl<l){
        cnt[num[cl]]--;
        wage-=cnt[num[cl]];
        cl++;
    }
    while(l<cl){
        cl--;
        wage+=cnt[num[cl]];
        cnt[num[cl]]++;
    }
    while(r<cr){
        cnt[num[cr]]--;
        wage-=cnt[num[cr]];
        cr--;
    }
    while(cr<r){
        cr++;
        wage+=cnt[num[cr]];
        cnt[num[cr]]++;
    }
}
void solve(int l,int r,int L,int R,int k) {
    if(l>r)return;
    int mid=(l+r)/2;
    int p=0;
    for(int i=L;i<=min(mid,R);i++){
        jump(i,mid);
        if(f[k-1][i-1]+wage<f[k][mid]){
            f[k][mid]=f[k-1][i-1]+wage;
            p=i;
        }
    }

    solve(l,mid-1,L,p,k);
    solve(mid+1,r,p,R,k);
}
int main(){
    ios::sync_with_stdio(false);

    int n,kk;cin>>n>>kk;
    for(int i=1;i<=n;i++)cin>>num[i];
    memset(f,0x3f,sizeof(f));
    for(auto & i : f)i[0]=0;
    for(int i=1;i<=n;i++){
        wage+=cnt[num[i]];
        cnt[num[i]]++;
        f[1][i]=wage;
    }
//    for(int i=1;i<=n;i++){
//        cout<<f[1][i]<<&#039; &#039;;
//    }
//    cout<<endl;
    wage=0;cl=1;cr=0;
    memset(cnt,0,sizeof(cnt));

    for(int i=2;i<=kk;i++){
        solve(1,n,1,n,i);
    }
    cout<<f[kk][n]<<endl;

}
```

