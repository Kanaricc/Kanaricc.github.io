# 一些筛法的题

## 技巧

### 自然溢出

* 自然溢出不会影响低位数据,所以有的时候你不需要取模,而是一个unsigned.

### 除法取模
对于式子

<div>$$
\frac {a \times b}{c} \mod p \equiv \frac{a \times b \mod cp}{c}
$$</div>



## 细节
* 注意数据类型,例如`6*(ll)(1<<30)`是要出问题的
* 注意函数在$f(1)$位置的取值,不要忘记初始化

## 单点$\phi$的求解

```cpp
ll euler_phi(ll n) {
    ll m = sqrt(n + 0.5);
    ll ans = n;
    for (ll i = 2; i <= m; i++)
        if (n % i == 0) {
            ans = ans / i * (i - 1);
            while (n % i == 0) n /= i;
        }
    if (n > 1) ans = ans / n * (n - 1);
    return ans;
}
```

## Divisor

Given $n$ and $m$ ($1 \leq n,m \leq 5 \times 10^4$),  please calculate

<!--more-->

### 分析
下面所有的除法都是舍去小数的整除.

<div>$$
\begin{aligned}
\sum_{i=1}^n\sum_{j=1}^m \sigma_0(ij) &= \sum_{i=1}^n\sum_{j=1}^m \sum_{p|i}\sum_{q|j}[p \perp q] \\
&=\sum_{i=1}^n\sum_{j=1}^m \sum_{p|i}\sum_{q|j}\sum_{d|(p,q)}\mu(d) \\
&=\sum_{i=1}^n\sum_{j=1}^m \sum_{p|i}\sum_{q|j}\sum_{d}\mu(d)[d|p][d|q] \\
&=\sum_d \mu(d)\sum_{i=1}^n\sum_{j=1}^m \sum_{pd} \sum_{qd}[pd|i][qd|j] \\
&=\sum_d \mu(d) \sum_{pd} \sum_{qd}\sum_{i=1}^{n/pd}\sum_{j=1}^{m/qd} \\
&=\sum_d \mu(d) \sum_{pd} \sum_{qd} \frac n {pd}\frac m {qd} \\
&=\sum_d \mu(d) \sum_p^{n/d} \frac n {pd} \sum_q^{m/d} \frac m {qd} \\
\end{aligned}
$$</div>

设$S(n)=\sum_{1\leq i \leq n} \frac n i$

<div>$$
\begin{aligned}
\sum_{i=1}^n\sum_{j=1}^m \sigma_0(ij) &=\sum_d \mu(d)S(n/d)S(m/d)
\end{aligned}
$$</div>

$n/d$的取值在一个区间中是相同的,因此可以把这个求和公式分块计算.(在分块后,需要获知该段区域内$\mu$的和,所以需要求前缀和)由于$n/d$的取值为$O(\sqrt n)$的级别,因此在知道$S$的值的情况下,每个询问可以这个复杂度中计算出来.

对于$\mu$,使用筛法,并求出其前缀和.

对于$S$,直接暴力计算.

### 代码
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
using ll=long long;
const int MAXN=50010;

bool isn_p[MAXN];
vector<int> primes;
int mu[MAXN];
int premu[MAXN];
void init_prime(int len){
    isn_p[1]=1;
    mu[1]=1;
    for(int i=2;i<=len;i++){
        if(!isn_p[i]){
            primes.push_back(i);
            mu[i]=-1;
        }

        for(int j=0;j<primes.size() && i*primes[j]<=len;j++){
            isn_p[i*primes[j]]=1;
            mu[i*primes[j]]=mu[i]*-1;

            if(i%primes[j]==0){
                mu[i*primes[j]]=0;
                break;
            }
        }
    }
    premu[0]=0;
    for(int i=1;i<=len;i++)premu[i]=premu[i-1]+mu[i];
}


ll S[MAXN];
void init_S(int len){
    for(int i=1;i<=len;i++){
        for(int l=1,r;l<=i;l=r+1){
	        //remember this line
            r=i/(i/l);
            S[i]+=(ll)(r-l+1)*(i/l);
        }
    }
}
int main(){
    ios::sync_with_stdio(false);
    init_prime(50000);
    init_S(50000);
    int kase;cin>>kase;
    while(kase--){
        int n,m;cin>>n>>m;
        ll ans=0;
        // the minimum one will fastly approach to 0, leading the extra parts of bigger one do nothing to the answer.
        int minn=min(n,m);
        for(int l=1,r;l<=minn;l=r+1){
            r=min(n/(n/l),m/(m/l));
            ans+=S[n/l]*S[m/l]*(premu[r]-premu[l-1]);
        }
        cout<<ans<<endl;
    }

    return 0;
}
```

## table
给出多组$n,m,a$,求

<div>$$
\sum_{i=1}^n \sum_{j=1}^m \sigma_1((i,j))[\sigma_1((i,j)) \geq a]
$$</div>

### 分析
> 这道题让谁都能看出来重点是如何处理条件
>
> <div>$$[\sigma_1((i,j)) \geq a]$$</div>

可是这我显然不知道该怎么做.

对询问作以$a$从小到大离线处理,询问前先处理新增$a$的影响.

在已经满足条件的前提下对公式作化简.

<div>$$
\begin{aligned}
\sum_{i=1}^n \sum_{j=1}^m \sigma_1((i,j)) &= \sum_d \sum_{i=1}^n \sum_{j=1}^m \sigma_1(d)[(i,j)=d] \\
&=\sum_d \sum_{i\leq \frac nd}\sum_{j\leq \frac md}\sigma_1(d)[(i,j)=1] \\
&=\sum_d \sum_{i\leq \frac n{td}}\sum_{j\leq \frac m{td}}\sigma_1(d) \sum_t \mu(t) \\
&=\sum_d \sum_t \mu(t) \sigma_1(d)  \sum_{i\leq \frac n{td}}\sum_{j\leq \frac m{td}} 1 \\
&=\sum_T \lfloor \frac nT \rfloor \lfloor \frac mT \rfloor \sum_{d|T}\mu(t)\sigma_1(\frac Td)
\end{aligned}
$$</div>

可以看到还是套路,引入d,引入$\mu$,之后胡乱化简.

按照这个式子,需要计算的就是$g(x)=\sum_{d|T}\mu(t)\sigma_1(\frac Td)$的前缀和.

接下来是如何处理条件...

当$a$每扩大一点,就有一部分$\mu(t)\sigma_1(\frac Td)$被加入到函数$g$的各个部分.使用一种数据结构来维护$g$的前缀和,例如树状数组.$a$最大到,每次受到影响的就是$d$的倍数,以此可以计算出总的时间复杂度为

<div>$$
WTF
$$</div>

至此,问题就解决了.

### 代码
又丑又长.
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cassert>
using namespace std;
using ll=long long;
const int MAXN=100010;
const int MAXQ=100010;
const int MAXFT=400010;
const ll P=(ll)1<<31;


ll FT[MAXFT];
ll lowbit(int x){
    return x&-x;
}
void ftadd(int pos,ll x){
    for(int i=pos;i<MAXN;i+=lowbit(i)){
        FT[i]=(FT[i]+x);
    }
}
ll ftget(int pos){
    ll res=0;
    for(int i=pos;i;i-=lowbit(i)){
        res=(res+FT[i]);
    }
    return res;
}

ll qpow(ll a,ll b,ll p){
    ll res=1;
    for(b;b;b>>=1,a=(a*a)%p){
        if(b&1)res=(res*a)%p;
    }
    return res;
}
ll qpow(ll a,ll b){
    ll res=1;
    for(b;b;b>>=1,a=(a*a)){
        if(b&1)res=(res*a);
    }
    return res;
}

bool is_np[MAXN];
ll sigma[MAXN],mu[MAXN];
int t[MAXN];
vector<int> primes;
struct Sig{
    ll sigma;
    int x;
    Sig(){}
    Sig(int x,ll sigma):x(x),sigma(sigma){}
};
vector<Sig> sigma_vec;

void init(int n){
    is_np[1]=1;
    sigma[1]=1;
    mu[1]=1;
    for(int i=2;i<=n;i++){
        if(!is_np[i]){
            primes.push_back(i);
            sigma[i]=i+1;
            t[i]=1;
            mu[i]=-1;
        }

        for(int j=0;j<primes.size() && i*primes[j]<=n;j++){
            is_np[i*primes[j]]=1;
            sigma[i*primes[j]]=sigma[i]*sigma[primes[j]];
            t[i*primes[j]]=1;
            mu[i*primes[j]]=mu[i]*-1;

            if(i%primes[j]==0){
                t[i*primes[j]]=t[i]+1;
                sigma[i*primes[j]]=sigma[i/qpow(primes[j],t[i])]*((ll)1-qpow(primes[j],(t[i]+1)+1))/(1-primes[j]);
                mu[i*primes[j]]=0;
                break;
            }
        }
    }
    for(int i=1;i<=n;i++){
        sigma_vec.push_back(Sig(i,sigma[i]));
    }
}

struct Q{
    int n,m,a;
    int i;
    ll ans;
    bool operator<(const Q &b)const{
        return a<b.a;
    }
} qs[MAXQ];

ll f(int n){
    return ftget(n);
}

int curidx=0;
void mergea(int newa){
    for(;curidx<sigma_vec.size();curidx++){
        Sig &sig=sigma_vec[curidx];
        if(sig.sigma>newa)break;
        for(int i=sig.x;i<MAXN;i+=sig.x){
            ftadd(i,mu[i/sig.x]*sig.sigma%P);
        }
    }
}

int main(){
    init(100000);
    sort(sigma_vec.begin(),sigma_vec.end(),[](Sig &a,Sig &b){
        return a.sigma<b.sigma;
    });

    int qlen;cin>>qlen;
    for(int i=0;i<qlen;i++){
        Q &q=qs[i];
        scanf("%d%d%d",&q.n,&q.m,&q.a);
        q.i=i;
    }
    sort(qs,qs+qlen,[](Q &a,Q &b){
        return a.a<b.a;
    });

    for(int i=0;i<qlen;i++){
        Q &q=qs[i];
        int n=q.n,m=q.m,a=q.a;
        ll &ans=q.ans=0;
        mergea(a);
        for(int l=1,r;l<=min(n,m);l=r+1){
            r=min(n/(n/l),m/(m/l));
            ans=ans+(n/l)*(m/l)*(f(r)-f(l-1));
        }
        
    }
    sort(qs,qs+qlen,[](Q &a,Q &b){
        return a.i<b.i;
    });
    for(int i=0;i<qlen;i++){
        printf("%lld\n",qs[i].ans%P);
    }
    return 0;
}
```

## product
定义斐波纳妾(?)(linux这输入法够魔性)函数$f(x)$

求

<div>$$
\prod_{i=1}^n \prod_{j=1}^m f((i,j))
$$</div>

### 分析
首先,先引个$d$是没错了.

但是

这道题,我又不会.我不知道该怎么处理$\prod$...蔡就完事了.

现在来看,当引入一个$d$后,在该求积公式里出现了相同项相乘.将该部分的计算调整为幂,剩下的就又都一样了.

<div>$$
\begin{aligned}
\prod_{i=1}^n \prod_{j=1}^m f((i,j)) &= \prod_d f(d)^{\sum_{i\leq n} \sum_{j \leq m} [(i,j)=d]} \\
&=\prod_d f(d)^{\sum_{i\leq \frac nd} \sum_{j \leq \frac md} [(i,j)=1]} \\
&=\prod_d f(d)^{\sum_t \mu(t) \sum_{i\leq \frac n{td}} \sum_{j \leq \frac m{td}} 1 } \\
&=\prod_d \prod_t f(d)^{\mu(t) \sum_{i\leq \frac n{td}} \sum_{j \leq \frac m{td}} 1 } \\
&=\prod_T (\prod_{d|T}  f(d)^{\mu(\frac Tt)})^{\lfloor \frac nT \rfloor \lfloor \frac mT \rfloor }
\end{aligned}
$$</div>

设$g(x)=\prod_{d|T}  f(d)^{\mu(\frac Tt)}$,求前缀和就完事了.

> 这道题需要使用**欧拉定理**来优化求幂的速度.

### 欧拉定理
当$(a,p)=1$,有以下式子

<div>$$
a^b \equiv a^{b \mod \phi(p)} \pmod p
$$</div>

当$(a,p)\neq 1$,有扩展欧拉定理

<div>$$
a^b \equiv a^{b \mod \phi(p)+ \phi(p)} \pmod p
$$</div>

根据这2个式子,可以在快速降幂,来加快运算.

### 代码
```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <cstring>
using namespace std;
using ll=long long;
const int MAXN=1000010;
const ll P=1e9+7;

inline ll qpow(ll a,ll b,ll p){
    ll res=1;
    for(;b;b>>=1,a=a*a%p){
        if(b&1)res=res*a%p;
    }
    return res;
}

inline ll get_inv(ll a,ll p){
    return qpow(a,p-2,p);
}

bool is_np[MAXN];
vector<int> primes;
ll f[MAXN], g[MAXN];
ll inv_f[MAXN];
ll preg[MAXN],inv_pg[MAXN];
ll mu[MAXN];
void init(int n){
    is_np[1]=1;
    mu[1]=1;
    for(int i=2;i<=n;i++){
        if(!is_np[i]){
            primes.push_back(i);
            mu[i]=-1;
        }

        for(int j=0;j<primes.size() && i*primes[j]<=n;j++){
            is_np[i*primes[j]]=1;
            mu[i*primes[j]]=mu[i]*-1;

            if(i%primes[j]==0){
                mu[i*primes[j]]=0;
                break;
            }
        }
    }

    f[1]=f[2]=1;
    for(int i=3;i<=n;i++){
        f[i]=(f[i-1]+f[i-2])%P;
    }
    for(int i=1;i<=n;i++){
        inv_f[i]=get_inv(f[i],P);
    }

    for(int i=1;i<=n;i++)g[i]=1;
    for(int i=1;i<=n;i++){
        for(int j=i;j<=n;j+=i){
            if(mu[j/i]==-1)g[j]=(g[j]*inv_f[i])%P;
            else if(mu[j/i]==1)g[j]=(g[j]*f[i])%P;
            //when mu==0,nothing happens
        }
    }
    preg[0]=1;
    for(int i=1;i<=n;i++)preg[i]=preg[i-1]*g[i]%P;
    inv_pg[0]=1;
    for(int i=1;i<=n;i++)inv_pg[i]=get_inv(preg[i],P)%P;
}

int main(){
    init(1000000);
    //cout<<"done"<<endl;
    /*
    
    for(auto i:primes){
        cout<<i<<" ";
    }
    cout<<endl;
    
    for(int i=1;i<=20;i++){
        cout<<g[i]<<" ";
    }
    cout<<endl;
    */
    

    int kase;
    scanf("%d",&kase);
    while(kase--){
        ll n,m;
        scanf("%d%d",&n,&m);
        if(n>m)swap(n,m);

        ll ans=1;
        for(ll l=1,r;l<=n;l=r+1){
            r=min(n/(n/l),m/(m/l));
            ll sum=preg[r]*inv_pg[l-1]%P;
            ans=ans*qpow(sum,(n/l)*(m/l)%(P-1),P)%P;
        }
        printf("%lld\n",ans);
    }
    return 0;
}
```

## phi3
给出$n$,求

<div>$$
(\sum_{i=1}^n\sum_{j=1}^n\sum_{k=1}^n \frac {\phi(i)\phi(j^2)\phi(k^3)}{\phi(i)\phi(j)\phi(k)} \phi((i,j,k))) \mod 2^{30}
$$</div>

### 分析
这道题重点在于

<div>$$\frac {\phi(i)\phi(j^2)\phi(k^3)}{\phi(i)\phi(j)\phi(k)}$$</div>

这堆东西的化简.很显然,我又不会.

观察$\phi(n)$的公式

<div>$$
\phi(n)=n\prod_i(1-\frac 1{\phi(i)})
$$</div>

可以得出上面那一堆等于$jk^2$.剩下的就又都一样了.

<div>$$
\begin{aligned}
\sum_{i=1}^n\sum_{j=1}^n\sum_{k=1}^n jk^2 \phi((i,j,k)) &=\sum_d\sum_{i=1}^n\sum_{j=1}^n\sum_{k=1}^n jk^2 \phi(d)[(i,j,k)=d] \\
&=\sum_d \sum_{t} \phi(d) \mu(t) t^3d^3 \sum_{i\leq \frac n{td}}\sum_{j \leq \frac n{td}}\sum_{k \leq \frac n{td}} jk^2 \\
&=\sum_T \sum_{i\leq \frac nT}\sum_{j \leq \frac nT} j\sum_{k \leq \frac nT}k^2 (T^3 \sum_{d|T} \phi(d) \mu(\frac Td))
\end{aligned} 
$$</div>

设$g(n)=\sum_{d|T} \phi(d) \mu(\frac Td)$,直接筛.最后计算前缀和的时候再乘上$n^3$就可以了.

> 这道题的取模还有这种处理方法: 直接取数组为`unsigned`并自然溢出.**二进制后30位**不会受到影响.

### 代码
```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <string>
#include <vector>
using namespace std;
using ull=unsigned long long;
using ll=long long;
const int MAXN=10000010;
const unsigned P=1<<30;

bool isnp[MAXN];
vector<int> primes;
int t[MAXN],M[MAXN];
unsigned Mt[MAXN];
unsigned g[MAXN];
ll preg[MAXN];

void init(int n){
    isnp[1]=1;
    g[1]=1;

    for(int i=2;i<=n;i++){
        if(!isnp[i]){
            primes.push_back(i);
            g[i]=i-1-1;

            t[i]=1;
            M[i]=Mt[i]=i;
        }
        for(int j=0;j<primes.size() && i*primes[j]<=n;j++){
            int newone=i*primes[j];
            isnp[newone]=1;
            g[newone]=g[i]*g[primes[j]]%P;
            t[newone]=1;
            M[newone]=Mt[newone]=primes[j];
            if(i%primes[j]==0){
                t[newone]=t[i]+1;
                Mt[newone]=Mt[i]*primes[j]%P;
                
                g[newone]=g[i/Mt[i]]*(Mt[newone]+Mt[i]/primes[j]-2*Mt[i])%P;
                break;
            }
        }
    }

    for(int i=1;i<=n;i++){
        preg[i]=preg[i-1]+(ll)i*i%P*i%P*g[i]%P;
        preg[i]%=P;
    }
}

int main(){
    init(10000000);

    int kase;
    scanf("%d",&kase);
    while(kase--){
        int n;
        scanf("%d",&n);
        ll ans=0;
        for(int l=1,r;l<=n;l=r+1){
            r=n/(n/l);
            ull lim=n/l;
            ull sum1=lim*(lim+1)%(2*P)/2;
            ull sum2=lim*(lim+1)%(6*P)*(2*lim+1)%(6*P)/6;
            ans=ans+lim*sum1%P*sum2%P*(preg[r]-preg[l-1])%P;
        }
        printf("%lld\n",ans%P);
    }
    return 0;
}
```
