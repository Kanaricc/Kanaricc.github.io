# [CCPC 2017] Master of Sequence

给出2个数列，$\{a_i\},\{b_i\}$.要求支持以下操作

1. 修改数列a的第i个为x
2. 修改数列b的第i个为x
3. 给出k，求最小的t，满足$\sum \lfloor \frac {t-b_i}{a_i} \rfloor \ge k$

操作数不多于100000，操作3不多于1000；数列长度不多于10000，**a的值域不大于1000**。

## 分析
将$\lfloor \frac {t-b_i}{a_i} \rfloor$拆开，观察前后可能出现的差。

<div>$$
\lfloor \frac {t}{a_i} \rfloor - \lfloor \frac {b_i}{a_i} \rfloor
$$</div>

设$t=k_1a_i+c_1,b_i=k_2a_i+c_2$，代入观察。

<div>$$
\begin{aligned}
\lfloor \frac {t-b_i}{a_i} \rfloor&=\lfloor \frac {(k_1-k_2)a_i+c_1-c_2}{a_i} \rfloor \\
\end{aligned}
$$</div>

可以发现，当把式子拆开时，有2种情况。

* $c_1 > c_2$，啥事没有。
* $c_1 < c_2$，拆开后会多统计一个1。

事先统计$\{b_i\}$模$a$的余数，那么有$t$后就能快速数出第2种情况的数量。

## 代码

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;
using ll=long long;
const int MAXA=1010,MAXB=100010;

int nlen,mlen;
int a[MAXB],acnt[MAXA];
int b[MAXB];

int fts[MAXA][MAXA];
int pre[MAXA][MAXA];

void force(int i){
	pre[i][0]=fts[i][0];
	for(int j=1;j<MAXA;j++){
		pre[i][j]=pre[i][j-1]+fts[i][j];
	}
}

ll base=0;

bool check(ll k,ll t){
	ll res=-base;
	for(int i=1;i<=1000;i++){
		res+=(t/i)*acnt[i];
		int c=t%i;
		ll offset=pre[i][1000]-pre[i][c];
		res-=offset;
	}
	return res>=k;
}

int main(){
	ios::sync_with_stdio(false);
	cin.tie(0);cout.tie(0);
	int kase;cin>>kase;
	while(kase--){
		memset(acnt,0,sizeof(acnt));
		memset(a,0,sizeof(a));
		base=0;
		memset(fts,0,sizeof(fts));
		
		cin>>nlen>>mlen;
		for(int i=1;i<=nlen;i++){
			cin>>a[i];
			acnt[a[i]]+=1;
		}
		for(int i=1;i<=nlen;i++){
			cin>>b[i];
			fts[a[i]][b[i]%a[i]]++;
			base+=b[i]/a[i];
		}
		for(int i=1;i<=1000;i++){
			force(i);
		}

		for(int i=1;i<=mlen;i++){
			int opt;
			cin>>opt;
			if(opt==1){
				int x,y;
				cin>>x>>y;

				acnt[a[x]]--;
				fts[a[x]][b[x]%a[x]]--;
				base-=b[x]/a[x];
				force(a[x]);

				a[x]=y;
				fts[a[x]][b[x]%a[x]]++;
				acnt[a[x]]++;
				base+=b[x]/a[x];
				force(a[x]);

				
			}else if(opt==2){
				int x,y;
				cin>>x>>y;

				fts[a[x]][b[x]%a[x]]--;
				base-=b[x]/a[x];
				force(a[x]);

				b[x]=y;

				fts[a[x]][b[x]%a[x]]++;
				force(a[x]);
				base+=b[x]/a[x];
			}else if(opt==3){
				int qk;cin>>qk;
				ll l=0,r=1e13;
				while(l+1<r){
					ll mid=(l+r)/2;
					if(check(qk,mid)){
						r=mid;
					}else l=mid+1;
				}
				for(ll i=l;i<=r;i++){
					if(check(qk,i)){
						cout<<i<<"\n";
						break;
					}
				}
			}
		}
	}
	return 0;
}
```
