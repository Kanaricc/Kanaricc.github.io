# 莫队算法

对于可以离线的区间询问问题，莫队算法提出了一种可以在$O(n\sqrt n)$(无修改)，$n^{5/3}$(带修改)内得出答案的方法。

主要的思路是对询问离线并分块，利用在2个区间间答案的**快速转移**（如果无法找到快速转移的方法，就没法用了）降低复杂度。

<!--more-->

## 无修改莫队
以$B=\sqrt{n}$，按照$(l/B,r)$对询问排序。

之后枚举每一个询问，将答案在相邻询问区间间暴力的+1-1转移。

### Problem: 小Y的袜子
没有在针对哪个人。

作为一个生活散漫的人，小Z每天早上都要耗费很久从一堆五颜六色的袜子中找出一双来穿。终于有一天，小Z再也无法忍受这恼人的找袜子过程，于是他决定听天由命……

具体来说，小Z把这N只袜子从1到N编号，然后从编号L到R(L 尽管小Z并不在意两只袜子是不是完整的一双，甚至不在意两只袜子是否一左一右，他却很在意袜子的颜色，毕竟穿两只不同色的袜子会很尴尬。

你的任务便是告诉小Z，他有多大的概率抽到两只颜色相同的袜子。当然，小Z希望这个概率尽量高，所以他可能会询问多个(L,R)以方便自己选择。

袜子的数量最多为50000（是真的🐂🍺）

#### 分析
这似乎是莫队的例题（

```cpp
#include <iostream>
#include <algorithm>
#include <cstdlib>
#include <cstring>
#include <cmath>
using namespace std;
using ll=long long;
const int MAXN=50010,MAXQ=50010;

ll gcd(ll a,ll b){
    return !b?a:gcd(b,a%b);
}
ll c2(ll n){
    if(n<2)return 0;
    return n*(n-1)/2;
}
int a[MAXN];
int block=0;
struct Q{
    int l,r;
    int i;
    ll ansu,ansd;
    bool operator<(const Q &b)const{
        if(l/block!=b.l/block)return l/block<b.l/block;
        return r<b.r;
    }
}qs[MAXQ];

int cnt[MAXN];
ll cup=0,cdown=0;
void remove(int ptr){
    cup-=c2(cnt[a[ptr]]);
    cnt[a[ptr]]--;
    cdown--;
    cup+=c2(cnt[a[ptr]]);
}
void add(int ptr){
    cup-=c2(cnt[a[ptr]]);
    cnt[a[ptr]]++;
    cdown++;
    cup+=c2(cnt[a[ptr]]);
}
int main(){
    int nlen,qlen;cin>>nlen>>qlen;
    for(int i=1;i<=nlen;i++)cin>>a[i];
    for(int i=0;i<qlen;i++)cin>>qs[i].l>>qs[i].r;
    for(int i=0;i<qlen;i++)qs[i].i=i;
    block=sqrt(nlen);
    sort(qs,qs+qlen);
    /*
    cout<<"current queries:"<<endl;
    for(auto q:qs){
        cout<<q.l<<" "<<q.r<<endl;
    }
    cout<<"====="<<endl;
*/
    int l=1,r=1;
    add(1);
    for(int i=0;i<qlen;i++){
        Q &q=qs[i];
        if(q.l==q.r){
            q.ansu=0;q.ansd=1;
            continue;
        }
        while(q.l<l)add(--l);
        while(r<q.r)add(++r);
        while(l<q.l)remove(l++);
        while(q.r<r)remove(r--);

        q.ansu=cup;
        q.ansd=cdown;
        //cout<<cup/c2(cdown)<<endl;
    }
    sort(qs,qs+qlen,[](const Q &a,const Q &b){
            return a.i<b.i;
            });
    for(int i=0;i<qlen;i++){
        if(qs[i].ansd<2){
            cout<<"0/1"<<endl;
            continue;
        }
        ll u=qs[i].ansu;
        ll d=c2(qs[i].ansd);
        ll g=gcd(u,d);
        if(g!=0)u/=g,d/=g;
        cout<<u<<"/"<<d<<endl;
    }


    return 0;
}
```

## 带修改莫队
> 一切都是石x门的选择！

将修改操作平铺在时间线上，计算每次询问所处的时间点，以$B=\sqrt{n}$，按照$(l/B,r/B,time)$对询问排序。

之后枚举每一个询问，将答案在不同时间线间暴力+1-1跳转，再暴力在相邻询问的区间间+1-1转移。

~~听起来有点中二的意思。~~

### Problem: Game
Again Alice and Bob is playing a game with stones. There are N piles of stones labelled from 1 to N, the i th pile has ai stones. 

First Alice will choose piles of stones with consecutive labels, whose leftmost is labelled with L and the rightmost one is R. After, Bob will choose another consecutive piles labelled from l to r (L≤l≤r≤R). Then they're going to play game within these piles.

Here's the rules of the game: Alice takes first and the two will take turn to make a move: choose one pile with nonegetive stones and take at least one stone and at most all away. One who cant make a move will lose.

Bob thinks this game is not so intersting because Alice always take first. So they add a new rule, which is that Bob can swap the number of two adjacent piles' stones whenever he want before a new round. That is to say, if the i th and i+1 pile have ai and ai+1 stones respectively, after this swapping there will be ai+1 and ai.

Before today's game with Bob, Alice wants to know, if both they play game optimally when she choose the piles from L to R, there are how many pairs (l, r) chosed by Bob that will make Alice *win*.

#### 分析
nim游戏输赢就是看异或和，异或和可以看前缀异或和内有多少个值相同的点。所以它就是在问一个区间里有多少个相同点对。

带单点修改的区间点对计数。

#### 代码
```cpp
#include <iostream>
#include <cstdio>
#include <cmath>
#include <algorithm>
#include <cstring>
#include <vector>
using namespace std;
using ll=long long;
const int MAXN=100010,MAXQ=100010;
const int MAXPOS=2e6+11;

int nlen,qlen;
int game[MAXN];
int pre[MAXN];
int modified[MAXQ],midx=0,belong[MAXN];
int block=0;
struct Q{
    int i;
    int l,r;
    int tick;

    bool operator<(const Q &b)const{
 //       if(l/block!=b.l/block)return l/block<b.l/block;
   //     if(r/block!=b.r/block)return r/block<b.r/block;
        if(belong[l]!=belong[b.l])
            return belong[l]<belong[b.l];
        if(belong[r]!=belong[b.r])
            return belong[r]<belong[b.r];
        return  tick<b.tick;
    }
}q[MAXQ];
int qidx=0;
ll cache=0;
int cnt[MAXPOS];

inline void add(int pos){
    int &c=cnt[pre[pos]];
    cache-=(ll)c*(c-1)/2;
    c++;
    cache+=(ll)c*(c-1)/2;
}
inline void rm(int pos){
    int &c=cnt[pre[pos]];
    cache-=(ll)c*(c-1)/2;
    c--;
    cache+=(ll)c*(c-1)/2;
}

int l=1,r=1,curt=0;
inline void jumpup(int tim){
    if(tim==0)return;
    int pos=modified[tim];
    int a=game[pos];
    int b=game[pos+1];
    swap(game[pos],game[pos+1]);
    if(l<=pos && pos<=r){
        rm(pos);
    }
    pre[pos]^=a;
    pre[pos]^=b;
    if(l<=pos && pos<=r){
        add(pos);
    }
}
inline void jumpdown(int tim){
    jumpup(tim);
}

ll qans[MAXQ];

int main(){
    //freopen("00.in","r",stdin);
    while(~scanf("%d%d",&nlen,&qlen)){
        block=pow(nlen,2.0/3);
        for(int i=1;i<=nlen;i++){
            scanf("%d",&game[i]);
            belong[i]=(i-1)/block;
        }
        pre[0]=0;
        for(int i=1;i<=nlen;i++)pre[i]=pre[i-1]^game[i];

        qidx=0,midx=0;
        for(int i=1;i<=qlen;i++){
            int opt;
            scanf("%d",&opt);
            if(opt==1){
                int l,r;
                scanf("%d%d",&l,&r);
                q[qidx].i=qidx;
                q[qidx].l=l;
                q[qidx].l--;
                q[qidx].r=r;
                q[qidx].tick=midx;
                qidx++;
            }else if(opt==2){
                scanf("%d",&modified[++midx]);
            }
        }
        sort(q,q+qidx);

        memset(cnt,0,sizeof(cnt));
        cache=0;
        l=r=1;curt=0;
        add(1);
        for(int i=0;i<qidx;i++){
            if(q[i].r-q[i].l+1<2){
                qans[q[i].i]=0;
                continue;
            }
            while(curt<q[i].tick)jumpup(++curt);
            while(q[i].tick<curt)jumpdown(curt--);

            while(q[i].l<l)add(--l);
            while(r<q[i].r)add(++r);
            while(l<q[i].l)rm(l++);
            while(q[i].r<r)rm(r--);
 
            //qans[q[i].i]=cache;
            ll len=r-l+1;
            qans[q[i].i]=len*(len-1)/2-cache;
        }
        for(int i=0;i<qidx;i++){
            printf("%lld\n",qans[i]);
        }
    }
    return 0;
}
```

## 注意
* 关于记录当前问题的区间的开闭问题，需要谨慎安排。
* 在确认了区间开闭后，关于最初始的状态，需要谨慎安排。
