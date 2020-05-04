# [CF 1165F2]Microtransactions

Ivan plays a computer game that contains some microtransactions to make characters look cooler. Since Ivan wants his character to be really cool, he wants to use some of these microtransactions — and he won't start playing until he gets all of them.

<!--more-->

Each day (during the morning) Ivan earns exactly one burle.

There are n types of microtransactions in the game. Each microtransaction costs 2 burles usually and 1 burle if it is on sale. Ivan has to order exactly ki microtransactions of the i-th type (he orders microtransactions during the evening).

Ivan can order any (possibly zero) number of microtransactions of any types during any day (of course, if he has enough money to do it). If the microtransaction he wants to order is on sale then he can buy it for 1 burle and otherwise he can buy it for 2 burles.

There are also m special offers in the game shop. The j-th offer (dj,tj) means that microtransactions of the tj-th type are on sale during the dj-th day.

Ivan wants to order all microtransactions as soon as possible. Your task is to calculate the minimum day when he can buy all microtransactions he want and actually start playing.

## 分析

首先，不关注到底买了哪个商品，因为每种商品的价格都一样，记$i$种商品需要$k_i$，全体商品总数$K$。

首先，如果没有任何优惠，需要$2K$天。在有优惠时，我们需要尽可能在优惠**当天**买齐需要的商品。我们放开的天数越多，钱越多，同时也越有可能遇到优惠（花的钱减少）。以此，题目具有单调性。

二分时间$t$。得出哪种商品有优惠，并从$2K$元内抵消优惠部分。尽量将买商品的决策拖到该商品最后一次优惠进行，若商品有优惠，买比不买好，我们要尽可能将手中积攒的钱扔出去。唯一的问题是越靠后的商品我们能够用之后的钱去买，但是更靠前的优惠可能会因为买了前述那种商品而没有更多钱购买。因此将每种商品购买时机拖得越晚越好，这样我们顶多会遇到没钱买下一个优惠的问题，不过二者贡献一样，这对最终答案没有影响。因为这玩意wa了一发..

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
const int MAXN=400010;

// 商品数目 打折次数
int n,m;
ll wanted[MAXN];
ll bwanted[MAXN];

vector<int> offs[MAXN];
int goodoff[MAXN];//第i种商品最后的打折时机
int maxday=0;
ll summ=0;

bool check(ll day){
	memset(goodoff,0,sizeof(goodoff));
	memcpy(wanted,bwanted,sizeof(wanted));

	for(int i=1;i<=min((ll)maxday,day);i++){
		for(auto t:offs[i]){
			goodoff[t]=i;
		}
	}
	/*
	cout<<"end at "<<day<<endl;
	for(int i=1;i<=n;i++){
		cout<<goodoff[i]<<" ";
	}
	cout<<endl;
	*/
	ll cost=summ*2;
	ll free=0;
	for(int i=1;i<=min((ll)maxday,day);i++){
		free++;
		for(auto t:offs[i]){
			if(goodoff[t]!=i)continue;
			if(free>wanted[t]){
				free-=wanted[t];
				cost-=wanted[t];
				wanted[t]=0;
			}else{
				wanted[t]-=free;
				cost-=free;
				free=0;
			}
		}
	}
	return cost<=day;
}

int main(){
    ios::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);

	
	cin>>n>>m;
	for(int i=1;i<=n;i++){
		cin>>wanted[i];
		summ+=wanted[i];
	}
	memcpy(bwanted,wanted,sizeof(bwanted));

	
	for(int i=1;i<=m;i++){
		int d,t;cin>>d>>t;
		maxday=max(maxday,d);
		offs[d].push_back(t);
	}

	ll l=0,r=5e5;
	while(r-l>1){
		ll mid=(l+r)/2;
		if(check(mid)){
			r=mid;
		}else l=mid;
	}
	cout<<r<<endl;

	return 0;
}
```
