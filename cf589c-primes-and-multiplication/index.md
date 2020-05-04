# [CF589C] primes and multiplication

设$primes(x)$为x的质因数的集合。

设$g(x,p)$表示可以整除x的最大的$p^k$。

设$f(x,y)$表示将x作分解后对每个质因子作用到1到y上求$g$的乘积。

求$f(x,n)$

## 分析

……最后你会发现，这题就是求阶乘质因子的乘积。

## 代码
```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <vector>
#include <cmath>
using namespace std;
const int MAXN=1010;
using ll=long long;
const ll P=1e9+7;
 
vector<ll> primes;
 
void unpack(ll x){
    ll b=x;
    for(int i=2;i<=sqrt(b)+1;i++){
        if(x%i==0)primes.push_back(i);
        while(x%i==0)x/=i;
    }
    if(x!=1)primes.push_back(x);
}
ll qpow(ll a,ll b,ll p){
    ll res=1;
    for(;b;b>>=1,a=a*a%p){
        if(b&1)res=res*a%p;
    }
    return res;
}
ll get(ll n,ll num){
    ll res=0;
    while(n){
        res=(res+n/num)%(P-1);
        n/=num;
    }
    return res;
}
int main(){
    ios::sync_with_stdio(false);
    ll x,n;cin>>x>>n;
    unpack(x);
 
    ll ans=1;
    for(ll item:primes){
        ans=ans*qpow(item,get(n,item),P)%P;
    }
    cout<<ans%P<<endl;
 
    return 0;
}
```
