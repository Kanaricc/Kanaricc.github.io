# Fansblog

Farmer John keeps a website called ‘FansBlog’ .Everyday , there are many people visited this blog.One day, he find the visits has reached P , which is a prime number.He thinks it is a interesting fact.And he remembers that the visits had reached another prime number.He try to find out the largest prime number Q ( Q < P ) ,and get the answer of Q! Module P.But he is too busy to find out the answer. So he ask you for help. ( Q! is the product of all positive integers less than or equal to n: n! = n * (n-1) * (n-2) * (n-3) *… * 3 * 2 * 1 . For example, 4! = 4 * 3 * 2 * 1 = 24 )


First line contains an number T(1<=T<=10) indicating the number of testcases.
Then T line follows, each contains a positive prime number P (1e9≤p≤1e14)

<!--more-->


# 分析
这题得知道2个结论，然而我都不知道。

## 威尔逊定理
当P为质数时，$(P-1)! \equiv -1 \pmod P$.

注意这里$!$是阶乘，不是取反的意思。

## 素数分布
当范围变大时，素数的出现频率增高，寻找一素数的相邻素数复杂度逐渐趋近于线性。

所以，寻找素数P的前一个素数可以直接暴力找。找到之后利用$(P-1)! \equiv -1 \pmod P$即可快速由$P-1$的阶乘通过逆元转到$Q$的阶乘，这题就做完了。

因为在计算逆元时会爆ll，使用快速乘法来避免，复杂度符合要求。

# 代码
```cpp
#include <iostream>
#include <cmath>
#include <cstdlib>
#include <algorithm>
using namespace std;
using ll=long long;
inline ll qmul(ll x,ll y,ll q){
    ll res=0;
    for(;y;y>>=1,x=(x+x)%q){
        if(y&1)res=(res+x)%q;
    }
    return res;
}


inline ll qpow(ll x,ll a,ll q){
    ll res=1;
    for(a;a;a>>=1,x=qmul(x,x,q)){
        if(a&1)res=qmul(res,x,q);
    }
    return res;
}

inline ll get_rev(ll x,ll q){
    return qpow(x,q-2,q);
}

inline bool is_prime(ll x){
    for(ll i=2;i<=sqrt(x);i++){
        if(x%i==0)return false;
    }
    return true;
}

inline ll factor(ll l,ll r,ll q){
    ll res=1;
    for(ll i=l;i<=r;i++)res=qmul(res,i,q);
    return res;
}

int main(){
    ios::sync_with_stdio(false);
    int kase;cin>>kase;
    while(kase--){
        ll x;cin>>x;
        ll prex=x-1;
        while(!is_prime(prex))prex--;

        ll ans=qmul(x-1,get_rev(factor(prex+1,x-1,x),x),x);
        cout<<ans<<endl;
    }

    return 0;
}
```
