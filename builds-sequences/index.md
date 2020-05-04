# [???] Builds Sequences

Alice 想要得到一个长度为 n 的序列，序列中的数都是不超过 m 的正整数，而且这 n 个数的和是 p 的倍数。

Alice 还希望，这 n 个数中，至少有一个数是质数。

Alice 想知道，有多少个序列满足她的要求。

<div>$$1 \leq n \leq 10 ^ 9, 1 \leq m \leq 2 \times 10 ^ 7, 1 \leq p \leq 100$$</div>

## 分析

这题做得很爽…

首先是一个套路，维护倍数可以转为维护当前和的模。因此如果递推的话，只需要100个状态，看起来是可做的。但是这个n的范围实在是有点，先列一下再说。

把2e7个数字全部模完，它们对答案的贡献只和剩余类有关。这样我们就能知道在和的模为a时有多少种方案能够凑到b了。得出递推方程

<div>$$
f(i,j)=\sum_{0 \leq k < p} f(i-1,(j-k) \mod 3)c(k)
$$</div>

其中$c(k)$是模为k的数有多少个。

至少有一个质数的条件转为无质数，从所有的c中扣去质数后再算一遍方案数，二者相减。

最后，关于n，这个公式能够用快速幂加速。

## 代码

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
using pii=pair<int,int>;
using ll=long long;
const int MAXN=110;
const int MAXM=20000010;
const int MOD=20170408;

bool isnp[MAXM];
vector<int> primes;

void init(int n){
    isnp[1]=1;
    for(int i=2;i<=n;i++){
        if(!isnp[i]){
            primes.push_back(i);
        }
        for(int j=0;j<primes.size() && i*primes[j]<=n;j++){
            isnp[i*primes[j]]=1;
            if(i%primes[j]==0){
                break;
            }
        }
    }
}

struct Mat{
    ll a[MAXN][MAXN];

    Mat(){
        memset(a,0,sizeof(a));
    }

    Mat operator*(const Mat &other){
        Mat res;
        for(int i=1;i<MAXN;i++){
            for(int j=1;j<MAXN;j++){
                for(int k=1;k<MAXN;k++){
                    res.a[i][j]=(res.a[i][j]+a[i][k]*other.a[k][j]%MOD)%MOD;
                }
            }
        }
        return res;
    }

    void debug(){
        for(int i=1;i<MAXN;i++){
            for(int j=1;j<MAXN;j++){
                cout<<a[i][j]<<" ";
            }
            cout<<endl;
        }
    }

    void normalize(){
        for(int i=1;i<MAXN;i++) {
            a[i][i] = 1;
        }
    }
};

Mat qpow(Mat a,int b){
    Mat res;
    res.normalize();

    for(;b;b>>=1,a=a*a){
        if(b&1)res=res*a;
    }

    return res;
}
Mat base;
int pool[MAXN],nopool[MAXN];
Mat incprime,noprime;
int main() {
    ios::sync_with_stdio(false);

    int n,m,p;cin>>n>>m>>p;
    for(int i=1;i<=m;i++){
        pool[i%p]++;
        nopool[i%p]++;
    }
    init(m);
    for(auto it:primes){
        nopool[it%p]--;
    }

//    for(auto it:nopool){
//        cout<<it<<" ";
//    }
//    cout<<endl;

    for(int j=1;j<=p;j++){
        for(int i=1;i<=p;i++){
            //cout<<((i-1-(j-1))%p+p)%p<<endl;
            incprime.a[i][j]=pool[((j-1-(i-1))%p+p)%p];
            noprime.a[i][j]=nopool[((j-1-(i-1))%p+p)%p];
        }
    }

    base.a[1][1]=1;

    incprime=base*qpow(incprime,n);
//    incprime.debug();
    noprime=base*qpow(noprime,n);
//    noprime.debug();

    cout<<((incprime.a[1][1]-noprime.a[1][1])%MOD+MOD)%MOD<<endl;

    return 0;
}
```
