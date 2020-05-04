# 一点点拉格朗日插值

抄起我的混凝土数学，复习了会上升下降积分，看了看斯特林数和伯努利数。结果误把伯努利数的递推式看成了$O(n)$的，改了半天一到3次方以上就翻车，这才反应过来。

8会啊…找了很长时间才知道是拉格朗日插值。

有这么一条定理。

> 平面上的n+1个点可以确定一个n次多项式。

而$\sum i^p$是一个$p+1$次多项式，况且在$p \leq 1000000$的前提下很容易找到前$p+1$个点。如果我们找到一种方式去构造这样一个多项式问题就解决了。

~~拉格朗~~提出了这么一种方法。

## 构造

设函数

<div>$$\ell_j(x)=\prod_{k \neq j} \frac{x-x_k}{x_j-x_k}$$</div>

注意其特性

* $x_j$被带入时，其为$1$
* 其他任何$x$被带入时，会出现$0$项。

因此这个东西就可以为任何一个点对创建在多项式中的单元$y_j \ell_j(x_j)$。剩下的问题是如何快速求解。

当我们只求单点（率先带入点），且整数x连续取值时，观察到分母和分子都变为阶乘形式，且分式的构成元素在前后两项之间具有递推的关系。

<div>$$
\ell_j(x)=\frac{(x-1)(x-2)\cdots (x-(j-1))(x-(j+1))\cdots (x-n)}{(j-1)(j-2) \cdots (j-(j-1))(j-(j+1)) \cdots (j-n))}
$$</div>

可以看到，分子和分母项都能够拆分为左右两个部分。分别维护前缀后缀与阶乘逆元即可。

*阶乘并不需要全部的*，仅仅是左侧和右侧的一小部分。

## 板子

嫖的，自己的写爆了而且巨丑。

```cpp
int lagrange(int n,int *x,int *y,int k) {
    int ans=0;
    for(int i=1;i<=n;++i) {
        int f=1,g=1;
        for(int j=1;j<=n;++j) if(i!=j) {
            f=1LL*f*(k-x[j]+mod)%mod;
            g=1LL*g*(x[i]-x[j]+mod)%mod;
        }
        upd(ans,1LL*y[i]*f%mod*inv(g)%mod);
    }
    return ans;
}
```

```cpp
int lagrange(int n,int *y,int k) {
    int ans=0;
    pre[0]=suf[n+1]=1;
    for(int i=1;i<=n;++i) pre[i]=1LL*pre[i-1]*(k-i)%mod;
    for(int i=n;i>=1;--i) suf[i]=1LL*suf[i+1]*(k-i)%mod;
    for(int i=1;i<=n;++i) {
        int a=1LL*pre[i-1]*suf[i+1]%mod*inv[i-1]%mod*inv[n-i]%mod;
        if((n-i)&1) a=mod-a;
        upd(ans,1LL*a*y[i]%mod);
    }
    return ans;
}
```

## 错误的代码

这份看错了伯努利数的递推复杂度

```cpp
#include <cstdio>
#include <algorithm>
#include <string>
#include <vector>
#include <cstring>
#include <iostream>
#include <queue>
#include <set>
#include <cmath>
#include <map>
using namespace std;
using ll=long long;
const int MAXN=1000010;
const int mod=1e9+7;


ll inv[MAXN];
double B[MAXN];
double C[MAXN];

void init(int n){
    //niyuan
    inv[1]=1;
    for(int i=2;i<=n+1;i++) {
        inv[i] = inv[mod % i] * (mod - mod / i) % mod;
    }

    //C
    C[0]=1;
    for(int i=1;i<=n+1;i++){
//        C[i]=C[i-1]*(n+1-i+1)%mod*inv[i]%mod;
        C[i]=C[i-1]*(n+1-i+1)/i;
    }
    
    
    //WRONG
    //伯努利数的递推是n^2级别的，理解式子上出现了失误。
    B[0]=1;
    double s=C[0];
    for(int i=1;i<=n;i++){
//        B[i]=-inv[i+1]*s%mod;
//        s=(s+B[i]*C[i]%mod)%mod;
        B[i]=-s/(i+1);
        s=(s+B[i]*C[i]);
    }
}

ll n2[MAXN];
int main(){
    ll n,m;cin>>n>>m;

    init(m);
    for(int i=0;i<=m;i++)cout<<C[i]<<" ";
    cout<<endl;
    for(int i=0;i<=m;i++)cout<<B[i]<<" ";
    cout<<endl;

    n2[0]=1;
    for(int i=1;i<=m+1;i++){
        n2[i]=n2[i-1]*n%mod;
    }

    for(int i=0;i<=m+1;i++)cout<<n2[i]<<" ";
    cout<<endl;

    double ans=0;
    for(int i=0;i<=m;i++){
//        ans=(ans+C[i]*B[i]%mod*n2[m+1-i]%mod)%mod;
        ans=(ans+C[i]*B[i]*n2[m+1-i]);
    }
//    (ans*=inv[m+1])%=mod;
    ans=ans/(m+1);
    cout<<ans<<endl;


}
```

这份自己yy的拉格朗日插值在阶乘的求法上有问题。
```cpp
#include <cstdio>
#include <algorithm>
#include <string>
#include <vector>
#include <cstring>
#include <iostream>
#include <queue>
#include <set>
#include <cmath>
#include <map>
using namespace std;
using ll=long long;
const int MAXN=1000010;
const int mod=1e9+7;

ll fac[MAXN];
ll facinv[MAXN];
ll inv[MAXN];

ll qpow(ll a,ll b,ll p=mod){
    ll res=1;
    for(;b;b>>=1,a=a*a%p){
        if(b&1){
            res=res*a%p;
        }
    }
    return res;
}

inline ll get_inv(ll a,ll p=mod){
    return qpow(a,p-2,p);
}

inline ll Fac(int n,bool pos=true){
    int flg=1;
    if(!pos && n%2==1)flg=-1;
    return fac[n]*flg;
}

inline ll Invfac(int n,bool pos=true){
    int flg=1;
    if(!pos && n%2==1)flg=-1;
    return facinv[n]*flg;
}


void init(int n){
    fac[0]=1;
    for(int i=1;i<=n;i++)fac[i]=fac[i-1]*i%mod;
    facinv[n]=get_inv(fac[n]);
    for(int i=n-1;i>=1;i--){
        facinv[i]=facinv[i+1]*(i+1)%mod;
    }
    facinv[0]=1;

    inv[1]=1;
    for(int i=2;i<=n;i++)
        inv[i] = inv[mod % i] * (mod-mod/i) % mod;
}

ll ans=0;
ll yy[MAXN];

int main(){
    int n,m;cin>>n>>m;

    int M=m+2;
    init(max(n,M));
    for(int i=1;i<=M;i++){
        yy[i]=qpow(i,m);
    }
    for(int i=1;i<=M;i++){
        yy[i]=(yy[i]+yy[i-1])%mod;
    }
    for(int i=1;i<=M;i++)cout<<yy[i]<<" ";
    cout<<endl;

    if(n<=M){
        cout<<yy[n]<<endl;
        return 0;
    }


    for(int i=1;i<=M;i++){
        ll cell=yy[i]*Fac(n-1)%mod*Invfac(n-M-1)%mod*inv[n-i]%mod;
        (cell*=Invfac(i-1))%=mod;
        (cell*=Invfac(abs(i-M),false))%=mod;
        (ans+=cell)%=mod;
//        ll cell=yy[i]*Fac(n-1)/Fac(n-M-1)/(n-i);
//        cell/=Fac(i-1);
//        cell/=Fac(abs(i-M),false);
//        ans+=cell;
    }

    cout<<(ans+mod)%mod<<endl;
}
```
