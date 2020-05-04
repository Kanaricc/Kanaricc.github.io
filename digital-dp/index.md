# 数位DP

## 例题-4 HDU2089 不要62
统计数位中没有出现4和62的数字个数。

### 分析
不要4，可以在4时直接不转移。对于62，可以维护一个上一个数字填了啥，就可以像4一样判断了。

<!--more-->

### 代码
```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;
using ll=long long;
constexpr int MAXN=20;
ll cache[MAXN][MAXN];
int digits[20];
ll solve(int pos,int last,int lim){
    if(pos==0)return 1;
    if(!lim && ~cache[pos][last])return cache[pos][last];

    int maxd=9;
    if(lim)maxd=digits[pos];
    ll res=0;
    for(int i=0;i<=maxd;i++){
        if(last*10+i==62 || i==4)continue;
        res+=solve(pos-1,i,lim && i==maxd);
    }
    if(!lim)cache[pos][last]=res;
    return res;
}

ll SOLVE(ll x){
    int ptr=0;
    while(x){
        digits[++ptr]=x%10;
        x/=10;
    }

    return solve(ptr,0,true);
}



int main(){
    ll l,r;
    memset(cache,-1,sizeof(cache));
    while(cin>>l>>r){
        if(l==0 && r==0)break;
        cout<<SOLVE(r)-SOLVE(l-1)<<endl;
    }
    return 0;
}
```

## 例题-3 HDU3555
统计所有出现过49的数字的个数。

### 分析
统计所有没出现过49的数字，就和上一道一样了。

### 代码
```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;
using ll=long long;
const int MAXN=50;

int digits[MAXN];
ll f[MAXN][MAXN];

ll solve(int pos,int last,int lim){
    if(pos==0)return 1;
    if(!lim && ~f[pos][last])return f[pos][last];

    int maxd=9;
    if(lim)maxd=digits[pos];
    ll res=0;
    for(int i=0;i<=maxd;i++){
        if(last*10+i==49)continue;
        res+=solve(pos-1,i,lim && i==maxd);
    }

    if(!lim)f[pos][last]=res;
    return res;
}

inline ll SOLVE(ll x){
    int ptr=0;
    while(x){
        digits[++ptr]=x%10;
        x/=10;
    }

    return solve(ptr,0,true);
}


int main(){
    ios::sync_with_stdio(false);
    int kase;cin>>kase;
    memset(f,-1,sizeof(f));
    while(kase--){
        ll r;cin>>r;
        cout<<r-SOLVE(r)+1<<endl;
    }
    return 0;
}
```

## 例题-2 POJ3252
The cows, as you know, have no fingers or thumbs and thus are unable to play Scissors, Paper, Stone&#039; (also known as &#039;Rock, Paper, Scissors&#039;, &#039;Ro, Sham, Bo&#039;, and a host of other names) in order to make arbitrary decisions such as who gets to be milked first. They can&#039;t even flip a coin because it&#039;s so hard to toss using hooves.

They have thus resorted to "round number" matching. The first cow picks an integer less than two billion. The second cow does the same. If the numbers are both "round numbers", the first cow wins,
otherwise the second cow wins.

A positive integer N is said to be a "round number" if the binary representation of N has as many or more zeroes than it has ones. For example, the integer 9, when written in binary form, is 1001. 1001 has two zeroes and two ones; thus, 9 is a round number. The integer 26 is 11010 in binary; since it has two zeroes and three ones, it is not a round number.

Obviously, it takes cows a while to convert numbers to binary, so the winner takes a while to determine. Bessie wants to cheat and thinks she can do that if she knows how many "round numbers" are in a given range.

Help her by writing a program that tells how many round numbers appear in the inclusive range given by the input (1 ≤ Start < Finish ≤ 2,000,000,000).

### 分析

这道题目将上界拆解为二进制数位。

### 代码
```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;
typedef long long ll;
const int MAXN=50;

int digits[MAXN];
ll f[MAXN][MAXN][MAXN];
int len=0;

ll solve(int pos,int one,int first, int lim){
    if(pos==0){
        //cout<<first<<" "<<one<<endl;
        if(first==0)return 0;
        if(first-one>=one)return 1;
        return 0;
    }
    if(!lim && ~f[pos][one][first])return f[pos][one][first];

    int maxd=1;
    if(lim)maxd=digits[pos];
    ll res=0;
    for(int i=0;i<=maxd;i++){
        res+=solve(pos-1,one+(i==1),(i==1 && !first? pos : first),lim && i==maxd);
    }

    if(!lim)f[pos][one][first]=res;
    return res;
}

inline ll SOLVE(ll x){
    int ptr=0;
    while(x){
        digits[++ptr]=x%2;
        x/=2;
    }

    return solve(ptr,0,0,true);
}


int main(){
    ios::sync_with_stdio(false);
    memset(f,-1,sizeof(f));
    ll l,r;cin>>l>>r;
    cout<<SOLVE(r)-SOLVE(l-1)<<endl;

    return 0;
}
```

## 例题-1 HDU3652
A wqb-number, or B-number for short, is a non-negative integer whose decimal form contains the sub- string "13" and can be divided by 13. For example, 130 and 2613 are wqb-numbers, but 143 and 2639 are not. Your task is to calculate how many wqb-numbers from 1 to n for a given integer n.

### 分析

维护额外2个状态，一个状态记录模13的余数，另一个进行编码：1为上一位为1，3为已经出现过13，0为其他。之后将转移条件描述清楚就可以了。

### 代码
```cpp
#include <iostream>
#include <cstring>
#include <vector>
#include <cstring>
using namespace std;
using ll=long long;
const int MAXN=20;

int num[MAXN];
ll f[MAXN][5][20];
ll dfs(int ptr,int last,int mod,bool lim){
	//cout<<"arrive"<<ptr<<" "<<last<<" "<<mod<<" "<<lim<<endl;
	if(ptr<=0){
		if(last==3 && mod==0)return 1;
		else return 0;
	}
	if(!lim && ~f[ptr][last][mod])return f[ptr][last][mod];

	int maxx=lim?num[ptr]:9;
	ll res=0;
	for(int i=0;i<=maxx;i++){
		if(last==3)res+=dfs(ptr-1,3,(mod*10+i)%13,lim && i==maxx);
		else if(last==1 && i==3)res+=dfs(ptr-1,3,(mod*10+i)%13,lim && i==maxx);
		else if(i==1)res+=dfs(ptr-1,1,(mod*10+i)%13,lim&&i==maxx);
		else res+=dfs(ptr-1,0,(mod*10+i)%13,lim&&i==maxx);
	}
	if(!lim)f[ptr][last][mod]=res;
	return res;
}
ll solve(ll r){
	int ptr=0;
	while(r){
		num[++ptr]=r%10;
		r/=10;
	}
	return dfs(ptr,0,0,1);
}
int main(){
	ios::sync_with_stdio(false);
	cin.tie(0);
	memset(f,-1,sizeof(f));
	ll r;
	while(cin>>r){
		cout<<solve(r)<<endl;
	}

	return 0;
}
```


## 例题0 HDU3709
A balanced number is a non-negative integer that can be balanced if a pivot is placed at some digit. More specifically, imagine each digit as a box with weight indicated by the digit. When a pivot is placed at some digit of the number, the distance from a digit to the pivot is the offset between it and the pivot. Then the torques of left part and right part can be calculated. It is balanced if they are the same. A balanced number must be balanced with the pivot at some of its digits. For example, 4139 is a balanced number with pivot fixed at 3. The torqueses are 4*2 + 1*1 = 9 and 9*1 = 9, for left part and right part, respectively. It&#039;s your job to calculate the number of balanced numbers in a given range [x, y].

### 分析
这题有点麻烦……

将杠杆在哪与题目分离出来，枚举杠杆定在何处。再维护是否平衡。平衡与否还是挺好考虑的，以杠杆为0，左右各赋值正与负的权值，当到达边接后全杆重量为0时即为平衡。

> 有一个小坑，就是0。如果这个数字全tm填0的时候，不管杠杆定在哪，它都平衡……你当然可以维护一个前导零标志，将所有0都去掉最后再加上。还有一种办法就是最后再将重复的0减去，相比起来更加简单。

### 代码

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;
using ll=long long;

int num[20];
ll f[20][20][2010];

ll dfs(int ptr,bool lim,int piv,int l){
	if(ptr<=0)return l==0;
	if(l<0)return 0;
	if(!lim && ~f[ptr][piv][l])return f[ptr][piv][l];

	int maxx=lim?num[ptr]:9;

	ll res=0;
	for(int i=0;i<=maxx;i++){
		res+=dfs(ptr-1,lim && i==maxx,piv,l+i*(ptr-piv));
	}

	if(!lim)f[ptr][piv][l]=res;
	return res;
}

ll solve(ll r){
	int ptr=0;
	while(r){
		num[++ptr]=r%10;
		r/=10;
	}
	ll ans=0;
	for(int i=1;i<=ptr;i++){
		ans+=dfs(ptr,1,i,0);
	}
	return ans-ptr+1;
}
int main(){
	ios::sync_with_stdio(false);
	cin.tie(0);cout.tie(0);
	int kase;cin>>kase;
	while(kase--){
		memset(f,-1,sizeof(f));
		ll l,r;cin>>l>>r;
		cout<<solve(r)+(l?-solve(l-1):0)<<'\n';
	}

	return 0;
}
```

## 例题1 CF55D
Volodya is an odd boy and his taste is strange as well. It seems to him that a positive integer number is beautiful if and only if it is divisible by each of its nonzero digits. We will not argue with this and just count the quantity of beautiful numbers in given ranges.

### 分析
维护数字对0-9每个数字的模，但是这样还需要维护某些数字是否出现，继续考虑。当数字能同时被$a$和$b$整除，等价为其能够被它俩的最小公倍数整除。

那么我们的问题可以稍微简化一下了。维护已经填写的数位的最小公倍数，并维护数字和。到达边界时，只有满足条件才计数。

> 这道题还有一点麻烦，就是内存它不够用。按照分析，至少需要开$2520\times 2520$，还要再加上个数位长度。内存就炸了。实际上，最小公倍数仅有了了几个，可以事先求出并先编码，就能降低一维状态。

### 代码
```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <vector>
#include <map>
using namespace std;
using ll = long long;
const int MAXN = 2620;
const int MAX_LCM = 2520;
//pos位数余rest模·10·的方案
ll cache[20][MAXN][50];
int digits[20];

int idx = 0;
int lcmref[MAXN];

ll gcd(ll a, ll b)
{
    return !b ? a : gcd(b, a % b);
}
ll lcm(ll a, ll b)
{
    return a / gcd(a, b) * b;
}
void init_lcm()
{
    for (int i = 0; i < (1 << 10); i++)
    {
        int temp = 1;
        for (int j = 1; j < 10; j++)
        {
            if ((i >> j) & 1)
            {
                temp = lcm(temp, j);
            }
        }
        if (!lcmref[temp])
        {
            //cout << temp << endl;
            lcmref[temp]=++idx;
        }
    }
}
ll solve(int pos, int rest, int l, bool lim)
{
    if (pos == 0)
    {
        if (rest % l == 0)
            return 1;
        return 0;
    }
    if (!lim && ~cache[pos][rest][lcmref[l]])
        return cache[pos][rest][lcmref[l]];

    ll res = 0;
    int maxd = 9;
    if (lim)
        maxd = digits[pos];
    for (int i = 0; i <= maxd; i++)
    {
        res += solve(pos - 1, (rest * 10 + i) % MAX_LCM, i == 0 ? l : lcm(l, i), lim && (i == maxd));
    }
    if (!lim)
        cache[pos][rest][lcmref[l]] = res;
    return res;
}

ll SOLVE(ll x)
{
    int ptr = 0;
    while (x)
    {
        digits[++ptr] = x % 10;
        x /= 10;
    }
    return solve(ptr, 0, 1, true);
}

int main()
{
    ios::sync_with_stdio(false);
    init_lcm();
    int kase;
    cin >> kase;
    memset(cache, -1, sizeof(cache));
    while (kase--)
    {
        ll l, r;
        cin >> l >> r;
        cout << SOLVE(r) - SOLVE(l - 1) << endl;
    }

    return 0;
}
```

## 例题2 
求$[l,r]$区间内的所有满足以下要求的数的平方和。

1. 数位上没有$7$
2. 不是$7$的倍数
3. 数位和不是$7$的倍数

区间为$10^{18}$级别，请对答案取模$10^{9}+7$。

### 分析
这题挺狠的……首先看如何判断上面的3个条件。

* 数位上没有$7$，可以直接在枚举填数时跳过$7$。
* 非 * 的倍数，可以维护已填数字的取模。取模为$0$则为倍数。
* 数位和，仍然可以直接维护。

这样就可以正确的判断某个数是不是满足条件。接下来是平方和。

当以数位来考虑平方和时，问题就稍微变得复杂了。假设我们已经知道了下一个状态的平方和为$x_n$，如何得到当前位$x$的平方和。

假设当前位$ptr$填了$i$。其数位所表达的意义是$10^{p}i$，那么以该位与下一状态的平方和，需要下一状态的**和**，定为$s_n$。可以得到

<div>$$
(10^p i+s_n)^2=10^{2p}i^2+2 \cdot 10^pis_n+s_n^2
$$</div>

另外，当前位的出现次数在实际计算时需要考虑进去。那么，还需要维护下一状态的**个数**，记$cnt$。那么，上式需要修改

<div>$$
10^{2p}i^2 \cdot cnt_n+2 \cdot 10^pis_n+s_n^2
$$</div>

后两项不乘以$cnt$是因为它们本身就已经是包括了下一状态的所有计数。这样，我们就得到了以数位为角度的平方和维护办法。

### 代码

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;
using ll=long long;
const int MAXN=22;
const ll MOD=1e9+7;

struct Status{
	ll cnt;
	ll sum,sqsum;
	Status(ll cnt=-1,ll sum=0,ll sqsum=0):cnt(cnt),sum(sum),sqsum(sqsum){}
} f[MAXN][9*20][10];
int num[MAXN];

ll qpow(ll a,ll b){
	ll res=1;
	for(;b;b>>=1,a=a*a%MOD){
		if(b&1)res=res*a%MOD;
	}
	return res;
}

Status dfs(int ptr,int sum,int mod,bool lim){
	if(ptr<=0){
		return Status(sum%7!=0 && mod!=0);
	}

	if(!lim && ~f[ptr][sum][mod].cnt)return f[ptr][sum][mod];

	int maxx=lim?num[ptr]:9;
	Status res(0);
	for(int i=0;i<=maxx;i++){
		if(i==7)continue;
		Status nex=dfs(ptr-1,sum+i,(mod*10+i)%7,lim&&i==maxx);
		res.cnt+=nex.cnt;
		res.cnt%=MOD;
		res.sum=(res.sum%MOD+qpow(10,ptr-1)*i%MOD*nex.cnt%MOD)%MOD;
		res.sum=(res.sum+nex.sum)%MOD;
		res.sqsum=(res.sqsum+nex.cnt%MOD*qpow(10,2*(ptr-1))%MOD*i%MOD*i%MOD)%MOD;
		res.sqsum=((res.sqsum+nex.sqsum)%MOD+2*qpow(10,ptr-1)%MOD*i%MOD*nex.sum%MOD)%MOD;
	}
	if(!lim)f[ptr][sum][mod]=res;
	return res;
}


ll solve(ll n){
	int ptr=0;
	while(n){
		num[++ptr]=n%10;
		n/=10;
	}
	return dfs(ptr,0,0,1).sqsum%MOD;
}
int main(){
	ios::sync_with_stdio(false);
	cin.tie(0);

	int kase;cin>>kase;
	while(kase--){
		ll l,r;cin>>l>>r;
		cout<<((solve(r)-(l?solve(l-1):0))%MOD+MOD)%MOD<<endl;
	}
	return 0;
}
```

## 数位DP
上面这道题目大概算是暴露了数位DP中需要考虑的事情。数位DP能够主要能够统计某2个数字区间内满足某些条件的数字的信息，比如个数、和、平方和……

首先数位DP不是单独考虑每一个数字，而是以数位为单位的考虑。这样在考虑条件满足和各类信息的维护时，就需要转一转。

先来看记忆化搜索。比较直观（

### 计数基础
初始化的值从何而来。当数位DP到达了边界，就是要考虑初始化的位置。一般为满足条件置$1$。

### 满足条件
这是我们需要考虑的状态。例如上面提到的。

* 数位没有某数：直接不转移。
* 数位和：直接维护一维数位和。
* 倍数关系：以递推方式维护取模。
* 数字出现次数：使用状压来对次数编码。例如，对每一种数字维护空间为3的状态（0，奇数，偶数）。

其实还可能有很多，主要考虑的问题是该状态经过编码后能不能保存下来，两个状态之间能不能递推过去。

### 计数
必须考虑$n-1$到$n$**位**之间的转移关系。首先一定会维护满足性质的数字个数，之后每一个位的计算都需要考虑下一状态的个数。
