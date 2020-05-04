# [CF 1156F]Card Bag

You have a bag which contains n cards. There is a number written on each card; the number on i-th card is ai.

<!--more-->

You are playing the following game. During each turn, you choose and remove a random card from the bag (all cards that are still left inside the bag are chosen equiprobably). Nothing else happens during the first turn — but during the next turns, after removing a card (let the number on it be x), you compare it with the card that was removed during the previous turn (let the number on it be y). Possible outcomes are:

if x < y, the game ends and you lose;
if x = y, the game ends and you win;
if x > y, the game continues.
If there are no cards left in the bag, you lose. Cards are not returned into the bag after you remove them.

You have to calculate the probability of winning in this game. It can be shown that it is in the form of PQ where P and Q are non-negative integers and Q≠0, P≤Q. Output the value of P⋅Q−1 (mod  998244353).

## 分析

数据范围足够通过累积每轮游戏中的胜率来算。先想办法算游戏运行到第$i$轮的概率。

定义$f(i,j)$为第i次抽卡抽出$j$且游戏将继续的概率，发现

* $f(i,j)$只能从$f(i,0 \to j-1)$转移。否则游戏结束。
* 由上条，第$i$次抽出$j$的概率也可知，因为前面抽卡没有抽过$j$。

记牌总数$n$，牌面为$i$的牌有$c(i)$张，有转移

<div>$$
\begin{aligned}
f(i,j)=\frac{c(j)}{n-i+1}\sum_{0 \leq k \leq j-1}f(i-1,k)
\end{aligned}
$$</div>

有答案

<div>$$
\text{ans}=\sum_{1 \leq i,j \leq n} f(i,j)\frac{c(j)-1}{n-i+1}
$$</div>

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
const int MAXN=5010;
const ll MOD=998244353;

// 游戏进行到第i轮,抽出j且没有结束的概率

ll f[MAXN][MAXN];

int card[MAXN];
ll inv[MAXN];

ll qpow(ll a,ll b,ll p){
	ll res=1;
	for(;b;b>>=1,a=a*a%p){
		if(b&1)res=res*a%p;
	}
	return res;
}

inline ll get_inv(ll a,ll p){
	return qpow(a,p-2,p);
}

void init_inv(int n){
	for(int i=1;i<=n;i++){
		inv[i]=get_inv(i,MOD);
	}
}

int main(){
    ios::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);


	int n;cin>>n;
	init_inv(n);
	for(int i=1;i<=n;i++){
		int x;cin>>x;
		card[x]++;
	}

	f[0][0]=1;

	ll ans=0;
	for(int i=1;i<=n;i++){
		ll cnt=0;
		for(int j=0;j<i;j++)cnt=(cnt+f[i-1][j])%MOD;

		for(int j=i;j<=n;j++){
			f[i][j]=((cnt*card[j])%MOD*inv[n-i+1])%MOD;
			//cout<<"f("<<i<<","<<j<<")"<<f[i][j]<<endl;
			cnt=(cnt+f[i-1][j])%MOD;
		}
		for(int j=1;j<=n;j++){
			ans=ans+((f[i-1][j]*(card[j]-1))%MOD*inv[n-i+1])%MOD;
			ans%=MOD;
		}
	}

	cout<<ans<<endl;

	return 0;
}
```
